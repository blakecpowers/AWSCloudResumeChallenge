# CORS with AWS Lambda and API Gateway

In this lesson, we will be configuring the API for our resume project. Since we're using AWS SAM, SAM actually provides some API structure set up for us already in the Amazon API Gateway.

## Contents
1. [CORS](#cors)
2. [API Gateway](#api-gateway)
3. [Conclusion](#conclusion)

## API Gateway
It was explained a little in the Intro but I want to be clear. The API Gateway has already been setup with SAM and we can navigate to API Gateway right now to find this API.

![apigateway](/images/apigateway.png)

Once clicked: 

![apigateway2p5](/images/apigateway2p5.png)

We can navigate to Stages and then Prod. Where we already have a GET method for this API and an invoke URL. 

![apigetp5](/images/apigetp5.png)

If we clicked on the URL right now we would see this error, telling us that we need to configure CORS. 

![corserrorp6](/images/corserrorp6.png)

## CORS
CORS (Cross-Origin Resource Sharing) is a browser security feature that allows web applications on different domains to safely interact with each other. It ensures that only trusted domains can access resources (such as data or assets) from other domains, protecting user privacy and preventing unauthorized access.

For this lesson, we're going to make CORS public. So we'll configure the API Server to allow cross-origin requests from any origin by setting the apropriate response headers. In a later lesson, we'll convert it so CORS is not public. 

In our codebase, we create a `main.go` file to represent our server code (coded in go, but we could also use python).

![maingohandler](/images/maingohandler.png)

This is the response from the lambda function. We're adding an additional headers property with a header of `Access-Control-Allow-Origin: "*"`, `Access-Control-Allow-Methods: "*"`, and `Access-Control-Allow-Headers: "*"` which allow any request from any client to request this API. 

This is a loose policy, we will change this later.

For the Body, we return a count. In a later lesson we'll get this "count" from the database to represent how many people have visited our website, but for now we just return a count of 2.

A status code of 200 is also sent. 

Above, we saw that if we navigated to the API URL the CORS error would appear. Adding these three headers to our lambda response will get rid of this cors error. Now we can deploy this using our handy make file command

```
make deploy-infra
```

Navigate to the API URL again where we see the counts

![apicountsp5](/images/apicountsp5.png)

## Conclusion
In conclusion, configuring CORS (Cross-Origin Resource Sharing) is essential when building web applications that interact with resources from different domains. In this tutorial, we focused on making CORS public, allowing cross-origin requests from any origin. By setting the appropriate response headers, such as `Access-Control-Allow-Origin: "*"`, `Access-Control-Allow-Methods: "*"`, and `Access-Control-Allow-Headers: "*"`, we enabled unrestricted access to our API from any client. However, it's important to note that this loose policy may have security implications, and in a later lesson, we will explore more secure configurations. By understanding and properly configuring CORS, we can ensure smooth communication between our web application and external resources while maintaining security and protecting user privacy.