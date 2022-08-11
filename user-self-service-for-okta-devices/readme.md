# User Self Service for Okta Devices

## Overview

  

Okta Devices is a Platform Service of the Okta Identity Cloud that embeds Okta on every device (via the Okta Verify app) to give organizations visibility into devices accessing Okta, enable contextual access decisions, and deliver a consistent, passwordless login experience for users. Additionally, the Okta Devices SDK provides application builders with the ability to white-label the MFA experience provided by Okta Verify, resulting in the same device registration and visibility within the respective Okta tenant.

  

Every registered device in Okta is a unique object within Okta Universal Directory. This gives the administrator visibility into the devices that access Okta and to make decisions about user access. It also gives the administrator the ability to suspend and deactivate a device when it is lost or stolen, thus preventing that device from connecting to Okta. Rather than the end user contacting their Okta Administrator, this workflow template provides the ability for the user to suspend and reactivate their devices directly.

  

Using Okta Workflows and Slack we can deliver a modern access request experience.

This implementation of the above use case will use Slack as the collaboration tool and interface to deliver a modern end-user experience during user device interaction. Okta Workflows will interact with your Okta Identity Cloud tenant as well as a custom Slack application that will allow your Slack workspace to interact with your Okta Organization. To view, suspend and activate a user's enrolled devices, the solution will leverage helper-flows, internal tables, the Okta connector, and Okta API, and the Slack API.

## Workflow Summary

This device review use case is implemented using a total of 16 flows. The flows follow a naming convention to help identify their purpose, but also to allow for extensibility. Currently, this use case only focuses on device review, but it is possible to extend this example to cover additional requirements.

### Template Flows

Here is a summary of the flows included in the flowpack:

[API Endpoint] Slack Interactivity URL

  

This flow is exposed as an API endpoint which is called by Slack for every interaction.

The flow calls the interaction router asynchronously and immediately responds to Slack with a 200. Slack requires an acknowledgment of interactive messages within 3 seconds.

  

[Route 0] Slack Interaction Router

  

This flow will be the first route executed for every request from Slack.

It will run a security check to ensure the request is coming from the correct Slack instance.

Based on the incoming request, this flow will call the appropriate helper flow (route) to process that interaction.

  

[Route 1] Start Interaction

  

This flow will be the first route executed when a user opens the Slack devices application.

It will set up the initial display.

  

[Route 2] Handle Device List Request

  

This flow will be the route executed when a user clicks the button to retrieve a list of their enrolled devices.

This will compile the initial list of a user's devices and return the list to Slack to display.

  

[Route 3] Update Device Status

  

This flow handles updates to a user's devices. If the user initiates a Suspend or Active on a respective device, then this flow will handle that request.

  

[util: Devices] Add Device Record

  

This flow creates an individual record in the Device Id table.

  

[util: Devices] Compose Device List

  

This flow will compile a list of a user's devices with the status for each. The device list will be in JSON format.

Modals are represented by JSON data that define one or more Blocks, which describe UX elements such as text, markdown, drop-down lists, radio buttons, action buttons, etc.

This flow creates the JSON for each device in the Device List table. Once completed, the complete list is inserted within the response JSON, which is then sent to Slack.

  

[util: Devices] Delete Device Id Row

  

Deletes a single record from the Device Ids table.

  

[util: Devices] Format Slack Request

  

This flow will compose the JSON required to display a single device in Slack.

The resulting JSON will be appended onto the current device list in the working table.

  

[util: Devices] Get Users Devices

  

[util: Devices] Get Users Devices.

  

[util: Devices] HTTP Post to Slack

  

This flow sends the response back to Slack via the API Connector.

  

[util: Devices] Populate Device List

  

This flow retrieves all the respective users' devices from the Device Id table. The flow will then format a Slack response message from the Device Id’s and store the formatted response in the Device List table.

  

[util: Devices] Suspend Device

  

Based on Device Id, this flow will suspend the device.

  

[util: Devices] Unsuspend Device

  

Based on Device Id, this flow will Un-Suspend a device.

Note, Activate is only used for a deactivated device.

  

[util: Security] Format Request Body for Verification

  

In order to verify the request from Slack, we need the raw request body exactly as sent (url encoded data inside a JSON object). This flow is used to ensure the request is in the original format.

  

[util: Security] Verify Slack Request Signature

  

This flow is used to verify the HMAC hash sent from Slack in a custom header. This ensures that the request came from our app in Slack. See [https://api.slack.com/authentication/verifying-requests-from-slack](https://api.slack.com/authentication/verifying-requests-from-slack)

### Template Tables

The Device Review template uses the following three tables:

  

Config Table

  

This table holds static values that are read by the flows at runtime.

  

Device Ids Table

  

This is a dynamic table used at runtime and holds individual device Id data.

  

Device List Table

  

This is a dynamic table used at runtime and holds the formatted JSON list of a user’s devices which is sent back in the response payload to Slack.

## Before you Get Started / Prerequisites

Before you get started, here are the things you’ll need:

-   Slack workspace with the ability to create a Slack Application
    
-   Okta tenant with Workflows enabled
    
-   A device with the latest version of Okta Verify installed
    

## Setup Steps

This section details the steps required to set up the Device Review workflow template.

  

The setup steps will do the following:

1.  Add the template to your workflow instance
    
2.  Create a custom Slack Application
    
3.  Configure your your workflow instance
    

### Step 1 - Add Template to Workflow Instance

In this step we will be adding the Device Review template to your workflow instance.

  

1.  Open the Templates page within the Workflows console and select the template titled Okta Device Review.
    
2.  Click Add template and a folder will be created containing all the template flows and tables.
    
3.  Go back to the template and download the sample data for the configuration table.
    
4.  Within the workflow console, open up the new folder that contains the flows and tables and select the tables tab. Three tables will be displayed. Open the Config table and click on the Import link on the top right. Then click Choose CSV File and select the downloaded data from the previous step. The table should now contain three records:
    

1.  device_icon - This is a link to an icon used within the generated Slack modal. It comes with a default link, but this can optionally be changed.
    
2.  slack_signing_request - This is a unique value that is used to ensure the requests are coming from your Slack app. This will be updated in a later step.
    
3.  domain - This is your underlying Okta domain in the following format test.okta.com or test.oktapreview.com. Update this value to your domain now.
    

6.  Within the workflow console, open up the new folder that contains the flows and tables and select the flows tab. Then open up the first flow titled [API Endpoint] Slack Interactivity URL.
    
7.  We need to configure the [API Endpoint] Slack Interactivity URL flow to be a public end-point API. To do that, click the cog at the end of the flow. Then click API Access. Confirm that the API Endpoint Settings is set to Exposed as Public Service.
    
8.  Copy the Invoke URL. We will use this information to set up the Slack App in a subsequent step.
    

### Step 2 - Create and Configure the Custom Slack Application

To create a custom app, follow these steps:

1.  Log in to your Slack workspace, and navigate to [https://api.slack.com/apps/](https://api.slack.com/apps/)
    
2.  Click the Create an App button and select the option to create the app from scratch.
    
3.  Provide a name to your application Okta Devices, select the correct development Slack workspace, and select Create App.
    

  

Once the application is created, we will be able to configure the app to interact with Okta Workflows. To configure your custom app, follow these steps:

#### Webhooks

1.  Select your newly created app and under the Features menu, click Incoming Webhooks.
    
2.  Activate Incoming Webhooks. This will allow Okta Workflows to communicate with the application.
    

#### Interactivity & Shortcuts

1.  From the left menu, select Interactivity & Shortcuts.
    
2.  Activate Interactivity.
    
3.  Populate the Request URL. This is the Invoke URL that we copied for flow [API Endpoint] Slack Interactivity URL. This enables Slack to communicate with our workflow.
    
4.  Next, we add a shortcut to our app, this way users can quickly invoke the app from Slack. Click Create New Shortcut.
    
5.  Select Global and click Next.
    
6.  Fill in the details with the following information, name “Review Devices”, short description “Review the devices currently connected to your Okta account”, and callback ID “review_devices”. Click Create to create the shortcut.
    
7.  Now save the changes, by clicking Save Changes at the bottom of the screen.
    

#### OAuth & Permissions

1.  From the left-menu click OAuth & Permissions.
    
2.  We need to inform the scopes our application requires in order to interact with Slack. Scroll down to the Scopes section of the OAuth & Permissions page.
    
3.  Under Bot Token Scopes, click Add an OAuth Scope.
    
4.  From the list search and select chat:write.
    
5.  Repeat the last two steps for the following scopes:
    

1.  chat:write.public
    
2.  links:write
    
3.  users:read
    
4.  users:read.email
    

7.  Below Bot Token Scopes, we have User Token Scopes. We need to add two additional scopes that are part of user token scopes. Click Add an OAuth Scope under the User Token Scopes section.
    
8.  From the list, search and select chat:write.
    
9.  Repeat the last two steps and add links:write.
    
10.  Now we need to install OAuth Tokens into our workspace. We do that by scrolling to the top of the page and clicking Install to Workspace.
    
11.  Review the information and select a channel for the Okta Devices app to post to. For the purposes of this tutorial, I’m using a channel that only the compliance and audit team have access to, this way every time a device is suspended or activated, the team receives a notification. Once you are done, click Allow.
    

Note: Once the app has been deployed, you will also need to invite the app to the channel if the channel is private. (eg. /invite @Devices Review)

11.  Once the installation is complete, you will have access to the OAuth Tokens. For the purpose of this demo, we will be using the Bot User OAuth Token, so go ahead and copy it for later use (you can always come back to this screen to copy the token).
    

### Step 4 - Okta Workflow Setup

In this step we are going to complete the configuration of the workflows generated by the template. Perform the following steps:

#### Update API Scopes

1.  Sign into your Okta tenant’s administration console and navigate to Applications and open the application titled Okta Workflows OAuth. This is the app that the Okta Workflows Connector uses to connect to your tenant.
    
2.  Within the application, open the Okta API Scopes tab. Grant access to the following scopes:
    

1.  okta.users.manage
    
2.  okta.users.read
    

Note: If you already have created an Okta Connector within workflows, you will need to re-authorize the connector.

#### Create Workflow Connections

1.  Create HTTP Connection to Slack.
    

1.  Go to the Okta Workflows console and click Connections and click New Connection.
    
2.  Choose API Connector from the list. For the new connection use Slack - Okta Requests as the connection nickname and for the auth type select Custom. Here we will use the bot user OAuth token that we copied from the previous step under OAuth & Permissions.
    

-   Header Name = Authorization
    
-   Header Value = Bearer <Bot_User_OAuth_Token>
    

(There is a space between the bearer and the actual token)

3.  Once you are done, click Create.
    

2.  Create an Okta Connection. If you already have an Okta connection you can skip this step. To create an Okta connection we need some information from the Okta Workflows OAuth application.
    

1.  Log in to your Okta administration console, and navigate to applications. Select Okta Workflows OAuth from the list, and select Sign On.
    
2.  Copy the client ID and client secret, we will use this information to create our Okta connection in Workflows.
    
3.  Go back into the Okta Workflows console and click Connections and select Okta from the list.
    
4.  For the new connection use Okta as the connection nickname, the domain will be the tenant you are using, the client ID and client secret is the information we copied from the Okta Workflows OAuth application.
    
5.  Once you are done, click Create.
    

4.  Create HTTP Connection to Okta. This separate Okta connection is required because at the time of writing this template, the operations required to interact with Okta Devices have not been added to the standard Okta connector. To create this connection, we need an API Token.
    

1.  Log in to your Okta administration console, and navigate to Security > API and select the Tokens tab.
    
2.  Click Create Token and give it a name of Device Review Workflow and copy the token.
    
3.  Go to the Okta Workflows console and click Connections and click New Connection. Choose API Connector from the list.
    
4.  For the new connection use Custom Okta API as the connection nickname and for the auth type select Custom. Give the connection a name of Custom Okta API and enter the following values:
    

-   Header Name = Authorization
    
-   Header Value = SSWS <API Token>
    

(There is a space between the SSWS and the actual token)

5.  Once you are done, click Create.
    

#### Associate Connections

1.  To associate the Slack - Okta Requests connector, go to the following flows and update ther API Connector card to use your newly created connector.
    

1.  [Route 1] Start Interaction
    
2.  [Route 2] Handle Device List Request
    
3.  [util: Devices] HTTP Post to Slack
    

3.  To associate the Okta connector, go to the following flows and update ther Okta Connector card to use your newly created connector.
    

1.  [util: Devices] Get Users Devices
    

5.  To associate the Custom Okta API connector, go to the following flows and update ther API Connector card to use your newly created connector.
    

1.  [util: Devices] Get Users Devices
    
2.  [util: Devices] Suspend Device
    
3.  [util: Devices] Unsuspend Device
    

  

Once all the connections have been associated, you can now toggle each flow to active.

#### Slack Signing Secret

In this step, we need to set a value for the slack_signing_secret in the workflow Config table. To do this, follow these steps:

1.  The Slack Signing Secret is copied from your Slack Application. Go to https://api.slack.com/ and select your application.
    
2.  From the basic information section, copy the Signing Secret.
    
3.  Go to the Okta Workflows console and click Flows and then select tables. Open the Config table and paste the copied Signing Secret as the value for slack_signing_secret.
    

## Testing this Flow

This is how to test the flow:

1.  Set up some test data in your Okta tenant. This is done by installing the latest version of Okta Verify on a device and adding an account to Okta Verify that exists in your Okta tenant. The respective device will then be enrolled and visible in the administration console under Directory > Devices. Repeat this step by adding additional devices for the same user.
    
2.  Log into the respective Slack workspace where the app has been created and configured. Ensure you log in with the same account used to enroll devices in step 1. Open the app by using the following command: “/Review Devices”. The app will open and provide a prompt to “Retrieve my current enrolled devices”.
    
3.  Click the Retrieve button. The app will be updated to display the users enrolled devices. Each device displayed includes an indicator that shows if the device is active or suspended. If the device is active, there will be a button to suspend the device. If the device is suspended, there will be a button to activate the device.
    
4.  Try suspending a device by clicking on the respective Suspend button. The app will be updated to indicate that the device is now suspended. Open the administration console and view the devices for the user under Directory > Devices. The respective device should be in a suspended state.
    
5.  Use the Slack app to reactivate the device.
    

#### Debugging the Flow

If the steps above do not result in the expected output, then follow these steps:

1.  Open the workflow console and open the flows within the Okta Devices Review folder. The History column will display the number of successful and failed invocations of each flow. Look for any flows with failed invocations.
    
2.  If a flow has failed invoications, then open the flow and click on the Flow History. Select any invocation that failed and the flow execution will be displayed. Then scroll from left to right and find where the flow failed. Note that it may be a helper flow that failed. If that's the case, then open the helper flow and perform the same steps.
    
3.  If the flows are not being executed in under 3 seconds, then the Slack app may be timing out. If this is the case, then we improve the execution speed by removing the security check. This can be done by opening the flow titled [Route 0] Slack Interaction Router and updating the static assignment for variable bypass_security to true.
    

## Limitations & Known Issues

-   Due to Slack’s API limitations, if the flow does not respond within 3 seconds some submits may fail. Future releases of Okta Workflows will allow for reduced response times for certain flows, which should mitigate this issue.
    
-   Minimal error handling is included in the flows. Note that Slack will respond with a 200 OK even if the Workflow takes longer than 3 seconds to respond. There will be an error message in the response body from Slack, generally indicating that the trigger_id has expired. That’s the first place to look if messages aren’t being sent. This should only occur if Workflows is experiencing unusually long latency.
