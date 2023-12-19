This project begins with a CloudFormation (CFN) template provided by Adrian Cantrill written in YAML. So instead of pretending I provisioned all the infrastructure myself
I'll go through the YAML file and briefly touch on all of the major details of the services being provisioned, and my take on the importance of certain aspects 

The block of code below defines the CloudFront distribution with an explicit dependency on 'appbucket'
this is important because CFN uses internal logic to determine which resources to create and when,
using an explicit dependency avoids errors in deployment. The Distribution is configured to be enabled
once provisioned and use 'appbucket' as the target origin and 'index.html' as the root object (the object
pull from the origin. 'https-only' set as the viewer protocol for data to be encrypted in transit.
The CFN template uses !GetAtt to use a value that will be produced by S3, and the OAI is set so that CloudFront
can't be bypassed directly for the bucket:

```
Resources:
  WebAppCDN:
    DependsOn: appbucket
    Type: AWS::CloudFront::Distribution
    Properties:
      DistributionConfig:
        Enabled: 'true'
        DefaultCacheBehavior:
          ForwardedValues:
            QueryString: 'true'
          TargetOriginId: appbucket
          ViewerProtocolPolicy: https-only
        DefaultRootObject: index.html
        Origins:
          - DomainName: !GetAtt appbucket.DomainName
            Id: "appbucket"
            S3OriginConfig:
              OriginAccessIdentity: ""
```
The block of code below provisions the S3 bucket which hosts the static site, as well as the bucket policy
that will be attached. The intention of the bucket is to allow public access to the objects inside as well
as point the website at the index.html document which hosts the front-end web server. There is code for a private
bucket in which the settings are not open to the public and is meant for private patches. I cut it out
to keep this part brief.

```
  appbucket:
    Type: AWS::S3::Bucket
    Properties:
      PublicAccessBlockConfiguration:
        BlockPublicAcls: false
        BlockPublicPolicy: false
        IgnorePublicAcls: false
        RestrictPublicBuckets: false
      AccessControl: PublicRead
      WebsiteConfiguration:
        IndexDocument: index.html
        ErrorDocument: error.html
  BucketPolicy:
    Type: AWS::S3::BucketPolicy
    Properties:
      PolicyDocument:
        Id: MyPolicy
        Version: 2012-10-17
        Statement:
          - Sid: PublicReadForGetBucketObjects
            Effect: Allow
            Principal: '*'
            Action: 's3:GetObject'
            Resource: !Join
              - ''
              - - 'arn:aws:s3:::'
                - !Ref appbucket
                - /*
      Bucket: !Ref appbucket

```
     
  The rest of the YAML document creates Custom functions within the stack to allow CloudFront to communicate with services it normally cannot such as Lambda, in this case to interact with buckets in Adrian's account. This leads into the next important area of the stack after the Lambda functions themselves and that's roles. Roles in AWS allow for authentication and authorization of resources within and between AWS accounts. In order for proper automation to take place between serverless apps and other AWS functions must have the proper role/permissions to perform certain tasks. The template uses created roles to grant necessary permissions for cross-account bucket access as well as IAM managed policies to apply common permissions to resources.
  
# **Stage 1**

- In the beginning stage of this project little is required, as resources are provisioned via a CloudFormantion (CFN) template. All that's left is to make sure that the resources were created and configured properly. 
- Rather than just applying the template I took my time in reading through it and seeing exactly what was happening inside the template. 
- Understanding YAML is relatively straightforward allowing me to read through and visualize these settings that are bring determined by code. 
- It was also insightful to see how instructions were to be passed into CFN, I got to see the logic of the service and how custom function can be integrated to make the service more robust.

# **Stage 2**

- Next in this serverless IdF project was to enable the Google Cloud API 
- Create an OAuth2.0 Client for authentication into the web app
- Configure the JS origin as the CloudFront distribution.
- This stage of the process is what would redirect users to the Google API and allow them to authenticate with their Google credentials, and if valid, an authentication token is provided by Google to be swapped with AWS credentials to access AWS resources. 

# **Stage 3**

- After creating the IdP via Google Cloud, Amazon Cognito federated identity pools must be configured to accept identites from the Google API. I did this via the AWS console and used [AWS Docs](https://docs.aws.amazon.com/cognito/latest/developerguide/identity-pools.html?icmpid=docs_cognito_console_help_panel) for clarification and guidance on settings.
- I configured Google Cloud for authenticated and guest access, which prompted me to created 2 new IAM roles for Cognito.
- I named them```PetIDFAuthenticatedRole``` and ```PetIDFGuestNonAuthenticatedRole```, these roles have a assumed trust relationship with the Cognito identity pool and have the attached policy ```cognito-identity:GetCredentialsForIdentity``` which allows for the role to be assumed by Cognito and generate AWS credentials.
- To finalize this stage of the serverless infrastructure I gave the Identity Pool a name, and left the remaining configurations blank as they provided the best performance/security.

# **Stage 4**

- In this final building stage of the project I was tasked with configuring the provided HTML and JS documents with the apporiate values to allow the front end web app to communicate with the Google IdP and Amazon Cognito, and subsequently provide access via presigned URL to the objects in the bucket.
- The Client ID for the Google API embedded into the HTML document. This is done so that the web app can redirect the user to the API that has been authorized to authenticate and use the identity pool.
- The JavaScript document is used to by the HTML document to swap out the Google token for AWS credentials generated by Cognito (Cognito does this by assuming the IAM role created during stage 3. It then uses those credentials to generate a presigned URL to provide authenicated access to the objects inside the bucket. 
