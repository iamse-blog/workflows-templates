
# Pagination with Okta Workflows
Okta Workflows makes it easy to automate identity processes at scale – without writing code. Using the if-this-then-that logic, Okta’s pre-built connector library and the ability to connect to any publicly available API, anyone can innovate with Okta.

When working with large result sets, its impractical to try and process them in one single block. Pagination turns big archives of data into smaller, more digestible pieces. Clicking through an archive of pictures, or turning the page of a book, are examples of pagination. How is this important in API structure?

When we use a GET API request to request information from a server via API endpoint, there could be thousands of entries in the returned JSON file. The API response sending us thousands of entries at once is a drain of resources and a waste of our time, both in sending the request as well as processing the results. We want to search through a database a little bit at a time, and paging helps us query databases efficiently. In Okta Workflows it also helps in processing the results faster as we can spin off a new instance of a Helper flow to process each page of results.

But how would you structure a flow to take advantage of pagination? This blog entry provides one way to do just that.

There are three main forms of pagination:

1.  Offset-based pagination - This is the most common form of paging, it’s very easily implemented using SQL based databases. Essentially, the list of records is split into divisions that are called pages. The endpoint accepts a page parameter that asks for the next page and a limit parameter (optional).
    
2.  KeySet-Based pagination (index based) - KeySet-based pagination uses a key parameter that should be used as the same key for the sort order. Often it’s the unique identifier (Id), and then a key parameter of since_id or since_created, etc.
    
3.  Cursor-based Pagination - A cursor is a piece of data that points to the next element (usually forwards or backwards). The server returns a cursor pointing to the next page in each request. Sometimes, the server also provides a reverse pointer too so you can go back to the previous page.
    

This workflow example is based on Cursor-based Pagination as used in the Microsoft Graph API.

## Paging Microsoft Graph Data

Some queries against Microsoft Graph return multiple pages of data either due to server-side paging or due to the use of the `$top` query parameter to specifically limit the page size in a request. When more than one query request is required to retrieve all the results, Microsoft Graph returns an `@odata.nextLink` property in the response that contains a URL to the next page of results.

For example, the following URL requests all the users in an organization with a page size of 5, specified with the `$top` query parameter:

`1https://graph.microsoft.com/v1.0/users?$top=5`

If the result contains more results, Microsoft Graph will return an `@odata.nextLink` property similar to the following along with the first page of results:

`1"@odata.nextLink": "https://graph.microsoft.com/v1.0/users?$top=5&$skiptoken=X%274453707 ... 6633B900000000000000000000%27"`

You can retrieve the next page of results by sending the URL value of the `@odata.nextLink` property to Microsoft Graph.

`1https://graph.microsoft.com/v1.0/users?$top=5&$skiptoken=X%274453707 ... 6633B900000000000000000000%27`
## Workflow Structure

This is how I structured my workflow to take advantage of the Graph API’s cursor-based pagination.

![](https://i0.wp.com/iamse.blog/wp-content/uploads/2022/09/image-5.png?resize=1024%2C660&ssl=1)

The processing logic is as follows:

1.  **[main] Query Azure AD Users** - This is the main flow that starts the process. It calls helper flow _[util] Load Configuration_ to load the static configuration from a workflow table and calls helper flow _[util] Get Access Token_ which uses the Graph API to generate an access token.
    
2.  **[util] Get Page of Results** - This helper flow does the following:
    
    1.  Calls the Graph API to get a page of users from the respective Azure AD instance. The call uses the `$top` parameter to define how many results are returned. The value of `$top` is defined in the workflows static configuration. If this flow is passed a cursor, then it will use that in preference to the default Graph API URL.
        
    2.  Once a page of results is returned, the flow then calls _[util] Process Page_ to process the results. This flow is called asynchronously, which allows the calling flow to continue without waiting for it to complete. This asynchronous call is key to maximising the use of workflow resources and reducing processing time.
        
    3.  If the results include a cursor to the next page (`@odata.nextLink`), then this flow calls itself passing the cursor as input.
        
3.  **[util] Process Page** - This helper flow iterates though the list of results and calls helper flow _[util] Process User_ for each item in the list.
    
4.  **[util] Process User** - This helper flow loads the user into a workflow table

## Sample Workflow

You can download the sample flows here: [Pagination Example Workflow](https://minhaskamal.github.io/DownGit/#/home?url=https://github.com/iamse-blog/workflows-templates/tree/main/cursor-based-pagination "https://minhaskamal.github.io/DownGit/#/home?url=https://github.com/iamse-blog/workflows-templates/tree/main/cursor-based-pagination")
Use the following steps to get the sample flow working.

### Pre-Requirements
-   An Okta tenant with Workflows enabled
-   An Azure AD tenant with a large number of users

### Step 1 - Create Azure App Registration

Log into the Azure AD Admin Centre for you Azure tenant and do the following:

1.  On the Overview page for your Azure tenant, copy the Tenant ID. This will be required in a subsequent step.
    
2.  Under App Registrations, create a new registration. Take note of the apps Application (client) ID as this will be required in a subsequent step.
    
3.  Under the Certificate and Secrets menu, create a new client secret. Take note of the secret value as this will be required in a subsequent step.
    
4.  Under the Authentication menu, ensure Access Tokens has been enabled.
    
5.  Under the API Permissions menu, select Add a Permission and choose the Microsoft Graph API. Then select Application Permissions. Add permission Directory.Read.All. Finally grant Admin Consent for that permission.
    

### Step 2 - Import and Configure Sample Flows

Log into the Okta Workflow console and do the following:

1.  Create a new folder and import the downloaded workflow artefact titled _cursorBasedPagination.folder_
    
2.  Open the workflow configuration table and import the sample data file titled _sample_data.csv_
    
3.  The sample data provides the name/value pairs required for the flow. The values will need to be updated to the following:
    
    1.  **page_size** - This is the page size of the returned results. Depending on the size of your data, you may need to increase this value.
        
    2.  **client_id** - Update to the Client Id saved from Step 1
        
    3.  **client_secret** - Update to the Client Secret saved from Step 1
        
    4.  **tenant_id** - Update to the Tenant Id saved from Step 1
        
4.  Create a new API Connector with a Auth Type of None. (The workflow will be doing authentication outside of the connector by passing a generated access token)
    
5.  Open the helper flows titled _[util] Get Access Token_ and _[util] Get Page of Results_ and update the API Connector cards to use your API Connector that was created in the previous step.
    
6.  Toggle each of the six sample flows to ON
    

The workflow is now ready to test.

### Testing the Sample Flows

Open the flow titled _[main] Query Azure AD Users_. This flow is currently defined as a helper flow. Click on the Test button to run the flow. If the flow executes successfully, then the workflow users table will be populated with all the users from your Azure AD tenant.

### References
[![](https://www.brcline.com/favicon.ico)Implementing Paging in a REST API - Brian Cline](https://www.brcline.com/blog/implementing-paging-in-a-rest-api)
[What is API Pagination? Technical topics explained simply](https://www.abstractapi.com/api-glossary/api-pagination)
[Paging Microsoft Graph data in your app - Microsoft Graph](https://docs.microsoft.com/en-us/graph/paging)
