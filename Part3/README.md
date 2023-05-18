# Deploying Portfolio Site with SAM

In this lesson, we will be deploying our portfolio (HTML & CSS) with AWS SAM

## Contents
1. [HTML CSS](#html-css)
2. [Deploy with SAM](#deploy-with-sam)
3. [Deploy Infrastructure](#deploy-infrastructure)
4. [Bucket Policy](#bucket-policy)
5. [CSS](#css)
6. [Conclusion](#conclusion)

## HTML CSS
The first step is to create some HTML/CSS that represents your portfolio websites. There are many examples and if you wish you can use my portfolio as a template [here](https://github.com/blakecpowers/blakecpowers.github.io).

## Deploy with SAM
We'll add to the template file we created in Part2 to allow us to read the index.html file in our bucket name.

![template3deploy](/images/templatePart3Deploy.png)

Then we want to deploy this change. We now have a public read on the access control and the website configuration with the index.html file.

```
sam build && aws-vault exec my-user --no-session
```

This essentially updates our S3 bucket to operate basically like a public website. We can navigate in AWS to see that our site is public 

![publicsite](/images/publicwebsite.png)

However, if we navigate using that link, we will get a 404 error because there aren't actually any files in this S3 bucket. Let's add our index.html using infrastructure as code.

First, let's make a new folder called "resume-site" and put our index.html in there.


## Deploy Infrastructure
Previously, when we deployed, we had a single build command in the Makefile. <br>
Now, we have two steps. We have a deploy-infra and deploy-site.

![deployInfra](/images/deployInfra.png)

The deploy-site command has some of the same stuff as before with the sam build, but has the new part of the command where we sync our local resume site directory with our previously created bucket. Now we can just run the following:

```
make deploy-site
```

This will take the index.html file we created above and upload it to the S3 bucket. <br> 
Now, if we navigate to our site again, we get a 403 (not a 404 anymore). This means the file exists but we just can't access it.

Let's add a bucket policy.


## Bucket Policy
To get access to the s3 bucket, we add a bucket policy to our template.yaml to set permissions for our bucket.

![bucketPolicy](/images/bucketPolicy2.png)

This allows public read on all of the objects on our S3. <br>

We now run the following to re-run our sam build using the makefile script:

```
make deploy-infra
```

![deployInfra2](/images/deployInfra2.png)

![deployedbucketpolicy](/images/deployedbucketpolicy.png)

Now if we navigate to the S3 bucket, we see a bucket policy was added. 

![awsbucketpolicy](/images/awsbucketpolicy.png)

Now, we can navigate to the website (using the AWS link) and see the index.html was deployed. 


## CSS
The index.html was deployed, but what about our CSS? 

Well, one way to do this is to directly add css into our html file by adding styles to the header. We would modify our index.html and add:

```
<style>
    CSS Here
</style>
```

We then need to deploy the site again since we made changes:

```
make deploy-site
```

Our site is now deployed with the CSS affects.


## Conclusion
In this lesson, we learned how to deploy an S3 bucket using AWS SAM, CloudFormation, and S3. We created HTML and CSS files for our portfolio website and used SAM build to update the S3 bucket. We added our index.html file and made it accessible by synchronizing our local directory with the bucket. To provide access, we added a bucket policy allowing public read permissions. Finally, we addressed CSS by adding styles to the HTML file and redeploying the site. This deployment process offers a scalable and efficient way to host and manage static websites on AWS.

