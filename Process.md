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
