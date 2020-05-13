![ANS](./images/ans_logo_small.png)

# Student Activity PowerApp using Microsoft Teams Data

For more information on the Student Activity PowerApp visit https://www.ans.co.uk

The application is deployed a PowerApp solution, PowerApps Portal and an Azure Function app to collect data from the Microsoft Graph API.

## Azure Pre-Requisites

The Azure Function requires access to the Microsoft Graph, to collect Azure AD user information and Teams Activity Report data, this is achieved using an Azure AD App Registration in the Azure Tenant hosting Microsoft Teams. After the data has been collected it is stored within an Azure Data Lake Gen2 storage account.

The solution requires you have the following resources and permissions, during the installation.

- A user with Global Administrator privileges in the Azure Active Directory Tenant that Microsoft Teams is associated to.
- A Microsoft Azure Subscription and permissions to the subscription of Owner.
- A Microsoft PowerApps Environment with available licenses.
  - PowerApps User/App licensing
  - Power Portal license
  - Power Portal Page View license
  - Common Data Service Capacity (Minimum of 2Gb)

### App Registration

To create the App Registration,

1. logon to the Azure Portal with an account that has **Global Administrator** permissions: https://portal.azure.com

2. Open the **Azure Active Directory** blade.

3. Click on **App Registrations**, then click on the **+ New registration** button.

4. Enter a name **Student Activity PowerApp**.

5. Select **Accounts in this organizational directory only (Default Directory only - Single tenant)** radio button.

6. Click **Register** button. And wait for it to complete.

7. **\*IMPORTANT: Record the **Application (client) ID** and **Directory (tenant) ID**, values.\***

8. Click **Certificates and secrets** from left-hand menu, then click **+ New client secret** button. **_IMPORTANT: Record the secret value, it is only displayed once._**

9. Click on **API Permissions** from left-hand menu, then click **+ Add a permission**. Add all the permissions in the table below.

   | Graph Permission | Permission Type |
   | ---------------- | --------------- |
   | Reports.Read.All | Application     |
   | Users.Read.All   | Application     |

10. Click on the **Grant admin consent for Default Directory** button.

### Deploy Azure Template

The Azure template will deploy an Azure Function App and an Azure Data Lake Gen2 storage account.

1. To deploy the Azure template, click on the Deploy button below, this will take you to the Azure Portal to deploy the template.

   [![Deploy to Azure](./images/azure_deploy.png)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Fans-group%2Fstudent-activity-powerapp-azure-template%2Fmaster%2Ftemplate%2Fazuredeploy.json)

2. Complete the information template form.

   - **Resource Group:** Create a new resource group to host the application.
   - **Prefix:** Enter a prefix that will be prepended to the resource’s names.
   - **Tenant Id:** Enter the Tenant Id that was recorded earlier.
   - **Client Id:** Enter the Client Id that was recorded earlier.
   - **Client Secret Id:** Enter the Client Secret that was recorded earlier.

3. Check **I agree to the terms and conditions stated above**

4. Click on the **Purchase** button, then wait 3-4 minutes for the deployment to complete.

5. Once complete click on **Outputs** on the left-hand menu, and record the URL value for **azureDataLkaeGen2**, this is required when configuring the PowerApps Dataflow to import the data into the Common Data Service later.

### Generate Initial Export

To allow PowerApps Dataflows to import the Teams Activity data, you need to run an initial export of the Azure AD and Teams data by running the Azure Function.

1. In the Azure Portal open the new Azure Function app, you will find this in the list of resources that have been provisioned by the template.

2. In the left-hand menu expand the **Functions** and click on **collectActivityData**, click on the **Run** button. A log window will appear at the bottom of the screen, look for a Succeeded message at the end of the log. If you see red text wait 5 minutes and try again, the function may still be importing modules. If the red text continues check the error messages, there may be a problem with the App Registration you created earlier.

### Azure AD Data Lake Role

An Azure AD User account is used to connect the PowerApp Dataflow to the Azure Data Lake, this user account must be assigned to at least the **Storage Blob Data Reader** role on the Data Lake storage account. Even a Global Administrator does not have permissions to the data without this additional role.

To assign the role,

1. logon to the Azure Portal with an account that has **Global Administrator** permissions: https://portal.azure.com

2. Open the **Storage accounts** blade.

3. Find the newly provisioned data lake storage account within the list, it should end **lakesa**, then click on it.

4. On the left-hand menu click on **Access Control (IAM)**

5. Click on **+ Add**, then **Add role assignment**

6. On the **Add role assignment** blade, for **Role** select the **Storage Blob Data Reader** or **Storage Blob Data Contributor** role.

7. Set **Assign access to** to **Azure AD user, group or service principle**

8. Select the user account you are going to use to connect the Power Platform Dataflow to the Azure Data Lake.

9. Click **Save** button.

### Azure AD Security Group

An Azure AD Security Group is required to limit the users that have access to the Power Platform environment. To create the Azure AD Group, follow the steps below.

To create the Azure AD Security Group,

1. logon to the Azure Portal with an account that has **Global Administrator** permissions: https://portal.azure.com

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

   - **Name:** **_Student Engagement - Prod_**
   - **Type:** Production
   - **Region:** United Kingdom
   - **Create a database:** Yes

   - **Language:** English
   - **Currency:** GBP (£)
   - **Security Group:** **_Group created in previous section_**

### PowerApps Portal Base Deployment

The solution requires a PowerApps portal, the base PowerApps Portal must be installed before the PowerApp Solution is imported, follow the steps below to complete this.

1. Open https://make.powerapps.com

2. From the top menu select the environment that the portal will be deployed into, this is the one we created in the previous section.

3. From the left hand menu click on **Apps**, then click on the **+ New app** menu at the top of the page and select **Portal**.

4. Complete the required fields

   - **Name:** Student Engagement Portal
   - **Address:**  Enter a meaningful name and record the full URL for later.
   - **Language:** English

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

   - Inbound Bot Webhook
   - Send email on new request

8. Click on the **Close** button, to complete, and **ANS_StudentActivity** should now appear in the list of solutions

### Configure PowerApps Dataflows

PowerApps Dataflows are used to import the Students and Activity from the Azure Data Lake into the Common data Service, follow the below steps to configure the Dataflows.

1. Open https://make.powerapps.com

2. From the top menu select the environment that the solution will be deployed into, this is the one we created earlier.

3. Click on the left-hand menu click on **Data**, then click on **Dataflows**.

4. Click on the **...** to the right of the **Import Students DataLake Gen2** Dataflow, then click **Edit** on the dropdown menu.

5. On the left-hand menu under **Queries**, click on **AzureDataLakeGen2** then in change the **Current Value** to match the value recorded at Step 5 of deploying the Azure Template.

6. Click on **StudentData** in the left-hand menu, then click on **Configure connection** button at the right of the yellow box.

7. Enter **AzureDataLakeGen2** as the connection name, then click **Sign in**

8. Enter the credentials of an account that has at least the **Storage Blob Data Reader** role assigned on the Azure Data Lake Storage account.

9. Click on the **Connect** button, you should now see a table containing your Student data, if your data is there click on the **Next** button.

10. Under **Load settings**, click on **Load to existing entity** radio button, from the entity dropdown select **Contact**

11. Check the **Delete rows that no longer exist in the query output** if you want deleted records to be removed.

12. Under **Field mapping** configure the mappings listed in the below table using the dropdown menus.

   | Source Column         | Destination Field |
   | --------------------- | ----------------- |
   | Column1.usageLocation | Address1_country  |
   | Column1.mail          | EmailAddress1     |
   | Column1.givenName     | FirstName         |
   | Column1.displayName   | FullName          |
   | Column1.surname       | LastName          |

13. Under **Queries** on the left, click on **AzureDataLakeGen2**.

14. Under **Load settings**, click on **Do not load** radio button, then click on **Next** button.

15. Select **Refresh automatically** radio button.

16. Set **Refresh every** to 1 Days and set the time to 00:00, then click **Create** button.

17. Click on the **...** to the right of the **Import Teams Data DataLake Gen2** Dataflow, then click **Edit** on the dropdown menu.

18. On the left-hand menu under **Queries**, click on **AzureDataLakeGen2** then in change the **Current Value** to match the value recorded at Step 5 of deploying the Azure Template.

19. Click on **TeamsActivityData** in the left-hand menu, then click on **Configure connection** button at the right of the yellow box.

20. Enter **AzureDataLakeGen2** as the connection name, then click **Sign in**

21. Enter the credentials of an account that has at least the **Storage Blob Data Reader** role assigned on the Azure Data Lake Storage account.

22. Click on the **Connect** button, you should now see a table containing your Student data, if your data is there click on the **Next** button.

23. Under **Load settings**, click on **Load to existing entity** radio button, from the entity dropdown select **Contact**

24. Leave the **Delete rows that no longer exist in the query output** unchecked.

25. Under **Field mapping** configure the mappings listed in the below table using the dropdown menus.

   | Source Column                   | Destination Field           |
   | ------------------------------- | --------------------------- |
   | Column1.callCount               | ans_CallCount               |
   | Column1.lastActivityDate        | ans_LastActivityDate        |
   | Column1.meetingCount            | ans_MeetingCount            |
   | Column1.privateChatMessageCount | ans_PrivateChatMessageCount |
   | Column1.reportRefreshDate       | ans_reportRefreshDate       |
   | Column1.teamChatMessageCount    | ans_teamChatMessageCount    |
   | Column1.mail                    | EmailAddress1               |

26. Under **Queries** on the left, click on **AzureDataLakeGen2**.

27. Under **Load settings**, click on **Do not load** radio button, then click on **Next** button.

28. Select **Refresh automatically** radio button.

29. Set **Refresh every** to 1 Days and set the time to 08:00, then click **Create** button.

### Update Power Automate Flow connections

To update the Power Automate Flow connections.

1. Open https://make.powerapps.com

2. From the top menu select the environment that the solution will be deployed into, this is the one we created earlier.

3. Click on the **ANS_StudentActivity** solution, then find **Send email on new Request** in the list of solution components and click on it. This will open a Power Automate window.

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

   - **Name:** Request
   - **Partial URL:** request

6. Click on the first section component in the page, this is located directly under the header component.

7. From the left-hand menu click on the second item down **Components**, then click on **Form**.

8. On the Component menu at the right hand side, complete the form as detailed below.

   - **Name:** Request
   - **Entity:** Request (ans_request)
   - **Form Layout:** Web Update Form
   - **Mode:** Edit

9. Click anywhere in the page to save the settings.

10. From the left-hand menu click on the top item **Pages**, find **Request** then click on the **...**, on the popup menu click on **Hide in default menu**

11. Now click on the **Sync Configuration** button at the top right of the page. Once this has completed the Portal is ready to allow users to respond to requests.

12. To apply branding and further customise the Portal, reference the Microsoft documentation here: https://docs.microsoft.com/en-us/powerapps/maker/portals/overview

### Enabling User Access

To enable users' access to the PowerApp Student Activity application, users must be added to PowerApps Security Roles.

To enable users, follow the steps below,

1. Open https://make.powerapps.com

2. From the top menu select the environment that the solution will be deployed into, this is the one we created earlier.

3. Click on the **Cog** icon to the right of the selected environment, on the menu click on **Advanced settings**.

4. Click on **Settings** in the top menu, and on the dropdown menu, click on **Security**.

5. Click on **Users**, then find the user you want to grant access to the application.

6. Click on the Users Full Name, then on the top menu click on **Manage Roles**.

7. Check the **ANS StudentActivity App Access** and **ANS StudentActivity Common Data Service** roles, then click on **OK** button.

### App URL

The Student Manager Application URL can be found by following these steps.

1. Open https://make.powerapps.com

2. From the top menu select the environment that the solution will be deployed into, this is the one we created earlier.

3. Click on **Apps** from the left-hand menu, find **Student Manager** in the list then click on the name to open it. It will open in a browser new tab or window.

4. From the browser new tab or window, copy the URL from the URL bar.

5. Optional:  You can also shorten the URL by removing the query parameters from the URL so that it looks like https://orgxxxxxx.crmxx.dynamics.com/main.aspx 
