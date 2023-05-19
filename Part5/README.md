# Route53 & DNS

In this lesson, we will be applying DNS and Route53 to use a custom domain name.

## Contents
1. [Apply DNS with Route 53 Record](#apply-dns-with-route-53-record)
2. [Create Certificate using ACM](#create-certificate-using-acm)
3. [Attaching a Cloudfront Custom Domain](#attaching-a-cloudfront-custom-domain)
4. [Conclusion](#conclusion)

## Apply DNS with Route 53 Record
To add DNS, we will add the following to our Template.yaml file:

MyRoute53Record:
  Type: "AWS :: Route53 :: RecordSetGroup"
  Properties: 
    HostedZoneId: Z06245993R6DDL18APSF6
    RecordSets:
      Name: website.cmcloudlab515.info 
      Type: A
      AliasTarget:
        HostedZoneId: Z2FDTNDATAQYW2
        DNSName: !GetAtt MyDistribution.DomainName

This is a route 53 record set with a hosted zone id, the new domain listed, and an alias through to the distribution domain name (this represents the cloudfront distribution domain name we created in Part 4). So, in short, it points `website.cmcloudlab515.info` to our cloudfront distribution domain. <br>

Let's look at what this does line by line: <br>

- `MyRoute53Record:`: This is the name or identifier assigned to the Route 53 RecordSetGroup resource. It can be any unique name you choose.

- `Type: "AWS :: Route53 :: RecordSetGroup"`: Specifies the resource type, which is an AWS Route 53 RecordSetGroup.

- `Properties:`: Denotes the properties associated with the Route 53 RecordSetGroup.

- `HostedZoneId: Z06245993R6DDL18APSF6`: Specifies the ID of the hosted zone in Route 53 where the record set will be created.

- `RecordSets:`: Defines the record sets within the record set group.

- `Name: website.cmcloudlab515.info`: Specifies the name (fully qualified domain name) for the record set. In this case, it is set to `website.cmcloudlab515.info`.

- `Type: A`: Specifies the type of the record set as "A", which is used for mapping a domain name to an IPv4 address.

- `AliasTarget:`: Specifies that the record set is an alias and points to another AWS resource.

- `HostedZoneId: Z2FDTNDATAQYW2`: Specifies the hosted zone ID of the resource to which the alias points. In this case, it is set to `Z2FDTNDATAQYW2`, which corresponds to the CloudFront distribution.

- `DNSName: !GetAtt MyDistribution.DomainName`: Retrieves the DNS name of the CloudFront distribution, using the `!GetAtt` function to reference the `DomainName` attribute of the `MyDistribution` resource. This ensures that the record set points to the CloudFront distribution's DNS name dynamically.
<br>
This code sets up a Route 53 RecordSetGroup to create a record set that maps the domain name `website.cmcloudlab515.info` to the CloudFront distribution using an alias. It enables seamless routing and resolves the domain name to the CloudFront distribution's DNS name, allowing the website to be accessed through the specified domain name. <br>

Now after seeing what it does, lets actually deploy it. <br>

```
make deploy-infra
```

![r53p5](/images/r53P5.png)

Now, when we make a request to `website.cmcloudlab515.info`, this should re-direct to the `d34yeshyar60z.Cloudfront.net`

At this moment, we would get a 403 error. To resolve this, we'll create our ACM (HTTP certificate) and apply it to our Cloudfront Distribution, and allow our custom domain access to the Cloudfront Distribution.

## Create Certificate using ACM
ACM is the amazon certificate manager and we can use it to create our custom certificate for the `website.cmcloudlab515.info` domain. Once created, the certificate will look like this:

![ACMCertP5](/images/ACMCertP5.png)

![createrecordR53P5](/images/createrecordR53P5.png)


Once we click the `Create record in Route 53` button, it will take the CNAME record and point it towards that `value`. <br>
The way that AWS Validates that you "own" this domain, it asks you to create this "Name" (random characters ending in the domain that own), and we will return a value (another seemingly random character string). <br>
So basically AWS hits this URL and if it gets this response, it's validated. <br>

If we go back into Route53, we can see that it creates us an entry. The same CNAME entry we saw in the ACM certificate.

![r53entry](/images/r53entry.png)

We can run a dig command, which is commonly used to query DNS servers and retrieve information about DNS records, such as A records, CNAME records, and others. It can be used to verify the DNS configuration of a domain and troubleshoot DNS-related issues.

![digCommandR53](/images/digCommandR53.png)

Now that this certificate has been created we need to attatch it to our Cloudfront Distribution. 

```
MyCertificate:
  Type: AWS::CertificateManager::Certificate
  Properties:
    DomainName: website.cmcloudlab515.info
    validationMethod: DNS
```

This represents an AWS CloudFormation resource of type `AWS::CertificateManager::Certificate`. Remember, this resource is used to create an SSL/TLS certificate in AWS Certificate Manager (ACM).

Let's break down the code line by line:

- `MyCertificate`: This is the logical name or identifier assigned to the ACM certificate resource. It can be any unique name you choose.

- `Type: AWS::CertificateManager::Certificate`: Specifies the resource type, which is an AWS Certificate Manager certificate.

- `Properties:` Denotes the properties associated with the ACM certificate.

- `DomainName: website.cmcloudlab515.info`: Specifies the domain name for which the certificate is being requested.

- `ValidationMethod: DNS`: Specifies the validation method for the certificate. In this case, it is set to `DNS` validation, which means that the certificate will be validated by creating DNS records in the domain's DNS configuration.

Note: When you create this CloudFormation resource, AWS Certificate Manager will initiate the certificate issuance process and provide you with instructions on how to complete the DNS validation. This usually involves creating specific DNS records with your DNS provider, as instructed by AWS Certificate Manager. Once the DNS records are created and validated, the certificate will be issued and available for use in other AWS services, such as Amazon CloudFront, Elastic Load Balancing, or Amazon API Gateway, to enable secure communication over HTTPS for your domain `website.cmcloudlab515.info`.

We also need to add the following to the Cloudfront Distribution so that the certificate is associated with our cloudfront distribution.

```
# MyDistribution:
  # Properties:
    ViewerCertificate: 
      AcmCertificateArn: !Ref MyCertificate
      SslSupportedMethod: sni-only
```

Note: This is a change made to another entry that already existed in our template.yaml file from previous lessons.

This is attaching a viewer certificate which is associated with the certificate we made.

Next we will add an alias for our cloudformation.

## Attaching a Cloudfront Custom Domain
As of right now, we set up ACM and attached it to our Cloudfront Distribution, but we get a 403 error at the moment because we haven't allowed our custom domain access to the Cloudfront Distribution. 

Ultimately, this is what we're after. Addint the Alternate Domain Names (CNAMEs) to the Cloudfront Distribution. We can also see the SSL which is what we created and attached above.

![cfsidresult](/images/cfsidresult.png)

In order to achieve this, we need to add the following to the template.yaml file:

```
Aliases:
  - website.cmcloudlab1035.info
```

This adds an aliases property to the custom domain name. This Alias will be placed in the MyDistribution/Properties section. Example:

```
# MyDistribution:
  # Properties:
    ViewerCertificate: 
      AcmCertificateArn: !Ref MyCertificate
      SslSupportedMethod: sni-only
    Aliases:
    - website.cmcloudlab1035.info
```

We then deploy just like normal using our makefile

```
make deploy-infra
```

And we can now navigate to that alias `website.cmcloudlab1035.info` , see that the connection is secure, and the corresponding certificate information. This is secure and serving content from our Cloudfront Distribution. 


## Conclusion
In this section, DNS and Route53 are implemented to enable the use of a custom domain name. The Template.yaml file is updated to include a Route 53 record set that maps the domain name `website.cmcloudlab515.info` to a CloudFront distribution using an alias. This ensures seamless routing and allows the website to be accessed through the specified domain name.

By integrating DNS and Route53, the custom domain name `website.cmcloudlab515.info` is properly connected to the CloudFront distribution, enabling users to access the website using the desired domain. This improves the user experience and provides a professional and branded online presence.

