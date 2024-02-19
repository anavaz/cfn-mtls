# cfn-mtls
This Repository Contain AWS Cloud-formation template To Launch ALB with mTLS listener.

Steps to Launch Cloudformation Template :
=========================================

Step1 : Create Certificate authority using AWS CA: Launch template using aws-ca.yaml . This template creates Root CA and SubOrdinate CA and also install Root and SubCA Certificates. Validity of Root CA is set to 10 years and Validity of Subordinate CA is set to 9 Years.  You can create a RootCAbundle.pem file by simply coping the subordinate CA cert and Root CA and upload it to Amazon S3. 

Step 2 : Create Trust Store and Launch ALB and Configure Mutual TLS Authentication : Launch CFN template using  alb-mtls.yaml. This template will launch ALB with 2 listeners :
    HTTP Listener on Port 80: Redirect All Traffic to HTTPS
    HTTPS Listsner on Port 443: Forward the Traffic to Lambda Function

 This will also create Trust Store using RootCAbundle.pem file uploaded into S3. HTTPS Listener is configured to perform mTLS in verify mode and then forwards the traffic to Lambda Function.

 Lambda Function will respond back with HTTP headers that it received from the ALB which contain information related to mTLS. 


 Steps to test mTLS :
 ==================

Step1 : Create Certificate Signing Request (CSR) 
openssl req -out client-csr.pem -new -newkey rsa:2048 -nodes -keyout client-key.pem

Step2 : 

 


