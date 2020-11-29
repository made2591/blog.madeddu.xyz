---
layout: post
title: "JarvisButton: how to invoke multiple AWS Lambda with one AWS IoT Button (not Enterprise ed.)"
date: 2018-03-18
polly: https://madeddu.xyz/mp3/aws/aws-jarvis-button.mp3
categories:
- coding
- js
- aws
tags:
- coding
- aws
- iot
- button
- lambda
- slack
---

### Introduction
If you have an [AWS account](http://aws.amazon.com) in Free Tier, bla bla bla ok stop: I am a AWS Lambda maniac. I only wrote about them ([here](https://madeddu.xyz/posts/aws-lambda), [here](https://madeddu.xyz/posts/aws-step-functions)). In this article, I want to talk about my new purchase that is - of course - related to AWS Lambda: the AWS IoT Button. It first made its appearance on the IoT scene in October of 2015 at AWS re:Invent with the introduction of the AWS IoT service. That year all re:Invent attendees received the AWS IoT Button providing them the opportunity to get hands-on with AWS IoT. So cute. Since that time, AWS IoT button has been made broadly available to anyone interested in the clickable IoT device. Here it is! 😎😎😎

<div class="img_container"><img src="https://i.imgur.com/p1bno7t.jpg" style="width: 100%; marker-top: -10px;"/></div>

Sooooooo expensive :P

### Ingredients
You will need:
- [AWS account](http://aws.amazon.com) (free tier it's ok)
- [AWS Lambda](https://aws.amazon.com/en/lambda/)
- [Slack](https://slack.com)

### Recipe
As I already said in a preview post on AWS Services, I recommend you to pay a lot of attention. You always have to know exactly what are you doing, to avoid surprise in billing in the end of the month. Fortunately, there are a lot of documentations on Amazon official site, so you only have to read them.

#### One click => One Lambda
The first thing you have to learn with AWS IoT Button is that you can do anything special: you simply download the app, you click on the button and, in a bunch of seconds, AWS create certificates, call the NSA, setup something you will pay forever, remove movies from your Netflix to-view-list, destroy your VPC, recreate your VPC, and then let your mobile application associate the button with one of your (previously created) AWS Lambda. And that's all.

<div class="img_container"><img src="https://i.imgur.com/0xacnZz.jpg" style="width: 100%; marker-top: -10px;"/></div>

### Scenario
Ok, my idea of the IoT Button was different: first, I thought to able to handle three different clicks - this is not, but in the end it is my fault. I didn't read anything - I mean, literaly - before purchasing the AWS IoT Button. I didn't want, I hadn't time, I didn't want to break a 3 dollars dash button and I hadn't so much interest in this button. I was simply bored, I bought it, as most of us do.

Just to be clear: there is an Enterprise version of the button (more [here](https://aws.amazon.com/it/blogs/aws/introducing-the-aws-iot-button-enterprise-program/)) but I think I got the occasion to create a sort of my own version of this - eventually, even more efficient.

### Goal
Did you ever seen IronMan? If not, I don't know what do you live for: in any case, in the movie J.A.R.V.I.S - (_Just A Rather Very Intelligent System_) is the name given to the personal assistant - actually, it is only a voice - of R. Downey Jr, that plays the role of the famous superhero. J.A.R.V.I.S knows everything, understands everythings, I'm pretty sure that in one movie of the saga is able to bypass the Oracle Cloud (?!) as if it were the Accenture's VPC (just kidding Accenture guys). The goal is to create something able to handle more than one click: so, I thought to use time, the only thing that the button _pass_ to the AWS Lambda. The only thing you have to do is defining a sort of _alphabet_ - something very similar to a morse code. But, it can't be a real morse code, because you only have one type of click. Fortunately, this is enough to create - with elapsing of time - a Turing machine equivalent system. Or maybe not? I'm still thinking about it.

### Grammar
My grammar is simple: I want my button able to do three things:
- Tell me the weather forecast;
- Gather news for me;
- Provide me navigation button;

Of course, it could tell me about movies, VPC status, home status, etc. I don't want to link this JarvisButton to everything I wrote until now. As I was saying, one bit is exactly what we need to do what we want to do.

| Field        | Values          |
|--------------|-----------------|
| Weather      | 1               |
| News         | 2               |
| Navigation   | 3               |

The desired behaviour is: I want to know about weather. I click and set the counter. I wait for 3 seconds: I click to execute the action. Done. If I want to invoke news gathering, I click two times to set the counter. I wait for 3 seconds: I click to execute the action. Done.

### Implementation
First, create a DynamoDB Table - actually, you could also use an s3 bucket - because you have to store only 1 record on this table [what? yes, 1 record]. The record is composed by three field:

{{< highlight json >}}
{ "id": 1, "count": 0, "timestamp": 1521307832 }
{{< / highlight >}}

The logic beside this is really simple. Let be:
- ```now``` a variable containing the request Unix timestamp in seconds (math rounded);
- ```data``` a variable containing the record described above - so that ```data['timestamp']``` will be the Unix timestamp of the record;
- ```COUNTER_LIMIT``` and ```UPPER_LIMIT``` two constants - for instance, 3 and 10;

Thus, I click the button and...:
- If the difference between ```now``` (request time) and ```data['timestamp']``` is minor than the ```COUNTER_LIMIT```, increment the counter (and update the timestamp);
- If the difference between ```now``` (request time) and ```data['timestamp']``` is between the ```COUNTER_LIMIT``` and the ```UPPER_LIMIT```, execute the action associated to value of the counter, then reset the counter;
- If the difference between ```now``` (request time) and ```data['timestamp']``` is major than the ```UPPER_LIMIT```, reset the counter;

<span style="color:#ffcc00; font-size: bold;">NOTE</span>: there are also two precautions.
- first, the increment has to be _circular_, because if it is not your code could look for value of counter not associated with any of the action.
- second, you have to reset the value of the counter to 1, without executing anything if the count is zero (so, if the action has been executed) but you click your button again without reset-time elapsed yet.

I implemented a short version of the algorithm below, but you can have a look at the code - one single lambda (despite I use a previous lambda to get my news) [here](https://github.com/made2591/jarvis-button).

{{< highlight js >}}
function increment(data, now) {
    if (data['count'] >= Object.keys(actions).length) {
        data['count'] = 1;
        console.log("Restart (new val: "+data['count']+")");
    } else {
        data['count'] += 1;
        console.log("Increment (new val: "+data['count']+")");
    }
    data['timestamp'] = now;
    return data;
}

function execute(data, now) {
    console.log("Execute action "+data['count']);
    data['count'] = 0;
    data['timestamp'] = now;
    return data;
}

function reset(data, now) {
    console.log("Reset and restart count!");
    data['count'] = 1;
    data['timestamp'] = now;
    return data;
}

function logic() {

    var reqdate = new Date();
    reqdate.setHours(reqdate.getHours() + 1); // this is for UTC/timezone/whatever
    var now = Math.round(reqdate.getTime()/1000);

    if ((now - data['timestamp']) < COUNTER_LIMIT) {
        data = increment(data, now);
    }
    if ((now - data['timestamp']) >= COUNTER_LIMIT && (now - data['timestamp']) < process.env.UPPER_LIMIT) {
        if (data['count'] == 0) {
            data = reset(data, now);
        } else {
            actions[data['count']](now);
            data = execute(data, now);
        }
    }
    if ((now - data['timestamp']) >= UPPER_LIMIT) {
        data = reset(data, now);
    }
}
{{< / highlight >}}

With this logic, you can implement the number of action you want! Of course, you can use a s3 to handle the JSON record needed to handle multiple click.

### JarvisButton mouth
I used Slack as a mouth for my JarvisButton: each morning, and during the day, I can click my button and get exactly what I want 🤘🤘🤘 😎😎😎

<div class="img_container"><img src="https://i.imgur.com/RCX7TfO.png" style="width: 100%; marker-top: -10px;"/></div>

<div class="img_container"><img src="https://i.imgur.com/2LjLlrW.png" style="width: 100%; marker-top: -10px;"/></div>

<div class="img_container"><img src="https://i.imgur.com/9ySOla8.png" style="width: 100%; marker-top: -10px;"/></div>

If you search for inspiration, have a look at my github repo [here](https://github.com/made2591/jarvis-button).

### My advise
- Start by logging the update of the record in the AWS Lambda Console, then add the logic to execute action (I used a dict of function and I simply call them, eventually invoking other lambda;
- Augment the number of seconds between each click because the AWS IoT Button shuts down after the call and you will need more than three second to power it on again, make another call without losing your time window opportunity;

Github repo [here](https://github.com/made2591/jarvis-button).

Thank you everybody for reading!

[^lambda]: You can find more information [here](https://aws.amazon.com/lambda/?nc1=h_ls)