# Okta Workflows How-To: Anything as a Source with Pagination

## Overview

Anything-as-a-Source allows you to integrate any source of truth with Okta, and realize the benefits of HR-driven provisioning from any source of truth. XaaS gives customers the flexibility to define the terms of synchronization between Okta and the source of truth.

![](https://i0.wp.com/iamse.blog/wp-content/uploads/2023/05/image-34.png?resize=1020%2C358&ssl=1)

XaaS with Okta Workflows

See my original blog on Anything-as-a-Source here:  [Okta Workflows How-To: Anything as a Source](https://oktawiki.atlassian.net/wiki/spaces/ASE/pages/2738241363). This provides an introduction to the Anything-as-a-Source functionality and a step-by-step guide on how to configure the functionality within Okta. My initial sample workflow and sample user data allows for 100 users to be synchronised from an AWS DynamoDB database into Okta. This blog entry does not take into account the limits applied to the Anything-as-a-Source bulk-upload operation.

### Anything-as-a-Source Constraints

The two constraints applied to the Anything-as-a-Source bulk-upload operation are:

1.  You can load up to 200 KB of data in a single bulk-load (/bulk-upsert or /bulk-delete) request for an Identity Source Session. This equates to  **200 user profiles**. To load more user profiles, you can make multiple bulk-load requests to the same session.
2.  The maximum number of bulk-load requests for a session is  **50**. If you exhaust the maximum number of bulk-load requests and you still need to load more user profiles, then create another Identity Source Session object for the additional user profiles.

For additional details, see Anything as a Source Okta documentation here:  [Build an Anything-as-a-Source custom client integration | Okta Developer](https://developer.okta.com/docs/guides/anything-as-a-source/)

Now lets look at how we can handle these two constraints within Okta Workflows. The key is obviously going to be how we can handle pagination or the ability to page over the results.

### Pagination Options

There are at least two main ways to handle pagination in Okta Workflows:

1.  Download all the users and store them in a workflow table. You can then loop through the table and load them into Okta via Anything-as-a-Source. You would use this approach if your external identity source does not support paging.
2.  Page through the users as they are retrieved from the external identity source. As you receive each page, use Anything-as-a-Source to load each page into Okta. This is the option that I’m documenting in the blog post.

I have also previously written a blog on pagination with Okta Workflows. See my Workflows How To: Pagination blog here:  [Pagination with Okta Workflows](https://iamse.blog/2022/09/07/pagination-with-okta-workflows/)  We are going to use the same principal in paginating results with Anything-as-a-Source.
## Pagination Logic

The key to how workflows paginates results is to continue to loop, based on the presence of a next page. Looping can be achieved by a flow calling itself to process the next page as per the following high level diagram:

![](https://i0.wp.com/iamse.blog/wp-content/uploads/2023/05/image-32.png?resize=997%2C459&ssl=1)

High Level Pagination Logic

In practice, the above logic equates to the following:

![](https://i0.wp.com/iamse.blog/wp-content/uploads/2023/06/image-23.png?resize=1024%2C824&ssl=1)

In the above flow, the link to the next page of results is extracted from the previous response payload. (next_link) If there is data in the value for next_link, then another page of data exists. In this case, the flow will call itself again, passing in the link to the next page. If there is no link, then thats the end of the results and the import is triggered and flow then terminates as all processing has been completed.

## Managing Page Size

The Anything-as-a-Source load requests can load up to a maximum of 200 users at one time, so therefore an API request to retrieve users from an external system only needs to retrieve a maximum of 200 users at one time. In my supplied pagination example, I’m using Azure AD as the external identity source as the Microsoft Graph API supports pagination. My example workflow includes the configuration of a static variable for page size. This value is then passed into the query when retrieving the users as per the screen shot below:

![](https://i0.wp.com/iamse.blog/wp-content/uploads/2023/06/image.png?resize=586%2C376&ssl=1)

Managing Page Size

Additionally, if the value for page size is set to be greater than 200, the flow will generate an error.

## Managing Bulk-Load Requests

The maximum number of bulk-load requests for a session is 50. So once 50 is reached, workflows needs to trigger an import based on the current Session Id and then open a new session for the remainder of bulk-load requests. Additionally, you can’t create a new Identity Source Session in less than five minutes after triggering an active session associated with the same identity source. So workflows will need to account for this and potentially perform a 5 minute wait within the flow.

This logic is represented by the following flow:

![](https://i0.wp.com/iamse.blog/wp-content/uploads/2023/06/image-25.png?resize=1024%2C520&ssl=1)

Managing Bulk-Load Requests

The logic in the above flow is the following:

1.  If the bulk-load counter equals 50, then do the following:
    1.  Reset the bulk-load counter back to zero.
    2.  Trigger an Import Session based on the current Session Id.
    3.  Wait for 5 minutes to allow for the import to complete.
    4.  Create a new Import Session and return that new Session Id to be used for future bulk-loads.
2.  If the bulk-load counter is less than 50, then just increment the counter and return the existing Session Id.
# End to End Enablement

This section details how to get Anything-as-a-Source with pagination functionality up and running within Okta Workflows with the supplied sample. These are the steps that we will be following:

1.  Configure a Custom Identity Source app in Okta via the Administration console. This Custom Identity Source app is an additional component that is provided as part of the XaaS release.
2.  Enable your identity source to be called by Okta Workflows. In this example, we will be using an Azure AD instance. The supplied workflow comes with sample users and utility flows that can be used to upload those users to your Azure AD instance.
3.  Import the supplied workflow example and configure the flows to retrieve users from your Azure AD instance and then bulk-upload the users into your Okta tenant via the Anything-as-a-Source API’s.

## Pre-Requirements

> Anything-as-a-Source is currently a Limited Early Access (LEA) feature, and it is available to a limited audience. To get it enabled, contact your Customer Success Manager (CSM) or  [Okta Support](https://support.okta.com/help/s/).

Once this new feature is enabled, your Okta tenant will have:

1.  Access to the new Custom Identity Source app
2.  Within Workflows, the Okta connector will include the additional operations to call the XaaS API
3.  The Okta Administration console will include an Import Monitoring page

This sample uses Azure AD to simulate your external repository of users, so you will need your own Azure AD instance with administrator access.

## Step 1 – Configure Custom Identity Source App

In this step, we are going to add the  **Custom Identity Source** app to Okta and configure the mapping from the incoming user profile to the Okta profile.

See the official Okta documentation on the Custom Identity Source application here:  [Use Anything-as-a-Source](https://help.okta.com/oie/en-us/Content/Topics/users-groups-profiles/usgp-use-anything-as-a-source.htm?cshid=ext-use-xaas).

### Add Application

In your Okta Administration console, go to  **Applications > Applications**  and then click on  **Browse App Catalog**. Then search for  _**Custom Identity Source**_.

Add the following application to your Okta tenant:

![](https://i0.wp.com/iamse.blog/wp-content/uploads/2023/06/image-2.png?resize=347%2C255&ssl=1)

Custom Identity Source App

### Configure Application

Once you have added the app to your Okta tenant, give the app a meaningful name. In my example, I have named it  _**AWS DynamoDB Identity Source**._

Next, go to the  **Provisioning tab**  and select the  **Integration**  option on the left menu. Click  **Edit**  and check the box for  _**Enable API Integration**_. In this example, we will not be importing Groups, so you can un-check the box for  **Import Groups**. Then click  **Save**. See the screenshot below:

![](https://i0.wp.com/iamse.blog/wp-content/uploads/2023/06/image-3.png?resize=882%2C304&ssl=1)

Enable API Integration

Next, under the  **Provisioning tab**, select the  **To Okta**  option on the left menu. In the section titled  **User Creation & Matching**, click  **Edit**  and check the boxes for the following settings:

1.  Auto-confirm exact matches
2.  Auto-confirm new users
3.  Auto-activate new users

Then click  **Save**. If these settings are not enabled, the administrator will have to manually confirm and activate the imports. See the screenshot below:

![](https://i0.wp.com/iamse.blog/wp-content/uploads/2023/06/image-4.png?resize=648%2C170&ssl=1)

Enable Auto Confirm and Activate

In the section titled  **Profile & Lifecycle Sourcing**, click  **Edit**  and check the box for  _**Allow Custom Identity**  **Source to source Okta users**._  You can also optionally check the boxes for  _**Reactivate suspended Okta**  **users**_  and  _**Reactivate deactivated Okta users**._  Then click  **Save**. See the screenshot below:

![](https://i0.wp.com/iamse.blog/wp-content/uploads/2023/06/image-5.png?resize=851%2C468&ssl=1)

Allow app to source Okta users

### Update Identity Source Profile

The default profile for the  **Custom Identity Source Application**  contains the following attributes; Username, First Name, Last Name, Email, Second Email and Mobile Phone. To support the profile coming from Azure AD, we need to add some additional custom attributes to the profile.

In your Okta Administration console, go to  **Directory > Profile Editor**  and select the profile for the  **Custom Identity Source Application**  added in the previous section. Then add the following custom attributes:

| **Display Name** | **Variable Name** | **Data Type** | **Attribute Type** |
|--|--|--|--|
| Display Name  | displayName  | string  | Custom |
| Department  | department  | string  | Custom |
| Title |  title |  string |  Custom |
| Street Address  | streetAddress |  string |  Custom |
| City  | city |  string |  Custom |
| State |  state |  string |  Custom |   
| Country Code  | countryCode |  string |  Custom |
| Zip Code |  zipCode |  string |  Custom |
| Organization |  organization |  string | Custom |

Once complete, click the  **Mappings**  button to bring up the profile mappings from the  **Custom Identity Source App**  profile to the Okta profile. Ensure all the default and custom attributes are mapped from source to destination. The mapping for the first three attributes is displayed in the screenshot below:

![](https://i0.wp.com/iamse.blog/wp-content/uploads/2023/06/image-6.png?resize=1024%2C399&ssl=1)

Map Attributes to Okta Profile

## Step 2 – Add Required Scopes to App

The additional XaaS operations that have been added to the workflow Okta connector, require some additional scopes. In your Okta Administration console, go to Applications > Applications and select the application titled Okta Workflows OAuth. Open the Okta API Scopes tab and grant consent for the following two scopes:

1.  okta.identitySources.manage
2.  okta.identitySources.read

![](https://i0.wp.com/iamse.blog/wp-content/uploads/2023/06/image-7.png?resize=776%2C139&ssl=1)

Update Workflow Apps API Scopes

In order for the Okta Connector to be able to access these additional scopes, it needs to be re-authorized. Open your workflow console and select Connections at the top of the page. Within your exiting connections, select the Okta connector. Click on the reauthorize icon on the right, and enter in your Domain, Client ID and Client Secret from the Okta Workflows OAuth app.

## Step 3 – Configure Azure AD

In this step we are going to configure your instance of Azure AD to be called by Okta.

1.  Log into the Azure Active Directory console and select the App registrations menu on the left. Then create a new application registration.
2.  Copy the Application (client) ID and the Directory (tenant) ID. We will need these values when configuring the sample workflow.
3.  Within the application, select Certificates & secrets on the left. Then click the plus sign next to New client secret. Give it a meaningful name and click Add. Then copy the client secret value. We will need this value when configuring the sample workflow.
4.  Within the application, select API permissions on the left. Then click the plus sign next to Add a permission. Select Microsoft Graph and then select Application Permissions. Then scroll down and find the user permissions and add User.ReadWrite.All. Once added, Grant admin consent for the permission. The permissions should now match the following:

In order for the Okta Connector to be able to access these additional scopes, it needs to be re-authorized. Open your workflow console and select Connections at the top of the page. Within your exiting connections, select the Okta connector. Click on the reauthorize icon on the right, and enter in your Domain, Client ID and Client Secret from the Okta Workflows OAuth app.

## Step 3 – Configure Azure AD

In this step we are going to configure your instance of Azure AD to be called by Okta.

1.  Log into the Azure Active Directory console and select the App registrations menu on the left. Then create a new application registration.
2.  Copy the Application (client) ID and the Directory (tenant) ID. We will need these values when configuring the sample workflow.
3.  Within the application, select Certificates & secrets on the left. Then click the plus sign next to New client secret. Give it a meaningful name and click Add. Then copy the client secret value. We will need this value when configuring the sample workflow.
4.  Within the application, select API permissions on the left. Then click the plus sign next to Add a permission. Select Microsoft Graph and then select Application Permissions. Then scroll down and find the user permissions and add User.ReadWrite.All. Once added, Grant admin consent for the permission. The permissions should now match the following:

![](https://i0.wp.com/iamse.blog/wp-content/uploads/2023/06/image-8.png?resize=1024%2C181&ssl=1)

Update Azure Apps API Permissions

Thats the completion of the Azure AD configuration.

## Step 4 – Configure the Sample Workflow

### Download Sample Flows

The supplied pagination workflow example can be downloaded from here:  [Pagination Workflow Example](https://minhaskamal.github.io/DownGit/#/home?url=https://github.com/iamse-blog/workflows-templates/tree/main/anything-as-a-source-pagination).

The sample contains the following flows:

1.  **[helper 1.0] Initialize Workflow**  – Loads configuration values
2.  **[helper 2.0] Generate New Import Session**  – Clears all previous import sessions and generates a new one
3.  **[helper 2.1] Delete Target Import Session** – Deletes a import session based on passed session id
4.  **[helper 3.0] Get Access Token**  – Calls Azure to generate an Access Token
5.  **[helper 4.0] Get Page of Results**  – Gets a page of Azure AD Users
6.  **[helper 4.1.1] Profile Map to Okta Format**  – This flow maps the incoming DynamoDB profile to the Okta profile
7.  **[helper 4.1.2] Bulk User Import**  – Manages the triggering of the bulk user import into Okta
8.  **[helper 4.1] Process Page of Results**  – Processes a batch of users in the form of an object list
9.  **[main 1.0] Scheduled Import Active Users**  – This flow synchronizes new users and user profile updates with Okta via Anything as a Source API Operations
10.  **[main 1.0] Scheduled Import Deactivate Users**  – This flow synchronizes deactivated users with Okta via Anything as a Source API Operations
11.  **[util] Add Random User To Table**  – This utility flow adds a random user to the sample users table. Only use this flow if you want to increase the number of sample users. (300 sample users supplied)
12.  **[util] Remove All Users from Okta**  – This utility flow removes all previously synchronized users (based on Sample Users table) from Okta
13.  **[util] Remove Single User from Okta**  – This utility flow removes a single user from Okta
14.  **[util] Upload All Sample Users**  – This utility flow uploads all users in the Sample Users table to Azure AD
15.  **[util] Upload Sample User to Azure AD**  – This utility flow uploads a single sample user to Azure AD

The sample contains the following tables:

1.  **Sample Users**  – Sample users used to upload into Azure AD
2.  **Configuration**  – Static name – value pairs
3.  **Bulk Load Requests**  – Maintains a counter of bulk load requests

### Import Flows and Table Data

Open the workflows console and create a new folder with a meaningful name. EG.  _Anything-as-a-Source Pagination_

Import the workflow sample (anythingAsASourcePagination.folder) into the folder you just created. This is done by clicking the three dots at the end of the folder name and selecting import. The pagination folder should now contain 15 flows and 3 tables as listed above.

Next, open the Tables tab and import the following data:

1.  Open the Sample Users table and select Import. Select the supplied file sampleUsers.csv. This will import 300 sample users that can be uploaded to Azure AD in a later step.
2.  Open the Configuration table and select Import. Select the supplied file configuration.csv. This will import 5 configuration values. The configuration value for page_size can be left as 200, but the remainder of the values require updating.
    1.  domain – Update this value to the Azure AD domain that you would like to add the sample users to. If you already have some sample users in your Azure AD domain, then you can ignore this value. The workflow will replace the  [example.com](http://example.com/)  domain from the sample users and replace it with this domain.
    2.  client_id – This is the saved Azure Client Id from step 3.
    3.  tenant_id – This is the saved Azure Tenant Id from step 3.
    4.  client_secret – This is the saved Azure Client Secret from step 3.

### Update and Active Flows

The following flows require the updating of the Okta connector cards to select your local Okta connector:

1.  [helper 2.0] Generate New Import Session
2.  [helper 2.1] Delete Target Import Session
3.  [helper 4.1.2] Bulk User Import
4.  [util] Remove Single User from Okta

In each of the above flows, select each Okta card and then select your local Okta connector. Additionally, for the first three helper flows, you will need to update the Okta cards to select your Custom Identity Source application that was created in Step 1.

For example:

![](https://i0.wp.com/iamse.blog/wp-content/uploads/2023/06/image-20.png?resize=498%2C398&ssl=1)

Update Okta Connector

![](https://i0.wp.com/iamse.blog/wp-content/uploads/2023/06/image-19.png?resize=476%2C374&ssl=1)

Select Custom App

Once each flow has been updated, save the flow.

The following flows require the updating of the generic API connector cards to select your local API Connector. Note that this API Connector should have an authentication type of  _**none**_. If you don’t already have an API connector with an authentication type of  _none_, then create one now by going to the connectors tab and clicking New Connection and select API Connector.

1.  [helper 3.0] Get Access Token
2.  [helper 4.0] Get Page of Results
3.  [util] Add Random User To Table
4.  [util] Upload Sample User to Azure AD

In each of the above flows, select each API Connector card and then select your local API connector. Then save each flow.

Now activate each imported flow by turn the ON/OFF toggle to ON.

### Load Sample Users

This is an optional step as you may have your own sample users. If you want to use the supplied sample users, then open the flow titled **_[util] Upload All Sample Users_**  and click the Test button to run the flow.

If workflows can not correctly connect to your Azure AD instance, then you will receive an error at this point. Errors could include incorrect Client and Tenant Id’s, incorrect permissions or an incorrect domain name.

If the flow runs successfully, your Azure AD domain will have an additional 300 users as shown in the screen shot below:

![](https://i0.wp.com/iamse.blog/wp-content/uploads/2023/06/image-11.png?resize=893%2C589&ssl=1)

Upload Sample Users to Azure AD

> **Note**: This only applies if you are  **not**  using the supplied sample users.
> 
> In order to only process the supplied sample users, and no other existing users in your Azure AD instance, the flow titled  _**[helper 4.1.1] Profile Map to Okta Format**_  contains a filter. Only users with a department of “Okta Workflows” will be processed. All other user records will be bypassed. If you are not using the sample users supplied, then you will need to update or remove the Continue If card within this flow. See screen shot below.
> 
> ![](https://i0.wp.com/iamse.blog/wp-content/uploads/2023/06/image-12.png?resize=470%2C414&ssl=1)
> 
> Filter Test Users by Department
> ## Step 5 – Run the Sample Workflow

Once configured, we can now run the workflow. We are going to test the following:

1.  Import all the users in Azure AD into Okta. If using the supplied sample users, then this step will add an additional 300 users to your Okta tenant.
2.  Update a subset of users. This will demonstrate how Anything as a Source can perform a delta and only update those users that have a profile change.
3.  Deactivate a subset of users. Users deactivate in Azure AD will be deactivated in Okta.

### Import Users

Now open the Workflow console and then open flow _**[main 1.0] Scheduled Import Active Users**_ and click the Test button.

Once the flow executes and the import completes, go to Reports > Import Monitoring in the Okta Administration console. The import log should indicate that 300 new users were added to your Okta tenant as displayed below:

![](https://i0.wp.com/iamse.blog/wp-content/uploads/2023/06/image-13.png?resize=824%2C570&ssl=1)

Import Monitoring – New Users

### Update Users

Now we are going to test Anything as a Source’s ability to apply user updates. Open the Azure AD console and select one of your test users. Open the user’s properties and make a modification. For example, update the users mobile phone. Then click Save. Update as many users as desired.

Now go back to the Okta Workflows console and open flow **_[main 1.0] Scheduled Import Active Users_**  and click the Test button.

Once the flow executes and the import completes, go to Reports > Import Monitoring in the Okta Administration console. The import log should indicate that 300 users were scanned and updates were only made to those users that were modified as displayed below:

![](https://i0.wp.com/iamse.blog/wp-content/uploads/2023/06/image-14.png?resize=816%2C560&ssl=1)

Import Monitoring – Update Users

### Deactivate Users

Finally we are going to test user deactivation. Open the Azure AD console and select one of your test users. Then edit the users account status and set the status to disabled. Save the status change. Disable as many users as desired.

Now go back to the Okta Workflows console and open flow **_[main 1.0] Scheduled Import Deactivate Users_**  and click the Test button.

Once the flow executes and the import completes, go to Reports > Import Monitoring in the Okta Administration console. The import log should indicate that one user was scanned and one user was removed (deactivated).

![](https://i0.wp.com/iamse.blog/wp-content/uploads/2023/06/image-15.png?resize=809%2C562&ssl=1)

Import Monitoring – Deactivate Users

## Clean Up

The workflow sample comes with some utility flows, including flows that will remove all the test users from your Okta tenant. If you would like to remove all the test users, run the flow titled _**[util] Remove All Users from Okta**._ Once the flow has completed, all 300 test users will have been removed from your tenant.

That ends the End to End Enablement of Anything as a Source.

## What you learned

In this tutorial, you learn’t how to build a workflow that leverages Okta’s new Anything as a Source feature. You learned:

-   The benefits of Okta’s new Anything as a Source feature as compared to the previous options
-   How to paginate results and bulk load users for each results page
-   How to setup and configure the new Custom Identity Source application
-   How to construct a workflow to use the new XaaS operations to sychronize users into Okta
-   How to access the in Import Monitoring page within the Okta Administration console

## Resources

The supplied pagination workflow example can be downloaded from here:  [Pagination Workflow Example](https://minhaskamal.github.io/DownGit/#/home?url=https://github.com/iamse-blog/workflows-templates/tree/main/anything-as-a-source-pagination).

See Anything as a Source Okta documentation here:  [Build an Anything-as-a-Source custom client integration | Okta Developer](https://developer.okta.com/docs/guides/anything-as-a-source/).

See the official Okta documentation on the Custom Identity Source application here:  [Use Anything-as-a-Source](https://help.okta.com/oie/en-us/Content/Topics/users-groups-profiles/usgp-use-anything-as-a-source.htm?cshid=ext-use-xaas).

See my original blog on Anything-as-a-Source here:  [Okta Workflows How-To: Anything as a Source](https://oktawiki.atlassian.net/wiki/spaces/ASE/pages/2738241363).
