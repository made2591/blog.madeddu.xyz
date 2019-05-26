---
layout: post
title: "SQS Extended: a serverless producer/consumer chain"
categories: [coding, aws, serverless]
tags: [coding, aws, cdk, guide, serverless]
---

### Introduction
A [few days ago](https://madeddu.xyz/posts/uploader-stack/) I wrote about a simple stack that leverage API Gateway and Lambda-proxy integration to create a safe upload endpoint to let unknown users push inside a bucket of your choice. The stack I will present today is actually a twin stack of this one. In fact, what I didn't mention a few week ago is that the upload stack - with really just a few lines of code - I'm serious, no more than 4 or 5 - can be used to build a producer-consumer chain, by implementing the *SQS Extended* pattern you can find in AWS exams. For the most curious, [here](https://github.com/made2591/immutable.templates/blob/master/templates/producer-consumer/lib/producer-consumer-stack.ts) you can find the core code.

<div class="img_container"><img src="https://i.imgur.com/rw6nQtf.jpg" style="width: 100%; marker-top: -10px;"/></div>

### Scenario
Amazon Simple Queue Service (Amazon SQS) is a fully managed message queuing service that makes it easy to decouple and scale microservices, distributed systems, and serverless applications. Amazon SQS moves data between distributed application components and helps you decouple these components. Unfortunately, one of the limit doesn't let you scale a lot in term of message dimension: the latter cannot exceed 256 KB. Fortunately, AWS provides a solution for this kind of scenario available on Github.

The [Amazon SQS Extended](https://github.com/awslabs/amazon-sqs-java-extended-client-lib) Client Library for Java enables you to manage Amazon SQS message payloads with Amazon S3. This is especially useful for storing and retrieving messages with a message payload size greater than the current SQS limit of 256 KB, up to a maximum of 2 GB. Specifically, you can use this library to specify whether message payloads are always stored in Amazon S3 or only when a message's size exceeds 256 KB, send a message that references a single message object stored in an Amazon S3 bucket, get the corresponding message object from an Amazon S3 bucket, delete the corresponding message object from an Amazon S3 bucket. But I'm not interested in Java in this moment, I'm sorry: instead, I'm more interested in putting in place something to support this idea and, thanks to Typescript inheritance, this will be super simple.

### Design
Let's have a look at the schema:

<div class="img_container"><img src="https://i.imgur.com/XS4Fyxd.png" style="width: 100%; marker-top: -10px;"/></div>

The *producer* asks to an API Gateway endpoint (1) for a pre-signed URL to produce his message. It doesn't need to know where it will be stored, neither the name of the temporary store or having any credentials. After that, the API Gateway will trigger a Lambda function (2) - i.e., the designed entity that runs with a role with attached the permissions to do a `PutObject` over the bucket designed to store the content of the file. The Lambda invokes the ```getSignedUrl``` URL action by using the s3 API (3) and provides back the URL to API Gateway (4) - that will forwards it directly to the producer (6). The producer is now able to push his file to s3 with the provided URL (7): when the file is uploaded, s3 will put a message over an sqs queue (8) and the consumer will be able to retrieve the reference to the *file* - message content, whatever it is - sent by polling the SQS Queue (9). With this object, the consumer can ask to API Gateway the permission to retrieve the message produced (10). Once the presigned URL is generated and sent back (11), it can retrieve safely the content of the message.

#### Surprise surprise
Ok now let's move to the funny part: the code!

{% highlight js %}
import cdk = require('@aws-cdk/cdk');
import sqs = require('@aws-cdk/aws-sqs');

import { UploadFormStack, UploadFormStackProps } from "../../upload-form/lib/upload-form-stack";

interface ProducerConsumerStackProps extends UploadFormStackProps {
}

export interface ProducerConsumerStack extends UploadFormStack {
  sqsQueue: sqs.Queue
}

export class ProducerConsumerStack extends UploadFormStack {
  constructor(scope: cdk.Construct, id: string, props: ProducerConsumerStackProps) {
    super(scope, id, props);

    this.sqsQueue = new sqs.Queue(this, 'Queue', {
      retentionPeriodSec: 1209600
    });
    this.contentBucket.onPutObject(props.stage.toString() + "-put-event");

  }
}
{% endhighlight %}

Done XD.

### Conclusion
The code is actually nothing more than an extention of the `UploadFormStack`: the stack just declare an SQS queue and create the put event from S3. The Lambda already accept which action the requester want to perform over a specified object and thus...we are done!

Thank you for reading!
