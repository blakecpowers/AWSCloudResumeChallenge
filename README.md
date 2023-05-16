# AWS Services for Cloud Service Technologies

If you're interested in gaining hands-on experience with cloud service technologies, AWS (Amazon Web Services) is a great option to explore. Follow the steps below to set up an AWS account and start working with their services.

## Account Setup

To get started, you'll need to set up a secure AWS account. Follow these steps:

1. **Setup an AWS Account:** Go to the AWS website and create an account. You will need to provide personal information and payment details.
![Setup](/images/setup.png)

2. **Setup MFA for the Root Account:** Multi-factor authentication (MFA) adds an extra layer of security to your account. To enable MFA for the root account, follow these steps:
   - Go to IAM (Identity and Access Management)
   - Click on Users, and find the user for the root account
   - Navigate to Security Credentials, and assign an MFA device.

3. **Create an IAM User:** To avoid using the root account for regular usage, create an IAM (Identity and Access Management) user. Follow these steps:
   - Create the IAM user
   - Once created, you will have an access key and secret access key which can be used for programmatic access.

4. **Optional: Install AWS Vault:** AWS Vault is a tool that allows you to securely store and use your IAM user credentials. To install AWS Vault:
   - Open your terminal and run "aws vault add my-user"
   - Provide the credentials from the IAM user
   - Now you can run "aws-vault exec my-user -- aws s3 ls" (assuming the AWS CLI is installed on your machine).
   - This will give a "permissions denied" error, since we haven't given our IAM user any permissions yet.
   - We need to give Amazon S3 Full Access to our IAM user, create a simple S3 bucket, and run the command again. This time, the error will no longer be thrown.