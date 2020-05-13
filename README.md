![ANS](./images/ans_logo_small.png)

# Student Activity PowerApp using Microsoft Teams Data

For more information on the Student Activity PowerApp visit https://www.ans.co.uk

The application is deployed a PowerApp solution, PowerApps Portal and an Azure Function app to collect data from the Microsoft Graph API.

## Azure Pre-Requisites

The Azure Function requires access to the Microsoft Graph, to collect Azure AD user information and Teams Activity Report data, this is achieved using an Azure AD App Registration in the Azure Tenant hosting Microsoft Teams. After the data has been collected it is stored within an Azure Data Lake Gen2 storage account.

The solution requires you have the following resources and permissions, during the installation.

* A user with Global Administrator privileges in the Azure Active Directory Tenant that Microsoft Teams is associated to.
* A Microsoft Azure Subscription and permissions to the subscription of Owner.
* A Microsoft PowerApps Environment with available licenses.
  * PowerApps User/App licensing
  * Power Portal license
  * Power Portal Page View license
  * Common Data Service Capacity (Minimum of 2Gb)

### App Registration

To create the App Registration,

1. logon to the Azure Portal with an account that has  **Global Administrator** permissions: https://portal.azure.com 

2. Open the **Azure Active Directory** blade.

3. Click on **App Registrations**, then click on the **+ New registration** button.

4. Enter a name **Student Activity PowerApp**.

5. Select **Accounts in this organizational directory only (Default Directory only - Single tenant)** radio button.

6. Click **Register** button. And wait for it to complete.

7. ***IMPORTANT: Record the **Application (client) ID** and **Directory (tenant) ID**, values.***

8. Click **Certificates and secrets** from left-hand menu, then click **+ New client secret** button. ***IMPORTANT: Record the secret value, it is only displayed once.***

9. Click on **API Permissions** from left-hand menu, then click **+ Add a permission**. Add all the permissions in the table below. 

    | Graph Permission | Permission Type |
    | --- | --- |
    | Reports.Read.All | Application |
    | Users.Read.All | Application |

10. Click on the **Grant admin consent for Default Directory** button.

### Deploy Azure Template

The Azure template will deploy an Azure Function App and an Azure Data Lake Gen2 storage account.

1. To deploy the Azure template, click on the Deploy button below, this will take you to the Azure Portal to deploy the template.

    [![Deploy to Azure](./images/azure_deploy.png)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Fans-group%2Fstudent-activity-powerapp-azure-template%2Fmaster%2Ftemplate%2Fazuredeploy.json)

2. Complete the information template form.
    * **Resource Group:** Create a new resource group to host the application.
    * **Prefix:** Enter a prefix that will be prepended to the resource’s names.
    * **Tenant Id:** Enter the Tenant Id that was recorded earlier.
    * **Client Id:** Enter the Client Id that was recorded earlier.
    * **Client Secret Id:** Enter the Client Secret that was recorded earlier.

3. Check **I agree to the terms and conditions stated above**

4. Click on the **Purchase** button.

### Generate Initial Export and SAS URLs

To allow PowerApps Dataflows to import the Teams Activity data, you need to run an initial export and generate SAS URL’s for both of the export files, these SAS URL’s are used within the PowerApps Dataflow to import the data into the Common Data Service

1. In the Azure Portal open the new Azure Function app, you will find this in the list of resources that have been provisioned by the template.

2. In the left-hand menu expand the **Functions** and click on **collectActivityData**, click on the **Run** button. A log window will appear at the bottom of the screen, look for a Succeeded message at the end of the log. If you see red text wait 5 minutes and try again, the function may still be importing modules. If the red text continues check the error messages, there may be a problem with the App Registration you created earlier.

3. Go back to the Azure Portal Home page and open Storage Accounts, in the list of storage account locate the account with name ending **lakesa**, then click to open.

4. In the middle of the page click on **Containers**, there should be two **teams-json** and **user-json**. Click on **teams-json**, there should be a file **teams_activity_data.json**, now click on the file name to open.

5. Towards the top of the page, click on the **Generate SAS** button, look for the **Expiry** field in the middle of the page and change the date, this is when the SAS URL is valid to, so set it at least 1 year into the future.

6. Click on the **Generate SAS token and URL** button, then record the **Blob SAS URL** from the bottom of the page, also note which file this is for.

7. Now go back to the Azure Portal Home page and open Storage Accounts, in the list of storage account locate the account with name ending **lakesa**, then click to open.

8. In the middle of the page click on **Containers**, click on **user-json**, there should be a file **user_data.json**, now click on the file name to open.

9. Towards the top of the page, click on the **Generate SAS** button, look for the **Expiry** field in the middle of the page and change the date, this is when the SAS URL is valid to, so set it at least 1 year into the future.

10. Click on the **Generate SAS token and URL** button, then record the **Blob SAS URL** from the bottom of the page, also note which file this is for.

### Azure AD Security Group

An Azure AD Security Group is required to limit the users that have access to the Power Platform environment. To create the Azure AD Group, follow the steps below.

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

The solution requires a PowerApps portal, the base PowerApps Portal must be installed before the PowerApp Solution is imported, follow the steps below to complete this.

1. Open https://make.powerapps.com

2. From the top menu select the environment that the portal will be deployed into, this is the one we created in the previous section.

3. From the left hand menu click on **Apps**, then click on the **+ New app** menu at the top of the page and select **Portal**.

4. Complete the required fields

    * **Name:** Student Engagement Portal
    * **Address:**  Enter a meaningful name and record the full URL for later.
    * **Language:** English  

    Wait for the new Student Engagement Portal and the Portal Management apps to deploy, this will take around 15 minutes to complete.  

### Deploy PowerApps Solution

The PowerApp solution

1. Download the Student Engagement PowerApps solution zip file here: https://github.com/ans-group/student-activity-powerapp-azure-template/raw/master/powerapp/student-activity-powerapp-solution.zip

2. Open https://make.powerapps.com

3. From the top menu select the environment that the solution will be deployed into, this is the one we created earlier.

4. On the left-hand menu click on **Solutions**, then from the top menu click on **Import**, a new window will open.

5. Click on **Choose File** and select the file you downloaded in step 1, then click **Next** button.

6. Click **Next** button again, then click **Import** button and wait for solution customisations to import.

7. If you receive a warning that relates to the below two items, it can safely be ignored.
    * Inbound Bot Webhook
    * Send email on new request

8. Click on the **Close** button, to complete, and **StudentActivity** should now appear in the list of solutions

### Update Power Automate Flow connections

To update the Power Automate Flow connections.

1. Open https://make.powerapps.com

2. From the top menu select the environment that the solution will be deployed into, this is the one we created earlier.

3. Click on the **StudentActivity** solution, then find **Send email on new Request** in the list of solution components and click on it. This will open a Power Automate window.

4. On the top menu click on **Edit**, each component in the Flow will be displaying a warning triangle click on the top component, to open it.

5. Click on **+Add new connection** link, it should automatically sign you in and create a new connection.

6. Click on the second component to open it, then click on **Common Data Service link** to re-use the same connection created on the first component.

7. Click on the third component to open it, then click on **Common Data Service link** to re-use the same connection created on the first component.

8. Click on the fourth component to open it, on the sign-in page login with an outlook account, this account will be used to send emails to the students, so you may want to setup a new email address for this purpose.

9. In the **To:** field, click on the **X** to remove the sample email address, if you are testing you may want to enter your own email address, if you’re ready to send notification to the students then click on **Add dynamic content**, from the list click on **Email** under the heading of **Get Contact Record**, this is going to use the email address stored in the student contact.

10. In the **Body** field update the Power Portal link, carefully updating the link to match the URL copied earlier during the PowerApps Portal Base Deployment, step 4.

11. Click on **Save** button and wait for save to complete.

12. Click on the back arrow at the top of the screen, then on the top menu click on **Turn on** button.

13. Return to the solution then find, **Convert Non Responded Requests to Tasks** in the list of solution components and click on it. This will open a PowerAutomate window.

14. On the top menu click on **Edit**, the second component in the Flow will be displaying a warning triangle, click on the component to open it, then click on **Common Data Service link** to re-use the connection.

15. Click on the third component to expand it, then click on each of the subcomponents with a warning triangle, for each of these click on **Common Data Service link** to re-use the connection.

16. Click on **Save** button and wait for save to complete.

17. Click on the back arrow at the top of the screen, then on the top menu click on **Turn on** button.

18. Return to the solution then find, **Inbound Bot Webhook** in the list of solution components and click on it. This will open a PowerAutomate window.

19. On the top menu click on **Edit**, the second component in the Flow will be displaying a warning triangle, click on the component to open it, then click on **Common Data Service link** to re-use the connection.

20. Click on the third component to expand it, then click on each of the subcomponents with a warning triangle, for each of these click on **Common Data Service link** to re-use the connection.

21. Click on **Save** button and wait for save to complete.

22. Click on the back arrow at the top of the screen, then on the top menu click on **Turn on** button.

23. Return to the solution then find, **Create Request when Low Engagement is Yes** in the list of solution components and click on it. This will open a PowerAutomate window.

24. On the top menu click on **Edit**, then click on the **Continue** button, then click on **Save** button.

25. Now click on the back arrow at the top of the screen, then on the top menu click on **Turn on** button.

26. Return to the solution then find, **Create Task from Request** in the list of solution components and click on it. This will open a PowerAutomate window.

27. On the top menu click on **Edit**, then click on the **Continue** button, then click on **Save** button.

28. Now click on the back arrow at the top of the screen, then on the top menu click on **Turn on** button.

### Configure Power Portal

To configure the Power Portal,

1. Open https://make.powerapps.com

2. From the top menu select the environment that the solution will be deployed into, this is the one we created earlier.

3. Click on **Apps** from the left-hand menu, find **Student Engagement Portal** in the list then click on the **...** to the right of the name, on the popup menu click on **Edit**

4. When the editor window opens, click on **+ New page**, then **Blank** to create a new page.

5. On the Component menu at the right hand side, complete the form as detailed below.
    * **Name:** Request
    * **Partial URL:** request

6. Click on the first section component in the page, this is located directly under the header component.

7. From the left-hand menu click on the second item down **Components**, then click on **Form**.

8. On the Component menu at the right hand side, complete the form as detailed below.
    * **Name:** Request
    * **Entity:** Request (ans_request)
    * **Form Layout:** Web Update Form
    * **Mode:** Edit

9. Click anywhere in the page to save the settings.

10. From the left-hand menu click on the top item **Pages**, find **Request** then click on the **...**, on the popup menu click on **Hide in default menu**

11. Now click on the **Sync Configuration** button at the top right of the page. Once this has completed the Portal is ready to allow users to respond to requests.

12. To apply branding and further customise the Portal, reference the Microsoft documentation here: https://docs.microsoft.com/en-us/powerapps/maker/portals/overview

### Enabling User Access

To enable users access to the PowerApp Student Activity application, users must be added to PowerApps Security Roles.

To enable users follwo the steps below,

1. Open https://make.powerapps.com

2. From the top menu select the environment that the solution will be deployed into, this is the one we created earlier.

3. Click on **Apps** from the left-hand menu, find **Student Manager** in the list, then click on the **...** to the right of the name, on the popup menu click on **Edit**

