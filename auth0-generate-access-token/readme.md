
# Generate Auth0 Access Token via Okta Workflows

Okta Workflows makes it easy to automate identity processes at scale – without writing code. Using the if-this-then-that logic, Okta’s pre-built connector library and the ability to connect to any publicly available API, anyone can innovate with Okta.

The Customer Identity Cloud (aka Auth0 Identity Platform), a product unit within Okta, takes a modern approach to identity and enables organizations to provide secure access to any application, for any user. Auth0 is a highly customizable platform that is as simple as development teams want, and as flexible as they need. In todays product landscape, it's becoming more common to see both Okta and Auth0 working together as there are many benefits in facilitating integration.

In this scenario, it's likely that Okta Workflows will be calling the [![](https://cdn.auth0.com/website/new-homepage/dark-favicon.png)Auth0 Management API v2](https://auth0.com/docs/api/management/v2). The primary way that this API is called is by passing an Access Token in the Authorization header. This blog entry looks at how we can generate an Access Token via Okta Workflows using the client credential flow. The Client Credential flow is designed for Machine to Machine communication.

## Create Auth0 Application

Within your Auth0 management console, go to Applications and create a new application of type Machine to Machine. You will need to authorize this application to perform the required tasks, so the token will be generated with the necessary scopes. If this is not correct, any request using the token will receive a 401 - Unauthorized.

Here is an example:

![enter image description here](https://i0.wp.com/iamse.blog/wp-content/uploads/2022/10/image.png?w=559&ssl=1)

The resulting token will include the following:

`1  "scope":  "read:users update:users",`

Take note of the following values once created.

1.  Domain
    
2.  Client ID
    
3.  Client Secret

EG:

![enter image description here](https://i0.wp.com/iamse.blog/wp-content/uploads/2022/10/image-1.png?w=532&ssl=1)

These values will be required as input to the following helper flow.

## Workflows Helper Flow

The sample helper flow can be downloaded here:

Here is an explanation of the helper flow.

1.  The Domain, Client Id and Client Secret (from the Auth0 M2M application) are passed into the flow. These values would usually be abstracted away in a workflows table, rather than hard coding them in the flow.
    
2.  Using the passed Domain, the flow manufactures the token endpoint URL. This is the endpoint that the flow will be calling to generate a token.
    
3.  Using the passed Domain, the flow manufactures the audience. This will be used in the JSON payload.

    ![enter image description here](https://i0.wp.com/iamse.blog/wp-content/uploads/2022/10/image-4.png?resize=1024,583&ssl=1)
    
4. Using the passed Client Id, Client Secret, manufactured Audience and a grant type of _client_credentials_, the flow then creates a JSON object. This will be the payload in the request.

5. The flow creates a header object with a Content-Type of _application/json_.

![enter image description here](https://i0.wp.com/iamse.blog/wp-content/uploads/2022/10/image-5.png?resize=768,747&ssl=1)

6. Now the flow does a HTTP Post to the token endpoint with the JSON payload.

![enter image description here](https://i0.wp.com/iamse.blog/wp-content/uploads/2022/10/image-6.png?w=269&ssl=1)

7. The flow then checks the returned status code from the call. If it's a 200, then the flow extracts the access token from the response. If a 200 is not returned, then the flow will return an error response.

![enter image description here](https://i0.wp.com/iamse.blog/wp-content/uploads/2022/10/image-7.png?resize=768,788&ssl=1)

8. Finally the helper flow returns the token to the calling workflow.

![enter image description here](https://i0.wp.com/iamse.blog/wp-content/uploads/2022/10/image-8.png?resize=259,300&ssl=1)
