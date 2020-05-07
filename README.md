![ANS](./images/ans_logo_small.png)

# Student Activity PowerApp using Microsoft Teams Data
For more information on the Student Activity PowerApp visit https://www.ans.co.uk

The application is deployed a PowerApp solution, PowerPortal and an Azure Function app to collect data from the Microsoft Graph API.

## Pre-Requisites
The Azure Function requires acces to the Microsoft Graph, to collect Azure AD user information and Teams Activity Report data, this is achieved using an Azure AD App Registration in the the Tenant hosting Microsoft Teams. After the data has been collected it is stored within an Azure Data Lake Gen2 storage account.

### App Registration
To create the App Registration, 

1. logon to the Azure Portal with an account that has  **Global Administrator** permissions: https://portal.azure.com 

2. Open the **Azure Active Directory** blade.

3. Click on **App Registrations**, then click on the **+ New registraion** button.

4. Enter a name **Student Activity PowerApp**.

5. Select **Accounts in this organizational directory only (Default Directory only - Single tenant)** radio button.

6. Click **Register** button. And wait for it to complete.

7. ***IMPORTANT: Record the **Application (client) ID** and **Directory (tenant) ID**, values.***

8. Click **Certificates and secrets** from left-hand menu, then click **+ New client secret** button. ***IMPORTANT: Record the secret value, it is only displayed once.***

9. Click on **API Permisisons** from left-hand menu, then click **+ Add a permission**. Add all the permissions in the table below. 

    | Graph Permission | Permission Type |
    | --- | --- |
    | Reports.Read.All | Application |
    | Users.Read.All | Application |

10. Click on the **Grant admin consent for Default Directory** button.

### Deploy Azure Template
The Azure template will deploy an Azure Function App and an Azure Data Lake Gen2 storage account.

To deploy the Azure template clicl on the Deploy button below, this will take you to the Azure Portal to deploy the template.

[![Deploy to Azure](./images/azure_deploy.png)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Fans-group%2Fstudent-activity-powerapp-azure-template%2Fmaster%2Ftemplate%2Fazuredeploy.json)



Please follow the steps below to complete the process.

1. 

