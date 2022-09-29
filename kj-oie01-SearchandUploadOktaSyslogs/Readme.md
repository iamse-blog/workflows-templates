**Use Case Overview****

Okta administrators sometimes get requested to selective filtering off Okta System logs for "User Behaviours" see this page: Behaviour Detection and feed the data into a SIEM endpoints or custom REST endpoint. 

Okta has got a detailed documentation on exporting Okta log data, see this page: Exporting Okta Log Data

Note: This blog focuses on uploading logs into a custom API REST endpoint (POST) where existing available integrations is not sufficient.

**Workflow Structure**

The processing logic is as following,

[main] Get Syslogs - This flow computes the date filter and passes a custom log filter query (debugContext.debugData.behaviors co "UNKNOWN") to Okta Search System Logs card to stream log results.

[helper] Post Syslogs - This flow processes the streamed record from main flow (Get Syslogs), where it extracts the required data from Okta System logs to load the records into a table (CustomSyslog) and also posts to a web hook endpoint.

**Pre-Requirements**

A Okta tenant

Okta Workflows enabled within the Okta tenant

Webhook/REST endpoint for loading data
