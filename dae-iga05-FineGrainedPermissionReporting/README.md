# README - IGA05 - Fine-grained Permission Reporting
This file provides some basic information for this sample set of workflows.

** *This is sample code only, and not officially supported by Okta* **

## Pre-reqs
Prior to using these flows you will need the following
- Okta Workflows configured for your environment
- A folder for these flows and table
- An Okta connector configured

## Installation
Select the folder you want to install the table/flows into and Import the .folder file. Activate the flows.

## Understanding the Flows/Tables
There are multiple flows and three tables.

- The first three flows (AD***) will extract Active Directory groups and determine users in those groups, and write the list to a workflows table
- The second two flows (O365***) will get all users assigned to the O365 app and for each, extract the fine-grained app assignment values and store them in a workflows table
- The last set of two flows (SF***) will get all users assigned to the Salesforce.com app and for each, extract the fine-grained app assignment values and store them in a workflows table

The resulting tables could be exported as CSV files and processed in a tool like Excel.


Most of the flows contain additional comments. More detailed information can be found in [Fine-Grained Entitlement Reporting with Workflows](https://iamse.blog/2022/03/22/fine-grained-entitlement-reporting-with-workflows/)
