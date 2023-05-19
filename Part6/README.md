# CORS with AWS Lambda and API Gateway

In this lesson, we will be configuring the API for our resume project. Since we're using AWS SAM, SAM actually provides some API structure set up for us already in the Amazon API Gateway.

## Contents
1. [CORS](#cors)
2. [Development](#development)
3. [API Gateway](#api-gateway)
4. [Conclusion](#conclusion)

## Development
Before we jump into this lesson, there's a couple things I want to outline in development.

```
sam local start-api
```

The command `sam local start-api` is used to start a local API Gateway server that emulates the behavior of AWS API Gateway locally on your development machine. It is part of the AWS Serverless Application Model (SAM) CLI, which allows you to develop and test serverless applications locally before deploying them to AWS.

When you run `sam local start-api`, it reads your SAM template (`template.yaml` or `template.yml` file) to determine the configuration of your API endpoints and their associated Lambda functions. It then starts a local HTTP server that listens for incoming requests on the specified port (default is 3000) and routes those requests to the corresponding Lambda functions.

This command is useful during development and testing stages, as it allows you to iterate quickly without the need to deploy your application to AWS. It provides a local environment for testing your API endpoints and their integration with Lambda functions, enabling you to debug and troubleshoot your code before deploying it to the cloud.

Now if we navigate to our local endpoint `http://localhost:3000/hello` this will invoke the lambda function.

The lambda function is setup in our template.yaml here:

```
...
Events:
    HelloWorld:
        Type: AWS::Serverless::Function
        Properties:
            Path: /hello
            Method: get
```

Which is a configuration for an event named "HelloWorld" in an AWS Serverless Application Model (SAM) template. It defines a serverless function that will be triggered by an HTTP GET request to the /hello path.

Here is a breakdown of the configuration:

`Events:`: This section specifies the events that trigger the serverless function.
`HelloWorld:`: This is the logical name or identifier for the event. It can be any unique name you choose.
`Type: AWS::Serverless::Function:` Specifies the type of event, which is an AWS Serverless Function.
`Properties:`: Denotes the properties associated with the serverless function.
`Path: /hello:` Specifies the URL path that triggers the function. In this case, it is set to /hello.
`Method: get:` Specifies the HTTP method that triggers the function. In this case, it is set to GET.
So, when an HTTP GET request is made to the /hello path, the serverless function defined in this configuration will be executed. The actual implementation of the function's logic would be provided elsewhere in the SAM template or in the referenced code file.
<br>

And the following in our template.yaml

```
...
    FirstFunction:
        Type: AWS::Serverless::Function
        Properties:
            CodeUri: hello_world/
            ...
```

Where this configures a serverless function named "FirstFunction" in an AWS Serverless Application Model (SAM) template. It specifies the code location for the function as the "hello_world/" directory.



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