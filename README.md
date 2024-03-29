The goal behind this project was to work with Amazon Cognito and see how it can intergrate with other IdP's, in this case the Google API. I wanted to see how Cognitio could be used as a scalable way to provide authentication and how permissions can scale securly in an enviroment with such high variablity.




# **Stage 1**

- In the beginning stage of this project little is required, as resources are provisioned via a CloudFormantion (CFN) template. All that's left is to make sure that the resources were created and configured properly. 
- Rather than just applying the template I took my time in reading through it and seeing exactly what was happening inside the template. 
- Understanding YAML is relatively straightforward allowing me to read through and visualize these settings that are bring determined by code. 
- It was also insightful to see how instructions were to be passed into CFN, I got to see the logic of the service and how custom function can be integrated to make the service more robust.

![Stage1_Architecture](https://github.com/dariusgatsby/aws-web-fed-project/blob/main/diagrams/Stage_1_Webapp.png)

# **Stage 2**

- Next in this serverless IdF project was to enable the Google Cloud API 
- Create an OAuth2.0 Client for authentication into the web app
- Configure the JS origin as the CloudFront distribution.
- This stage of the process is what would redirect users to the Google API and allow them to authenticate with their Google credentials, and if valid, an authentication token is provided by Google to be swapped with AWS credentials to access AWS resources.

![Stage2_Architecture](https://github.com/dariusgatsby/aws-web-fed-project/blob/main/diagrams/Stage_2_Webapp.png)


# **Stage 3**

- After creating the IdP via Google Cloud, Amazon Cognito federated identity pools must be configured to accept identites from the Google API. I did this via the AWS console and used [AWS Docs](https://docs.aws.amazon.com/cognito/latest/developerguide/identity-pools.html?icmpid=docs_cognito_console_help_panel) for clarification and guidance on settings.
- I configured Google Cloud for authenticated and guest access, which prompted me to created 2 new IAM roles for Cognito.
- I named them```PetIDFAuthenticatedRole``` and ```PetIDFGuestNonAuthenticatedRole```, these roles have a assumed trust relationship with the Cognito identity pool and have the attached policy ```cognito-identity:GetCredentialsForIdentity``` which allows for the role to be assumed by Cognito and generate AWS credentials.
- To finalize this stage of the serverless infrastructure I gave the Identity Pool a name, and left the remaining configurations blank as they provided the best performance/security.

![Stage3_Architecture](https://github.com/dariusgatsby/aws-web-fed-project/blob/main/diagrams/Stage_3_Webapp.png)

# **Stage 4**

- In this final building stage of the project I was tasked with configuring the provided HTML and JS documents with the apporiate values to allow the front end web app to communicate with the Google IdP and Amazon Cognito, and subsequently provide access via presigned URL to the objects in the bucket.
- The Client ID for the Google API embedded into the HTML document. This is done so that the web app can redirect the user to the API that has been authorized to authenticate and use the identity pool.
- The JavaScript document is used to by the HTML document to swap out the Google token for AWS credentials generated by Cognito (Cognito does this by assuming the IAM role created during stage 3. It then uses those credentials to generate a presigned URL to provide authenicated access to the objects inside the bucket.

![Stage4_Architecture](https://github.com/dariusgatsby/aws-web-fed-project/blob/main/diagrams/Stage_4_Webapp.png)

# aws-web-fed-project
This project is a web federation application using serverless AWS services like S3 for static hosting, CloudFront for content delivery, and AWS Cognito with Google as the IdP for user authentication. 
I'm doing this project to gain hands-on experience with AWS web federation and serverless architecture. This project was done with the text guidance of Adrian Cantrill and AWS documentation. (Sorry for the ugly diagrams).
