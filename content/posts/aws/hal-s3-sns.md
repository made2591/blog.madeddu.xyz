---
layout: post
title: "HAL: AWS s3-sns based single-slack-command bot to handle your VPC"
date: 2018-03-24
polly: https://madeddu.xyz/mp3/aws/hal-s3-sns.mp3
categories:
- coding
- python
- aws
tags:
- coding
- python
- aws
- s3
- sns
- ec2
- lambda
- slack
- bot
katex: true
markup: "mmark"
---

### Introduction
I recently build a [Slack](https://slack.com) command to help me handle actions on my VPC. The only thing you need is an [AWS account](http://aws.amazon.com) - Free Tier it's ok. I [recently wrote](https://madeddu.xyz/posts/free-tier-cloudwatch) about how to maximize resources, with particular focus on the number of hours you have in Free Tier - using specific CloudWatch Rules. In this article, I want to describe how I extended my architecture to invoke _actions_ - potentially, all the action provided by Amazon Web Services official SDK(s) - with a single Slack command. I decided to call this slack command HAL because I think it's a really dangerous command 😜

<div class="img_container"><img src="https://i.imgur.com/sIQhMOm.jpg" style="width: 100%; marker-top: -10px;"/></div>

### Ingredients
You will need:
- [AWS account](http://aws.amazon.com) (free tier it's ok)
- [AWS Lambda](https://aws.amazon.com/en/Lambda/)
- [AWS S3](https://aws.amazon.com/en/s3/)
- [AWS SNS](https://aws.amazon.com/en/sns/)
- [Slack](https://slack.com)

#### General schema
The schema I created to handle my actions is the following

<div class="img_container"><img src="https://i.imgur.com/R8uA19x.png" style="width: 100%; marker-top: -10px;"/></div>

### Recipe
As I already said in a preview post on AWS Services, I recommend you to pay a lot of attention. You always have to know exactly what are you doing, to avoid surprise in billing in the end of the month. Fortunately, there are a lot of documentations on Amazon official site, so you only have to read them.

#### One slack command to rule them all
Immagine to have a chatbot able to recognize sentence orders:

    HAL, start my jenkins instances and stop the kafka cluster.

Or

    Schedule my-Lambda to run in the next 3 hours.

This sentences both invoke particular predefined actions in your VPC, but if you you want a bot able to understand them, you have no choice: you need something able to understand natural language. This is tricky because even if you can do this efficiently - I mean, recognize instances, actions, contexts, retrieve ids of resources and so on - in the end you have to deal with the mapping between what you (actually, your bot) understood and what you effectively want to do in your VPC (or house, etc). But...if you think about it, why should you use natural language? I mean, you use regular language to talk to machines every day, without any problems: git add, docker-compose up, ps aux, etc are all specific commands precisely understood by machines, like clicks in web applications are able to reach specific url. The _parameters_ and parametric thinking solve - in a sense - the problems of create something as much as possible generic, durable, but...in the end, formally defined. Regular.

And that's why I decided to implement a grammar - the old style way. Imagine to have two sets: $$C$$, the set of contexts (ec2, Lambda, etc) and $$A_c$$ the set of action for the specific context $$c \in C$$. You can invoke one single command with one or more specified context, each of them followed by one or more actions available in the specific context, each of them followed by 0 or more parameters - if needed. It seems difficult to create something like this, but - believe me - it is not. The grammar is really simple:

- S := /hal (C;)+
- C := $$c \in C$$ (-> A(c)) \| $$c \in C$$ (-> A(c),)+;
- A(c) := $$a \in A_c$$ (a-zA-Z0-9)\*;

Ok, I mixed a little bit of notation: first, don't get confused by the uppercase symbols (S, C, A) and the the set C and A. The grammar start symbol is S: it produces the string _"/hal"_ followed by the result of at least one (I used regex expression +) production of the symbol C followed by _";"_. Thus, our command would be something like

    /hal [result of C]; ... [result of C]; from 1 to n

The C symbol produces a string in the form _"c"_ with $$c \in C$$, followed by the result of 1 or more (I used regex expression + and or symbol \| to prevent insert a comma _","_) concatenation of _"->"_ with the production of the symbol A (related to the chosen context c) followed by _","_. Let be c = _"ec2"_, our C symbol would produce something like:

    ec2 -> [result of A(ec2)]

Or, if more than one action is produced, each one except the last is followed by the commma _","_ and the result will be something like (for instance, for three actions over the same context):

    ec2 -> [result of A(ec2), result of A(ec2), result of A(ec2)]

Finally, given a context c, the symbol $$A(c)$$ produces a concatenation of one of the actions $$a$$ available in the context (i.e., with $$a \in A_c$$) followed by 0-n parameters. Given a = _"start"_, with $$a \in A_c$$, then the symbol A(c) will produce:

    start all

or

    start docker jenkins

or simply

    start

Putting all toghether and extending the set of context and actions, we get something like:

    /hal ec2 -> start docker jenkins, -> stop kafka; Lambda -> create new nodejs;

This command is easy to write, readable and easily to parse using a little bit of string manipulation without going crazy. Of course, you can define your own grammar!

#### A single AWS Lambda to execute actions
Let's forget for a moment the grammar. Let say that you have the result of your commands in a JSON request in the form:

{{< highlight json >}}
[
    {
        "context" : "value",
        "action" : "value",
        "parameters" : []
    },
    ...
    {
        "context" : "value",
        "action" : "value",
        "parameters" : []
    }
]
{{< / highlight >}}

Ok, this is simple to handle. The only things you need to do is mapping context and actions to specific action in your VPC - using an AWS Lambda function. You will use this mapping also to _validate_ your command later, so I decided to wrote a JSON file to a closed S3 bucket. In the next paragraph, I describe this JSON configuration file and how to create a s3 bucket.

#### S3: unique configuration endpoint
Have a look at the configuration below

{{< highlight json >}}
{
  "ec2" : {
    "start" : "start_instances_action",
    "stop" : "stop_instances_action"
  },
  "Lambda" : {
    "invoke" : "invoke_Lambda",
    "create" : "create_Lambda",
    "schedule" : "schedule_Lambda"
  }
}
{{< / highlight >}}

The keys at the first level provide the set $$C$$ of specific contexts you want to handle with your grammar. Keys of each context provide the set of specific actions $$A(c)$$ available in the context and the value of each action key is the name of the method to invoke during the AWS Lambda execution. To create an s3 bucket, follow [this](https://docs.aws.amazon.com/AmazonS3/latest/gsg/CreatingABucket.html) guidelines, then put your JSON configuration file in the bucket. You will need the name of the bucket and the of the file in it late to setup AWS Lambda(s). Let's create the core Lambda that will execute passed actions.

#### AWS Lambda: VPC Actions
I created this Lambda using Python: you can of course working with the other supported language. The code is available [in this Github Gist](https://gist.github.com/made2591/6c4c35590831101f203f8b5b26d4e2f9).

The ```VPCAction``` class implements the methods defined in the configuration file created and uploaded to the s3 bucket: in my example, the Lambda context methods are missing but this is not a problem because the ```getActionConfiguration``` method looks (this is done using the ```getattr``` method) for an acation in the ```VPCAction``` class and if there isn't a match, assign an empty action to prevent errors during execution of (eventually not-defined) actions (better, defined in configuration so available in commands, but actually not implemented yet).

If the action is allowed and pass each check, the response of the respective method called on the specific context with given parameters is saved in a dictionary and then returned as result.

Another interesting point is the ```isinstance``` check done in the handler method is to prevent errors while testing your Lambda. In fact, this Lambda will be invoked by using ```AWS SNS```, so it has to be able to deal with events coming from the SNS publish-subscribe system. I will talk later about this. After you give to the role assumed by this Lambda the right policy to get into the s3 and to work with ec2 instances, you can test it by creating a test event like the ones defined in the previous paragraph.

{{< highlight json >}}
[
  {
    "context": "ec2",
    "action": "stop",
    "parameters": [
      "docker",
      "jenkins"
    ]
  }
]
{{< / highlight >}}

Done? Ok. Let' attach the SNS trigger to the Lambda. It's really simple: starting from the web console, you can add a trigger to your Lambda and the only thing you have to do is provide the ARN of the topic. To define a topic for HAL and get an ARN, follow [this](https://docs.aws.amazon.com/sns/latest/dg/CreateTopic.html) guidelines.

Let's go ahead by creating another Lambda to handle slack command. I used Node.js for my ```SlackEntryPoint``` AWS Lambda: for the moment, create an empty function, just to test message coming from the Slack command we will create later: just put a log and return the event passed for debug purpose.

Let's define an endpoint for the empty Lambda

#### API Gateway
- In AWS go to the API Gateway section;
- Click the “Create API” button and fill in the details;
- Click the Create method button in the top right, and select POST from the drop down on the left (and click the small tick);
- In the Integration type select Lambda function, and select the region your AWS account is set to and start typing - for instance, in my case ```SlackEntryPoint``` (the empty Lambda) - for a list of your Lambdas to appear;
- Click “Save”;
- On the following screen click the box Integration Request and scroll down to the Mapping Requests section to define a Body Mapping template;
- Click Add mapping template and type application/json into the box and click the small tick;
- Click the pen icon next to the word Input passthrough and select Mapping template from the dropdown;
- In the Template box put the code available [here](https://gist.github.com/made2591/f173b4af0a2d6b4e87887507b59260f9);
- Click the small tick to save these settings and select the Deploy API button;

At the top of the page you should now be given a HTTPS URL (similar to http://xyz.execute-api.zone-1.amazonaws.com/stage). This URL is needed for the creation of the Slack command /hal.

#### Slack command
When you create a Slack application (see step 4 of [my previous post](https://madeddu.xyz/posts/free-tier-cloudwatch) for more details), you can easily add a command to invoke request. The only things you need is a URL as endpoint - the one we have just defined above. If you have a look a look at the [documentation](https://api.slack.com/slash-commands), you find out that each time you invoke the Slack command, the message (and its data) will be sent to the configured external URL via HTTP POST. For example, imagine a workspace at example.slack.com installed an app with a command called /weather. If someone on that workspace types /weather 94070 in their #test channel and hits enter, this data would be posted to the external URL:

{{< highlight sh >}}
token=gIkuvaNzQIHg97ATvDxqgjtO
team_id=T0001
team_domain=example
enterprise_id=E0001
enterprise_name=Globular%20Construct%20Inc
channel_id=C2147483705
channel_name=test
user_id=U2147483697
user_name=Steve
command=/weather
text=94070
response_url=https://hooks.slack.com/commands/1234/5678
trigger_id=13345224609.738474920.8088930838d88f008e0
{{< / highlight >}}

This data will be sent to your URL as a HTTP POST with a ```Content-type``` header set as ```application/x-www-form-urlencoded```: because we defined a Body Mapping template to deal with x-www-form-urlencoded content-type, we can use the link provided by API Gateway console after the deploy and start testing our command (better if in a private channel). If you type and return /hal, you could see a json version of the information provided by Slack.

#### AWS Simple Notification Service
You can fill your ```SlackEntryPoint``` Lambda with the code available [in this Github Gist](https://gist.github.com/made2591/9db86b9c3256ea60620b7923f9ee9297) - I know, it's not so good, but it's ok for a test. That's the part where AWS SNS is used: SNS is the way I delegate the action parsed by my SlackEntryPoint Lambda to my VPCAction Lambda to both reduced timeout error - still there, if my Lambda are not cached (1st call) - returned by Slack and to split in pieces without invoking a Lambda from a Lambda (it is possible, but I think slower). In particular, SlackEntryPoint in order:

- checkSecurity (validate Slack provided token, and in my case alse my ID, my private channel and so on);
- parseText (provided as arguments of my /hal command);
- checkIntegrity (this parts use the same configuration file of the bucket previously created);
- publishActionOnSNSTopic (and invoke the VPCActions Lambda - excluding grammar errors);

#### Optional: AWS Lambda to send information on change of status
I recently wrote a Lambda to let me know the change of status: of course, you can create a Lambda able to send generic message, and let VPC Actions publish the message over a SNS topic to provide slack response customized by the action you invoked. My third Lambda for this project is available [here](https://gist.github.com/made2591/acc61010fa119929718e122ca69d8a92).

<div class="img_container"><img src="https://i.imgur.com/J8KcD4P.png" style="width: 100%; marker-top: -10px;"/></div>

Thank you everybody for reading!

[^Lambda]: You can find more information [here](https://aws.amazon.com/Lambda/?nc1=h_ls)