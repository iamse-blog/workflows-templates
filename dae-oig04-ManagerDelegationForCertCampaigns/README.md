# README - OIG04 - Manager Delegation for Certification Campaigns
This README file provides some basic information for this sample set of workflows.

** *This is sample code only, and not officially supported by Okta* **

## Overview
This set of flows implements a manager delegation function for Access Certification Campaigns. It is based on the assumption that an external mechanism (like a HR Feed) would manage some attributes on the Okta user profile (details below) and these flows would run periodically (perhaps daily) to reassign Access Certification reviews from one manager to another. It is implemented using a series of workflows and tables.

There are some assumptions made in the implementation of this:
- Users have a managerId (manager username) in the user.profile.managerId attribute
- Three new attributes are added to the user profile: reassignStartDate, reassignTempManager and reassignStopDate that are managed externally to this solution (such as via a HR Feed)
- When a manager is reassigned across a number of users, the same new (temp) manager is assigned and the same start/stop dates of the reassignment are used for all users
- This solution will run through any active campaigns and look for any active reassignments (i.e. current date is between the start/stop dates). It does not revert any entries outside of the period (i.e. if the reassign manager does not complete their reviews before the stop date, these is no process to revert the reviews to the normal manager)
- This solution currently only handles Access Certification, not Access Requests
- This solution only looks at managers associated with users, not other reviewer types

See the relevant https://iamse.blog article for more details.

## Pre-reqs
Prior to using these flows you will need the following
- Okta Workflows configured for your environment
- A folder for these flows and table
- An Okta connector configured
- An access token to use with the OIG API calls

## Installation
Select the folder you want to install the table/flows into and Import the .folder file. Activate the flows.

## Configuration
There are multiple configuration steps required: add new attributes to the Okta user profile (and populate) and setup environment variables.

### Add Attributes to the Okta User Profile
Three new (custom) attributes must be added to the Otka user profile. They can have any Display Name, but must use the following Variable Names:
- reassignStartDate - the flows expect the date to be YYYY-MM-DD format,
- reassignTempManager - the username for the new/temp manager, and
- reassignStopDate - the flows expect the date to be YYYY-MM-DD information

To test the flows, you will need some users (who can be in a campaign) with these values set.

### Set Environment variables
There are two environment variables to be set in the Environment Variables table:
- varName = apiToken, varValue = your API token from your Okta instance
- varName = oktaDomain, varValue = the fullname of your Okta instance without the https:// (e.g. deadwoods-oig.oktapreview.com)

## Execution
There are two flows to be run (on-demand for testing, but scheduled for prod):
- First *M00 - Search Users with Reassignment* to get all users with manager reassignment and store them in a table,
- Then *M10 - Reassign Reviewers in Active Campaigns* to process all the reassignments

Note that there is a standalone reporting flow (*R10 - Get all User Managers*) that can be run to see all users and their managers (via the user.profile.managerId attribute).

## Understanding the Flows/Tables
Details of the flows can be found in the article on https://iamse.blog.
