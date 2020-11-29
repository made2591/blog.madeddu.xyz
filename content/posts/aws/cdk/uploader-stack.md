---
layout: post
title: "Build an upload form with 45 lines of Typescript"
date: 2019-05-29
polly: https://madeddu.xyz/mp3/aws/cdk/uploader-stack.mp3
categories:
- coding
- aws
- serverless
tags:
- coding
- aws
- cdk
- guide
- serverless
---

### Introduction
The [AWS CDK](https://docs.aws.amazon.com/cdk/latest/guide/what-is.html) is becoming day by day pretty easy to use. I use Typescript, and today I will talk about a common use case: a simple Upload Endpoint for your API Gateway than like a LEGO can be built with a few instructions and of course...without the need of any server. For the most curious, [here](https://github.com/made2591/immutable.templates/blob/master/templates/upload-form/lib/upload-form-stack.ts) you can find the core code.

<div class="img_container"><img src="https://i.imgur.com/YltF5n6.jpg" style="width: 100%; marker-top: -10px;"/></div>

### Scenario
You want to provide an endpoint to upload object: where? S3, of course. How? With a pre-signed a URL! What is it? A pre-signed URL it's a URL that gives someone access to the object identified in the URL, provided that the creator of the pre-signed URL has the permissions to access that object. That is, if you receive a pre-signed URL to upload an object, you can upload the object only if the creator of the pre-signed URL has the necessary permissions to upload that object.

To create a pre-signed URL, you must provide security credentials - or something that acts with a Role with permissions over the bucket - and then specify a bucket name, an object key, an HTTP method (PUT for uploading objects), and expiration date and time. The pre-signed URLs are valid only for the specified duration.

The nice thing is that anyone who receives a valid pre-signed URL can then programmatically upload an object.

### Design
Let's have a look at the schema:

<div class="img_container"><img src="https://i.imgur.com/veA8UUs.png" style="width: 65%; marker-top: -10px;"/></div>

The user asks to API Gateway (1) for a pre-signed URL to upload a file. It doesn't need to know where it will be stored, neither the name of the bucket or having any credentials: this covers our scenario in which a generic user just want to upload a file into our platform, and only owns the file - in this case, it will also provide the name of it, but could even be ignored, depending on the logic of your application. After that, API Gateway will trigger a Lambda function (2) - i.e., the designed entity that runs with a role with attached the permissions to do a PutObject over the bucket designed to store the content of the user. The Lambda invokes the ```getSignedUrl``` URL action by using the s3 API (3) and provides back the URL to API Gateway (4) - that will forwards it directly to the user (6). The user is now able to push his file to s3 with the provided URL (7).

#### Deep dive
Ok now let's move to the funny part: the code!

##### The Bucket
First of all, we need to define the storage on s3. Pretty simple as usual:

{{< highlight js >}}
// create content s3 bucket
const contentBucket = new s3.Bucket(this, props.stage.toString() + "-content", {
    encryption: BucketEncryption.S3Managed,
    publicReadAccess: false
})
this.contentBucketRef = contentBucket.export();
{{< / highlight >}}

##### The Lambda
The Lambda will need the right permissions to detach respective pre-signed URL for the user, thus:

{{< highlight js >}}
// create lambda policy statement for operating over s3
var lambdaS3PolicyStatement = new iam.PolicyStatement(PolicyStatementEffect.Allow)
lambdaS3PolicyStatement.addActions(
    's3:PutObject',
    's3:GetObject'
)
lambdaS3PolicyStatement.addResources(
    contentBucket.bucketArn + "/*"
);
{{< / highlight >}}

The `GetObject` is required to extend the use case also to getting uploaded object - could be for registered user, or premium users, once again: it depends on your use case. I will write about a scenario with the double permission soon.

### The architecture
Before deep dive into the code, excluding the s3/Cloudfront arrival (1) I already discussed [here](http://madeddu.xyz/posts/cloudformation-to-cdk/) - if you visit it, please go to directly to the repo, the post is already outdated 😂 - and the Click Submit POST request to the API Endpoint (2) - let's discuss the steps from 3 to 5.

#### API Lambda Proxy Integration
Setting up an API endpoint for this use case is pretty simple: this time, instead of going for API Gateway Service Integration as I did in my [previous post](https://madeddu.xyz/posts/contact-form/), we are going to use Lambda Integration with Proxy mode. What does it mean? Let's do one step back.

The API Gateway Lambda integration provides a complex, but also more controlled way - at least in the transmission of data - way of integrating an HTTP endpoint. As I did with the DynamoDB integration, the request can be modified before it is sent to Lambda and the response can be modified after it is sent from lambda. This can be done by using the [mapping templates](https://docs.aws.amazon.com/apigateway/latest/developerguide/models-mappings.html) that follow the Apache Velocity Template specification which transforms the payload, as per the user customizations. As we discovered the last time, the API Gateway uses Velocity Template Language (VTL) engine to process body mapping templates for the integration request and integration response.

The API Gateway Lambda-Proxy integration instead provides a more simple but powerful integration, perfect for our use case: with the proxy mode enable, all the requests to the APIGateway URL are forwarded straight to the Lambda and the response is sent from Lambda. This means, no modifications to the request(query params, body, variables) and response(status code, message) are done by the APIGateway. The code below easily integrates the Lambda by following this schema:

{{< highlight js >}}
// defines an API Gateway REST API resource backed by our s3 uploader function.
this.uploadApiAuthorizer = new apigateway.LambdaRestApi(this, props.stage.toString() + "-apigtw", {
    handler: this.s3AuthLambda
});
{{< / highlight >}}

#### The Lambda
The Lambda function is pretty simple: it just expects a JSON in the following form.

{{< highlight json >}}
{
    "object_key" : "[the_name_of_the_file]",
    "action" : "putObject",
}
{{< / highlight >}}

You were wondering why it required to specify the action wanted: this is something you will discover soon, but not in this post 😘 (well, not properly). By the way, the code of the Lambda can be found [here](https://github.com/made2591/immutable.templates/blob/master/templates/upload-form/lib/s3-authorizer/index.js) and it's super simple to understand.

#### Pack and deploy everything
To test the stack, simply clone the code, move to the upload-form stack and run the install of the packages, compile the Typescript and start the deploy.

{{< highlight sh >}}
git clone https://github.com/made2591/immutable.templates.git
cd immutable.templates/templates/upload-stack/
npm i
npm run build
cdk deploy # or cdk synth if you want to have a look at the Cloudformation generated by the process
{{< / highlight >}}

#### Test the endpoint
To test the endpoint, you can easily try upload a file to the bucket (in the sample, I upload the README.md generated by the `cdk init` command in the upload-stack form folder):

{{< highlight sh >}}
PRESIGNE_URL=$(curl -X POST -d "{\"object_key\": \"README.md\", \"action\": \"putObject\"}" <YOUR_API_GATEWAY_ENDPOINT>)
curl -X PUT -v -H "Content-Type: multipart/form-data" --upload-file ./README.md "$PRESIGNED_URL"
# this is to test the availability of the object
curl -X POST -d "{\"object_key\": \"README.md\", \"action\": \"getObject\"}" <YOUR_API_GATEWAY_ENDPOINT>)
{{< / highlight >}}

where <YOUR_API_GATEWAY_ENDPOINT> is provided as output after the deploy of the stack - if everything was fine, of course (and if it's not please let me know).

### Conclusion
As you discovered, with just a few lines of code we can deploy the infrastructure we need to let unknown users upload safely content to an s3 bucket, in a completely serverless manner that doesn't require you to provision anything: it simply works and scales automatically. You can, of course, extend the stack by authenticating the endpoint: if you do it please don't hesitate in creating a pull request with a new folder or made this parametric in the same stack!

Thank you for reading!
