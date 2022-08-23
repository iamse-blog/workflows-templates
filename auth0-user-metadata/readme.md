

# Auth0 Integration with Okta Workflows
The Customer Identity Cloud (aka Auth0 Identity Platform), a product unit within Okta, takes a modern approach to identity and enables organizations to provide secure access to any application, for any user. Auth0 is a highly customizable platform that is as simple as development teams want, and as flexible as they need. In todays product landscape, it’s becoming more common to see both Okta and Auth0 working together. This could be within a single enterprise where Okta handles internal workforce identities and Auth0 looks after customer identities, or in cross enterprise environments like B2B, where a business partner may be using one product or the other.

Okta Workflows makes it easy to automate identity processes at scale – without writing code. Using the if-this-then-that logic, Okta’s pre-built connector library, the Connector Builder and the ability to connect to any publicly available API, anyone can innovate with Okta. In this blog entry we are going to look at using Okta Workflows to facilitate the integration between Okta and Auth0.

## Use Case

Let start with a real simple use case so we can look at how we can integrate the two products. There may be situations like B2B, where the same identity exists in both Okta and Auth0. If the identity is deactivated in one environment, then the other environment will need to know about it. So in this example, if a user is deactivated in Okta, we are going to update the users metadata in Auth0 to indicate that status change. Auth0 can then pick up that change and deny the user access.

This simple flow is described in the following diagram.

![](https://i0.wp.com/iamse.blog/wp-content/uploads/2022/08/image-10.png?resize=594%2C130&ssl=1)

In the above diagram, the processing sequence is as flows:

1.  A user in Okta is deactivated (The same user also exists in Auth0).
2.  A call is made to Okta to retrieve the users email address.
3.  Using the email address, we then call Auth0 to retrieve the users unique Auth0 identifier.
4.  Using the identifier we update the users metadata to include a flag that indicates the user has been deactivated in Okta.
5.  Using an Auth0 Action, we check the users metadata during login. If the metadata indicates the user has been deactivated in Okta, then the user is denied access.

## Pre-Requirements

To implement is example, you will need the following:

1.  An Okta tenant
2.  An Auth0 tenant
3.  Okta Workflows enabled in your Okta tenant

## Step 1 – Create Auth0 Application

Within your Auth0 management console, go to Applications and create a new application of type Machine to Machine. This application will be using the Auth0 Management API and will need to be able to read and update users.

![](https://i0.wp.com/iamse.blog/wp-content/uploads/2022/08/image-11.png?resize=559%2C423&ssl=1)

Take note of the following values once created.

1.  Domain
2.  Client ID
3.  Client Secret

Here is an example:

![](https://i0.wp.com/iamse.blog/wp-content/uploads/2022/08/image-12.png?resize=532%2C362&ssl=1)

We will be using these values in subsequent steps where we need to generate an Auth0 Access Token.

## Step 2 – Download and Import Sample Flows

The sample flows that demonstrate this use case can be found here:  [auth0UserMetaData.folder](https://github.com/iamse-blog/workflows-templates/blob/main/auth0-user-metadata/auth0UserMetaData.folder)

Create a folder within Okta Workflows and import the artefact titled auth0UserMetaData.folder

Once imported, you should have the following flows:

1.  **[main] User Deactivated**  – This is the main flow and will initiated every time a user in your respective Okta tenant is deactivated.
2.  **[util] Get Access Token**  – This helper flow is used to call your Auth0 tenant to generate an Access Token
3.  **[util] Get Auth0 User Id**  – This helper flow is used to retrieve the respective users Auth0 identifier based on the users Okta email address
4.  **[util] Get Configuration**  – This helper floe retrieves some static values from the configuration table
5.  **[util] Update Auth0 User** – This helper flow is used to update the Auth0 user and set a flag in their user metadata

You will also have the following table:

1.  **configuration**  – Used to store some static configuration values

## Step 3 – Update Sample Flows and Table

The next step is to update the configuration table to include name / value pairs for domain, client_id and client_secret. These are the values saved from Step 1. Once completed, your configuration table should look like the following example:

![](https://i0.wp.com/iamse.blog/wp-content/uploads/2022/08/image-13.png?resize=903%2C153&ssl=1)

> **Note:**  These values can be hidden by encompassing some of the Auth0 logic within its own workflow connector. These values can be used in the connector authorization and will therefore not be accessible to users once the connector has been initialized.

Next, create an Okta Connector for your local Okta tenant if one does not already exists in your workflow instance.

Then open the flow titled  **[main] User Deactivated**  and set the Okta connector for the two Okta cards (User Deactivated and Read User). As an example, the start of the sample main flow is displayed below. We can see that the User Deactivated and Read User cards are using the MS2 Okta connector. Update these cards to use your own Okta connector.

![](https://i0.wp.com/iamse.blog/wp-content/uploads/2022/08/image-14.png?resize=1024%2C527&ssl=1)

Finally, ensure all of your flows have been activated.

Here is the remainder of my main flow:

![](https://i0.wp.com/iamse.blog/wp-content/uploads/2022/08/image-15.png?resize=1100%2C323&ssl=1)

The processing of the main flow is as follows:

1.  Using the passed User Id, the user is read to retrieve the users primary email.
2.  A call to a helper flow retrieves the values stored in the configuration table.
3.  A call to a helper flow retrieves a Auth0 Access Token based on the stored domain, client Id and client secret. This help flow uses the Client Credential flow.
4.  Using the Access Token, a call to a helper flow is made to retrieve the users unique Auth0 identifier.
5.  Finally, a call to a helper floe is made to update the users metadata to indicate that the user has been deactivated in Okta.

## Step 4 – Create Auth0 Action

There are a number of ways to prevent a user from accessing your website or application that is protected by Auth0. One way is to use Auth0 Actions to evaluate the users metadata during login. Based on the value, the user can then be prevented from authenticating. I have included an example below.

`exports.onExecutePostLogin = async (event, api) => {`

`if (event.user.user_metadata.okta.deactivated === "true") {`

``api.access.deny(`Your account has been deactivated!`);``

`};`

`};`

## Testing the Workflow

To test your workflow, ensure you have the same user existing in both Okta and Auth0. Then deactivate your user in Okta. The workflow should then run and update the users metadata in Auth0.

Here is an example of the updated metadata:

![](https://i0.wp.com/iamse.blog/wp-content/uploads/2022/08/image-16.png?resize=367%2C228&ssl=1)

Now if you try and authenticate as that user in Auth0, you should be prevented.
