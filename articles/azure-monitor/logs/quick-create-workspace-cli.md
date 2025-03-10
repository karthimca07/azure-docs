---
title: Create a Log Analytics workspace using Azure CLI | Microsoft Docs
description: Learn how to create a Log Analytics workspace to enable management solutions and data collection from your cloud and on-premises environments with Azure CLI.
ms.topic: conceptual
author: bwren
ms.author: bwren
ms.date: 05/26/2020

---

# Create a Log Analytics workspace with Azure CLI 2.0

The Azure CLI 2.0 is used to create and manage Azure resources from the command line or in scripts. This quickstart shows you how to use Azure CLI 2.0 to deploy a Log Analytics workspace in Azure Monitor. A Log Analytics workspace is a unique environment for Azure Monitor log data. Each workspace has its own data repository and configuration, and data sources and solutions are configured to store their data in a particular workspace. You require a Log Analytics workspace if you intend on collecting data from the following sources:

* Azure resources in your subscription  
* On-premises computers monitored by System Center Operations Manager  
* Device collections from Configuration Manager  
* Diagnostic or log data from Azure storage  


[!INCLUDE [quickstarts-free-trial-note](../../../includes/quickstarts-free-trial-note.md)]

[!INCLUDE [azure-cli-prepare-your-environment.md](../../../includes/azure-cli-prepare-your-environment.md)]

- This article requires version 2.0.30 or later of the Azure CLI. If using Azure Cloud Shell, the latest version is already installed.

## Create a workspace
Create a workspace with [az deployment group create](/cli/azure/deployment/group#az_deployment_group_create). The following example creates a workspace in the *eastus* location using a Resource Manager template from your local machine. The  JSON template is configured to only prompt you for the name of the workspace, and specifies a default value for the other parameters that would likely be used as a standard configuration in your environment. Or you can store the template in an Azure storage account for shared access in your organization. For further information about working with templates, see [Deploy resources with Resource Manager templates and Azure CLI](../../azure-resource-manager/templates/deploy-cli.md)

For information about regions supported, see [regions Log Analytics is available in](https://azure.microsoft.com/regions/services/) and search for Azure Monitor from the **Search for a product** field.

The following parameters set a default value:

* location - defaults to East US
* sku - defaults to the new Per-GB pricing tier released in the April 2018 pricing model

>[!WARNING]
>If creating or configuring a Log Analytics workspace in a subscription that has opted into the new April 2018 pricing model, the only valid Log Analytics pricing tier is **PerGB2018**.
>

### Create and deploy template

1. Copy and paste the following JSON syntax into your file:

    ```json
    {
    "$schema": "https://schema.management.azure.com/schemas/2014-04-01-preview/deploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {
        "workspaceName": {
            "type": "String",
            "metadata": {
              "description": "Specifies the name of the workspace."
            }
        },
        "location": {
            "type": "String",
            "allowedValues": [
              "eastus",
              "westus"
            ],
            "defaultValue": "eastus",
            "metadata": {
              "description": "Specifies the location in which to create the workspace."
            }
        },
        "sku": {
            "type": "String",
            "allowedValues": [
              "Standalone",
              "PerNode",
              "PerGB2018"
            ],
            "defaultValue": "PerGB2018",
            "metadata": {
            "description": "Specifies the service tier of the workspace: Standalone, PerNode, Per-GB"
        }
          }
    },
    "resources": [
        {
            "type": "Microsoft.OperationalInsights/workspaces",
            "name": "[parameters('workspaceName')]",
            "apiVersion": "2015-11-01-preview",
            "location": "[parameters('location')]",
            "properties": {
                "sku": {
                    "Name": "[parameters('sku')]"
                },
                "features": {
                    "searchVersion": 1
                }
            }
          }
       ]
    }
    ```

2. Edit the template to meet your requirements. Review [Microsoft.OperationalInsights/workspaces template](/azure/templates/microsoft.operationalinsights/2015-11-01-preview/workspaces) reference to learn what properties and values are supported.
3. Save this file as **deploylaworkspacetemplate.json** to a local folder.   
4. You are ready to deploy this template. Use the following commands from the folder containing the template. When you're prompted for a workspace name, provide a name that is unique in your resource group.

    ```azurecli
    az deployment group create --resource-group <my-resource-group> --name <my-deployment-name> --template-file deploylaworkspacetemplate.json
    ```

The deployment can take a few minutes to complete. When it finishes, you see a message similar to the following that includes the result:

![Example result when deployment is complete](media/quick-create-workspace-cli/template-output-01.png)

## Troubleshooting
When you create a workspace that was deleted in the last 14 days and in [soft-delete state](../logs/delete-workspace.md#soft-delete-behavior), the operation could have different outcome depending on your workspace configuration:
1. If you provide the same workspace name, resource group, subscription and region as in the deleted workspace, your workspace will be recovered including its data, configuration and connected agents.
2. Workspace name must be unique per resource group. If you use a workspace name that is already exists, also in soft-delete in your resource group, you will get an error *The workspace name 'workspace-name' is not unique*, or *conflict*. To override the soft-delete and permanently delete your workspace and create a new workspace with the same name, follow these steps to recover the workspace first and perform permanent delete:
   * [Recover](../logs/delete-workspace.md#recover-workspace) your workspace
   * [Permanently delete](../logs/delete-workspace.md#permanent-workspace-delete) your workspace
   * Create a new workspace using the same workspace name

## Next steps
Now that you have a workspace available, you can configure collection of monitoring telemetry, run log searches to analyze that data, and add a management solution to provide additional data and analytic insights.  

* To enable data collection from Azure resources with Azure Diagnostics or Azure storage, see [Collect Azure service logs and metrics for use in Log Analytics](../essentials/resource-logs.md#send-to-log-analytics-workspace).  
* Add [System Center Operations Manager as a data source](../agents/om-agents.md) to collect data from agents reporting your Operations Manager management group and store it in your Log Analytics workspace.  
* Connect [Configuration Manager](../logs/collect-sccm.md) to import computers that are members of collections in the hierarchy.  
* Review the [monitoring solutions](../insights/solutions.md) available and how to add or remove a solution from your workspace.

