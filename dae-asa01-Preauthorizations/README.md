# README - ASA01 - Preauthorizations
This file provides some basic information for this sample set of workflows.

** *This is sample code only, and not officially supported by Okta* **

## Pre-reqs
Prior to using these flows you will need the following
- Okta Advanced Server Access configured into your Okta environment and a project with pre-auth set
- Okta Workflows configured for your environment
- A folder for these flows and table
- An Okta connector configured
- An Okta ASA connector configured

## Installation
Select the folder you want to install the table/flows into and Import the .folder file. Activate the flows.

## Understanding the Flows/Tables
There are three flows and one table.

The flows are:
- **MAIN - Add User to ASA Preauth** - triggered by user being added to a specific dummy application in Okta, it will get details of the user from the event, determine now+2hrs, create the preauthorization (against a hardcoded project) and write the user/time to the PreAuth History table
- **MAIN - Remove Expired Preauths** - is run on a schedule and will poll the PreAuth History table for expired preauths and for each one call a sub flow
- **SUB - Remove single preauth** - is called from the MAIN above and will remove the user from the dummy app in Okta and update the PreAuth History table


More information can be found in [ASA PreAuthorization with Okta Workflows](https://iamse.blog/2022/04/11/asa-preauthorization-with-workflows/)
