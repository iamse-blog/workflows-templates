# README - IGA00 - SoD (Separation of Duties)
This file provides some basic information for this sample set of workflows.

** *This is sample code only, and not officially supported by Okta* **

## Pre-reqs
Prior to using these flows you will need the following
- Okta Workflows configured for your environment
- A folder for these flows and table
- An Okta connector configured
- An email and Slack connector configured 

## Installation
Select the folder you want to install the table/flows into and Import the .folder file. Activate the flows.

## Understanding the Flows/Tables
There are multiple flows and two tables.

The flows are:
- Main and sub flows:
-- **M00 - User Add to Group Check SoD** - this flow will run when a user is added to a group. It will check if this group is one of the SoD groups and if so, check against the SoD policies in the SoD policy table. If found and the policy is flagged for enforce, the user is removed from the new group. Then email and slack notifications are sent out.
-- **S00 - Compare Curr to SoD Group Lists** - this flow compares a users current group list to the SoD policy group list to determine a match
-- **S90 - Is SoD Group** - check to see if the passed group is in the SoD group table
- Utility flows:
-- **S99 - Get Flow Variable** - this set of flows use environment variables stored in a table (**T9**). This flow is for extracting a specific environment variable.
-- **U00 - Build SoD Groups EnvVar** - work through the SoD policy table to find all groups in any SoD policy and store them in a single environment variable (done for performance)
-- **U00a - Process a list of groups** - sub flow for the U00 flow
- There are also some test (Z0n) flows included 


Most of the flows contain additional comments. More information can be found in [Separation of Duties (SoD) With Okta Workflows](https://iamse.blog/2022/04/22/separation-of-duties-sod-with-okta-workflows/)
