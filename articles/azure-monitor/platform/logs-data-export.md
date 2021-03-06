---
title: Datenexport im Log Analytics-Arbeitsbereich in Azure Monitor (Vorschau)
description: Der Log Analytics-Datenexport ermöglicht es Ihnen, Daten ausgewählter Tabellen aus Ihrem Log Analytics-Arbeitsbereich bei der Sammlung fortlaufend in ein Azure Storage-Konto oder in Azure Event Hubs zu exportieren.
ms.subservice: logs
ms.topic: conceptual
ms.custom: references_regions, devx-track-azurecli
author: bwren
ms.author: bwren
ms.date: 10/14/2020
ms.openlocfilehash: bb4987550e4962ba044e0a6aafbfd00145319e94
ms.sourcegitcommit: fc8ce6ff76e64486d5acd7be24faf819f0a7be1d
ms.translationtype: HT
ms.contentlocale: de-DE
ms.lasthandoff: 01/26/2021
ms.locfileid: "98804942"
---
# <a name="log-analytics-workspace-data-export-in-azure-monitor-preview"></a>Datenexport im Log Analytics-Arbeitsbereich in Azure Monitor (Vorschau)
Der Datenexport im Log Analytics-Arbeitsbereich in Azure Monitor ermöglicht es Ihnen, Daten aus ausgewählten Tabellen in Ihrem Log Analytics-Arbeitsbereich bei der Sammlung fortlaufend in ein Azure Storage-Konto oder in Azure Event Hubs zu exportieren. In diesem Artikel werden dieses Feature und die Schritte zum Konfigurieren des Datenexports in Ihren Arbeitsbereichen ausführlich beschrieben.

## <a name="overview"></a>Übersicht
Nachdem der Datenexport für den Log Analytics-Arbeitsbereich konfiguriert wurde, werden alle neuen Daten, die an die ausgewählten Tabellen im Arbeitsbereich gesendet werden, automatisch in Ihr Speicherkonto oder nahezu in Echtzeit in Ihren Event Hub exportiert.

![Übersicht über den Datenexport](media/logs-data-export/data-export-overview.png)

Alle Daten aus eingeschlossenen Tabellen werden ohne Filter exportiert. Wenn Sie z. B. eine Datenexportregel für die Tabelle *SecurityEvent* konfigurieren, werden alle Daten, die an die Tabelle *SecurityEvent* gesendet werden, ab dem Zeitpunkt der Konfiguration exportiert.


## <a name="other-export-options"></a>Weitere Exportoptionen
Mit dem Datenexport im Log Analytics-Arbeitsbereich werden kontinuierlich Daten aus einem Log Analytics Arbeitsbereich exportiert. Darüber hinaus gibt es die folgenden Möglichkeiten zum Exportieren von Daten für bestimmte Szenarien:

- Geplanter Export aus einer Protokollabfrage mithilfe einer Logik-App. Dies ähnelt dem Datenexportfeature, ermöglicht Ihnen aber das Senden von gefilterten oder aggregierten Daten an Azure Storage. Diese Methode unterliegt jedoch [Beschränkungen für Protokollabfragen](../service-limits.md#log-analytics-workspaces). Informationen dazu finden Sie unter [Archivieren von Daten aus dem Log Analytics-Arbeitsbereich in Azure Storage mithilfe von Logic Apps](logs-export-logic-app.md).
- Einmaliger Export mit einer Logik-App. Informationen hierzu finden Sie unter [Connector für Azure Monitor-Protokolle für Azure Logic Apps und Power Automate](logicapp-flow-connector.md).
- Einmaliger Export auf lokalen Computer mit einem PowerShell-Skript. Informationen hierzu finden Sie unter [Invoke-AzOperationalInsightsQueryExport](https://www.powershellgallery.com/packages/Invoke-AzOperationalInsightsQueryExport).


## <a name="limitations"></a>Einschränkungen

- Die Konfiguration kann zurzeit mithilfe der Befehlszeilenschnittstelle oder über REST-Anforderungen vorgenommen werden. Azure-Portal oder PowerShell werden noch nicht unterstützt.
- Die Option ```--export-all-tables``` in CLI und REST wird nicht unterstützt und entfernt. Sie müssen die Liste der Tabellen in den Exportregeln explizit angeben.
- Unterstützte Tabellen sind zurzeit auf die im Abschnitt [Unterstützte Tabellen](#supported-tables) weiter unten beschränkt. 
- Wenn die Datenexportregel eine nicht unterstützte Tabelle umfasst, wird der Vorgang erfolgreich ausgeführt, es werden jedoch für diese Tabelle keine Daten exportiert bis die Tabelle unterstützt wird. 
- Wenn die Datenexportregel eine nicht vorhandene Tabelle enthält, verursacht sie den Fehler ```Table <tableName> does not exist in the workspace```.
- Ihr Log Analytics-Arbeitsbereich kann sich in einer beliebigen Region mit Ausnahme der folgenden befinden:
  - Schweiz, Norden
  - Schweiz, Westen
  - Azure Government-Regionen
- Sie können zwei Exportregeln in einem Arbeitsbereich erstellen: eine Regel für den Event Hub und eine Regel für das Speicherkonto.
- Das Zielspeicherkonto oder der Ziel-Event Hub muss sich in derselben Region wie der Log Analytics-Arbeitsbereich befinden.
- Die Namen der zu exportierenden Tabellen dürfen bei einem Speicherkonto nicht länger als 60 Zeichen und bei einem Event Hub nicht länger als 47 Zeichen sein. Tabellen mit längeren Namen werden nicht exportiert.

> [!NOTE]
> Der Log Analytics-Datenexport schreibt Daten als Anfügeblob, das sich zurzeit für Azure Data Lake Storage Gen2 in der Vorschauphase befindet. Sie müssen eine Supportanfrage erstellen, bevor Sie den Export in diesen Speicher konfigurieren. Machen Sie in der Anfrage die folgenden Angaben.
> - Problemtyp: Technisch
> - Abonnement: Ihr Abonnement
> - Dienst: Data Lake Storage Gen2
> - Ressource: Der Name Ihrer Ressource
> - Zusammenfassung: Anforderung einer Abonnementregistrierung zum Akzeptieren von Daten aus dem Log Analytics-Datenexport.
> - Problemtyp: Konnektivität
> - Problemuntertyp: Konnektivitätsproblem

## <a name="data-completeness"></a>Datenvollständigkeit
Der Datenexport versucht bis zu 30 Minuten lang, Daten zu senden, wenn das Ziel nicht verfügbar ist. Wenn es nach 30 Minuten immer noch nicht verfügbar ist, werden die Daten verworfen, bis das Ziel wieder verfügbar ist.

## <a name="cost"></a>Cost
Für die Datenexportfunktion fallen zurzeit keine zusätzlichen Gebühren an. Die Preise für den Datenexport werden später bekannt gegeben, und vor Abrechnungsbeginn erhalten Sie eine entsprechende Benachrichtigung. Falls Sie sich dafür entscheiden, den Datenexport über den Benachrichtigungszeitraum hinaus zu verwenden, wird dies Ihnen zum entsprechenden Tarif in Rechnung gestellt.

## <a name="export-destinations"></a>Exportziele

### <a name="storage-account"></a>Speicherkonto
Daten werden, sobald sie Azure Monitor erreichen, nahezu in Echtzeit an Speicherkonten gesendet. Die Datenexportkonfiguration erstellt für jede Tabelle im Speicherkonto einen Container mit dem Namen *am-* , gefolgt vom Namen der Tabelle. Beispielsweise würde die Tabelle *SecurityEvent* an einen Container mit dem Namen *am-SecurityEvent* gesendet.

Der Blobpfad im Speicherkonto lautet *WorkspaceResourceId=/subscriptions/subscription-id/resourcegroups/\<resource-group\>/providers/microsoft.operationalinsights/workspaces/\<workspace\>/y=\<four-digit numeric year\>/m=\<two-digit numeric month\>/d=\<two-digit numeric day\>/h=\<two-digit 24-hour clock hour\>/m=00/PT1H.json*. Da Anfügeblobs auf 50.000 Schreibvorgänge im Speicher beschränkt sind, kann sich die Anzahl der exportierten Blobs erhöhen, wenn die Anzahl der Anfügevorgänge hoch ist. Das Benennungsmuster für Blobs ist in diesem Fall „PT1H_#.json“, wobei # die inkrementelle Blobanzahl ist.

Das Speicherkonto-Datenformat ist [JSON Lines](./resource-logs-blob-format.md). Dies bedeutet, dass die einzelnen Datensätze jeweils durch einen Zeilenumbruch getrennt sind und dass kein externes Datensatzarray und keine Kommas zwischen JSON-Datensätzen verwendet werden. 

[![Speicherbeispieldaten](media/logs-data-export/storage-data.png)](media/logs-data-export/storage-data.png#lightbox)

Durch den Log Analytics-Datenexport können Anfügeblobs in unveränderliche Speicherkonten geschrieben werden, wenn bei zeitbasierten Aufbewahrungsrichtlinien die Einstellung *allowProtectedAppendWrites* aktiviert ist. Dadurch wird das Schreiben neuer Blöcke in ein Anfügeblob ermöglicht, wobei gleichzeitig der Unveränderlichkeitsschutz und die Konformität aufrechterhalten bleiben. Weitere Informationen finden Sie unter [Zulassen von Schreibvorgängen in geschützten Anfügeblobs](../../storage/blobs/storage-blob-immutable-storage.md#allow-protected-append-blobs-writes).

### <a name="event-hub"></a>Event Hub
Daten werden, sobald sie Azure Monitor erreichen, nahezu in Echtzeit an Event Hub gesendet. Für jeden Datentyp, den Sie exportieren, wird ein Event Hub mit dem Namen *am-* erstellt, gefolgt vom Namen der Tabelle. Beispielsweise würde die Tabelle *SecurityEvent* an einen Event Hub mit dem Namen *am-SecurityEvent* gesendet. Wenn für die exportierten Daten ein bestimmter Event Hub als Ziel verwendet werden soll oder Sie eine Tabelle mit einem Namen haben, der den Grenzwert von 47 Zeichen überschreitet, können Sie den Namen Ihrer eigenen Event Hub-Instanz angeben und alle Daten für definierte Tabelle in diese exportieren.

Überlegungen:
1. Die Event Hub SKU „Basic“ unterstützt ein niedrigeres [Limit](../../event-hubs/event-hubs-quotas.md#basic-vs-standard-tiers) der Ereignisgröße, und einige Protokolle in Ihrem Arbeitsbereich können diese überschreiten und gelöscht werden. Es wird empfohlen, das Event Hub „Standard“ oder „Dedicated“ als Exportziel zu verwenden.
2. Die Menge der exportierten Daten nimmt im Laufe der Zeit häufig zu, und die Event Hub-Skalierung muss erhöht werden, um größere Übertragungsraten zu bewältigen und Drosselungsszenarien und Datenlatenz zu vermeiden. Verwenden Sie die Funktion „Automatische Vergrößerung“ von Event Hubs, um die Anzahl von Durchsatzeinheiten automatisch hochzuskalieren und so den Nutzungsanforderungen gerecht zu werden. Weitere Informationen finden Sie unter [Automatisches Hochskalieren von Azure Event Hubs-Durchsatzeinheiten](../../event-hubs/event-hubs-auto-inflate.md).

## <a name="prerequisites"></a>Voraussetzungen
Im Folgenden sind die Voraussetzungen aufgeführt, die vor der Konfiguration des Log Analytics-Datenexports erfüllt sein müssen.

- Das Speicherkonto bzw. der Event Hub muss bereits erstellt worden sein, und zwar in derselben Region wie der Log Analytics-Arbeitsbereich. Wenn Sie Ihre Daten in andere Speicherkonten replizieren müssen, können Sie eine der [Azure Storage Redundanzoptionen](../../storage/common/storage-redundancy.md) verwenden.  
- Beim Speicherkonto muss es sich um StorageV1 oder StorageV2 handeln. Klassischer Speicher wird nicht unterstützt  
- Wenn Sie Ihr Speicherkonto so konfiguriert haben, dass der Zugriff von ausgewählten Netzwerken aus möglich ist, müssen Sie eine Ausnahme in den Einstellungen Ihres Speicherkontos hinzufügen, damit Azure Monitor in den Speicher schreiben darf.

## <a name="enable-data-export"></a>Aktivieren des Datenexports
Die folgenden Schritte müssen ausgeführt werden, um den Log Analytics-Datenexport zu aktivieren. Ausführliche Informationen finden Sie in den entsprechenden Abschnitten weiter unten.

- Registrieren Sie den Ressourcenanbieter.
- Lassen Sie vertrauenswürdige Microsoft-Dienste zu.
- Erstellen Sie mindestens eine Datenexportregel, mit der die zu exportierenden Tabellen und deren Ziel definiert werden.

### <a name="register-resource-provider"></a>Registrieren des Ressourcenanbieters
Der folgende Azure-Ressourcenanbieter muss für Ihr Abonnement registriert werden, um den Log Analytics-Datenexport zu aktivieren. 

- Microsoft.Insights

Dieser Ressourcenanbieter ist bei den meisten Azure Monitor-Benutzern wahrscheinlich bereits registriert. Um dies zu überprüfen, gehen Sie im Azure-Portal zu **Abonnements**. Wählen Sie Ihr Abonnement aus, und klicken Sie dann im Bereich **Einstellungen** im Menü auf **Ressourcenanbieter**. Suchen Sie nach **Microsoft.Insights**. Wenn der Status **Registriert** lautet, ist die Registrierung bereits erfolgt. Andernfalls klicken Sie auf **Registrieren**, um die Registrierung vorzunehmen.

Sie können auch eine der verfügbaren Methoden zum Registrieren eines Ressourcenanbieters verwenden, die unter [Azure-Ressourcenanbieter und -typen](../../azure-resource-manager/management/resource-providers-and-types.md) beschrieben sind. Im Folgenden finden Sie einen Beispielbefehl für PowerShell:

```PowerShell
Register-AzResourceProvider -ProviderNamespace Microsoft.insights
```

### <a name="allow-trusted-microsoft-services"></a>Zulassen vertrauenswürdiger Microsoft-Dienste
Wenn Sie Ihr Speicherkonto so konfiguriert haben, dass der Zugriff von ausgewählten Netzwerken aus möglich ist, müssen Sie eine Ausnahme hinzufügen, damit Azure Monitor in das Konto schreiben darf. Wählen Sie unter **Firewalls und virtuelle Netzwerke** für Ihr Speicherkonto **Vertrauenswürdigen Microsoft-Diensten den Zugriff auf dieses Speicherkonto erlauben** aus.

[![„Firewalls und virtuelle Netzwerke“ unter Ihrem Speicherkonto](media/logs-data-export/storage-account-vnet.png)](media/logs-data-export/storage-account-vnet.png#lightbox)


### <a name="create-or-update-data-export-rule"></a>Erstellen oder Aktualisieren der Datenexportregel
Eine Datenexportregel definiert Daten, die für eine Gruppe von Tabellen an ein bestimmtes Ziel exportiert werden sollen. Sie können für jedes Ziel eine einzelne Regel erstellen.


# <a name="azure-portal"></a>[Azure portal](#tab/portal)

–

# <a name="powershell"></a>[PowerShell](#tab/powershell)

–

# <a name="azure-cli"></a>[Azure-Befehlszeilenschnittstelle](#tab/azure-cli)

Zeigen Sie mit dem folgenden CLI-Befehl Tabellen in Ihrem Arbeitsbereich an. Mit dem Befehl können Sie die gewünschten Tabellen kopieren und in die Datenexportregel einschließen.

```azurecli
az monitor log-analytics workspace table list --resource-group resourceGroupName --workspace-name workspaceName --query [].name --output table
```

Verwenden Sie den folgenden Befehl, um mithilfe der CLI eine Datenexportregel für ein Speicherkonto zu erstellen.

```azurecli
$storageAccountResourceId = '/subscriptions/subscription-id/resourceGroups/resource-group-name/providers/Microsoft.Storage/storageAccounts/storage-account-name'
az monitor log-analytics workspace data-export create --resource-group resourceGroupName --workspace-name workspaceName --name ruleName --tables SecurityEvent Heartbeat --destination $storageAccountResourceId
```

Verwenden Sie den folgenden Befehl, um mithilfe der CLI eine Datenexportregel für einen Event Hub zu erstellen. Für jede Tabelle wird ein separater Event Hub erstellt.

```azurecli
$eventHubsNamespacesResourceId = '/subscriptions/subscription-id/resourceGroups/resource-group-name/providers/Microsoft.EventHub/namespaces/namespaces-name'
az monitor log-analytics workspace data-export create --resource-group resourceGroupName --workspace-name workspaceName --name ruleName --tables SecurityEvent Heartbeat --destination $eventHubsNamespacesResourceId
```

Verwenden Sie den folgenden Befehl, um mithilfe der CLI eine Datenexportregel für einen bestimmten Event Hub zu erstellen. Alle Tabellen werden in den angegebenen Event Hub exportiert. 

```azurecli
$eventHubResourceId = '/subscriptions/subscription-id/resourceGroups/resource-group-name/providers/Microsoft.EventHub/namespaces/namespaces-name/eventHubName/eventhub-name'
az monitor log-analytics workspace data-export create --resource-group resourceGroupName --workspace-name workspaceName --name ruleName --tables SecurityEvent Heartbeat --destination $eventHubResourceId
```

# <a name="rest"></a>[REST](#tab/rest)

Verwenden Sie die folgende Anforderung, um mithilfe der REST-API eine Datenexportregel zu erstellen. Die Anforderung sollte die Bearertokenautorisierung und den Inhaltstyp „application/json“ verwenden.

```rest
PUT https://management.azure.com/subscriptions/<subscription-id>/resourcegroups/<resource-group-name>/providers/Microsoft.operationalInsights/workspaces/<workspace-name>/dataexports/<data-export-name>?api-version=2020-08-01
```

Der Anforderungstext gibt das Tabellenziel an. Im Folgenden sehen Sie einen Beispieltext für die REST-Anforderung für ein Speicherkonto.

```json
{
    "properties": {
        "destination": {
            "resourceId": "/subscriptions/subscription-id/resourcegroups/resource-group-name/providers/Microsoft.Storage/storageAccounts/storage-account-name"
        },
        "tablenames": [
            "table1",
            "table2" 
        ],
        "enable": true
    }
}
```

Im Folgenden sehen Sie einen Beispieltext für die REST-Anforderung für einen Event Hub.

```json
{
    "properties": {
        "destination": {
            "resourceId": "/subscriptions/subscription-id/resourcegroups/resource-group-name/providers/Microsoft.EventHub/namespaces/eventhub-namespaces-name"
        },
        "tablenames": [
            "table1",
            "table2"
        ],
        "enable": true
    }
}
```

Es folgt ein Beispieltext für die REST-Anforderung für einen Event Hub, bei dem der Event Hub-Name angegeben wird. In diesem Fall werden alle exportierten Daten an diese Event Hub-Instanz gesendet.

```json
{
    "properties": {
        "destination": {
            "resourceId": "/subscriptions/subscription-id/resourcegroups/resource-group-name/providers/Microsoft.EventHub/namespaces/eventhub-namespaces-name",
            "metaData": {
                "EventHubName": "eventhub-name"
        },
        "tablenames": [
            "table1",
            "table2"
        ],
        "enable": true
    }
  }
}
```

# <a name="template"></a>[Vorlage](#tab/json)

Verwenden Sie den folgenden Befehl, um mithilfe der Vorlage eine Datenexportregel für ein Speicherkonto zu erstellen.

```
{
    "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {
        "workspaceName": {
            "defaultValue": "workspace-name",
            "type": "String"
        },
        "workspaceLocation": {
            "defaultValue": "workspace-region",
            "type": "string"
        },
        "storageAccountRuleName": {
            "defaultValue": "storage-account-rule-name",
            "type": "string"
        },
        "storageAccountResourceId": {
            "defaultValue": "/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/resource-group-name/providers/Microsoft.Storage/storageAccounts/storage-account-name",
            "type": "String"
        }
    },
    "variables": {},
    "resources": [
        {
            "type": "microsoft.operationalinsights/workspaces",
            "apiVersion": "2020-08-01",
            "name": "[parameters('workspaceName')]",
            "location": "[parameters('workspaceLocation')]",
            "resources": [
                {
                  "type": "microsoft.operationalinsights/workspaces/dataexports",
                  "apiVersion": "2020-08-01",
                  "name": "[concat(parameters('workspaceName'), '/' , parameters('storageAccountRuleName'))]",
                  "dependsOn": [
                      "[resourceId('microsoft.operationalinsights/workspaces', parameters('workspaceName'))]"
                  ],
                  "properties": {
                      "destination": {
                          "resourceId": "[parameters('storageAccountResourceId')]"
                      },
                      "tableNames": [
                          "Heartbeat",
                          "InsightsMetrics",
                          "VMConnection",
                          "Usage"
                      ],
                      "enable": true
                  }
              }
            ]
        }
    ]
}
```

Verwenden Sie den folgenden Befehl, um mithilfe der Vorlage eine Datenexportregel für einen Event Hub zu erstellen. Für jede Tabelle wird ein separater Event Hub erstellt.

```
{
    "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {
        "workspaceName": {
            "defaultValue": "workspace-name",
            "type": "String"
        },
        "workspaceLocation": {
            "defaultValue": "workspace-region",
            "type": "string"
        },
        "eventhubRuleName": {
            "defaultValue": "event-hub-rule-name",
            "type": "string"
        },
        "namespacesResourceId": {
            "defaultValue": "/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/resource-group-name/providers/microsoft.eventhub/namespaces/namespaces-name",
            "type": "String"
        }
    },
    "variables": {},
    "resources": [
        {
            "type": "microsoft.operationalinsights/workspaces",
            "apiVersion": "2020-08-01",
            "name": "[parameters('workspaceName')]",
            "location": "[parameters('workspaceLocation')]",
            "resources": [
              {
                  "type": "microsoft.operationalinsights/workspaces/dataexports",
                  "apiVersion": "2020-08-01",
                  "name": "[concat(parameters('workspaceName'), '/', parameters('eventhubRuleName'))]",
                  "dependsOn": [
                      "[resourceId('microsoft.operationalinsights/workspaces', parameters('workspaceName'))]"
                  ],
                  "properties": {
                      "destination": {
                          "resourceId": "[parameters('namespacesResourceId')]"
                      },
                      "tableNames": [
                          "Usage",
                          "Heartbeat"
                      ],
                      "enable": true
                  }
              }
            ]
        }
    ]
}
```

Verwenden Sie den folgenden Befehl, um mithilfe der Vorlage eine Datenexportregel für einen bestimmten Event Hub zu erstellen. Alle Tabellen werden in den angegebenen Event Hub exportiert.

```
{
    "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {
        "workspaceName": {
            "defaultValue": "workspace-name",
            "type": "String"
        },
        "workspaceLocation": {
            "defaultValue": "workspace-region",
            "type": "string"
        },
        "eventhubRuleName": {
            "defaultValue": "event-hub-rule-name",
            "type": "string"
        },
        "namespacesResourceId": {
            "defaultValue": "/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/resource-group-name/providers/microsoft.eventhub/namespaces/namespaces-name",
            "type": "String"
        },
        "eventhubName": {
            "defaultValue": "event-hub-name",
            "type": "string"
        }
    },
    "variables": {},
    "resources": [
        {
            "type": "microsoft.operationalinsights/workspaces",
            "apiVersion": "2020-08-01",
            "name": "[parameters('workspaceName')]",
            "location": "[parameters('workspaceLocation')]",
            "resources": [
              {
                  "type": "microsoft.operationalinsights/workspaces/dataexports",
                  "apiVersion": "2020-08-01",
                  "name": "[concat(parameters('workspaceName'), '/', parameters('eventhubRuleName'))]",
                  "dependsOn": [
                      "[resourceId('microsoft.operationalinsights/workspaces', parameters('workspaceName'))]"
                  ],
                  "properties": {
                      "destination": {
                          "resourceId": "[parameters('namespacesResourceId')]",
                          "metaData": {
                              "eventHubName": "[parameters('eventhubName')]"
                          }
                      },
                      "tableNames": [
                          "Usage",
                          "Heartbeat"
                      ],
                      "enable": true
                  }
              }
            ]
        }
    ]
}
```

---

## <a name="view-data-export-rule-configuration"></a>Anzeigen der Konfiguration der Datenexportregel

# <a name="azure-portal"></a>[Azure portal](#tab/portal)

–

# <a name="powershell"></a>[PowerShell](#tab/powershell)

–

# <a name="azure-cli"></a>[Azure-Befehlszeilenschnittstelle](#tab/azure-cli)

Verwenden Sie den folgenden Befehl, um die Konfiguration einer Datenexportregel mithilfe der CLI anzuzeigen.

```azurecli
az monitor log-analytics workspace data-export show --resource-group resourceGroupName --workspace-name workspaceName --name ruleName
```

# <a name="rest"></a>[REST](#tab/rest)

Verwenden Sie die folgende Anforderung, um die Konfiguration einer Datenexportregel mithilfe der REST-API anzuzeigen. Die Anforderung sollte die Bearertokenautorisierung verwenden.

```rest
GET https://management.azure.com/subscriptions/<subscription-id>/resourcegroups/<resource-group-name>/providers/Microsoft.operationalInsights/workspaces/<workspace-name>/dataexports/<data-export-name>?api-version=2020-08-01
```

# <a name="template"></a>[Vorlage](#tab/json)

–

---

## <a name="disable-an-export-rule"></a>Deaktivieren einer Exportregel

# <a name="azure-portal"></a>[Azure portal](#tab/portal)

–

# <a name="powershell"></a>[PowerShell](#tab/powershell)

–

# <a name="azure-cli"></a>[Azure-Befehlszeilenschnittstelle](#tab/azure-cli)

Exportregeln können deaktiviert werden, damit Sie den Export beenden können, wenn über einen bestimmten Zeitraum hinweg keine Daten aufbewahrt werden müssen, z. B. während des Testens. Verwenden Sie den folgenden Befehl, um eine Datenexportregel mithilfe der CLI zu deaktivieren.

```azurecli
az monitor log-analytics workspace data-export update --resource-group resourceGroupName --workspace-name workspaceName --name ruleName --enable false
```

# <a name="rest"></a>[REST](#tab/rest)

Exportregeln können deaktiviert werden, damit Sie den Export beenden können, wenn über einen bestimmten Zeitraum hinweg keine Daten aufbewahrt werden müssen, z. B. während des Testens. Verwenden Sie die folgende Anforderung, um eine Datenexportregel mithilfe der REST-API zu deaktivieren. Die Anforderung sollte die Bearertokenautorisierung verwenden.

```rest
PUT https://management.azure.com/subscriptions/<subscription-id>/resourcegroups/<resource-group-name>/providers/Microsoft.operationalInsights/workspaces/<workspace-name>/dataexports/<data-export-name>?api-version=2020-08-01
Authorization: Bearer <token>
Content-type: application/json

{
    "properties": {
        "destination": {
            "resourceId": "/subscriptions/subscription-id/resourcegroups/resource-group-name/providers/Microsoft.Storage/storageAccounts/storage-account-name"
        },
        "tablenames": [
"table1",
    "table2" 
        ],
        "enable": false
    }
}
```

# <a name="template"></a>[Vorlage](#tab/json)

Exportregeln können deaktiviert werden, damit Sie den Export beenden können, wenn über einen bestimmten Zeitraum hinweg keine Daten aufbewahrt werden müssen, z. B. während des Testens. Legen Sie ```"enable": false``` in der Vorlage fest, um einen Datenexport zu deaktivieren.

---

## <a name="delete-an-export-rule"></a>Löschen einer Exportregel

# <a name="azure-portal"></a>[Azure portal](#tab/portal)

–

# <a name="powershell"></a>[PowerShell](#tab/powershell)

–

# <a name="azure-cli"></a>[Azure-Befehlszeilenschnittstelle](#tab/azure-cli)

Verwenden Sie den folgenden Befehl, um eine Datenexportregel mithilfe der CLI zu löschen.

```azurecli
az monitor log-analytics workspace data-export delete --resource-group resourceGroupName --workspace-name workspaceName --name ruleName
```

# <a name="rest"></a>[REST](#tab/rest)

Verwenden Sie die folgende Anforderung, um eine Datenexportregel mithilfe der REST-API zu löschen. Die Anforderung sollte die Bearertokenautorisierung verwenden.

```rest
DELETE https://management.azure.com/subscriptions/<subscription-id>/resourcegroups/<resource-group-name>/providers/Microsoft.operationalInsights/workspaces/<workspace-name>/dataexports/<data-export-name>?api-version=2020-08-01
```

# <a name="template"></a>[Vorlage](#tab/json)

–

---

## <a name="view-all-data-export-rules-in-a-workspace"></a>Anzeigen aller Datenexportregeln in einem Arbeitsbereich

# <a name="azure-portal"></a>[Azure portal](#tab/portal)

–

# <a name="powershell"></a>[PowerShell](#tab/powershell)

–

# <a name="azure-cli"></a>[Azure-Befehlszeilenschnittstelle](#tab/azure-cli)

Verwenden Sie den folgenden Befehl, um alle Datenexportregeln in einem Arbeitsbereich mithilfe der CLI anzuzeigen.

```azurecli
az monitor log-analytics workspace data-export list --resource-group resourceGroupName --workspace-name workspaceName
```

# <a name="rest"></a>[REST](#tab/rest)

Verwenden Sie die folgende Anforderung, um alle Datenexportregeln in einem Arbeitsbereich mithilfe der REST-API anzuzeigen. Die Anforderung sollte die Bearertokenautorisierung verwenden.

```rest
GET https://management.azure.com/subscriptions/<subscription-id>/resourcegroups/<resource-group-name>/providers/Microsoft.operationalInsights/workspaces/<workspace-name>/dataexports?api-version=2020-08-01
```

# <a name="template"></a>[Vorlage](#tab/json)

–

---

## <a name="unsupported-tables"></a>Nicht unterstützte Tabellen
Wenn die Datenexportregel eine nicht unterstützte Tabelle umfasst, wird die Konfiguration erfolgreich ausgeführt, es werden jedoch für diese Tabelle keine Daten exportiert. Wenn die Tabelle zu einem späteren Zeitpunkt unterstützt wird, werden die entsprechenden Daten dann exportiert.

Wenn die Datenexportregel eine nicht vorhandene Tabelle umfasst, tritt der Fehler „Tabelle <tableName> ist im Arbeitsbereich nicht vorhanden“ auf.


## <a name="supported-tables"></a>Unterstützte Tabellen
Die Unterstützung für Tabellen ist zurzeit auf die unten genannten beschränkt. Es werden alle Daten aus der Tabelle exportiert, sofern keine Einschränkungen angegeben werden. Diese Liste wird aktualisiert, sobald weitere Tabellen unterstützt werden.


| Tabelle | Einschränkungen |
|:---|:---|
| AADDomainServicesAccountLogon | |
| AADDomainServicesAccountManagement | |
| AADDomainServicesDirectoryServiceAccess | |
| AADDomainServicesLogonLogoff | |
| AADDomainServicesPolicyChange | |
| AADDomainServicesPrivilegeUse | |
| AADManagedIdentitySignInLogs | |
| AADNonInteractiveUserSignInLogs | |
| AADProvisioningLogs | |
| AADServicePrincipalSignInLogs | |
| ADAssessmentRecommendation | |
| ADFActivityRun | |
| ADFPipelineRun | |
| ADFTriggerRun | |
| ADReplicationResult | |
| ADSecurityAssessmentRecommendation | |
| ADTDigitalTwinsOperation | |
| ADTEventRoutesOperation | |
| ADTModelsOperation | |
| ADTQueryOperation | |
| ADXCommand | |
| ADXQuery | |
| AegDeliveryFailureLogs | |
| AegPublishFailureLogs | |
| Warnung |Teilweise unterstützt. Einige Daten für diese Tabelle werden über das Speicherkonto erfasst. Diese Daten werden zurzeit nicht exportiert. |
| Anomalien | |
| ApiManagementGatewayLogs | |
| AppCenterError | |
| AppPlatformSystemLogs | |
| AppServiceAppLogs | |
| AppServiceAuditLogs | |
| AppServiceConsoleLogs | |
| AppServiceFileAuditLogs | |
| AppServiceHTTPLogs | |
| AppServiceIPSecAuditLogs | |
| AppServicePlatformLogs | |
| AuditLogs | |
| AutoscaleEvaluationsLog | |
| AutoscaleScaleActionsLog | |
| AWSCloudTrail | |
| AzureAssessmentRecommendation | |
| AzureDevOpsAuditing | |
| BehaviorAnalytics | |
| BlockchainApplicationLog | |
| BlockchainProxyLog | |
| BlockchainProxyLog | |
| CommonSecurityLog | |
| CommonSecurityLog | |
| ComputerGroup | |
| ConfigurationData | Teilweise unterstützt. Einige Daten werden durch interne Dienste erfasst, die für den Export nicht unterstützt werden. Diese Daten werden zurzeit nicht exportiert. |
| ContainerImageInventory | |
| ContainerInventory | |
| ContainerLog | |
| ContainerNodeInventory | |
| ContainerServiceLog | |
| CoreAzureBackup | |
| DatabricksAccounts | |
| DatabricksClusters | |
| DatabricksDBFS | |
| DatabricksInstancePools | |
| DatabricksJobs | |
| DatabricksNotebook | |
| DatabricksSecrets | |
| DatabricksSQLPermissions | |
| DatabricksSSH | |
| DatabricksWorkspace | |
| DnsEvents | |
| DnsInventory | |
| Dynamics365Activity | |
| Ereignis | Teilweise unterstützt. Einige Daten für diese Tabelle werden über das Speicherkonto erfasst. Diese Daten werden zurzeit nicht exportiert. |
| ExchangeAssessmentRecommendation | |
| FailedIngestion | |
| FunctionAppLogs | |
| HDInsightAmbariClusterAlerts | |
| HDInsightAmbariSystemMetrics | |
| HDInsightGatewayAuditLogs | |
| HDInsightHadoopAndYarnLogs | |
| HDInsightHadoopAndYarnMetrics | |
| HDInsightHBaseLogs | |
| HDInsightHBaseMetrics | |
| HDInsightHiveAndLLAPLogsSample | |
| HDInsightKafkaLogs | |
| HDInsightKafkaMetrics | |
| HDInsightOozieLogs | |
| HDInsightSecurityLogs | |
| HDInsightSparkApplicationEvents | |
| HDInsightSparkBlockManagerEvents | |
| HDInsightSparkEnvironmentEvents | |
| HDInsightSparkEventsLog | |
| HDInsightSparkExecutorEvents | |
| HDInsightSparkExtraEvents | |
| HDInsightSparkJobEvents | |
| HDInsightSparkLogs | |
| HDInsightSparkSQLExecutionEvents | |
| HDInsightSparkStageEvents | |
| HDInsightSparkStageTaskAccumulables | |
| HDInsightSparkTaskEvents | |
| HDInsightStormLogs | |
| HDInsightStormMetrics | |
| HDInsightStormTopologyMetrics | |
| Heartbeat | |
| HuntingBookmark | |
| InsightsMetrics | Teilweise unterstützt. Einige Daten werden durch interne Dienste erfasst, die für den Export nicht unterstützt werden. Dieser Teil fehlt derzeit im Export. |
| IntuneAuditLogs | |
| IntuneDeviceComplianceOrg | |
| IntuneOperationalLogs | |
| KubeEvents | |
| KubeHealth | |
| KubeMonAgentEvents | |
| KubeNodeInventory | |
| KubePodInventory | |
| KubeServices | |
| KubeServices | |
| LAQueryLogs | |
| McasShadowItReporting | |
| MicrosoftAzureBastionAuditLogs | |
| MicrosoftDataShareReceivedSnapshotLog | |
| MicrosoftDataShareSentSnapshotLog | |
| MicrosoftDataShareShareLog | |
| MicrosoftHealthcareApisAuditLogs | |
| NWConnectionMonitorDestinationListenerResult | |
| NWConnectionMonitorDNSResult | |
| NWConnectionMonitorPathResult | |
| NWConnectionMonitorPathResult | |
| NWConnectionMonitorTestResult | |
| NWConnectionMonitorTestResult | |
| OfficeActivity | Teilweise unterstützt. Einige der Daten werden über Webhooks aus Office 365 in Log Analytics erfasst. Diese Daten werden zurzeit nicht exportiert. |
| Vorgang | Teilweise unterstützt. Einige Daten werden durch interne Dienste erfasst, die für den Export nicht unterstützt werden. Diese Daten werden zurzeit nicht exportiert. |
| Perf | Teilweise unterstützt. Zurzeit werden nur Windows-Leistungsdaten unterstützt. Linux-Leistungsdaten werde derzeit nicht exportiert. |
| ProtectionStatus | |
| SCCMAssessmentRecommendation | |
| SCOMAssessmentRecommendation | |
| SecurityAlert | |
| SecurityBaseline | |
| SecurityBaselineSummary | |
| SecurityDetection | |
| SecurityEvent | |
| SecurityIncident | |
| SecurityIoTRawEvent | |
| SecurityNestedRecommendation | |
| SecurityRecommendation | |
| SfBAssessmentRecommendation | |
| SfBOnlineAssessmentRecommendation | |
| SharePointOnlineAssessmentRecommendation | |
| SignalRServiceDiagnosticLogs | |
| SigninLogs | |
| SPAssessmentRecommendation | |
| SQLAssessmentRecommendation | |
| SucceededIngestion | |
| SynapseGatewayEvents | |
| SynapseRBACEvents | |
| syslog | Teilweise unterstützt. Einige Daten für diese Tabelle werden über das Speicherkonto erfasst. Diese Daten werden zurzeit nicht exportiert. |
| ThreatIntelligenceIndicator | |
| Aktualisieren | Teilweise unterstützt. Einige Daten werden durch interne Dienste erfasst, die für den Export nicht unterstützt werden. Diese Daten werden zurzeit nicht exportiert. |
| UpdateRunProgress | |
| UpdateSummary | |
| Verwendung | |
| UserAccessAnalytics | |
| UserPeerAnalytics | |
| Watchlist | |
| WindowsEvent | |
| WindowsFirewall | |
| WireData | Teilweise unterstützt. Einige Daten werden durch interne Dienste erfasst, die für den Export nicht unterstützt werden. Diese Daten werden zurzeit nicht exportiert. |
| WorkloadMonitoringPerf | |
| WVDAgentHealthStatus | |
| WVDCheckpoints | |
| WVDConnections | |
| WVDErrors | |
| WVDFeeds | |
| WVDManagement | |


## <a name="next-steps"></a>Nächste Schritte

- [Abfragen der exportierten Daten aus Azure Data Explorer](azure-data-explorer-query-storage.md)
