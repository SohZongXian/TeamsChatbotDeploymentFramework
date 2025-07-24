# Provision and Publish a Python Bot with Azure CLI

This guide explains how to use the Azure CLI to create resources, prepare, and deploy a Python bot to Azure using the Bot Framework SDK v4.

> **Note**: This repository already contains the echo bot template. For other samples, use samples from the [Bot Framework Samples repository](https://github.com/microsoft/BotFramework-Samples).

> **Tip**: This guide creates an Azure Bot resource. Existing bots using Web App Bot or Bot Channels Registration resources will continue to work, but new bots cannot use these resource types.

## Alternative Options
- **Microsoft 365 Agents SDK**: Build agents with customizable AI services, orchestration, and knowledge in Python, C#, or JavaScript. Learn more at [aka.ms/agents](https://aka.ms/agents).
- **Microsoft Copilot Studio**: A SaaS-based agent platform.
- **Migration**: Update existing Bot Framework SDK bots to Agents SDK. See [Bot Framework SDK to Agents SDK migration guidance](https://learn.microsoft.com/en-us/azure/bot-service/).

## Prerequisites
- **Azure Account**: An active Azure subscription. [Create a free account](https://azure.microsoft.com/free/).
- **Azure CLI**: Install Azure CLI version **2.55.0 or later** for Python bots. See [Install the Azure CLI](https://learn.microsoft.com/en-us/cli/azure/install-azure-cli).
- **ARM Templates**: Copy the `deploymentTemplates` folder for Python to your bot project from the [Bot Framework Samples repository](https://github.com/microsoft/BotFramework-Samples).

> **Note**: Additional resources (e.g., storage or language services) must be deployed separately.

## Plan Your Deployment
Before starting, decide:
- **Identity Management**:
  - **User-assigned managed identity**: No manual credential management.
  - **Single-tenant app**: Supported in Python SDK.
  - **Multi-tenant app**: Deprecated after July 31, 2025; supported in Python SDK, Composer, Emulator, Dev Tunnels.
- **Resource Group**: Use one resource group initially for simplicity. See [Manage Azure resources](https://learn.microsoft.com/en-us/azure/azure-resource-manager/management/manage-resource-groups-portal).
- **Regional or Global Bot**: See [Regionalization in Azure AI Bot Service](https://learn.microsoft.com/en-us/azure/bot-service/).

> **Important**: Python bots cannot be deployed to resource groups with Windows services or bots. Use a separate resource group for other services.

### Supported App Types
| App Type                     | Supported In                                                                 |
|------------------------------|-----------------------------------------------------------------------------|
| User-assigned managed identity| Azure AI Bot Service; Python, C#, JavaScript SDKs                           |
| Single-tenant                | Azure AI Bot Service; Python, C#, JavaScript SDKs                           |
| Multi-tenant (Deprecated)    | Azure AI Bot Service; all SDKs, Composer, Emulator, Dev Tunnels             |

### Azure Resources Needed
- Azure subscription
- One or more resource groups
- User-assigned managed identity or Microsoft Entra ID app registration
- App Service Plan resource
- App Service resource
- Azure Bot resource

### Information to Record
| Information            | Where Generated                            | Where Used                                                                 |
|------------------------|--------------------------------------------|---------------------------------------------------------------------------|
| Tenant ID             | Sign in and select subscription            | Create App Service, Create/Update Azure Bot, Update project settings       |
| App Type              | Create identity resource                   | Create App Service, Create/Update Azure Bot, Update project settings       |
| Client ID             | Create identity resource                   | Create App Service, Create/Update Azure Bot, Update project settings       |
| Base App Service URL  | Create App Service                         | Create/Update Azure Bot                                                   |
| App Service Name      | Create App Service                         | Publish bot to Azure                                                      |

> **Caution**: Many IDs and passwords are sensitive. Follow [Bot Framework security guidelines](https://learn.microsoft.com/en-us/azure/bot-service/).

## Steps to Provision and Publish

### 1. Sign In and Select Subscription
1. Open a command window.
2. Sign in to Azure:
   ```bash
   az login
   ```
   - A browser window will open to complete sign-in.
   - On success, the command lists accessible subscriptions.
3. Set the subscription:
   ```bash
   az account set --subscription "<subscription>"
   ```
   Replace `<subscription>` with the subscription ID or name.
4. For user-assigned managed identity or single-tenant bots, record the `tenantId` from the subscription.

> **Tip**: For non-public clouds, see [Azure cloud management with the Azure CLI](https://learn.microsoft.com/en-us/cli/azure/azure-cloud).

### 2. Create Resource Groups
If needed, create a resource group:
```bash
az group create --name "<group>" --location "<region>"
```
- `name`: Resource group name.
- `location`: Region for the resource group (e.g., `eastus`).

See [Manage Azure resource groups with the Azure CLI](https://learn.microsoft.com/en-us/cli/azure/group).

### 3. Create an Identity Resource
1. Create a user-assigned managed identity:
   ```bash
   az identity create --resource-group "<group>" --name "<identity>"
   ```
   - `resource-group`: Resource group name.
   - `name`: Identity resource name.
2. Record:
   - Resource group name
   - Identity resource name
   - `clientId` from the command output

See [az identity reference](https://learn.microsoft.com/en-us/cli/azure/identity).

### 4. Create Resources with ARM Templates
Use ARM templates with the `az deployment group create` command to create:
1. **App Service Resource**: Create within a new or existing App Service Plan. See [Use Azure CLI to create an App Service](https://learn.microsoft.com/en-us/azure/app-service/quickstart-python?tabs=cli).
2. **Azure Bot Resource**: See [Use Azure CLI to create or update an Azure Bot](https://learn.microsoft.com/en-us/azure/bot-service/provision-and-publish-a-bot?view=azure-bot-service-4.0&tabs=singletenant%2Cpython#provision-the-bot-service).

> **Important**: You can create these in any order, but if the Azure Bot is created first, update its messaging endpoint after creating the App Service.

### 5. Update Project Configuration Settings
Add identity information to your bot’s `config.py` file:
| Property                | Value                                            |
|-------------------------|--------------------------------------------------|
| `MicrosoftAppType`      | `"UserAssignedMSI"`                              |
| `MicrosoftAppId`        | Client ID of the user-assigned managed identity  |
| `MicrosoftAppPassword`  | Leave blank (e.g., `""`)                         |
| `MicrosoftAppTenantId`  | Tenant ID of the user-assigned managed identity  |

Example `config.py`:
```python
MicrosoftAppType = "UserAssignedMSI"
MicrosoftAppId = "<client-id>"
MicrosoftAppPassword = ""
MicrosoftAppTenantId = "<tenant-id>"
```

### 6. Prepare Your Project Files
1. Switch to your project’s root folder (containing `config.py`).
2. Ensure all dependencies are installed (e.g., run `pip install -r requirements.txt`).
4. Create a zip file containing all files and subfolders in the root folder.

### 7. Publish Your Bot to Azure
Deploy the zip file to your App Service:
```bash
az webapp deploy --resource-group "<group>" --name "<app-service-name>" --src "<path-to-zip>"
```
- `resource-group`: Resource group name.
- `name`: App Service name.
- `src`: Path to the zipped project file.

> **Note**: Deployment may take a few minutes, and the bot may take additional time to be available.

> **Tip**: Use the `--slot` parameter to deploy to a non-production slot. See [az webapp deploy](https://learn.microsoft.com/en-us/cli/azure/webapp#az-webapp-deploy).

### 8. Test in Web Chat
1. In the [Azure portal](https://portal.azure.com/), go to your bot resource.
2. Open the **Test in Web Chat** pane.
3. Interact with your deployed bot.

See [Register a bot with Bot Service](https://learn.microsoft.com/en-us/azure/bot-service/).

## Clean Up Resources
If you don’t plan to publish the bot:
1. In the [Azure portal](https://portal.azure.com/), open the resource group and select **Delete resource group**.
   - Confirm by entering the resource group name and select **Delete**.
2. For single-tenant or multi-tenant apps:
   - Go to the **Microsoft Entra ID** blade in the [Azure portal](https://portal.azure.com/).
   - Locate and delete the app registration used for your bot.

## Additional Resources
| Subject                          | Article                                                                 |
|----------------------------------|------------------------------------------------------------------------|
| Azure CLI                        | [What is the Azure CLI?](https://learn.microsoft.com/en-us/cli/azure/) |
| Azure subscription management    | [Manage Azure subscriptions with the Azure CLI](https://learn.microsoft.com/en-us/cli/azure/account) |
| Azure regions                    | [Regions and availability zones](https://learn.microsoft.com/en-us/azure/availability-zones/) |
| Resource groups                  | [Manage Azure resources](https://learn.microsoft.com/en-us/azure/azure-resource-manager/management/manage-resource-groups-portal) |
| Managed identities               | [What are managed identities for Azure resources?](https://learn.microsoft.com/en-us/azure/active-directory/managed-identities-azure-resources/) |
| Single-tenant and multi-tenant apps | [Tenancy in Microsoft Entra ID](https://learn.microsoft.com/en-us/azure/active-directory/develop/single-and-multi-tenant-apps) |
| Web applications                 | [App Service](https://learn.microsoft.com/en-us/azure/app-service/) |
| Compute resources for web apps   | [App Service plans](https://learn.microsoft.com/en-us/azure/app-service/overview-hosting-plans) |
| ARM templates                    | [What are ARM templates?](https://learn.microsoft.com/en-us/azure/azure-resource-manager/templates/) and [How to use ARM templates with Azure CLI](https://learn.microsoft.com/en-us/cli/azure/deployment) |
| Azure billing                    | [Billing and cost management](https://learn.microsoft.com/en-us/azure/cost-management-billing/) |

### Kudu Files
The `az webapp deploy` command uses Kudu for Python bots. Ensure the zip file includes all built code and dependencies (e.g., from `requirements.txt`), as Kudu assumes no additional build steps (e.g., `pip install`). See [Deploy files to App Service](https://learn.microsoft.com/en-us/azure/app-service/deploy-zip).


### Next Steps 
Please refer to [Publish Bot To Teams Channel](https://learn.microsoft.com/en-us/MicrosoftTeams/manage-apps) to publish bot to MS Teams.