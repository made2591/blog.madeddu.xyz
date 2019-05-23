---
layout: post
title: "How to deploy a serverless contact form with API Gateway, DynamoDB and SNS"
categories: [coding, aws]
tags: [coding, aws, cdk, guide, impression]
---

### Introduction
Hi everybody, thanks for the claps, it was a great month - rain rain rain again - now I'm back. The only GOOD THING of this terrible May is that [AWS CDK](https://docs.aws.amazon.com/cdk/latest/guide/what-is.html) came to simplify our life and I started using it (just a little) bit - still, enough to say, sincerely: it's awesome. I used the Typescript version, everything is broken 2 release out of 3 but the time you save exploring the interfaces instead of looking for Cloudformation documentation online worths the time spending in troubleshooting the ongoing changes.
Today I'm here to write about a common use case, a simple stack, and that's all I have to say.

<div class="img_container"><img src="https://i.imgur.com/Cb1D4nt.jpg" style="width: 100%; marker-top: -10px;"/></div>

### Scenario
Quick and dirty: you all bloggers, startuppers, freelancers, enterprises (mmm?) but mostly **mums** have a simple static blog - it's fancy and s3 is cheaper than a Wordpress with a digital ocean of machines to maintain, no? - indeed, you would like to have a contact form on it. I mean, a real, simple, cool, cheap, no-frontend-for-free-in-this-blog-post-I'm-sorry contact form, am I wrong? Then you are in the right place! Because thanks to [REST API Integrations](https://docs.aws.amazon.com/apigateway/latest/developerguide/how-to-integration-settings.html) of API Gateway you can do it pretty easily.

So the mamas from the other blog you hate a lot will reach your bucket, and will just send a "holy shit! I need a contact form like this!" message in your text-area then Click, POST, and the chain begins.

Before going ahead, if you want to go straight to the code, [here you are](https://github.com/made2591/immutable.templates/blob/master/templates/contact-form/lib/contact-form-stack.ts) the main Typescript.

<div class="img_container"><img src="https://i.imgur.com/z20B4rG.png" style="width: 100%; marker-top: -10px;"/></div>

### The architecture
Before deep dive into the code, excluding the s3/Cloudfront arrival (1) I already discussed [here](http://madeddu.xyz/posts/cloudformation-to-cdk/) - if you visit it, please go to directly to the repo, the post is already outdated 😂 - and the Click Submit POST request to the API Endpoint (2) - let's discuss the steps from 3 to 5. 

#### API Service Integration
Setting up an API method is a simple as writing something like this:

{% highlight js %}
// define the api gateway and map the integration
this.api = new apigateway.RestApi(this, props.stage.toString() + '-contacts-api');
this.api.root.addMethod('ANY');
var contacts = this.api.root.addResource('contacts');
contacts.addMethod('POST', dynamoIntegration, {
    methodResponses: methodResponses
});
{% endhighlight %}

You defined the rest API, you define a resource and then you can integrate it with an endpoint in the backend. A backend endpoint is called by AWS the *integration endpoint* and can be a Lambda function, an HTTP webpage, or... AWS service action, that is the equivalent to say "Ehy AWS: please take care of my APIs."

<div class="img_container"><img src="https://i.imgur.com/OsZZs0h.gif" style="width: 100%; marker-top: -10px;"/></div>

The API integration consists of an integration request and an integration response. An integration request encapsulates an HTTP request to be received by the backend. An integration response is an HTTP response encapsulating the output returned by the backend.

#### Setting up things
Setting up an integration request first involves configuring how to pass client-submitted method requests to the backend: in our case, the request as is passed from the POST method is fine for us. It will be something like this

{% highlight json %}
{
    "name" : "Who is contacting", 
    "email" : "whois.contacting@gmail.com", 
    "content" : "Lorem ipsum"
}
{% endhighlight %}

Fair enough.

The second step consists in configuring how to transform the request data, if necessary, to the integration request data: since we are going to prepare a request to the DynamoDB service, this request has to be compliant to the backend specification - in this case DynamoDB. The following piece of code both specify the AWS service and respective action to invoke; furthermore, it includes an [Apache Velocity Template](https://docs.aws.amazon.com/apigateway/latest/developerguide/models-mappings.html) specification to transform the request initially received by the client into a DynamoDB request.

{% highlight js %}
// create the proxy service integration to dynamodb
var dynamoIntegration = new apigateway.AwsIntegration({
    proxy: false,
    service: "dynamodb",
    integrationHttpMethod: "POST",
    action: "PutItem",
    options: {
    credentialsRole: apigatewayRole,
    requestTemplates: {
        "application/json": "{\
        \"TableName\": \""+ this.contactstable.tableName + "\",\
        \"Item\": {\
            \""+ props.partitionKey + "\": {\
            \"S\": \"$context.requestId\"\
            },\
            \"name\": {\
            \"S\": \"$input.path('$.name')\"\
            },\
            \"email\": {\
            \"S\": \"$input.path('$.email')\"\
            },\
            \"content\": {\
            \"S\": \"$input.path('$.content')\"\
            }\
        }\
        }",
    },
    integrationResponses: integrationResponses,
    }
});
{% endhighlight %}

To set up an integration response, you create an IntegrationResponse resource and use it to set its target method response. You then configure how to map backend output to the method response: and this is the part of the code that does that.

{% highlight js %}
// create integration response programmatically:
var statuses: { [index: string]: string; } = {
    "200": "",
    "400": "[\s\S]*\[400\][\s\S]*",
    "401": "[\s\S]*\[401\][\s\S]*",
    "403": "[\s\S]*\[403\][\s\S]*",
    "404": "[\s\S]*\[404\][\s\S]*",
    "422": "[\s\S]*\[422\][\s\S]*",
    "500": "[\s\S]*(Process\s?exited\s?before\s?completing\s?request|\[500\])[\s\S]*",
    "502": "[\s\S]*\[502\][\s\S]*",
    "504": "([\s\S]*\[504\][\s\S]*)|(^[Task timed out].*)"
}

// create integration response
var integrationResponses: apigateway.IntegrationResponse[] = [];
for (let status in statuses) {
    var selectionPattern = statuses[status];
    integrationResponses.push({
    statusCode: status,
    selectionPattern: selectionPattern,
    responseParameters: {
        "method.response.header.Access-Control-Allow-Origin": "'''*'''"
    },
    responseTemplates: {}
    })
}

// create method response
var methodResponses: apigateway.MethodResponse[] = [];
for (let status in statuses) {
    var selectionPattern = statuses[status];
    methodResponses.push({
    statusCode: status,
    responseParameters: {
        "method.response.header.Access-Control-Allow-Origin": true
    },
    responseModels: {}
    })
}
{% endhighlight %}

#### DynamoDB
The DynamoDB part is the easiest one: you only have to define a table and remember to enable a DynamoDB Stream: this is the simple snippet to do it - **NOTE**: some props I defined are part of an interface that extends the props of the contact-form stack I created. In this way, you will be able to define more easily your preferences about almost everything inside the code, just by adding a property.

{% highlight js %}
// create contact tables for registration
this.contactstable = new dynamodb.Table(this, props.stage.toString() + '-contacts-table', {
    readCapacity: props.readCapacity,
    writeCapacity: props.writeCapacity,
    partitionKey: {
    name: props.partitionKey,
    type: AttributeType.String
    },
    streamSpecification: StreamViewType.NewImage
})
{% endhighlight %}

We already discussed how to attach things, it's just one method call but...

### Attach things
I lied. Unfortunately, I didn't find a way to properly integrate two services to the same POST method and thus I had to create a lambda to publish the message to the SNS topic. Let's move in order.

#### SNS Topic
Nothing to say, it's just an SNS topic:

{% highlight js %}
// create amazon topic
this.snstopic = new sns.Topic(this, props.stage.toString() + '-contacts-topic', {
    displayName: 'Client subscription topic'
});
{% endhighlight %}

After the deployment, you will have to subscribe your email to the deployed topic and confirm the subscription to receive the emails with messages people send to you.

#### DynamoStream and Lambda
Since there's no direct integration from DynamoDB Stream to SNS, I had to write a Lambda to push my updates: you can find the code of the Lambda [here](https://github.com/made2591/immutable.templates/blob/master/templates/contact-form/lib/sns-lambda/index.js) and the core CDK declaration part below:

{% highlight js %}
// create sns lambda
this.snslambda = new lambda.Function(this, props.stage.toString() + "-contacts-sns", {
    runtime: lambda.Runtime.NodeJS810,
    handler: 'index.handler',
    code: lambda.Code.asset("lib/sns-lambda"),
    environment: {
        "TOPIC_ARN": this.snstopic.topicArn,
    },
    initialPolicy: [lambdaDynamoPolicyStatement, lambdaSNSPolicyStatement, lambdaDynamoStreamPolicyStatement]
})

// create event source
new lambda.EventSourceMapping(this, props.stage.toString() + "-contacts-dynamo-stream", {
    eventSourceArn: this.contactstable.tableStreamArn,
    target: this.snslambda,
    startingPosition: StartingPosition.Latest
});
{% endhighlight %}

Don't forget to provide the right permissions to invoke the lambda!

{% highlight js %}
// give to pipeline permission to invoke the sns lambda
new lambda.CfnPermission(this, props.stage.toString() + "-contacts-lambda", {
    functionName: this.snslambda.functionArn,
    action: "lambda:InvokeFunction",
    principal: "dynamodb.amazonaws.com"
})
{% endhighlight %}

### Conclusion
With just a few lines of Typescript, we are able to have a contact form, completely serverless, that will rock in the neighborhood.

<div class="img_container"><img src="https://i.imgur.com/3fF6oaV.jpg" style="width: 100%; marker-top: -10px;"/></div>

Thank you for reading!

PS: I didn't deploy it yet here, I'm sorry XD 