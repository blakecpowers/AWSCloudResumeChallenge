# AWS CloudFront Distribution

In this lesson, we will be setting up Cloudfront Distribution, providing us with a custom domain name.

## Contents
1. [Cloudfront Distribution](#cloudfront-distribution)
2. [Cloudfront Invalidations](#cloudfront-invalidations)
3. [Conclusion](#conclusion)

## Cloudfront Distribution
First, we need to add our Cloudfront Distribution to the template.yaml file. Remember, Cloudfront is a CDN, so we have different servers all over the world which caches our content so it can be distributed as quick as possible to users. In AWS, it also allows us to attach HTTPS and SSL certificate to our S3 bucket so we have a secure site. <br>

```
MyDistribution:
  Type: "AWS :: CloudFront ::Distribution"
  Properties:
    DistributionConfig:
      DefaultCacheBehavior:
        ViewerProtocolPolicy: allow-all
        TargetOriginId:my-fantastic-website.s3-website-us-east-1.amazonaws.com
        DefaultTTL: 0
        MinTTL: 0
        MaxTTL: 0
        ForwardedValues:
          QueryString: false
        Origins:
          - DomainName: my-fantastic-website.s3-website-us-east-1.amazonaws.com
            Id: my-fantastic-website.s3-website-us-east-1.amazonaws.com
            CustomOriginConfig:
              OriginProtocolPolicy: match-viewer
        Enabled: "true"
        DefaultRootObject: index.html
```

There's 2 main things to see with this distribution. <br>
- Distribution config
    DefaultCacheBehavior: Defines the default caching behavior for the distribution. <br>
    ViewerProtocolPolicy: allow-all - Sets the viewer protocol policy to "allow-all," which means both HTTP and HTTPS protocols are allowed.<br>
    TargetOriginId: my-fantastic-website.s3-website-us-east-1.amazonaws.com: Identifies the target origin for the default cache behavior. In this case, it points to the S3 bucket hosting the website. <br>
    TTL - We have the TTL values of our CDN Set to 0. This essentially invalidates our caches. So everytime we hit cloudfront, cloudfront will hit S3.  <br> It will not store any cached values. In a professional setting, we probably wouldn't do this. <br>
    QueryString: false: Indicates that CloudFront does not include query strings when caching or forwarding requests. <br>
- Origins
    DomainName: my-fantastic-website.s3-website-us-east-1.amazonaws.com: Specifies the domain name of the S3 bucket as the origin. <br>
    Id: my-fantastic-website.s3-website-us-east-1.amazonaws.com: Provides an ID to identify the specific origin. <br>
    CustomOriginConfig:: Configures the custom origin settings. <br>
    OriginProtocolPolicy: match-viewer: Sets the origin protocol policy to "match-viewer," which aligns the protocol used by the viewer (client) with the one used to fetch content from the origin. <br>
    Enabled: "true": Enables the CloudFront Distribution. <br>
    DefaultRootObject: index.html: Sets the default root object to index.html, which is the main entry point of the website. <br>

We can now deploy using the config we setup in the makefile in our previous Part3 lesson.

```
make deploy-infra
```

After navigating to AWS Cloudfront Distributions, we can see the CFF Distribution is deployed.

![CFDistributionP4.png](/images/CFDistributionP4.png)

By default, AWS gives us a domain name, which  we could navigate to.

![DomainNameP4](/images/DomainNameP4.png)

## Cloudfront Invalidations

If we wanted to make a change on our deployed site, we could make the change, deploy using `make deploy-site`, and this change would be reflected on both the original S3 resource and the new domain url just now provided. <br>
This is becase our caches are invalidated in the template.yaml file above. If we didn't add that, we actually wouldn't see the change in the new domain url site. 

If we wanted a different approach instead of invalidating our caches each time, we would need to Create an Invalidation on the Cloudfront Distribution here:

![InvalidCFDistributionP4](/images/InvalidCFDistributionP4.png)


## Conclusion
In this lesson, we learned how to set up an AWS CloudFront Distribution to assign a custom domain name to our website. CloudFront acts as a CDN, caching and distributing content globally for faster delivery. We configured the CloudFront Distribution in our `template.yaml` file, enabling HTTPS and SSL certificate attachment to our S3 bucket. After deploying the distribution, we could access it through the provided domain name. To update the site, we could redeploy changes using `make deploy-site`. Alternatively, we could manage cache invalidations or create specific file or directory refreshes through CloudFront's invalidation feature. Overall, CloudFront provides improved performance and flexibility for our website. <br>
In the next video, we'll assign a custom domain name of our choosing using Route53.

