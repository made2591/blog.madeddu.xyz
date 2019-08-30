---
layout: post
title: "SQS Extended: a serverless producer/consumer chain"
categories: [coding, aws, serverless]
tags: [coding, aws, cdk, guide, serverless]
---

### Introduction
A [few days ago](https://madeddu.xyz/posts/uploader-stack/) I wrote about a simple stack that leverage API Gateway and Lambda-proxy integration to create a safe upload endpoint to let unknown users push inside a bucket of your choice. The stack I will present today is can be used to build a producer-consumer chain, by implementing the *SQS Extended* pattern you can find in AWS exams. For the most curious, [here](https://github.com/made2591/immutable.templates/blob/master/templates/producer-consumer/lib/producer-consumer-stack.ts) you can find the core code.

<div class="img_container"><img src="https://i.imgur.com/rw6nQtf.jpg" style="width: 100%; marker-top: -10px;"/></div>

### Scenario
You were wondering: why Tweedledum e Tweedledee? Go ahead :)

Amazon Simple Queue Service (Amazon SQS) is a fully managed message queuing service that makes it easy to decouple and scale microservices, distributed systems, and serverless applications. Amazon SQS moves data between distributed application components and helps you decouple these components. Unfortunately, one of the limit doesn't let you scale a lot in term of message dimension: the latter cannot exceed 256 KB. Fortunately, AWS provides a solution for this kind of scenario available on Github.

The [Amazon SQS Extended](https://github.com/awslabs/amazon-sqs-java-extended-client-lib) Client Library for Java enables you to manage Amazon SQS message payloads with Amazon S3. This is especially useful for storing and retrieving messages with a message payload size greater than the current SQS limit of 256 KB, up to a maximum of 2 GB. Specifically, you can use this library to specify whether message payloads are always stored in Amazon S3 or only when a message's size exceeds 256 KB, send a message that references a single message object stored in an Amazon S3 bucket, get the corresponding message object from an Amazon S3 bucket, delete the corresponding message object from an Amazon S3 bucket. But I'm not interested in Java in this moment, I'm sorry: instead, I'm more interested in putting in place something to support this idea and, thanks to Typescript inheritance, this will be super simple.

### Design
Let's have a look at the schema:

<div class="img_container"><img src="https://i.imgur.com/lCzt2ON.png" style="width: 100%; marker-top: -10px;"/></div>

The *producer* asks to an API Gateway endpoint (1) for a pre-signed URL to produce his message. It doesn't need to know where it will be stored, neither the name of the temporary store or having any credentials. After that, the API Gateway will trigger a Lambda function (2) - i.e., the designed entity that runs with a role with attached the permissions to do a `PutObject` over the bucket designed to store the content of the file. The Lambda invokes the ```getSignedUrl``` URL action by using the S3 API (3) and provides back the URL to API Gateway (4) - that will forwards it directly to the producer (6). The producer is now able to push his file to S3 with the provided URL (7).

Until now, this stack seems similar to the one I discussed [previously](https://madeddu.xyz/posts/uploader-stack/)...wait. It's actually the same stack:

<div class="img_container"><img src="https://i.imgur.com/7Z3KeHf.png" style="width: 100%; marker-top: -10px;"/></div>

Ok, so what changed? Well, when the file is uploaded, S3 will put a message over an SQS queue (8). I added a piece: the consumer will be able to retrieve the reference to the *file* sent - message content, whatever it is - by polling the SQS Queue (9). With this message, the consumer can ask to API Gateway the permission to retrieve the original content produced (10). Once the presigned URL is generated and sent back (11), it can retrieve safely the content of the message directly from S3 (12).

#### Surprise surprise
Ok now let's move to the funny part: the code!

{% highlight js %}
import cdk = require('@aws-cdk/cdk');
import cloudtrail = require('@aws-cdk/aws-cloudtrail');
import iam = require('@aws-cdk/aws-iam');
import sqs = require('@aws-cdk/aws-sqs');

import { UploadFormStack, UploadFormStackProps } from "../../upload-form/lib/upload-form-stack";
import { PolicyStatementEffect } from '@aws-cdk/aws-iam';
import { ReadWriteType } from '@aws-cdk/aws-cloudtrail';

export interface ProducerConsumerStackProps extends UploadFormStackProps {
}

export interface ProducerConsumerStack extends UploadFormStack {
  sqsQueue: sqs.Queue
}

export class ProducerConsumerStack extends UploadFormStack {
  constructor(scope: cdk.Construct, id: string, props: ProducerConsumerStackProps) {
    super(scope, id, props);

    // create trail to enable events on S3
    const trail = new cloudtrail.Trail(this, props.stage.toString() + 'content-bucket-trail');
    trail.addS3EventSelector([this.contentBucket.bucketArn + "/"], {
      includeManagementEvents: false,
      readWriteType: ReadWriteType.All,
    });

    // create sqs queue
    this.sqsQueue = new sqs.Queue(this, props.stage.toString() + "-sqs-content-queue", {
      retentionPeriodSec: 1209600,
      visibilityTimeoutSec: 360,
      deadLetterQueue: {
        queue: new sqs.Queue(this, props.stage.toString() + "-sqs-dead-queue", {
          retentionPeriodSec: 1209600
        }),
        maxReceiveCount: 5
      }
    });

    // create sqs policy statement to allow cloudwatch pushing the message
    var s3SQSStatement = new iam.PolicyStatement(PolicyStatementEffect.Allow)
    s3SQSStatement.addActions(
      "sqs:SendMessage",
      "sqs:SendMessageBatch",
      "sqs:GetQueueAttributes",
      "sqs:GetQueueUrl",
    )
    s3SQSStatement.addResources(
      this.sqsQueue.queueArn
    );
    s3SQSStatement.addAnyPrincipal()
    s3SQSStatement.addCondition("ArnLike", { "aws:SourceArn": this.contentBucket.bucketArn })

    this.contentBucket.addObjectCreatedNotification(this.sqsQueue, {
      prefix: "input/"
    })

  }
}
{% endhighlight %}

The most interesting thing to notice is the declaration of the `ProducerConsumerStack`: it extends the `UploadFormStack`, by actually inherits everything inside it ^^.

#### Deep dive
The code declare a trail for the bucket - in the future, could be used to trigger `CloudWatch Events Rule`, that right now doesn't support yet SQS Queue as Target (I'm working on it and I hope my pull request will be merged soon). After that, there's the creation of an SQS Queue with a `Dead Letter Queue` (soon I will wrote about why the dead queue letter is required). Finally, to avoid using Event Target rule, I use directly the S3 Service to push notification of elements under a certain prefix over the queue. To do that we have to provide the SQS the right policy to let the bucket `SendMessage` over it. And voilà!

### Conclusion
The code is actually nothing more than an extention of the `UploadFormStack`: the stack just declare an SQS queue and create a message over it with the put event from S3. The Lambda already accept JSON request with specified action the requester wants to perform over a specified object and thus...we are done!

The next stack - step 3 will build over this one as well so...stay tuned!

Thank you for reading!
