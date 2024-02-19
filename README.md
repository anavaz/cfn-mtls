# cfn-mtls
This Repository Contain AWS Cloud-formation template To Launch ALB with mTLS listener.

Steps to Launch Cloudformation Template :
=========================================

Step1 : Create Certificate authority using AWS CA: 

[![Launch AWS CloudFormation stack](https://s3.amazonaws.com/cloudformation-examples/cloudformation-launch-stack.png)](https://console.aws.amazon.com/cloudformation/home?region=us-east-1#/stacks/new?stackName=MyStack&templateURL=https://console.aws.amazon.com/cloudformation/home?region=us-east-1#/stacks/new?stackName=Issue-Certficate-using-private-ca&templateURL=https://raw.githubusercontent.com/anavaz/cfn-mtls/main/issue-certificate-using-aws-private-ca.yaml
) . 

This template creates Root CA and SubOrdinate CA and also install Root and SubCA Certificates. Validity of Root CA is set to 10 years and Validity of Subordinate CA is set to 9 Years.  You can create a RootCAbundle.pem file by simply coping the subordinate CA cert and Root CA and upload it to Amazon S3. 



Step 2 : Create Trust Store and Launch ALB and Configure Mutual TLS Authentication : 

Launch CFN [template](https://github.com/anavaz/cfn-mtls/blob/main/issue-certificate-using-aws-private-ca.yaml). This template will launch ALB with 2 listeners :
    HTTP Listener on Port 80: Redirect All Traffic to HTTPS
    HTTPS Listener on Port 443: Forward the Traffic to Lambda Function

 This will also create Trust Store using RootCAbundle.pem file uploaded into S3. HTTPS Listener is configured to perform mTLS in verify mode and then forwards the traffic to Lambda Function.

 Lambda Function will respond back with HTTP headers that it received from the ALB which contain information related to mTLS. 


 Steps to test mTLS :
 ==================

Step1 : Create Certificate Signing Request (CSR)** 
```
openssl req -out client-csr.pem -new -newkey rsa:2048 -nodes -keyout client-key.pem
```
Step2 : Issue a Certificate using the Private CA that you have created. This should return the ARN of Issued certificate .
```
aws acm-pca issue-certificate \
      --certificate-authority-arn <arn-of-subordinate-ca \
      --csr fileb://client_csr.pem \
      --signing-algorithm "SHA256WITHRSA" \
      --validity Value=30,Type="DAYS" 
```
Step3: You can Get the issued the certificate and Download it to your local machine.
```
aws acm-pca get-certificate \
      --certificate-authority-arn <arn-of-subordinate-ca \
      --certificate-arn <arn-of-isssued-cert | \
      jq -r .'Certificate' > client_cert.cert
 ```

Step 4: Test the mTLS using cURL.
```
curl -ivk --cert client_cert.cert --key client-key.pem https://<elb-name>
```
