# README - IGA03 - Continuous Certification
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
There are multiple flows and three tables.

The concept of the flows is that when a user changes group or application membership, you want to notify their manager of the change and allow review. The main flows will respond to four Okta events: user added to group, user assigned to application, user removed from group and user unassigned from application.

There are multiple sub flows and utility flows used by the main flows, including flows to write audit events. 


Most of the flows contain additional comments. More detailed information can be found in [Continuous Certification With Okta Workflows](https://iamse.blog/2022/04/05/continuous-certification-with-okta-workflows/)
