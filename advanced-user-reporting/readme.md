

# Workflows How To: Advanced User Reporting
Okta provides a number of Out of the Box reports for Okta’s Workforce Identity Cloud (WIC) and Customer Identity Solution (CIS). These reports are based on the System Log and are therefore derived by user activity. A list of these reports can be found here:  [Report types | Okta](https://help.okta.com/en-us/Content/Topics/Reports/report-types.htm)

But what if you want a report based on the status of each user, including the total number of users in your tenant? This type of report can be produced by calling the Okta API. The Okta REST based API provides endpoints to configure and query objects within your tenant. The Okta Users API provides operations to manage users in your organization. This includes operations to search and list users. See the Users API doco here:  [Users | Okta Developer](https://developer.okta.com/docs/reference/api/users/#list-users-with-search)

Writing scripts to call the API, calculate totals and formatting and sending a report is time consuming and resource intensive. The good news is that Okta Workflows can do this for you.

Here is a sample of the type of report that can be produced with Okta Workflows:

![](https://i0.wp.com/iamse.blog/wp-content/uploads/2023/05/image.png?resize=1020%2C613&ssl=1)

Sample User Status Report

The above report can be generated and emailed to a configured list of recipients on a set schedule. It also maintains a history of the previous report and uses the saved data to provide a comparison. The flows used to generate this report can be downloaded from here:  [Advanced User Reporting Sample Flows](https://minhaskamal.github.io/DownGit/#/home?url=https://github.com/iamse-blog/workflows-templates/tree/main/advanced-user-reporting)

## Sample Flow Overview

The workflow has been constructed to process the data in a number of steps. By doing it this way, we can componentize the the logic into separate helper flows which helps with both reuse and in isolating issues.

The workflow starts by extracting all the users with their respective status and dumping the data in a working table (User Export table). This is done by the flow titled _[helper] Extract All Users_.

![](https://i0.wp.com/iamse.blog/wp-content/uploads/2023/05/image-7.png?resize=722%2C404&ssl=1)

Flow: [helper] Extract All Users

This flow uses the List Users with Search card to retrieve all users, regardless of status. The returned data is then streamed to another helper flow  _[helper] Write User to Table_  which creates a record on the workflow User Export table for every user returned.

Next, the helper flow titled  _[helper] Calculate Status Totals_  searches the User Export table for users with a particular status. The total users for the respective status is then derived using the List Length card. The total count is then written to the User Summary table.

An extract from the  _[helper] Calculate Status Totals_  flow is displayed below:

![](https://i0.wp.com/iamse.blog/wp-content/uploads/2023/05/image-9.png?resize=913%2C430&ssl=1)

Flow: [helper] Calculate Status Totals

Finally, the helper flow titled  _[helper] Send Summary Email_  is called to format and send the report. An extract of this flow is displayed below:

![](https://i0.wp.com/iamse.blog/wp-content/uploads/2023/05/image-10.png?resize=624%2C535&ssl=1)

Flow: [helper] Send Summary Email

## Sample Flow Setup

The rest of this article details how to get the sample flows up and running.

> **Note:** This workflow sample is not suitable for Okta tenants with over 100k of users.

### **Step 1 – Import Workflow Sample**

Within the Okta Workflow console, create a folder for the sample flows. The click on the three dots at the end of the folder name and select Import. Then import the previously downloaded sample workflow folder.

![](https://i0.wp.com/iamse.blog/wp-content/uploads/2023/05/image-1.png?resize=272%2C244&ssl=1)

Import into Folder

Once imported, the folder should contain the following flows:

1.  **[helper] Calculate Status Totals**  – Calculates the totals for each user status
2.  **[helper] Extract All Users**  – Get all users from underlying Okta org
3.  **[helper] Initialize Summary Table**  – Moves current totals to previous totals
4.  **[helper] Send Summary Email** – Sends a user status summary report via email
5.  **[helper] Write User to Table**  – Creates individual user record in the export table
6.  **[main] Run User Status Report** – This is the main flow that is used to run the report

The folder will also contain the following tables:

1.  **Config**  – Configuration table that holds static name/value pairs
2.  **Execution Dates**  – Dynamic table that stores the report execution dates
3.  **User Summary**  – Dynamic table that holds the calculated user status totals
4.  **User Export**  – Dynamic table that holds all the users extracted from the respective tenant

### **Step 2 – Configure Workflow Sample**

The flows use two connectors:

1.  Okta Connector
2.  Office 365 Mail Connector (Can be replaced with GMail Connector)

If either of these connectors don’t exist in your workflow instance, then add them from the connector catalogue now. The Office 365 Mail Connector will require a user account that will be used to send the report.

Next, open flow  **[helper] Extract All Users**  and update the Okta List Users with Search card to use your local Okta connector.

Next, open flow  **[helper] Send Summary Email**  and update the Send Email card at the end of the flow to use your local Office 365 Mail connector. This card can be replaced with the GMail Send Email card if using Google.

Next, open table  **Config**  and import the  **sample-config-data.csv**  file downloaded with the sample flows. Once imported, update the value column to your desired values as per the table below:

**Name**

**Description**

timezone

Timezone for Date/Time display. See the correct format here:  [List of tz database time zones](https://en.wikipedia.org/wiki/List_of_tz_database_time_zones)

recipients

A comma delimited list of email recipients

logo

A link to the logo on the resulting email

org

The name of your Okta Tenant. This can be in any format and is just used as a display in the resulting report.

Config Table

Next, open flow  **[main] Run User Status Report**  and click on the clock icon at the bottom of the Schedule Flow card:

![](https://i0.wp.com/iamse.blog/wp-content/uploads/2023/05/image-2.png?resize=241%2C265&ssl=1)

Schedule Flow Card

This will bring up the schedule popup.

![](https://i0.wp.com/iamse.blog/wp-content/uploads/2023/05/image-3.png?resize=435%2C363&ssl=1)

Schedule Flow Popup

Set the schedule to your desired interval.

Finally, turn each flow on by using the toggle ON/OFF switch. Once all the flows are on, we can now run a test.

### **Step 3 – Test Report Execution**

Even though the main execution flow has been scheduled to run at a particular time, the workflow can be tested at any time.

Open flow  **[main] Run User Status Report**  and click the test button at the top of the flow.

![](https://i0.wp.com/iamse.blog/wp-content/uploads/2023/05/image-4.png?resize=741%2C185&ssl=1)

Execute Flow via Test Button

The flow will then start execution and the console view will switch to debug. Depending on the number of users in your tenant, the flow may take a few minutes to complete. Additionally, the first time the flow runs, the execution data history will be empty, so the resulting report will just have zeros for the previous set of figures.

The flow history should indicate that execution was successful. Successful flow execution will result in a report being sent to each configured email recipient. If the flow history indicates an error, then flow that caused the error will be indicated with a red exclamation mark. Open the respective flow and go to the flow history. The flow history will indicate why the flow terminated with an error.

### **What you learned**

In this tutorial, you learned how to build a workflow that leverages Okta’s REST based API to create a custom report on user status. You learned:

-   The benefits of using Okta Workflows to call Okta’s API and generate a report rather than using custom scripts.
-   How the List Users with Search card can be used to retrieve users from an Okta tenant and stream the returned data to a helper flow.
-   How the List utility can be used to calculate totals.
-   How a HTML report can be formatted and sent via an Email Connector.

### **More resources**

-   🍫 Get help from Workflows specialists during weekly [community office hours](https://calendly.com/oktaworkflows/group-office-hours-okta-workflows).
-   📺 Learn from [Workflows videos](https://youtube.com/playlist?list=PLIid085fSVdvyK8F4xuk49EchBPmAVNHG).
-   🛟 Get help from support: [discuss a Workflows topic](https://support.okta.com/help/s/group/0F91Y000000PueUSAS/workflows?language=en_US) or [ask a question](https://support.okta.com/help/s/global-search/%40uri?language=en_US#t=All&f:ProductFacet=[Workflows]&f:ContentTypeFacet=[Discussions]).
-   🙋🏻‍♀️ Join the [#okta-workflows channel on MacAdmins Slack](http://macadmins.org/) to learn and get help from the community.

