---
title: Erstellen einer VM aus einem generalisierten Image mithilfe der Azure-Befehlszeilenschnittstelle
description: Erstellen Sie eine VM aus einem generalisierten Image mithilfe der Azure-Befehlszeilenschnittstelle.
author: cynthn
ms.service: virtual-machines
ms.subservice: imaging
ms.topic: how-to
ms.workload: infrastructure
ms.date: 05/04/2020
ms.author: cynthn
ms.custom: devx-track-azurecli
ms.openlocfilehash: ec589848625e1114dedd8c58b41f7ecbc991f311
ms.sourcegitcommit: aaa65bd769eb2e234e42cfb07d7d459a2cc273ab
ms.translationtype: HT
ms.contentlocale: de-DE
ms.lasthandoff: 01/27/2021
ms.locfileid: "98881973"
---
# <a name="create-a-vm-from-a-generalized-image-version-using-the-cli"></a>Erstellen einer VM aus einer generalisierten Imageversion mithilfe der Befehlszeilenschnittstelle

Erstellen Sie eine VM aus einer [generalisierten Imageversion](./shared-image-galleries.md#generalized-and-specialized-images), die in einem Katalog mit freigegebenen Images gespeichert ist. Informationen zum Erstellen einer VM aus einem spezialisierten Image finden Sie unter [Erstellen einer VM aus einem spezialisierten Image](vm-specialized-image-version-powershell.md). 


## <a name="get-the-image-id"></a>Abrufen der Image-ID

Listen Sie die Imagedefinitionen in einem Katalog mithilfe von [az sig image-definition list](/cli/azure/sig/image-definition#az-sig-image-definition-list) auf, um den Namen und die ID der Definitionen anzuzeigen.

```azurecli-interactive 
resourceGroup=myGalleryRG
gallery=myGallery
az sig image-definition list --resource-group $resourceGroup --gallery-name $gallery --query "[].[name, id]" --output tsv
```

## <a name="create-the-vm"></a>Erstellen des virtuellen Computers

Erstellen Sie einen virtuellen Computer mit [az vm create](/cli/azure/vm#az-vm-create). Wenn Sie die neueste Version des Images verwenden möchten, legen Sie `--image` auf die ID der Imagedefinition fest. 

Ersetzen Sie bei Bedarf die Ressourcennamen in diesem Beispiel. 

```azurecli-interactive 
imgDef="/subscriptions/<subscription ID where the gallery is located>/resourceGroups/myGalleryRG/providers/Microsoft.Compute/galleries/myGallery/images/myImageDefinition"
vmResourceGroup=myResourceGroup
location=eastus
vmName=myVM
adminUsername=azureuser


az group create --name $vmResourceGroup --location $location

az vm create\
   --resource-group $vmResourceGroup \
   --name $vmName \
   --image $imgDef \
   --admin-username $adminUsername \
   --generate-ssh-keys
```

Sie können auch eine bestimmte Version verwenden, indem Sie die Imageversion mit der ID für den `--image`-Parameter verwenden. Um beispielsweise die Imageversion *1.0.0* zu verwenden, geben Sie ein: `--image "/subscriptions/<subscription ID where the gallery is located>/resourceGroups/myGalleryRG/providers/Microsoft.Compute/galleries/myGallery/images/myImageDefinition/versions/1.0.0"`.

## <a name="next-steps"></a>Nächste Schritte

[Azure Image Builder (Vorschauversion)](./image-builder-overview.md) hilft beim Automatisieren der Erstellung von Imageversionen. Sie können den Dienst sogar zum Aktualisieren und [Erstellen einer neuen Imageversion aus einer vorhandenen](./linux/image-builder-gallery-update-image-version.md) verwenden.