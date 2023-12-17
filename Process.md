# **Stage 1**

In the first step of this project all that was required was essentially load up a CFN template and make sure that the resources were created and accessed properly. Rather than just applying the template I took my time in reading through it and seeing exactly what was happening inside the template. Understanding YAML is relatively straightforward allowing me to read through and visualize these settings that are bring determined by code. It was also insightful to see how instructions were to be passed into CFN, I got to see the logic of the service and how custom function can be integrated to make the service more robust.

# **Stage 2**

Next in this serverless IdF project was to enable the Google API, create an OAuth2.0 Client for authentication into the web app, and configure the JS origin as the CloudFront distribution. This stage of the process is what would redirect users to the Google API and allow them to authenticate with their Google credentials, and if valid, an authentication token is provided by Google to be swapped with AWS credentials to access AWS resources. 
