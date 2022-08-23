# Okta Workflows How-To: Retrieve Slack user status and other info using Custom API cards
##### By [Bryan Barrows](https://www.linkedin.com/in/bbarrows89/)

---
Okta Workflows includes an ever-growing library of connectable applications with pre-built actions and events that allow you to quickly automate business processes with ease.

However, there are times when we might know that an API that we're leveraging has a different endpoint or some other method by which we could extract additional necessary information.

These situations are what the Custom API Action cards included with each app connector are made for.

---

In this example, I've leveraged [Slack - Custom API Cards](https://help.okta.com/wf/en-us/Content/Topics/Workflows/connector-reference/slack/actions/httprequest.htm) to retrieve the full user profile as well as their status _(active/away)_. 

These details aren't available in the default `Slack - Read User` card, but by leveraging the Custom API Action, we're able to take advantage of more of the functionality of Slack's API.


![Flow Design - Get Slack User Status and Details](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/b54rsgkp10qnthh9pe0d.png)


To accomplish this, we'll reference [Slack's API docs](https://api.slack.com/) to learn more about the [/users.info](https://api.slack.com/methods/users.info) and [/users.getPresence](https://api.slack.com/methods/users.getPresence) methods.

A simple `GET` to those Relative URLs should do the trick...

1. Use a `Slack - Read User` get the Slack User ID.
2. Use an `Object - Construct` with a key of `user` and input the Slack ID from the previous step. We'll use this query object in both of our Slack - Custom API cards to specify which user we want information about. 
3. Use two `Slack - Custom API` cards with a method of `GET` and the following Relative URLs:
  * `/users.info`
  * `/users.getPresence`
4. Drag the object output from Step 2 into the "Query" input of each Custom API card.

And that's all there is to it! Looks like it worked:

![Successful Flow Execution - Get Slack User Details and Status](https://dev-to-uploads.s3.amazonaws.com/uploads/articles/ksq75y9gr3x3pkdm135a.png)

Now you can use a pair of [Object - Get Multiple](https://help.okta.com/wf/en-us/Content/Topics/Workflows/function-reference/Object/object_pick.htm) cards to parse whatever data you need out of each response body. 

#### Notes 
* I've used `Okta - User Added to Group` for sake of example. You can substitute this to meet your needs. Just make sure to use the `Slack - Read User` to get the Slack User ID needed for the query object. 
* If you zoom in on my flow, you'll also notice that I built some error handling into my flow by putting my Custom API calls within an "If Error" card and building outputs in the case of a success as well as a failure. It's never a bad idea to build some error handling into your flows to make them more robust. 

Feel free to [download](https://github.com/bbarrows89/oktaworkflows/blob/main/guides/getSlackUserStatus/getSlackUserDetailsAndStatus.flow) this example and import it to your environment.

**_Download steps:_** 
* _right click "view raw"_ 
* _click "Save Link As"_
* _be sure the filename ends in `.flow` !_

_Read more about how to import and export flows [here](https://help.okta.com/wf/en-us/Content/Topics/Workflows/build/export-import-flows.htm)._ 

---

Hope this helps! Find me on [LinkedIn](https://www.linkedin.com/in/bbarrows89/) or join us at a [Community Office Hours](https://calendly.com/oktaworkflows) session if you have any questions or want to chat about what you're working on - would love to see you there!
