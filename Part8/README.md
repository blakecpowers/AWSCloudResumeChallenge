# Continuous Integration / Continuous Deployment
In this lesson, we will be implementing CI / CD in our project. We'll take a look at what CI / CD means, HERe--------------


## Contents
1. [Continuous Integration Continuous Deployment](#continuous-integration-continuous-deployment)
2. [Unit Testing in Github Actions](#unit-testing-in-github-actions)

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
  build-infra:
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

This "main" file is triggered by a push event. So when you push changes to the github repo. The workflow includes one job named "build-infra" that runs on an Ubuntu environment and has a timeout of 2 minutes.

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





## Conclusion

