---
title: Erstellen von ContentKeys mit .NET
description: In diesem Artikel wird gezeigt, wie symmetrische Schlüssel mithilfe von .NET erstellt werden können. Diese Schlüssel ermöglichen einen sicheren Zugriff auf Medienobjekte.
services: media-services
documentationcenter: ''
author: Juliako
manager: femila
editor: ''
ms.assetid: 225b05e5-7d30-409c-b5b7-3ef0634310c7
ms.service: media-services
ms.workload: media
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 03/18/2019
ms.author: juliako
ms.custom: devx-track-csharp
ms.openlocfilehash: d5efbc30c80a0052ed99cb129c2e5f9030482012
ms.sourcegitcommit: 32c521a2ef396d121e71ba682e098092ac673b30
ms.translationtype: HT
ms.contentlocale: de-DE
ms.lasthandoff: 09/25/2020
ms.locfileid: "91261225"
---
# <a name="create-contentkeys-with-net"></a>Erstellen von ContentKeys mit .NET

[!INCLUDE [media services api v2 logo](./includes/v2-hr.md)]
 
> [!div class="op_single_selector"]
> * [REST](media-services-rest-create-contentkey.md)
> * [.NET](media-services-dotnet-create-contentkey.md)
> 
> 

Media Services ermöglicht das Erstellen und Übermitteln verschlüsselter Medienobjekte. Ein **ContentKey** ermöglicht den sicheren Zugriff auf Ihre **Medienobjekte**. 

Beim Erstellen eines neuen Medienobjekts (z.B. vor dem [Hochladen von Dateien](media-services-dotnet-upload-files.md)) können Sie die folgenden Verschlüsselungsoptionen angeben: **StorageEncrypted**, **CommonEncryptionProtected** oder **EnvelopeEncryptionProtected**. 

Wenn Sie Medienobjekte an Ihre Clients übermitteln, können Sie mithilfe einer der beiden folgenden Verschlüsselungen die [dynamische Verschlüsselung von Medienobjekten konfigurieren](media-services-dotnet-configure-asset-delivery-policy.md): **DynamicEnvelopeEncryption** oder **DynamicCommonEncryption**.

Verschlüsselte Medienobjekte müssen **ContentKeys**zugeordnet werden. In diesem Artikel wird beschrieben, wie ein Inhaltsschlüssel erstellt wird.

> [!NOTE]
> Beim Erstellen eines neuen **StorageEncrypted**-Medienobjekts mithilfe des Media Services .NET SDK wird der **ContentKey** automatisch erstellt und mit dem Medienobjekt verknüpft.
> 
> 

## <a name="contentkeytype"></a>ContentKeyType
Einer der Werte, die Sie beim Erstellen eines Inhaltsschlüssels festlegen müssen, ist der Typ des Inhaltsschlüssels. Wählen Sie unter folgenden Werten aus. 

```csharp
    public enum ContentKeyType
    {
        /// <summary>
        /// Specifies a content key for common encryption.
        /// </summary>
        /// <remarks>This is the default value.</remarks>
        CommonEncryption = 0,

        /// <summary>
        /// Specifies a content key for storage encryption.
        /// </summary>
        StorageEncryption = 1,

        /// <summary>
        /// Specifies a content key for configuration encryption.
        /// </summary>
        ConfigurationEncryption = 2,

        /// <summary>
        /// Specifies a content key for Envelope encryption.  Only used internally.
        /// </summary>
        EnvelopeEncryption = 4
    }
```

## <a name="create-envelope-type-contentkey"></a><a id="envelope_contentkey"></a>Erstellen eines "ContentKey" vom Typ "Umschlagverschlüsselung"
Im folgenden Codeausschnitt wird ein Inhaltsschlüssel vom Typ „Umschlagverschlüsselung“ erstellt. Anschließend wird der Schlüssel dem angegebenen Medienobjekt zugeordnet.

```csharp
    static public IContentKey CreateEnvelopeTypeContentKey(IAsset asset)
    {
        // Create envelope encryption content key
        Guid keyId = Guid.NewGuid();
        byte[] contentKey = GetRandomBuffer(16);

        IContentKey key = _context.ContentKeys.Create(
                                keyId,
                                contentKey,
                                "ContentKey",
                                ContentKeyType.EnvelopeEncryption);

        asset.ContentKeys.Add(key);

        return key;
    }

    static private byte[] GetRandomBuffer(int size)
    {
        byte[] randomBytes = new byte[size];
        using (RNGCryptoServiceProvider rng = new RNGCryptoServiceProvider())
        {
            rng.GetBytes(randomBytes);
        }

        return randomBytes;
    }

call

    IContentKey key = CreateEnvelopeTypeContentKey(encryptedsset);
```


## <a name="create-common-type-contentkey"></a><a id="common_contentkey"></a>Erstellen eines "ContentKey" vom Typ "Allgemeine Verschlüsselung"
Im folgenden Codeausschnitt wird ein Inhaltsschlüssel vom Typ „Allgemeine Verschlüsselung“ erstellt. Anschließend wird der Schlüssel dem angegebenen Medienobjekt zugeordnet.

```csharp
    static public IContentKey CreateCommonTypeContentKey(IAsset asset)
    {
        // Create common encryption content key
        Guid keyId = Guid.NewGuid();
        byte[] contentKey = GetRandomBuffer(16);

        IContentKey key = _context.ContentKeys.Create(
                                keyId,
                                contentKey,
                                "ContentKey",
                                ContentKeyType.CommonEncryption);

        // Associate the key with the asset.
        asset.ContentKeys.Add(key);

        return key;
    }

    static private byte[] GetRandomBuffer(int length)
    {
        var returnValue = new byte[length];

        using (var rng =
            new System.Security.Cryptography.RNGCryptoServiceProvider())
        {
            rng.GetBytes(returnValue);
        }

        return returnValue;
    }
call

    IContentKey key = CreateCommonTypeContentKey(encryptedsset); 
```

## <a name="media-services-learning-paths"></a>Media Services-Lernpfade
[!INCLUDE [media-services-learning-paths-include](../../../includes/media-services-learning-paths-include.md)]

## <a name="provide-feedback"></a>Feedback geben
[!INCLUDE [media-services-user-voice-include](../../../includes/media-services-user-voice-include.md)]

