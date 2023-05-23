# Continuous Integration / Continuous Deployment
In this lesson, we will be implementing CI / CD in our project. We'll take a look at what CI / CD means, adding testing using github actions, deploying our infrastructure using github actions and lastly deploying to S3 in order to update our site using github actions.


## Contents
1. [Continuous Integration Continuous Deployment](#continuous-integration-continuous-deployment)
2. [Unit Testing in Github Actions](#unit-testing-in-github-actions)
3. [Deploying AWS SAM with Github Actions](#deploying-aws-sam-with-github-actions)
4. [Deploying an S3 Website with Github Actions](#deploying-an-s3-website-with-github-actions)
5. [Conclusion](#conclusion)

## Continuous Integration Continuous Deployment
Before we implement CI/CD (Continuous Integration/Continuous Deployment) in our project, let's take a look at what these mean.

- Continuous Integration (CI):
    Continuous Integration is a development practice that focuses on frequent and automated integration of code changes into a shared repository. The main goal of CI is to enable developers to integrate their code changes regularly and detect integration issues early in the development process. With CI, developers typically commit their code changes to the repository multiple times a day, and each commit triggers an automated build process, which compiles the code, runs tests, and performs other necessary checks. This approach helps to identify any conflicts or errors early on, allowing teams to address them quickly. By continuously integrating code, CI promotes collaboration, reduces integration problems, and improves the overall software quality.
- Continuous Deployment (CD):
    Continuous Deployment is an extension of continuous integration that takes automation a step further by automatically deploying the successfully built and tested code changes to a production environment. With CD, once the code changes pass all the necessary tests and checks in the CI process, they are automatically deployed to the live production environment, making them immediately available to end-users. This automation eliminates the need for manual intervention in the deployment process, reducing the risk of human error and enabling frequent and reliable software releases. Continuous Deployment aims to deliver new features, bug fixes, and improvements to users rapidly and efficiently, ensuring a smooth and continuous delivery pipeline.
In summary, Continuous Integration focuses on regularly integrating and testing code changes, whereas Continuous Deployment builds upon CI by automating the process of deploying code changes to production environments. Both practices are integral to modern software development methodologies, promoting collaboration, quality assurance, and faster delivery of software.

## Unit Testing in Github Actions
GitHub Actions is a workflow automation and CI/CD platform that allows developers to define custom workflows as code, automate tasks, and integrate with various tools and platforms to streamline the software development process.

In order to use github actions, we need to create a folder named ".github", within that folder another folder named "workflow", and within that is where all the github action files / workflow lives. This is a yml file, and in our case we chose to name it "main.yml".

Our main.yml file looks like this - 

```
name: main
on: push
env:
  GO_VERSION: 1.15.x

jobs:
  test-infra:
    runs-on: ubuntu-latest
    timeout-minutes: 2
    steps:
      - uses: actions/checkout@v2
      - uses: actions/setup-go@v2.1.3
        with:
          go-version: ${{ env.GO_VERSION }}
      - name: test get-function
        run: cd cloud-resume-challenge/get-function && go test -v ./ && cd ../../
      - name: test put-function
        run: cd cloud-resume-challenge/put-function && go test -v ./ && cd ../../
```

This "main" file is triggered by a push event. So when you push changes to the github repo. The workflow includes one job named "test-infra" that runs on an Ubuntu environment and has a timeout of 2 minutes.

The job consists of several steps:

- Checking out the repository using the actions/checkout action.
- Setting up the Go programming language version 1.15.x using the actions/setup-go action (so that it can test this Go code).
- Running tests for the get-function by executing the go test command in the cloud-resume-challenge/get-function directory.
- Running tests for the put-function by executing the go test command in the cloud-resume-challenge/put-function directory.

![githubactionsp8](/images/githubactionsp8.png)

Here we can see our main workflow that we defined. And if we click into that, our job that we defined with the individual steps..

![buildinfrap8](/images/buildinfrap8.png)

We can also click into the test to see the tests that were ran and whether they passed or not, running each time a push is made. 

![testp8](/images/testp8.png)

## Deploying AWS SAM with Github Actions
Next, we'll be deploying infrastructure from Github Actions into AWS. Here, we'll be adding onto our main.yml file again by making github actions build our AWS SAM and deploy it. Before this, we've been doing this locally each time by running the command in our Makefile. Now, this command will be ran to deploy our AWS SAM each time code is committed. Lets take a look - 

```
build-and-deploy-infra:
    needs: test-infra
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2
      - uses: actions/setup-python@v2
      - uses: aws-actions/setup-sam@v1
      - uses: aws-actions/configure-aws-credentials@v1
        with:
          aws-access-key-id: ${{ secrets.AWS_ACCESS_KEY_ID }}
          aws-secret-access-key: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
          aws-region: us-east-1
      - run: sam build
        working-directory: cloud-resume-challenge
      - run: sam deploy --no-confirm-changeset --no-fail-on-empty-changeset
        working-directory: cloud-resume-challenge
```

At the top we see "needs: test-infra". This means that this build and deploy step requires the test-infra step to be completed first. 

We also check out our repo again. We use python here since it's required for AWS SAM and we configure our AWS credentials. 

We pass in our access key and secret access key. 

We run the sam build and sam deploy (which is what we were running locally). Note: We're also passing in extra parameters which lets us skip the "confirming" steps when deploying. 

Note: Notice we're passing in secrets using "secrets.", this actually comes from the repository secrets which is handled by Github. In our case there would be two, named "AWS_ACCESS_KEY_ID" and "AWS_SECRET_ACCESS_KEY"

![secret](/images/secret.png)

## Deploying an S3 Website with Github Actions
Here, we'll be going through the part of uploading into S3 using github actions. This is to actually update our website once we push to github. 

Once again, we'll be adding to our main.yaml the following - 

```
deploy-site:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2
      - uses: jakejarvis/s3-sync-action@master
        with:
          args: --delete
        env:
          AWS_S3_BUCKET: open-up-the-cloud-website
          AWS_ACCESS_KEY_ID: ${{ secrets.AWS_ACCESS_KEY_ID }}
          AWS_SECRET_ACCESS_KEY: ${{ secrets.AWS_SECRET_ACCESS_KEY }}
          SOURCE_DIR: cloud-resume-challenge/resume-site
```

This checks out the code again, syncs the directories (including a delete if neccessary), passes in the S3 bucket name (defined in our template.yaml file), and lastly points it to our directory that contains the code for our website (the index.html).  

Now our github actions workflow looks like the following: 

test-infra ---> build-and-deploy-infra
deploy-site

This means that our build-and-deploy-infra is dependent on the test-infra build, but deploy-site will build as a stand alone. 

![builds](/images/builds.png)


## Conclusion
In conclusion, this lesson covers the implementation of Continuous Integration and Continuous Deployment (CI/CD) in our project. We explored the concepts of CI and CD and leveraged GitHub Actions (a powerful workflow automation and CI/CD platform) to automate our development tasks and integrate with various tools and platforms.

We started by setting up unit testing using GitHub Actions. We created a workflow file, main.yml, which executed tests for our infrastructure-related functions. By utilizing the actions/checkout and actions/setup-go actions, we ensured that our code was checked out and tested in an Ubuntu environment with Go version 1.15.x.

Next, we extended our workflow to include deploying our AWS SAM infrastructure using GitHub Actions. By incorporating the actions/setup-python and aws-actions/setup-sam actions, we configured the necessary dependencies and AWS credentials. This allowed us to build and deploy our AWS SAM application automatically, eliminating the need for manual deployment steps.

Finally, we added the deployment of our S3 website to the GitHub Actions workflow. We utilized the jakejarvis/s3-sync-action action to sync our website code with an S3 bucket, ensuring that our website was updated every time we pushed changes to GitHub.

By combining these CI/CD practices and leveraging the automation capabilities of GitHub Actions, we streamlined our development process, enhanced collaboration, and achieved faster and more reliable software delivery.

