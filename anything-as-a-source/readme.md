
# Okta Workflows How-To: Anything as a Source
## End to End Enablement

This section details how to get the new Anything-as-a-Source functionality up and running within Okta Workflows. These are the steps that we will be following:

1.  Configure a  **Custom Identity Source**  app in Okta via the Administration console. This  **Custom**  **Identity Source**  app is an additional component that is provided as part of the XaaS release.
2.  Enable your identity source to be called by Okta Workflows. In this example, we will be using an  **AWS DynamoDB**  table that has been exposed via the  **AWS API Gateway**.
3.  Create a workflow that retrieves the user data from an API endpoint and then calls the new XaaS API’s available in the  **Okta Connector.**

## Pre-Requirements

> This is currently a  **Limited Early Access**  (LEA) feature, and it is available to a limited audience. To get it enabled, contact your  **Customer Success Manager**  (CSM) or  [Okta Support](https://support.okta.com/help/s/?language=en_US).

Once enabled, your Okta tenant will have:

1.  Access to the new Custom Identity Source app
2.  Within Workflows, the Okta connector will include the additional operations to call the XaaS API
3.  The Okta Administration console will include an Import Monitoring page

## Step 1 – Configure Custom Identity Source App

In this step, we are going to add the  **Custom Identity Source** app to Okta and configure the mapping from the incoming user profile to the Okta profile.

See the official Okta documentation on the Custom Identity Source application here:  [Use Anything-as-a-Source](https://help.okta.com/oie/en-us/Content/Topics/users-groups-profiles/usgp-use-anything-as-a-source.htm?cshid=ext-use-xaas)

### Add Application

In your Okta Administration console, go to  **Applications > Applications**  and then click on  **Browse App Catalog**. Then search for  _**Custom Identity Source**_.

Add the following application to your Okta tenant:

![](https://i0.wp.com/iamse.blog/wp-content/uploads/2023/03/image-96.png?resize=347%2C255&ssl=1)

### Configure Application

Once you have added the app to your Okta tenant, give the app a meaningful name. In my example, I have named it  _**AWS DynamoDB Identity Source**._

Next, go to the  **Provisioning tab**  and select the  **Integration**  option on the left menu. Click  **Edit**  and check the box for  _**Enable API Integration**_. In this example, we will not be importing Groups, so you can un-check the box for  **Import Groups**. Then click  **Save**. See the screenshot below:

![](https://i0.wp.com/iamse.blog/wp-content/uploads/2023/03/image-97.png?resize=882%2C304&ssl=1)

Next, under the  **Provisioning tab**, select the  **To Okta**  option on the left menu. In the section titled  **User Creation & Matching**, click  **Edit**  and check the boxes for the following settings:

1.  Auto-confirm exact matches
2.  Auto-confirm new users
3.  Auto-activate new users

Then click  **Save**. If these settings are not enabled, the administrator will have to manually confirm and activate the imports. See the screenshot below:

![](https://i0.wp.com/iamse.blog/wp-content/uploads/2023/03/image-98.png?resize=648%2C170&ssl=1)

In the section titled  **Profile & Lifecycle Sourcing**, click  **Edit**  and check the box for  _**Allow Custom Identity**  **Source to source Okta users**._  You can also optionally check the boxes for  _**Reactivate suspended Okta**  **users**_  and  _**Reactivate deactivated Okta users**._  Then click  **Save**. See the screenshot below:

![](https://i0.wp.com/iamse.blog/wp-content/uploads/2023/03/image-99.png?resize=851%2C468&ssl=1)

### Update Identity Source Profile

The default profile for the  **Custom Identity Source Application**  contains the following attributes; Username, First Name, Last Name, Email, Second Email and Mobile Phone. If you are using the AWS DynamoDB example, then we need to add some additional custom attributes to the profile.

In your Okta Administration console, go to  **Directory > Profile Editor**  and select the profile for the  **Custom Identity Source Application**  added in the previous section. Then add the following custom attributes:

| Display Name | Variable Name | Data Type | Attribute Type |
|--|--|--|--|
| Title | title | string | Custom |
| Display Name | displayName|  string | Custom | 
| Primary Phone | primaryPhone | string | Custom | 
| Zip Code | zipCode | string | Custom| 
| Street Address | streetAddress | string | Custom| 
| City | city | string | Custom| 
| State | state | string | Custom| 
| Country Code | countryCode | string | Custom| 

Once complete, click the  **Mappings**  button to bring up the profile mappings from the  **Custom Identity Source App**  profile to the Okta profile. Ensure all the default and custom attributes are mapped from source to destination. The mapping for the first three attributes is displayed in the screenshot below:

![](https://i0.wp.com/iamse.blog/wp-content/uploads/2023/03/image-100.png?resize=1024%2C379&ssl=1)

## Step 2 – Integrate your Custom Identity Source

In this step, we are going to set up our custom identity source and populate it with sample user records. As an example custom identity source, I exposed a  **DynamoDB**  table as an external service with  **Lambda** and the  **AWS API Gateway**.

![](https://i0.wp.com/iamse.blog/wp-content/uploads/2023/03/image-101.png?resize=1024%2C181&ssl=1)

To implement the example, follow the documentation here:  [https://docs.aws.amazon.com/apigateway/latest/developerguide/http-api-dynamo-db.html](https://docs.aws.amazon.com/apigateway/latest/developerguide/http-api-dynamo-db.html)

The only alteration I made to the AWS example, is to update the  **PUT**  operation in the Lambda function. I updated the code to account for the supplied sample users.

Here is my updated  **PUT**  operation in Lambda.

```
case "PUT /items":
        let requestJSON = JSON.parse(event.body);
        await dynamo.send(
          new PutCommand({
            TableName: tableName,
            Item: {
              id: requestJSON.id,
              active: requestJSON.active,
              title: requestJSON.title,
              first_name: requestJSON.first_name,
              last_name: requestJSON.last_name,
              street_number: requestJSON.street_number,
              street_name: requestJSON.street_name,
              city: requestJSON.city,
              state: requestJSON.state,
              country: requestJSON.country,
              postcode: requestJSON.postcode,
              email: requestJSON.email,
              phone: requestJSON.phone,
              cell: requestJSON.cell,
              dob: requestJSON.dob
            },
          })
        );
```

Here is a sample JSON payload to test your  **PUT**  operation:

```
{
    "id": "albert.gambaro@example.com",
    "active": true,
    "title": "Mr",
    "first_name": "Albert",
    "last_name": "Gambaro",
    "street_number": "2478",
    "street_name": "Calle del Arenal",
    "city": "Las Palmas de Gran Canaria",
    "state": "Andalucía",
    "country": "ES",
    "postcode": "15465",
    "email": "albert.gambaro@example.com",
    "phone": "973-093-421",
    "cell": "630-851-127",
    "dob": "2000-11-05T21:22:37.049Z"
}
```

Once the  **PUT**  operation has been successfully completed, log into your DynamoDB instance and check the contents of the table.

![](https://i0.wp.com/iamse.blog/wp-content/uploads/2023/03/image-102.png?resize=928%2C212&ssl=1)

Ensure your  **PUT**  and  **GET**  operations are working before proceeding to the next step.

## Step 3 – Configure the Sample Workflow

In this step, we are going to import and configure a sample workflow. The supplied sample workflow can be downloaded from here:  [Sample Workflow](https://minhaskamal.github.io/DownGit/#/home?url=https://github.com/iamse-blog/workflows-templates/tree/main/anything-as-a-source)

### Add Required Scopes

The additional XaaS operations that have been added to the workflow Okta connector, require some additional scopes. In your Okta Administration console, go to  **Applications > Applications** and select the application titled  **Okta Workflows OAuth**. Open the  **Okta API Scopes**  tab and grant consent for the following two scopes:

1.  okta.identitySources.manage
2.  okta.identitySources.read

![](https://i0.wp.com/iamse.blog/wp-content/uploads/2023/03/image-103.png?resize=776%2C139&ssl=1)

In order for the  **Okta Connector** to be able to access these additional scopes, it needs to be re-authorized. Open your workflow console and select Connections at the top of the page. Within your exiting connections, select the  **Okta connector.**  Click on the reauthorize icon on the right, and enter in your Domain, Client ID and Client Secret from the  **Okta Workflows OAuth** app.

### Import Sample Flows

The contents of the downloaded zip file is as follows:

1.  anythingAsASource.folder – The workflow folder import
2.  readme.md – Install Instructions
3.  sampleUsers.csv – Sample user data

Open your workflow console and create a new folder and give it a meaningful name. In my workflow instance, I called my folder  **Anything-as-a-Source**. Click the three dots at the end of the folder name and select  **Import**. Import the file titled anythingAsASource.folder

Once the import is complete, the folder will contain the following flows:

1.  [main] Populate DynamoDB Table – This flow is used to synchronize the sample users supplied in the workflow table, with the AWS DynamoDB table.
2.  [util] Upload Sample User – This flow is called by [main] Populate DynamoDB Table and uploads an individual user to the AWS DynamoDB table.
3.  [main] Scheduled Import Active Users – This flow orchestrates a bulk upload of active users from the DynamoDB table into Okta.
4.  [main] Scheduled Import Deactive Users – This flow orchestrates a bulk upload of deactive users from the DynamoDB table into Okta.
5.  [util] Profile Map to Okta Format – This flow takes the user data from the DynamoDB user record and formats it into the expected format for the XaaS upload API.
6.  [util] Delete Target Import Session – This flow is used to clean up any old import sessions.
7.  [util] Filter Active Users – This flow will return a value of True if the user is in a active state.
8.  [util] Filter Deactive Users – This flow will return a value of True if the user is in a de-active state.

### Import Sample Data

The workflow import in the previous step would have created a table called  **Sample Users**. Select the  **Tables tab**  and select the  **Sample Users** table. Click  **Import**  and select the supplied  **sampleUsers.csv** data file. Once the import is complete, the table should contain exactly 100 sample users.

### Update Flow Configuration

#### Okta Connector

The following flows have  **Okta Connector**  cards that require updating:

1.  [main] Scheduled Import Active Users
2.  [main] Scheduled Import Deactive Users
3.  [util] Delete Target Import Session

Within each flow, select the respective  **Okta Connector** card and update the connector to use your local connector.

#### API Connector

The following flows have  **API Connector** cards that require updating:

1.  [main] Scheduled Import Active Users
2.  [main] Scheduled Import Deactive Users
3.  [util] Upload Sample User

Within each flow, select the respective  **API Connector**  card and update the connector to use your local connector. The  **Auth Type** of your  **API Connector**  should be a type of  **None**. As this is just an example Custom Identity Source, there is no security required.

#### Custom Identity Source URL

The following flows have  **AWS API Gateway URL’s**  that require updating:

1.  [main] Scheduled Import Active Users
2.  [main] Scheduled Import Deactive Users
3.  [util] Upload Sample User

Within each flow, select the respective  **API Connector** card and update the URL to point to your  **AWS API Gateway e**ndpoint.

#### XaaS Workflow Cards

The XaaS workflow cards need to point to your  **Custom Identity Source**  application created in Step 1. Open flow  **[main] Scheduled Import Active Users**  and select the  **Options**  for each of the following XaaS cards:

1.  List Import Sessions
2.  Create an Import Session
3.  Bulk User Import
4.  Trigger Import Session

Then under the  **Application**  option, select your  **Custom Identity Source**  application from the drop down menu and then select save. Do the same for flow  **[main] Scheduled Import Deactive Users**.

![](https://i0.wp.com/iamse.blog/wp-content/uploads/2023/03/image-104.png?resize=912%2C334&ssl=1)

Once the workflow configuration has been updated, turn each flow  **On**.

### Initiate Data Upload to DynamoDB

In this step we are going to upload the user records in the  **Sample Users**  workflow table to your  **DynamoDB**  table using your  **API Gateways PUT operation**. The sample flows include two helper flows that will do this for you. To initiate this process, open the flow titled  **[main] Populate DynamoDB Table**  and click the  **Test**  icon to run the flow.

![](https://i0.wp.com/iamse.blog/wp-content/uploads/2023/03/image-105.png?resize=934%2C652&ssl=1)

Once the flow has completed, log into your  **DynamoDB**  instance and check the contents of the table. You should have  **100 records**  (in addition to any existing test records).

![](https://i0.wp.com/iamse.blog/wp-content/uploads/2023/03/image-106.png?resize=872%2C284&ssl=1)

Ensure your  **DynamoDB**  table has been populated before proceeding to the next step.

## Step 4 – Run The Import Flows

In this step, we are going to run the sample flows and see how the workflow can synchronize users from the external repository with Okta. The two scheduled flows that will sychronize users are  _**[main] Scheduled Import Active Users**_  and  _**[main] Scheduled Import Deactive Users**._  In production, these will run at the desired schedule to sychronize new users, user profile updates and user status updates.

The contents of these flows is as follows:

![](https://i0.wp.com/iamse.blog/wp-content/uploads/2023/03/image-107.png?resize=1024%2C535&ssl=1)

The flow starts with a call to  **List Import Sessions** to retrieve any sessions that have not completed. If any are found, then these sessions are deleted. The flow then calls the  **AWS API Gateway**  endpoint to retrieve the contents of the  **DynamoDB**  table. The results are then parsed into a list.

![](https://i0.wp.com/iamse.blog/wp-content/uploads/2023/03/image-108.png?resize=704%2C484&ssl=1)

The flow then filters the user list to get all the users with an  **Active**  status. The corresponding flow  _**[main] Scheduled Import Deactive Users**_  will filter the list for  **Inactive**  users at the same point. This is because there is a separate operation for synchronising inactive users. The flow then takes the active user list and maps it to the required format for import. The flow then cleans the list by removing the “output” tag.

![](https://i0.wp.com/iamse.blog/wp-content/uploads/2023/03/image-109.png?resize=701%2C425&ssl=1)

The final step in the flow is to  **Create**  a  **Import Session**. The  **Session ID**  is then used in the  **Bulk User Import**  and the  **Trigger Import Session**  cards.

Now run the flow by clicking on the  **Test**  icon. Once the flow completes, it may take a few additional minutes for import session to add the 100 users to  **Universal Directory.**  In your Okta Administration console, go to  **Directory > People**. The user count should now indicate that you have 100 additional users. Now go to  **Reports > Import Monitoring**. The page should display a completed import session. When you open the log, it should show that a total of 100 users have been created.

![](https://i0.wp.com/iamse.blog/wp-content/uploads/2023/03/image-110.png?resize=759%2C510&ssl=1)

Now we are going to test a user profile update. Open your  **DynamoDB table**  and update any attribute on any user. Now run the  _**[main] Scheduled Import Active Users**_  flow again. Once complete, check the respective attribute on the user in Okta. It should be showing the updated value. Additionally, the **Import Monitoring**  log should show the following:

![](https://i0.wp.com/iamse.blog/wp-content/uploads/2023/03/image-111.png?resize=755%2C513&ssl=1)

Finally, we are going to test the deactivation of a user. In your  **DynamoDB**  table, update a users  **active**  status to  **false**. Then save the update.

![](https://i0.wp.com/iamse.blog/wp-content/uploads/2023/03/image-113.png?resize=966%2C214&ssl=1)

Next, run the flow titled  _**[main] Scheduled Import Deactive Users**_. Once complete, check the respective user in the administration console. The users status should now be  **Deactivated**. Additionally, the  **Import**  **Monitoring**  log should show the following:

![](https://i0.wp.com/iamse.blog/wp-content/uploads/2023/03/image-114.png?resize=761%2C510&ssl=1)

That ends the End to End Enablement of Anything as a Source.

## What you learned

In this tutorial, you learn’t how to build a workflow that leverages Okta’s new Anything as a Source feature. You learned:

-   The benefits of Okta’s new Anything as a Source feature as compared to the previous options
-   How to setup and configure the new Custom Identity Source application
-   How to construct a workflow to use the new XaaS operations to sychronize users into Okta
-   How to access the in Import Monitoring page within the Okta Administration console
