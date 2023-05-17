# Deploying S3 Using AWS SAM - CloudFormation, IAM, and S3

In this lesson, we will be using AWS SAM (Serverless Application Model) to create an S3 bucket for storing our static website. 

## Contents
1. Installing the SAM CLI
2. Initializing SAM
3. Deploying SAM
4. Modifying the Template.yaml to Create an S3 Bucket
5. Defining "Infrastructure as Code"
6. Checking Our S3 Buckets

## Installing the SAM CLI
To use the SAM CLI, we must first install it. We can do this by running `sam init` and providing a project name, runtime to use, etc.

## Initializing SAM
After installing the SAM CLI, it will give us a `template.yaml` file, which is our CloudFormation, and a "hello-world" lambda function. We will look at this lambda function in a later lesson.

## Deploying SAM
To deploy SAM, we will run `sam build`, which will compile our templates, getting them ready to deploy. Before deploying, we need to give our user some permissions for SAM, including CloudFormation, IAM, Lambda, and API Gateway (`IAMFullAccess`, `AWSCloudFormationFullAccess`, `AWSLAmbda_FullAccess`, and `AmazonAPIGatewayAdministrator`). Then, we can run `sam deploy` using `aws-vault`, which we installed in Part 1. This will allow us to set up some default options, and now our file path has some default configs we don't need to type out again. 
```
sam build 
aws vault exec my-user --no-session -- sam deploy --guided 
```

![deploySAM](/images/deploySAM.png)

Our file structure now looks like this: (containing default configs)

![fileStruc](/images/fileStructure.png)

## Cloudformation in AWS
Cloudformation resources in AWS now exist, which were just created by SAM. This includes two stacks. One stack representing the base resources that allow SAM to function. The other stack is the one we created to represent our cloud resume challenge. Each of these stacks have resources associated with them.
![CFStack](/images/CFStack.png)
![CFSAM](/images/CFSAM.png)

Our cloudformation stack of "cloud-resume-challenge" has resources created related to our function like the lambda permissions, default API Gateway, IAM role, etc.
![CFCloudResume](/images/CFCloudResumeChallenge.png)

All of these resources were deployed by SAM and were all defined in our template in this resources section:
![CFTEmplate](/images/template.png)



## Modifying the Template.yaml to Create an S3 Bucket
Since we are deploying a static site using S3, we will modify the Resources in `template.yaml` to accept a new resource named "MyWebsite" with type "s3". We can run `sam build` again to compile down the template and deploy it using `aws-vault`.

Add resources to the template

![addResource](/images/newResource.png)

Deploy the new template with added resource
```
Sam build
aws-vault exec my-user --no-session -- sam deploy
```

This creates a new S3 bucket


## Checking Our S3 Buckets
We can run the following to "ls" our S3 buckets.
```
aws-vault exec my-user -- aws s3 ls
```


## Defining "Infrastructure as Code"
What we're doing here is defining "Infrastructure as Code". We'll come back this more in later lessons.


## Conclusion
Using AWS SAM, we were able to create an S3 bucket to store our static website. We learned about CloudFormation, IAM, and S3 and how to define "Infrastructure as Code".
