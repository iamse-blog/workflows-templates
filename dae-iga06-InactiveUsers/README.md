# README - IGA06 - Inactive Users
This file provides some basic information for this sample set of workflows.

** *This is sample code only, and not officially supported by Okta* **

## Pre-reqs
Prior to using these flows you will need the following
- Okta Workflows configured for your environment
- A folder for these flows and table
- An Okta connector configured
- Optionally a GDrive and email connector configured 
- The `okta.logs.read` scope granted for the Okta Workflows OAuth app in Okta. In Okta Workflows, go to connectors and reauthorize.

## Installation
Select the folder you want to install the table/flows into and Import the .folder file. Activate the flows.

## Understanding the Flows/Tables
There are eight flows and one table.

The flows are:
- Find inactive Okta users (i.e. Okta user logins):
-- **M10 - Find inactive Okta users** - this will search for any Okta users who have not logging in for a period of time
-- **M11 - Store Okta No Recent Login** - for each found (inactive) user, log their details to the **T1** table
- Get all application logins for a period (i.e. application SSO events):
-- **M30 - Get All App Logins for Period** - search for all `user.authentication.sso` events in the syslog indicating a successful SSO in the last 30 days
-- **M31 - Process Individual SSO** - for each successful login, write the details to the **T3** table
- Get O365 users and check if they've logged in - knowing all the app logins (M3* flows) we can check for users of a specific app (O365 in this example) to see if they logged in over that time and if not, they are inactive
-- **M40 - Get O365 Users and Check Login** - list all users assigned to the specific app, check if they are in the list of recent logins, and if not, write them to a table (**T4**) and then send the table as a report to a nominated owner. *This flow is a sample for any application inactive account reports*.
- Utilities (common flows used by other flows)
-- **U00 - Check User in Login Table** - this flow is passed a user and application and determines if they are in the recent logins table (**T3**) and if not, writes the record to an app-specific inactive logins table
-- **U10 - Send Report via Email** - this flow will export a nominated table as CSV, save it to a Google drive, then email the nominated report recipient
-- **U90 - Get Flow Variable** - this set of flows use environment variables stored in a table (**T9**). This flow is for extracting a specific environment variable.

The main flows (M10, M30 and M40) are run on demand. M40 expects that M3* has been run as it's dependent on the recent logins table to be populated. 
