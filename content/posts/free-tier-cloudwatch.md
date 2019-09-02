---
layout: post
title: "AWS Free Tier, Docker and Jenkins: smart resources handling with CloudWatch Events and Slack"
date: 2018-03-11
categories:
- coding
- golang
- js
- aws
tags:
- coding
- aws
- lambda
- cloudwatch
- rules
- event
- ec2
- slack
- docker
- jenkins
---

### Introduction
If you have an [AWS account](http://aws.amazon.com) in Free Tier, you have (updated: March, 13th 2018) 750 hours/month to run EC2 (small ones) in your VPC. You also have a lot of other resources, such as AWS Lambda functions (I wrote about them [here](https://madeddu.xyz/posts/aws-lambda) and [here](https://madeddu.xyz/posts/aws-step-functions)) and CloudWatch Events. In this article, I talk about smart resources handling and some trick - actually, not so smart XD - I setup to take the best from the services. Attention!!! Picture Spoiler

<div class="img_container"><img src="https://i.imgur.com/K9B0I3F.png" style="width: 100%; marker-top: -10px;"/></div>

### Ingredients
For this article, you will need the following:
- An [AWS account](http://aws.amazon.com) (free tier it's ok, but API Gateway is not included);
- [AWS EC2](https://aws.amazon.com/ec2/?nc1=h_ls);
- [AWS Lambda](https://aws.amazon.com/en/lambda/);
- [AWS CloudWatch Events](https://aws.amazon.com/cloudwatch/?nc1=h_ls);
- [Slack](https://slack.com) - it's a plus;

### Recipe
As I already said in a preview post on AWS Services, I recommend you to pay a lot of attention. You always have to know exactly what are you doing, to avoid surprise in billing in the end of the month. Fortunately, there are a lot of documentations on Amazon official site, so you only have to read them.

#### What and when
Ok, let's start from the AWS EC2. You can create really slow and not-efficient machines of type t2-what? ssh-session expired. Just kidding.

The crucial part is that you have 750 hours free. Despite the case you can change Amazon Time, this implies that you can have an instance up and running for 1 year: in fact, 24\*31 = 744 < 750. That's a huge amount of time before closing your AWS account. Of course, you will not use all this hours of execution in a month. Let's suppose you will use 12 hours a day - it's too much, but let's keep it simple. This was my initial setup: in this case, you can run 2 instances for 12 hours a day for 1 years. Not's so bad, even if the instances are really slow. You can scale di reasoning up to 6 instances for 4 hours a day...even 24 for 1 hours: if you're wondering what you can do with 24 1 vCPU with 1 GB of RAM, ask to those crazy guys in the wab that built a 64-Raspberry PI Cluster. Got it?

### Scenario
I will talk about a really simple (also, not so efficient) scenario: 2 EC2 machines, the first one to run a Jenkins and the second one to run docker-daemon. I will not go through the details about how to setup a Jenkins CI/CD pipeline, there are plenty of guides better than mine. Instead, I will show you the step to automatically reach your instances after each reboot: this problem is arised by AWS, because each time you reboot an instance, if no Elastic IP (more [here](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/elastic-ip-addresses-eip.html)) is attached, another Public IP will be assigned and you don't know a priori which one. First, let's talk about scheduling of start and stop.

#### Step 1/5: Start and Stop your instances
There are many ways (maybe?) to start and stop your instances: I decided to use AWS Lambda because you can easily create and manage them. This time I decided to use Python to build my StartAndStop Lambda. Before going ahead with code, please remember to assign the right policy to the Role used by the Lambda function (you can choose the role during the setup after the click of Create Function button). From the official AWS Doc, the Statement to add to the policy attached to the Role you choose is the following:

{{< highlight json >}}
{
    "Effect": "Allow",
    "Action": [
        "ec2:Start*",
        "ec2:Stop*"
    ],
    "Resource": "*"
}
{{< / highlight >}}

The code of the function is really simple:

{{< highlight python >}}
import boto3, os

def lambda_handler(event, context):
    ec2 = boto3.client('ec2', region_name=os.environ['region'])

    if len(event['instances']) == 0:
        return { "message" : "No instances passed" }

    if event['action'].encode("utf-8") != 'start' and event['action'].encode("utf-8") != 'stop':
        return { "message" : "Passed action not allowed: '" + event['action'].encode("utf-8") + " over '" + event['action'].encode("utf-8")+"'" }

    if event['action'].encode("utf-8") == 'start':
        response = ec2.start_instances(InstanceIds=event['instances'])

    if event['action'].encode("utf-8") == 'stop':
        response = ec2.stop_instances(InstanceIds=event['instances'])

    return response
{{< / highlight >}}

This AWS Lambda use ```boto3``` library to start and stop instances: the expected request is like the following:

{{< highlight json >}}
{
  "action": "stop",
  "instances": [
    "YourJenkinsInstanceID", "YourDockerInstanceID"
  ]
}
{{< / highlight >}}

Done? Let's go with CloudWatch Events.

#### Step 2/5: Schedule AWS Lambda execution
Amazon CloudWatch Events delivers a near real-time stream of system events that describe changes in Amazon Web Services (AWS) resources. Using simple rules that you can quickly set up, you can match events and route them to one or more target functions or streams. More in details, I used [ScheduledEvents](https://docs.aws.amazon.com/AmazonCloudWatch/latest/events/ScheduledEvents.html) function to schedule my Lambda: with Scheduled Events, you can create rules that self-trigger on an automated schedule in CloudWatch Events using cron or rate expressions. All scheduled events use UTC time zone and the minimum precision for schedules is 1 minute.

| Field        | Values          | Wildcards     |
|--------------|-----------------|---------------|
| Minutes      | 0-59            | , - * /       |
| Hours        | 0-23            | , - * /       |
| Day-of-month | 1-31            | , - * ? / L W |
| Month        | 1-12 or JAN-DEC | , - * /       |
| Day-of-week  | 1-7 or SUN-SAT  | , - * ? L #   |
| Year         | 1970-2199       | , - * /       |

I want to start my docker-daemon and jenkins server both at 9am and stop them at 9pm. To do that:

- First, open [CloudWatch Console](https://console.aws.amazon.com/cloudwatch/), then click on __Rules__ on the left;
- Click on the blue button Create Rule, select Schedule, select cron expression and place your expression. My cron expression to start each day (forever) my instances is the following, but you have to deal with UTC and your timezone of course.

<span style="color:#A04279; font-size: bold;">&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;0 8 * * ? *</span>

- Then, on the right panel, click on the Add target button. Choose Lambda from the select box, then select the AWS Lambda function created in the Step 1. In Configure input form, click on Constant and copy and paste - with your instance ids:

{{< highlight json >}}
{ "action": "start", "instances": [ "YourJenkinsInstanceID", "YourDockerInstanceID"] }
{{< / highlight >}}

- Finally, click on configure details and - it's just a suggest - write the Instance IDs involved in the request in your description. To test if the rule triggers you can setup cron to run after a few mitutes and then change to the desired time.

<div class="img_container"><img src="https://i.imgur.com/pPakEsW.png" style="width: 100%; marker-top: -10px;"/></div>

To stop instances, create another rule with	0 20 * * ? * - remember that it is UTC time zone!!! - and invoke the same AWS Lambda function with action value equal to "stop" in your Constant JSON input.

#### Step 3/5: Have always updated DNS
The best way to deal with Public DNS is by assigning an Elastip IP to your instances, or setup a third-level-domain service inside each of your instance (like no-ip)...or use again Lambda (and Slack) to alert you whanever there is a change of status.

This time, the IAM Policy System requires from you (actually, your Lambda function) something more: this function should be able to ask for instance details. To do that, it needs - at least - read access to EC2 Instances details (I think there is a policy to do that). In any case, you don't have to deal with super restrictive policy, because your Lambda is not exposed through API Gateway and is called only by a CloudWatch Event. Thus, you can add - at least, for this experiment - the _AmazonEC2FullAccess_ managed policy ([this](https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_policies_examples_ec2_region.html) is the description page, I guess) to your Role.

For this Lambda, I used Node.js (just to make things a little bit confusing for you). First, install ```slack-node``` in the folder you will upload to your AWS Lambda Console with the command

{{< highlight json >}}
npm install slack-node --save
{{< / highlight >}}

Then, create a ```index.js``` file in the folder and copy and paste the following.

{{< highlight js >}}
const Slack = require("slack-node");
// Load the AWS SDK for Node.js
const AWS = require("aws-sdk");
// Set the region
AWS.config.update({region: process.env.REGION});

// Create EC2 service object
var ec2 = new AWS.EC2();

function composeMsg(infos) {

    var msg = "";

    for (var info of infos) {
        msg += "Instance: "+info["name"]+" (id: "+info["id"]+", ip: "+info["ip"]+") has status "+info["status"]+"\n"+"PublicDNS: "+info["dns"]+"\n\n";
    }

    return msg;

}

function slackNotifier(infos) {

    var msg = composeMsg(infos);

    var webhookUri = process.env.SLACK_AWS_WEBHOOK;

    var slack = new Slack();
    slack.setWebhook(webhookUri);

    slack.webhook({
        channel: process.env.SLACK_AWS_CHANNEL,
        username: process.env.SLACK_AWS_BOT_NAME,
        text: msg
    }, function(err, response) {
        if (err) {
            console.log(err);
        }
    });

    return msg;

}

exports.handler = (event, context, callback) => {

    var params = {};

    ec2.describeInstances(params, function(err, data) {

        if (err) {
            console.log(err, err.stack);
        }
        else {
            var count = 0;
            var infos = [];

            data = JSON.parse(JSON.stringify(data));

            // for each instance
            for (var reservartion of data["Reservations"]) {

                var instance = reservartion["Instances"][0];
                infos.push({});

                // find id
                infos[count]["id"] = instance["InstanceId"];

                // find name
                for (var tag of instance["Tags"]) {
                    if (tag["Key"] == "Name") {
                        infos[count]["name"] = tag["Value"];
                        break;
                    }
                }

                // save status
                infos[count]["status"] = instance["State"]["Name"];

                // save dns and ip
                infos[count]["dns"] = instance["PublicDnsName"];
                infos[count]["ip"] = instance["PublicIpAddress"];

                count = count + 1;

            }

            console.log(infos);

            // send message
            var msg = slackNotifier(infos);
            callback(null, msg);
        }
    });
};
{{< / highlight >}}

The code simply calls AWS SDK to get informations - in particular, the instance name, id, status, public dns and ip, even if included in the addreess, to copy and past faster - about your running EC2 instances, parse and format them in a pretty format. Then, a message is sent to a private slack channel (see later). So...zip, upload. Done. Environment variable needed are the REGION (aws-region), SLACK_AWS_BOT_NAME (the name of your slack bot), SLACK_AWS_CHANNEL (the name of your channel) and SLACK_AWS_WEBHOOK (the web.. ok, you got it). You can complete missing env vars later: keep them empty.

There are two things to complete yet: 1) attach this rules to a Cloudwatch Event (of course, the change of state for your EC2 instances) and 2) effectively create a slack channel, a slack-app and corresponding webhook for the created channel. What do you want to do first? Don't worry, I decide for you: let's create a Slack channel.

#### Step 4/5: Create a Slack Channel (and App and so on)

_Preamble_: first of all: if you don't have a Slack account, sign up [here](https://slack.com). Then, create a work space (it shoule be easy by following instruction after the first access). Done? After that, you should be able to create a channel (I suggetst a private one if this is an experiment). Done? Ok. If you have problem to complete these steps, well... Actually, I can't help you: I don't remember, you have to google about them.

To create a Slack App, follow these steps:

- Go [here](https://api.slack.com/apps?new_app=1);
- Choose a name for your app, such as AWSStartAndStopBot, or whatever you want, then select the workspace (you should have at least one);

<div class="img_container"><img src="https://i.imgur.com/qNc50Fc.png" style="width: 100%; marker-top: -10px;"/></div>

- Click on the Create App button. You should be redirect to the page of your application a Add features and functionality section;

<div class="img_container"><img src="https://i.imgur.com/oA64Ipp.png" style="width: 100%; marker-top: -10px;"/></div>

- Click on the Incoming Weebhooks below, then setup to ON the switch button in right corner;

<div class="img_container"><img src="https://i.imgur.com/As2lD6c.png" style="width: 100%; marker-top: -10px;"/></div>

- Go to the bottom of the page and click on the Add new webhook to workspace button. You should be redirected to a page with a select box with a Post to label. Select the private channel created in the preamble, then click on Authorize button;

<div class="img_container"><img src="https://i.imgur.com/ZWlrfO0.png" style="width: 100%; marker-top: -10px;"/></div>

- You should be redirected to a previous page with the webhook url (you have to update your SLACK_AWS_WEBHOOK env variable for your AWS Lambda, the Node.js one);

#### Step 5/5: CloudWatch Event to sent Slack Message
The last step is create a CloudWatch Rule - a new one, keep the start and stop previously created rule untouched - triggered: this time, the trigger is not a Schedule event at fixed time, but a an Event Pattern.

- First, open [CloudWatch Console](https://console.aws.amazon.com/cloudwatch/), then click on __Rules__ on the left;
- Click on the blue button Create Rule, select Event Pattern, then select EC2 from Service Name select box;
- Click on EC2 Instance State-change Notification, then choose the involved state (you can exlude pending, if you want);
- On the left, click on Add trigger, then select the AWS Lambda (Node.js) function and configure input as matched event (it will be ignored);

<div class="img_container"><img src="https://i.imgur.com/F7Nlupl.png" style="width: 100%; marker-top: -10px;"/></div>

To test the setup, try to stop and start your instances, and you should see something appear on your slack channel!

<div class="img_container"><img src="https://i.imgur.com/siw2Tnw.png" style="width: 100%; marker-top: -10px;"/></div>

### Other possible scenario
I think I will work on Slack features like button to handle my VPC with a sort of question-answer event-driven bot. But...what if you want to add more worker node to Jenkins? For instance, using a 9-17 setup, you can have a three machine running at time.

<div class="img_container"><img src="https://i.imgur.com/RHaXqi4.png" style="width: 100%; marker-top: -10px;"/></div>

But I think this setup is more interesting :D. In the end, having a 4 node k8s cluster with a Jenkins for deploy, for 4 hours each day for one year....for free, is not so bad. I will try, I think performance are around 1980 :D

<div class="img_container"><img src="https://i.imgur.com/O5PHV4Z.png" style="width: 100%; marker-top: -10px;"/></div>

Thank you everybody for reading!

[^lambda]: You can find more information [here](https://aws.amazon.com/lambda/?nc1=h_ls)