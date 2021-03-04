# Getting the Latest Image Version ResourceID from Shared Image Gallery
This code is designed to be used in a DevOps AZ CLI task, to get the lastest SIG version, you can then pass it into a DevOps variable, and use the resourceID in the proceeding Azure VM Image Builder Task.

## Using AZ CLI

```bash
# set resource group name
sigResourceGroup=aibwinsig

# SIG subscription, use current subscription 
subscriptionID=$(az account show | grep id | tr -d '",' | tr -d '[:space:]' | cut -c4- )
echo $subscriptionID

# name of the shared image gallery to used, e.g. myCorpGallery
sigName=my22stSIG

# name of the image definition to be used, e.g. ProdImages
imageDefName=winSvrimages

# image distribution metadata reference name
runOutputName=w2019SigRo2

sigDefImgVersionId=$(az sig image-version list \
   -g $sigResourceGroup \
   --gallery-name $sigName \
   --gallery-image-definition $imageDefName \
   --subscription $subscriptionID --query [].'id' -o json | grep 0. | tr -d '"' | tr -d '[:space:]' )
```

### bash script to get all image versions for a DevOps Task
```bash
echo INFO Get All Available Image Versions:

IFS=","
arr=($sigDefImgVersionId)
for i in "${!arr[@]}";do
echo Item $i:"${arr[$i]}"
done
unset IFS
echo "Completed"
latestSigId=${arr[@]:(-1)}

echo INFO Latest Image version : $latestSigId

#emit to DevOps var
echo "##vso[task.setvariable variable=sigLatest]$latestSigId"
```


## PowerShell

```powerShell
# get all versions from SIG def
$getAllImageVersions=$(Get-AzGalleryImageVersion -ResourceGroupName $imageResourceGroup  -GalleryName $sigGalleryName -GalleryImageDefinitionName $imageDefName)

# get name and expand publishing date for a version
$versionPubList=$($getAllImageVersions | Select-Object -Property Name -ExpandProperty PublishingProfile)

# order by published date (leaving in more columns for induvidual validation)
$sortedVersionList=$($versionPubList | Select-Object Name, PublishedDate | Sort-Object PublishedDate -Descending | Select-Object Name -First 1)

# get latest version resource id
$sigDefImgVersionId=$(Get-AzGalleryImageVersion -ResourceGroupName $imageResourceGroup  -GalleryName $sigGalleryName -GalleryImageDefinitionName $imageDefName -Name $sortedVersionList.name).Id
```
