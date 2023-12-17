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
