---
layout: post
title: "A serverless OCR with Polly and Rekognition unveils the power of stack inheritance in CDK"
date: 2019-06-13
polly: https://madeddu.xyz/mp3/aws/cdk/serverless-ocr.mp3
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
- polly
- rekognition
---

### Introduction
In the last two weeks, I released a few CDK stacks: I made some experiments around API Gateway and service integration that came out in two serverless forms, the [Contact Form](https://madeddu.xyz/posts/contact-form/) and the [Upload Form](https://madeddu.xyz/posts/uploader-stack/), read to be deployed in your static web page 😎. Actually, with CDK you can do so much more and so much more easily. The last stack I released - a *producer-consumer* chain presented [here](https://madeddu.xyz/posts/producer-consumer/) - it was a way I used to introduce how you can leverage Typescript inheritance to recycle an old stack and build on top of it. In this last step of this cycle, I will extend this concept once again to build - DRUMROLL - an OCR Serverless solution to be integrated into your application. The use cases are multiple: imagine you want to create podcasts of course lessons on top of your notes, to be able to listen to them during your trip to office or university or whenever you want. Imagine an application to help blind people - like the [Be My Eyes](https://www.bemyeyes.com/) app - to read documents without the help of no ones but AWS Services. Well, the use cases are endless so... let's go!

<div class="img_container"><img src="https://i.imgur.com/3IGyGe0.png" style="width: 100%; marker-top: -10px;"/></div>

### Architecture recap
Ok, before going ahead: how cool is the image above?! Sorry, I had to say it. So as already mentioned, I started with a simple an [Upload Form](https://madeddu.xyz/posts/contact-form/). I then extended this stack with an SQS Queue to enable a consumer to *consume* data uploaded: this is the [Producer/Consumer](https://madeddu.xyz/posts/producer-consumer/) stack. Finally, the Document Reader stack will leverage this chain to let a Lambda consume the document upload, rekognize the test on it and produce an mp3 with the extracted sentences read for us. For the most impatient, you can have a look at the stack [here](https://github.com/made2591/immutable.templates/blob/master/templates/document-reader/lib/document-reader-stack.ts). Before going ahead, let's recap the previous stacks.

#### Stack 1: Upload Form
The schema is pretty simple, let's have a look to it once again.

<div class="img_container"><img src="https://i.imgur.com/veA8UUs.png" style="width: 65%; marker-top: -10px;"/></div>

The functioning of this schema is pretty simple: the user asks to API Gateway (1) for a pre-signed URL to upload a document. It doesn't need to know where it will be stored, neither the name of the bucket or having any credentials: this covers our scenario in which a generic user just want to upload a document into our application, and only owns the document. The API Gateway will trigger a Lambda function (2) that will invoke the ```getSignedUrl``` URL action by using the S3 API (3) and provides back the URL to API Gateway (4) - that will forwards it directly to the user (6). The user is now able to push his document to S3 with the provided URL (7).

#### Stack 2: Producer/Consumer chain
The second stack is just as simple as producing a message with the document uploaded by the users:

<div class="img_container"><img src="https://i.imgur.com/7Z3KeHf.png" style="width: 100%; marker-top: -10px;"/></div>

This stack doesn't introduce a lot of changes: when the document is uploaded, S3 will put a message over an SQS queue (8). The consumer will be able to retrieve the reference to the *document* sent by pollyng (ops) the SQS Queue (9). With this message, the consumer can ask to API Gateway the permission to retrieve the original document produced (10). Once the pre-signed URL is generated and sent back (11), it can retrieve safely the content of the message directly from S3 (12).

#### Stack 3: Document Reader a.k.a. Serverless OCR
Finally, the third stack. The schema is below:

<div class="img_container"><img src="https://i.imgur.com/hogQluu.png" style="width: 95%; marker-top: -10px;"/></div>

Nice, isn't it? So I just added the Dead Letter Queue I missed in the producer-consumer schema above - but is actually not a key element of the stack. It's the small SQS Queue in the bottom part. So, the first important thing is: this stack implements the *consumer* I spoke about before. We will have a look in a few paragraphs. The steps completed are pretty simple: the consumer Lambda provides the document retrieved (12) to Rekognition service (13) and get back the extracted text (13) if any. After that, it sends this text to Polly (15) and gets back and AudioStream (14) ready to be uploaded to S3. Before going ahead, it saves the references of the document, the extracted test, and the produced output to a DynamoDB table. Finally, it saves the AudioStream as a *.mp3* file to S3, where the document was originally stored by the user. The interesting part is that this - from an architectural perspective - was done by leveraging the two previous stacks, as shown in the picture above.

<div class="img_container"><img src="https://i.imgur.com/xDa4Rmp.png" style="width: 100%; marker-top: -10px;"/></div>

### The OCR Stack
First of all, the stack declaration:

{{< highlight ts >}}
interface DocumentReaderStackProps extends ProducerConsumerStackProps {
  readCapacity: number;
  writeCapacity: number;
  partitionKey: string;
}

export interface DocumentReaderStack extends ProducerConsumerStack {
  parsedDocument: dynamodb.Table;
  workerLambda: lambda.Function;
}

export class DocumentReaderStack extends ProducerConsumerStack {
  constructor(scope: cdk.Construct, id: string, props: ProducerConsumerStackProps) {
    super(scope, id, props);
    ...
    // code
    ...
  }
}
{{< / highlight >}}

As you can see, it extends the ProducerConsumerStack - that extends the UploadFormStack. The important thing to notice is that if you extend things these have to be visible from outside the module they are declared into: this is always true for the classes that implement the Stacks but could be false for the interfaces that define the properties of them. Thus, I exported the interface `ProducerConsumerStackProps` - as the `UploaderFormStackProps` before - to have visibility of them from outside respective module.

Now, let's cover the *code* part. Since this new Stack uses mainly *services*, we don't have to provision a lot of things: it's mainly about creating the required DynamoDB Table and providing the consumer Lambda the right permissions to leverage the involved services. In order, the DynamoDB Table

{{< highlight ts >}}
// create tables for images, text and registration references
this.parsedDocument = new dynamodb.Table(this, props.stage.toString() + '-parsed-document', {
    readCapacity: props.readCapacity,
    writeCapacity: props.writeCapacity,
    partitionKey: {
        name: props.partitionKey,
        type: AttributeType.String
    }
});
{{< / highlight >}}

and the worker Lambda:

{{< highlight ts >}}
// create worker lambda
this.workerLambda = new lambda.Function(this, props.stage.toString() + "-worker-lambda", {
    runtime: lambda.Runtime.NodeJS810,
    handler: 'index.handler',
    code: lambda.Code.asset("/Users/matteo/immutable.templates/templates/document-reader/lib/worker-lambda"),
    timeout: 60,
    environment: {
        "DYNAMO_TABLE": this.parsedDocument.tableName,
        "CONTENT_BUCKET": this.contentBucket.bucketName
    },
    initialPolicy: [
        lambdaS3PolicyStatement,
        lambdaRekognitionPolicyStatement,
        lambdaPollyPolicyStatement,
        lambdaDynamoPolicyStatement,
        lambdaSQSStatement
    ]
})
{{< / highlight >}}

Finally, as explained in the Amazon documentation, it is possible to create a standard Amazon SQS queue to serve as an event source for the consumer Lambda. The queue must be configured to allow time for the consumer Lambda to process each batch of events — and for Lambda to retry in response to throttling errors as it scales up. If a message fails to process multiple times, Amazon SQS can send it to a dead letter queue. This is the reason it was configured in the producer-consumer stack, to retain messages that failed to process for troubleshooting. I also set the `maxReceiveCount` on the queue's redrive policy to at least 5 to avoid sending messages to the dead letter queue due to throttling. The SQS integration that enables Long Polling of the Lambda can be done with just a few lines of code:

{{< highlight ts >}}
// create event source
new lambda.EventSourceMapping(this, props.stage.toString() + "-worker-lambda-source", {
    eventSourceArn: this.sqsQueue.queueArn,
    target: this.workerLambda,
});
{{< / highlight >}}

### The consumer
Let's talk about the consumer or the Lambda that orchestrate the required steps. The code of the Lambda is available [here](https://github.com/made2591/immutable.templates/blob/master/templates/document-reader/lib/worker-lambda/index.js).

#### Step 1: Rekognize the text
Amazon Rekognition makes it super easy to extract text from images. Actually, there's a specific service also to work on this topic and it's called - surprise - [Amazon Textract](https://aws.amazon.com/textract/?n=sn&p=r). So you were wondering why I didn't use this service and the reason is: no reason, I didn't know about it when I build the consumer Lambda 😅 but, hey! Leveraging this service doesn't change the overall functioning of the solution. So, assuming you want to use Rekognition, this is how you can do it easily:

{{< highlight ts >}}
function detectTextFromBytes(bytes) {
    return rekognition.detectText({Image: {Bytes: bytes}}).promise()
}

function getBase64BufferFromS3(objectKey) {
    return s3.getObject({
            Bucket: contentBucket,
            Key: objectKey,
        }).promise()
        .then(response => response.Body)
        .catch(error => {
            console.error(`error in detecting text: ${error}`)
        })
}

async function rekognizeText(rawObjectKey, maxLabels, minConfidence, attributes) {
    const bytes = await getBase64BufferFromS3(rawObjectKey)
    if (!bytes) return noText
    let text = await detectTextFromBytes(bytes)
    text = text && text.TextDetections ? text.TextDetections.map(i => i.DetectedText).join(" ") : noText
    return text
}
{{< / highlight >}}

The three methods above just get the content of the image provided encoded in base64. Then, they detect the text from the images and output the extracted text. We are ready to provide the text to Polly.

#### Step 2: Read the text with Polly
Reading the text with Polly is pretty too! You can do it using the `synthesizeSpeech` API - more on the [official doc](https://docs.aws.amazon.com/AWSJavaScriptSDK/latest/AWS/Polly.html#synthesizeSpeech-property). The following two methods produce the `AudioStream` needed to create the mp3 file and save the result on S3.

{{< highlight ts >}}
function savePollyResultToS3(bucket, objectKey, audioStream) {
    return s3.putObject({
            Bucket: bucket,
            Key: objectKey,
            Body: audioStream,
            ContentType: 'audio/' + outputFormat
        }).promise()
        .catch(error => {
            console.error(`error in save object to S3: ${error}`)
        })
}

async function readText(text, voiceId, outputObjectKey) {
    let audio = await polly.synthesizeSpeech({
        Text: text,
        OutputFormat: outputFormat,
        VoiceId: voiceId
    }).promise()
    if (audio.AudioStream instanceof Buffer) {
        await savePollyResultToS3(contentBucket, outputObjectKey, audio.AudioStream)
        return outputObjectKey
    } else {
        console.error(`audiostream is not a buffer`)
    }
}
{{< / highlight >}}

And finally the references of the object analyzed is stored into DynamoDB with a simple `putObject`.

### Conclusion
I finish this series of stacks concatenated together to implement different use cases scenario: we pass from an Upload Form to a Serverless solution to read your document, without having nothing to manage excluding your content! The next step would be to build the piece of infrastructure to notify your producer that the document is read: I have already some idea in my mind - and I think you also have it - but it really depends on the use case you want to build. I hope you enjoyed these three articles and this stack.

Thank you everybody for reading!
