![ANS](./images/ans_logo_small.png)

# Student Activity PowerApp using Microsoft Teams Data

For more information on the Student Activity PowerApp visit https://www.ans.co.uk

The application is deployed a PowerApp solution, PowerPortal and an Azure Function app to collect data from the Microsoft Graph API.

## Azure Pre-Requisites

The Azure Function requires acces to the Microsoft Graph, to collect Azure AD user information and Teams Activity Report data, this is achieved using an Azure AD App Registration in the the Tenant hosting Microsoft Teams. After the data has been collected it is stored within an Azure Data Lake Gen2 storage account.

The soluion requires you have the following resources and permissions, during the installation.

* A user with Global Administrator privileges in the Azure Active Directory Tenant that Micosoft Teams is associated to.
* A Microsoft Azure Subscription and permissions to the subscription of Owner.
* A Microsoft PowerApps Environment with available licenses.
  * PowerApps User/App licensing
  * Power Portal license
  * Power Portal Page View license
  * Common Data Service Capacity (Minimum of 2.5Gb)

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

1. To deploy the Azure template click on the Deploy button below, this will take you to the Azure Portal to deploy the template.

    [![Deploy to Azure](./images/azure_deploy.png)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Fans-group%2Fstudent-activity-powerapp-azure-template%2Fmaster%2Ftemplate%2Fazuredeploy.json)


2. Complete the information template form.
    * **Resource Group:** Create a new resource group to hows the application.
    * **Prefix:** Enter a prefix that will be prepended to the resources names.
    * **Tenant Id:** Enter the Tenant Id that was recorded earlier.
    * **Client Id:** Enter the Client Id that was recorded earlier.
    * **Client Secret Id:** Enter the Client Secret that was recorded earlier.

3. Check **I agree to the terms and conditions stated above**

4. Click on the **Purchase** button.

### Generate Intial Export and SAS Urls

To allow PowerApps Dataflows to import the Teams Activity data, you need to run an initial export and generate SAS Url's for both of the export files, these SAS Url's are used within the PowerApps Dataflow to import the data into the Common Data Service

1. In the Azure Portal open the new Azure Function app, you will find this in the list of resources that have been provisioned by the template.

2. In the left-hand menu expand the **Functions** and click on **collectActivityData**, click on the **Run** button. A log window will appear at the bottom of the screen, look for a Succeeded message at the end of the log. If you see red text wait 5 minutes anf try again, the function may still be importing modules. If the red text continues check the error messages, there may be a problem with the App Registration you created earlier.

3. Go back to the Azure Portal Home page and open Storage Accounts, in the list of storage account locate the account with name ending **lakesa*, then click to open.

4. In the middle of the page click on **Containers**, there should be two **teams-json** and **user-json**. Click on **teams-json**, there should be a file **teams_activity_data.json**, now click on the file name to open.

5. Towards the top of the page, click on the **Generate SAS** button, look for the **Expiry** field in the middle of the page and change the date, this is when the SAS Url is valid to, so set it at least 1 year into the future.

6. Click on the **Generate SAS token and URL** button, then record the **Blob SAS URL** from the bottom of the page, also note which file this is for.

7. Now go back to the Azure Portal Home page and open Storage Accounts, in the list of storage account locate the account with name ending **lakesa*, then click to open.

8. In the middle of the page click on **Containers**, click on **user-json**, there should be a file **user_data.json**, now click on the file name to open.

9. Towards the top of the page, click on the **Generate SAS** button, look for the **Expiry** field in the middle of the page and change the date, this is when the SAS Url is valid to, so set it at least 1 year into the future.

10. Click on the **Generate SAS token and URL** button, then record the **Blob SAS URL** from the bottom of the page, also note which file this is for.

### Azure AD Security Group

An Azure AD Security Group is required to limit the users that have access to the Power Platform environment. To create the Azure AD Group follow the steps below.

To create the Azure AD Security Group,

1. logon to the Azure Portal with an account that has  **Global Administrator** permissions: https://portal.azure.com 

2. Open the **Azure Active Directory** blade.

3. Click on **Groups**, then click on the **+ New group** button.

4. Set **Group type** to **Security**

5. Complete the **Group name** and **Group description**

6. Add at least your Admin user as a Member of the Group.

7. Click **Create**

### Create a PowerApps Environment

The application should be deployed into a dedicated PowerApps Environment, to create a new PowerApps Environment follow the steps below, it is important the new Environment is attached to a security group to restrict user access, pay special attention to this setting.

1. Login to https://admin.powerplatform.microsoft.com/environments 

2. Click on the **+ New** button and create new Environment. Use a name that makes it easy to identify the environments purpose and include Prod or Dev to identify if it is production or development.  

    * **Name:** ***Student Engagement - Prod***
    * **Type:** Production
    * **Region:** United Kingdom
    * **Create a database:** Yes

    * **Language:** English
    * **Currency:** GBP (£)
    * **Security Group:** ***Group created in previous section***

### PowerApps Portal Base Deployment

The solution requires a PowerApps portal, The base PowerApps Portal must be installed before the PowerApp Solution is imported, follow the steps below to complete this.

1. Open https://make.powerapps.com

2. From the top menu select the environment that the portal will be deployed into, this is the one we created in the previous section.

3. From the left hand menu click on **Apps**, then click on the **+ New app** menu at the top of the page and select **Portal**.

4. Complete the required fields

    * **Name:** Student Engagement Portal
    * **Address:**  Enter a meaningfull name
    * **Language:** English  

    Wait for the new Student Engagement Portal and the Portal Management apps to deploy, this will take around 15 minutes to complete.  

### Deploy PowerAppS Solution

The PowerApp solution

1. Download the Student Engagement PowerApps solution zip file here: https://github.com/ans-group/student-activity-powerapp-azure-template/raw/master/powerapp/student-activity-powerapp-solution.zip

2. Open https://make.powerapps.com

3. From the top menu select the environment that the solution will be deployed into, this is the one we created earlier.

4. On the left-hand menu click on **Solutions**, then from the top menu click on **Import**, a new window will open.

5. Click on **Choose File** and select the file you downloaded in step 1, then click **Next**

6. 

