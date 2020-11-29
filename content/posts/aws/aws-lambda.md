---
layout: post
title: "AWS Lambda, GoLang and Grafana to perform sentiment analysis for your company / business"
date: 2018-01-30
polly: https://madeddu.xyz/mp3/aws/aws-lambda.mp3
categories:
- coding
- golang
- aws
tags:
- coding
- aws
- lambda
- golang
- grafana
- sentiment
- analysis
---

### Introduction
In this article I will talk about my experience with AWS Lambda + API Gateway, GoLang (of course) and Grafana to build a sentiment analysis tool over customizable topics. Who should you read this post? Don't know, maybe a CIO, a CTO, a CEO, a generic Chief or a MasterChef, for sure an AWS and GoLang fan like me. First of all: to better understand how to use Elasticsearch, read my previous post [Elasticsearch over My home Network Attached Storage](https://madeddu.xyz/posts/elasticnas): it's not so exciting as it seems, but you will have a general idea about what is Elasticsearch and how can you use it. Second: if you don't know about AWS Lambda, study it. I personally believe that it represents one of the most interesting services currently offered by AWS: as they state, _AWS Lambda lets you run code without provisioning or managing servers_. You pay only for the compute time you consume and there is no charge when your code is not running. The amazing thing is that with a Free Tier trial you have 1 milions requests for free - O.O - to run code of any type of application or backend service - all with zero administration: you just upload your code - unfortunately the online editor for GoLang is not supported yet - and AWS Lambda[^lambda] takes care of everything required to run and scale your code with high availability. You can even set up your code to automatically trigger from other AWS services - as I have done with API Gateway - or call it directly from any web or mobile app. And...last but definetly not the least, why I'm writing this post!? Because starting from [15 January 2018](https://aws.amazon.com/it/blogs/compute/announcing-go-support-for-aws-lambda/), AWS Lambda support GoLang!!!

Ingredients after the image.

<div class="img_container"><img src="https://i.imgur.com/lDDWtbW.jpg" style="width: 100%; marker-top: -10px;"/></div>

### Ingredients
For this article, you will need the following:
- A Grafana + Elasticsearch setup (wherever you want: if you want to run both of them locally, go [here](https://github.com/ftes/grafana-elasticsearch-docker);
- An [AWS account](http://aws.amazon.com) (free tier it's ok, but API Gateway is not included);
- Python or Bash to perform queries;
- A [Newsapi](http://newsapi.org) account to gather news from several sources (the free tier it's ok for our purpose);
- A [Aylien](http://aylien.com) account to do some sentiment analysis (the free tier it's ok for our purpose);

### Recipe
There are a lot of quite simple steps. I recommend you to pay a lot of attention with AWS. You always have to know exactly what are you doing, to avoid surprise in billing in the end of the month. Fortunately, there are a lot of documentations on Amazon official site, so you only have to read them.

#### 1/7 Create an AWS Account
Create an AWS Account is simple, you only need to have a credit cart and no fear of Amazon (what?!) :D You can start from [here](http://aws.amazon.com). After the creation, I strongly suggest to study a little more how the IAM Roles work. After you have created your account, you can start from the [IAM Dashboard](https://console.aws.amazon.com/iam/) by following the 5 points to ensure your account is secured. I am talking about

<div class="img_container"><img src="https://i.imgur.com/0M8vfbi.png" style="width: 100%; marker-top: -10px;"/></div>

Following those steps you guarantee - in practise - to:
- Create a secondary user with admin rights, possibly with MFA enabled (I use [2stp](http://thomasrzhao.com/2stp-support/) even if it is not supported anymore, because it works and it includes what I need - and nothing more - from a 2-step virtual authenticator device);
- Create IAM password policy and start to understand the use of groups to assign permissions;

#### 2/7 Create Newsapi and Aylien Account
These services are replaceable with any other service you want to lambd-_ize_. I chose the first one because it is really a great service for aggregating news and highlights from different sources. I chose the second one because the service simply work for my trial purpose, but I would like to compare it with Google Cloud Platform and Microsoft Azure as soon as possible, and I already have the idea of who will be the winner (spoiler: in my opinion it is not Google).
Start [here](http://newsapi.org) for news and [here](http://aylien.com) for sentiment.

#### 3/7 Build News gatherer over AWS Lambda
Lambda currently supports different languages: C#, Java, Node.js, Python and now Go. First of all, you need to know how to write code: online editor is not supported yet so you will have to write your lambda offline. A Lambda ready GoLang file is a single ```.go``` file with a function, the ```handler``` and a ```main``` function to link the handler function to the lambda. And that's all. The only dependencies you need to install, if you want to run your lambda locally, is the ```aws-lambda-go``` sdk provided by Amazon and available on Github.

{{< highlight sh >}}
go get github.com/aws/aws-lambda-go/lambda
{{< / highlight >}}

{{< highlight go >}}
package main

import (
	"os"
	"fmt"
	"time"
	"errors"
	"strconv"
	"net/http"
	"io/ioutil"
	"encoding/json"

	"github.com/aws/aws-lambda-go/lambda"
)

var (
	API_KEY      = os.Getenv("API_KEY")
	API_URL      = os.Getenv("API_URL")
	API_DATE_FMT = os.Getenv("API_DATE_FMT")
	ErrorBackend = errors.New("Something went wrong")
)

type Request struct {
	Q *string 		 		`json:"q"`
	Sources *string  		`json:"sources"`  // comma separated https://newsapi.org/sources
	Domains *string  		`json:"domains"`  // comma separated https://newsapi.org/domains
	From *string     		`json:"from"`
	To *string       		`json:"to"`
	Language *string 		`json:"language"`
	SortBy *string   		`json:"sortBy"`
	PageSize *int    		`json:"pageSize"`
	Page *int   	 		`json:"page"`
}

type NewsApiResponse struct {
	Status string   		`json:"status"`
	News []News 			`json:"articles"`
}

type Source struct {
	ID   string 			`json:"id"`
	Name string 			`json:"name"`
}

type News struct {
	Source	  	Source      `json:"source"`
	Author    	string		`json:"author"`
	Title     	string		`json:"title"`
	Description string  	`json:"description"`
	URL       	string		`json:"url"`
	Image     	string 		`json:"urlToImage"`
	Published 	time.Time	`json:"publishedAt"`
}

func GatherRecentNewsAboutTopic(request Request) ([]News, error) {

	// concat api key
	url := fmt.Sprintf(API_URL, API_KEY)

	// create client to ask for apis
	client := &http.Client{}

	// error in external source
	req, err := http.NewRequest("GET", url, nil)
	if err != nil {
		return []News{}, ErrorBackend
	}

	// parse request parameters
	if request.Q != nil {
		p := req.URL.Query()
		p.Add("q", *request.Q)
		req.URL.RawQuery = p.Encode()
	}

	// parse request source
	if request.Sources != nil {
		p := req.URL.Query()
		p.Add("sources", *request.Sources)
		req.URL.RawQuery = p.Encode()
	}

	// parse request domains
	if request.Domains != nil {
		p := req.URL.Query()
		p.Add("domains", *request.Domains)
		req.URL.RawQuery = p.Encode()
	}

	// parse request from
	if request.From != nil {
		p := req.URL.Query()
		if t, err := time.Parse(API_DATE_FMT, *request.From); err != nil {
			p.Add("from", t.Format(API_DATE_FMT))
			req.URL.RawQuery = p.Encode()
		}
	}

	// parse request to
	if request.To != nil {
		p := req.URL.Query()
		if t, err := time.Parse(API_DATE_FMT, *request.To); err != nil {
			p.Add("to", t.Format(API_DATE_FMT))
			req.URL.RawQuery = p.Encode()
		}
	}

	// parse request language
	if request.Language != nil {
		p := req.URL.Query()
		p.Add("language", *request.Language)
		req.URL.RawQuery = p.Encode()
	}

	// parse request sort by
	if request.SortBy != nil {
		p := req.URL.Query()
		p.Add("sortBy", *request.SortBy)
		req.URL.RawQuery = p.Encode()
	}

	// parse request page size
	if request.PageSize != nil {
		p := req.URL.Query()
		p.Add("pageSize", strconv.Itoa(*request.PageSize))
		req.URL.RawQuery = p.Encode()
	}

	// parse request page
	if request.Page != nil {
		p := req.URL.Query()
		p.Add("page", strconv.Itoa(*request.Page))
		req.URL.RawQuery = p.Encode()
	}

	// debug
	fmt.Println(req.URL)

	// make request and defer response
	resp, err := client.Do(req)
	if err != nil {
		return []News{}, ErrorBackend
	}
	defer resp.Body.Close()

	// parse news results
	var data NewsApiResponse
	if err := json.NewDecoder(resp.Body).Decode(&data); err != nil {
		return []News{}, ErrorBackend
	}

	// return response
	return data.News, nil

}

// handle request
func main() {

	// handle request
	lambda.Start(GatherRecentNewsAboutTopic)

}
{{< / highlight >}}

I also found [this](https://github.com/lambci/docker-lambda) beautiful docker image that let you test your lambda (and support also GoLang) with a single docker run. You can pass the parameters as a string (payload requests), as shown in the method above: of course, you first have to compile your lambda for linux.

{{< highlight sh >}}

GOOS=linux go build -o MyCompiledLambda MyLambda.go

docker run --rm -v $PWD:/var/task lambci/lambda:go1.x MyCompiledLambda '{"parameter": "value"}'

{{< / highlight >}}

To upload your Lambda in AWS, in the creation steps specify you want to upload a Go 1.x Lambda, then zip your build (in the example, ```MyCompiledLambda```)

{{< highlight sh >}}

zip MyCompiledLambda.zip MyCompiledLambda

{{< / highlight >}}

and upload from an S3 bucket or manually.

<div class="img_container"><img src="https://i.imgur.com/BdIDvDr.png" style="width: 100%; marker-top: -10px;"/></div>

__NOTE__: the most important things is to setup the handler name to the name of the compiled binary inside your zip - exactly the same name. Otherwise, during testing a path error will be arised because AWS will look for the wrong file name to run your lambda.

As you can see from the code above, there are environment variables to setup API KEY and API Endpoint (whatever your want). From the Lambda setup page you can setup this environment variable to let your code gather the information from AWS, in a secure way.

<div class="img_container"><img src="https://i.imgur.com/bkDyH0E.png" style="width: 100%; marker-top: -10px;"/></div>

After you have succesfully setup your lambda with the right execution role (have a look at the documentation step, or follow the wizard to automatically create an execution role), you can test your Lambda configuring and using the test menu near to the save button in the right corner of the page. You can click on create your test (they will be available for each lambda separately), you can specify the same payload - you passed before as parameter - to the lambda as a request payload in the editor - using json format.

<div class="img_container"><img src="https://i.imgur.com/hkgoLJU.png" style="width: 100%; marker-top: -10px;"/></div>

#### 4/7 Build Sentiment analyzer over AWS Lambda
I build a second AWS Lambda to create a sentiment analyzer that take advantage of free tier plan kindly granted by [Aylien Team](http://aylien.com). The code below:

{{< highlight go >}}
package main

import (
	"errors"
	"net/http"
	"os"

	"github.com/AYLIEN/aylien_textapi_go"
	"github.com/aws/aws-lambda-go/lambda"
)

var (
	AYLIEN_API_URL                = os.Getenv("AYLIEN_API_URL")
	AYLIEN_API_KEY                = os.Getenv("AYLIEN_API_KEY")
	AYLIEN_API_ID                 = os.Getenv("AYLIEN_API_ID")
	SentimentAnalysisErrorBackend = errors.New("Something went wrong")
)

type Request struct {
	Text     *string `json:"text"`
	Url      *string `json:"url"`
	Mode     *string `json:"mode"`
	Language *string `json:"language"`
}

func SentimentAnalysisOverRequest(request Request) (textapi.SentimentResponse, error) {

	// concat api key
	auth := textapi.Auth{AYLIEN_API_ID, AYLIEN_API_KEY}

	// error in external source
	_, err := http.NewRequest("GET", AYLIEN_API_URL, nil)
	if err != nil {
		return textapi.SentimentResponse{}, SentimentAnalysisErrorBackend
	}

	// create client to ask for apis
	client, err := textapi.NewClient(auth, true)

	// error in external source
	if err != nil {
		panic(err)
	}

	text := ""
	url := ""
	mode := ""
	lang := "auto"

	// parse request parameters
	if request.Text != nil {
		text = *request.Text
	}
	if request.Url != nil {
		url = *request.Url
	}
	if request.Mode != nil {
		mode = *request.Mode
	}
	if request.Language != nil {
		lang = *request.Language
	}

	// create sentiment parameters
	sentimentParams := &textapi.SentimentParams{Text: text, URL: url, Language: lang, Mode: mode}

	// return sentiment
	sentiment, err := client.Sentiment(sentimentParams)

	if err != nil {
		return textapi.SentimentResponse{}, SentimentAnalysisErrorBackend
	}

	// return response
	return *sentiment, nil

}

// handle request
func main() {

	// handle request
	lambda.Start(SentimentAnalysisOverRequest)

}
{{< / highlight >}}

#### 5/7 Setup API Gateway
First, you need to create an API Endpoint. This is simple, you have to go [here](https://console.aws.amazon.com/apigateway/) and click on "Create API" button.

<div class="img_container"><img src="https://i.imgur.com/Ecmgrmj.png" style="width: 100%; marker-top: -10px;"/></div>

After that, you have to create a Resource clicking on the action menu and specifying your api endpoint. Then click on "Create Resource".

<div class="img_container"><img src="https://i.imgur.com/A2xFlC5.png" style="width: 100%; marker-top: -10px;"/></div>

You can now create your "Action Method": as integration type choose "Lambda Function", then specify the region you deployed your lambda and the Lambda function (it should appear).

<div class="img_container"><img src="https://i.imgur.com/AP871Qn.png" style="width: 100%; marker-top: -10px;"/></div>

When you will click on create, a popup will appear to warn you that the action will setup the execution role for the API Gateway service - don't remember exactly the step, eventually you can create your onw policy for API Gateway following the [Build an API Gateway API with Lambda Integration](https://docs.aws.amazon.com/apigateway/latest/developerguide/getting-started-with-lambda-integration.html) tutorial.

After that, I strongly suggest you to set up an api key and a usage plan to your api - it's really simple, from the left menu shown the Create-API-focused-image you can do that. This is to prevent consume your API from stranger (they will be available over the Internet).

#### 6/7 Fill you Elasticsearch
Ok, now it's time to fill your elasticsearch of news. I will assume you have a dump of the news about the topic you want to sentimentanalyze in a json file called ```"localDump.json"```. If you don't have, you can cUrl like a PRO and dump to file. Then, I used Python to build a reasonable index of fake data - of course, you have to call your AWS Lambda function, to _sentiment_ (nice, the new _to sentiment_ expression, or not?!) your news but I'm poor, and I didn't want to make DOS Attack to my own VPC. I simulate because of money lack: I'm afraid of Amazon.

{{< highlight py >}}
polarity     = ["neutral", "neutral", "negative"]
subjectivity = ["subjective", "objective"]

# read dump
localSource = json.load(open("localDump.json", "r"))

newsCleaned = []
newsHashing = []

# local cleaning
for news in localSource:

	if hashlib.sha256(news["description"].encode('utf-8')).hexdigest() not in newsHashing:
		newsHashing.append(hashlib.sha256(news["description"].encode('utf-8')).hexdigest())
		news["id"] = hashlib.sha256(news["description"].encode('utf-8')).hexdigest()
		newsCleaned.append(news)

print len(localSource), len(newsCleaned)

for news in newsCleaned:
	news["text"] = "document"
	news["polarity"] = polarity[randint(0, len(polarity)-1)]
	news["subjectivity"] = subjectivity[randint(0, len(subjectivity)-1)]
	news["polarity_confidence"] = random.uniform(0, 1)
	news["subjectivity_confidence"] = random.uniform(0, 1)

with open("elasticReady.json", "w") as f:
	for news in newsCleaned:
		f.write("{\"index\":{\"_id\":\""+news["id"]+"\"}}\n")
		f.write(json.dumps(news)+"\n")

{{< / highlight >}}

What does this code do? It's very simple: it loops over the dump of the news - don't forget that the format of each news is the one defined by the ```News``` structure in the code of the first AWS Lambda function. In the first loop, duplicates are removed, if the paginated queries - look at the ```Request``` wrapper struct in the second AWS Lambda - have been returned. Then, it simulates a Sentiment evaluation query to the second AWS Lambda and a _merge_ operation over dict key (the News object), with the informations that AWS Lambda currently returns: the polarity and subjectivity of the article, with relative confidences. In the end, a file is created with an identified - hash on the article extract - to be uploaded to Elasticsearh... how? With this unique call - (of course you have to run first a docker container / ec2 instance / any-kind-of Elasticsearch node). Locally:

{{< highlight sh >}}
curl -H "Content-Type: application/json" -XPOST "localhost:9200/news/sentimented/_bulk?pretty&refresh" --data-binary "@elasticReady.json"
{{< / highlight >}}

#### 7/7 Setup your Grafana Dashboard
And this is the most exiting part: with grafana you can setup Elasticsearch as Datasource (don't need to explain this, simply fill the host field). Then, playing with some graphs and lucene queries / aggregation, you can create for instance an Heat map that shows how much bad or good are the feedback of the topic you are looking for. You can discover who is the top influencer in term of _how much it talks about the topic_, or simply show the number of neutral / negative feedback from highlights and setup an alarm if they reach a huge number - this is only an idea. Look at my Grafana dashboard ^^

<div class="img_container"><img src="https://i.imgur.com/UuSj8Fl.png" style="width: 100%; marker-top: -10px;"/></div>

Thank you everybody for reading!

[^lambda]: You can find more information [here](https://aws.amazon.com/lambda/?nc1=h_ls)