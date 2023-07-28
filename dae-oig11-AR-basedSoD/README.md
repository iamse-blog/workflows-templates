# README - IGA11 - AR-based SoD (Separation of Duties)
This file provides some basic information for this sample set of workflows.

** *This is sample code only, and not officially supported by Okta* **

## Pre-reqs
Prior to using these flows you will need the following
- Workflows:
-- Okta Workflows configured for your environment
-- A folder for these flows and table
-- An Okta connector configured
- OIG:
-- Okta Identity Governance (OIG) with Access Requests enabled and working
-- Workflows OAuth app has the okta.governance.accessRequests.manage scope granted
-- ATSPOKE_WORKFLOWS feature flag enabled

## Installation
Select the folder you want to install the tables/flows into and Import the .folder file. Activate the flows. Due to the new mapping requirement, you may want to enable the flows in reverse alphabetical order (sub flows then main flows).

## Initial Setup
You will need to do the following to use the flows.

### Configure Environment Variables table
There is a table called **Environment Variables** that needs two values:
- oktaDomain - that has your Okta org domain in the form of mydomain.okta.com (or mydomain.oktapreview.com)
- apiToken - you need to create an API token so that workflows can run raw HTTP API requests 

### Configure Resource Type Entitlements table
The table **Resource Type Entitlements** contains the mapping of Access Request Request Types (flows) to the entitlements they grant. You will need this for the E** set of flows. The entitlements are specified as a comma-separated list and are entered in the form X:name (X = A for Apps and G for groups, and Name is the Okta app label or group name).

### Configure SoD Policies table
You will need to define one or more SoD policies in the **SoD Policies** table. It contains an id (not used), name, description, scope (# allowed before violation), entitlements (in the same X:name format) and severity. An example is included in this folder.

### Assign Worflow to an Access Request
To use the D** flows from an Access Request, you must assign the workflow to the Request Type. If unsure of how to do this, see https://iamse.blog/2023/07/20/oig-access-requests-posting-additional-information-into-a-request/. The workflow to include is D00 - Update Request with User Data (AR-called). It requires the user email, entitlementId and requestSubject to be passed from the Access Request.

### Define Request Types for E** Flows
The E** flows are triggered off the access.request.create event. There is no filtered hooks for this event yet, so every event from every access request will trigger this workflow. The E00 flow has a Continue If card that contains a list of flows to do the SoD checking for. Update that list (value b in the card)

## Understanding the Flows/Tables
There are multiple flows and three tables.

### Tables
There are three tables that must be populated before any of the flows are run:
1. **Environment Variables** - some variables used throughout the flows
2. **Resource Type Entitlements** - mapping of Request Types to entitlements 
3. **SoD Policies** - Separation of Duties policies

### Flows
The sets of flows are:
- **Dnn** - The D00 Delegated Workflow is called from a newly created request in Access Requests. It uses the D0* subflows  for data setup and then calls the S00 flow to run the SoD check.
- **Enn** - The E00 flow is triggered by the access.request.create event in the Okta System Log. It uses the E0* subflows for data setup and then calls the S00 flow to run the SoD check.
- **Snn** - The S00 flow is the main flow for the SoD check (called by either D00 or E00). It uses a number of S** subflows to perform the SoD violation checks.
- **Unn** - Various utility flows

Most of the flows contain additional comments. More information can be found in [OIG Access Requests and Workflows – Checking SoD In An Access Request](https://iamse.blog/2023/07/26/oig-checking-sod-in-an-access-request/)
