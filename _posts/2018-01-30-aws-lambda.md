---
title: "AWS Lambda, GoLang and Grafana to perform Sentiment Analysis"
tags: [coding, aws, lambda, golang, grafana, sentiment, analysis]
---

### Introduction
In this article I will talk about my experience with AWS Lambda + API Gateway, GoLang (of course) and Grafana to build a sentiment analysis tool over customizable topics. To better understand how to use Elasticsearch as a efficient database for information, read my previous post [Elasticsearch over My home Network Attached Storage](https://made2591.github.io/posts/elasticnas). If you don't know about Lambda, I personally believe that it represents one of the most interesting services currently offered by AWS: as they state, _AWS Lambda lets you run code without provisioning or managing servers_. You pay only for the compute time you consume - there is no charge when your code is not running. The amazing thing is that with a Free Tier trial you have 1 milions requests for free, to run code of any type of application or backend service - all with zero administration: you just upload your code - unfortunately the online editor for GoLang is not supported yet - and Lambda[^lambda] takes care of everything required to run and scale your code with high availability. You can even set up your code to automatically trigger from other AWS services or call it directly from any web or mobile app - as I have done with API Gateway. Ok, I'm writing this post because there is an amazing news: starting [15 January 2018](https://aws.amazon.com/it/blogs/compute/announcing-go-support-for-aws-lambda/), AWS Lambda support GoLang!!! Ingriedents after the image.

<p align="center"><img src="https://images.everyeye.it/img-notizie/tramite-gli-steam-awards-giocatori-chiedono-arrivo-half-life-3-v3-278335-1280x720.jpg" style="width: 100%; marker-top: -10px;"/></p>

### Ingredients 
For this article, you will need the following:
- A Grafana + Elasticsearch setup (wherever you want: if you want to run both of them locally, go [here](https://github.com/ftes/grafana-elasticsearch-docker);
- An [AWS account](http://aws.amazon.com) (free tier it's ok, but API Gateway is not included);
- Python or Bash to perform queries;
- A [Newsapi](http://newsapi.org) account to gather news from several sources (the free tier it's ok for our purpose);
- A [Aylien](http://aylien.com) account to do some sentiment analysis (the free tier it's ok for our purpose);

### Recipe
There are a lot of quite simple steps: the first one is that with AWS you have to pay attentions and you have to know exactly what are you doing. Fortunately, there are a lot of documentations on Amazon official site, so you only have to read them.

#### 1/7 Create an AWS Account
Create an AWS Account is simple, you only need to have a credit cart and no fear :D You can start from [here](http://aws.amazon.com). After the creation, I strongly suggest to study a little more how the IAM Roles work. After you have created your account, you can start from the [IAM Dashboard](https://console.aws.amazon.com/iam/) by following the 5 points to ensure your account is secured. The steps consist in practise in:
- Create a secondary user with admin rights, possibly MFA (I use [2stp](http://thomasrzhao.com/2stp-support/) even if it is not supported anymore, because it works and it includes what I need - and nothing more - from a 2-step virtual authenticator device);
- Create IAM password policy and start to understand the use of groups to assign permissions;

#### 2/7 Create Newsapi and Aylien Account
These services are replaceable with any other service you want to lambd-_ize_. I chose the first one because it is really a great service for aggregating news and highlights from different sources. I chose the second one because the service simply work for my trial purpose, but I would like to compare it with Google Cloud Platform and Microsoft Azure as soon as possible, and I already have the idea of who will be the winner (spoiler: in my opinion it is not Google).
Start [here](http://newsapi.org) for news and [here](http://aylien.com) for sentiment.

#### 3/7 Build News gatherer over AWS Lambda
Lambda currently supports different languages: C#, Java, Node.js, Python and now Go. First of all, you need to know how to write code: online editor is not supported yet so you will have to write your lambda offline. A Lambda ready GoLang file is a single ```.go``` file with a function, the ```handler``` and a ```main``` function to link the handler function to the lambda. And that's all. The only dependencies you need to install, if you want to run your lambda locally, is the ```aws-lambda-go``` sdk provided by Amazon and available on Github.

{% highlight sh %}
go get github.com/aws/aws-lambda-go/lambda"
{% endhighlight %}

{% highlight go %}
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
{% endhighlight %}

I also found [this](https://github.com/lambci/docker-lambda) beautiful docker image that let you test your lambda (and support also GoLang) with a single docker run. You can pass the parameters as a string (payload requests), as shown in the method above: of course, you first have to compile your lambda for linux.

{% highlight sh %}

GOOS=linux go build -o MyCompiledLambda MyLambda.go

docker run --rm -v $PWD:/var/task lambci/lambda:go1.x MyCompiledLambda '{"parameter": "value"}'

{% endhighlight %}

To upload your Lambda in AWS, in the creation steps specify you want to upload a Go 1.x Lambda, then zip your build (in the example, ```MyCompiledLambda```) and upload from an S3 bucket or manually. 

{% highlight sh %}

zip MyCompiledLambda.zip MyCompiledLambda

{% endhighlight %}

__NOTE__: the most important things is to setup the handler name to the name of the compiled binary inside your zip - exactly the same name. Otherwise, during testing a path error will be arised because AWS will look for the wrong file name to run your lambda. 

As you can see from the code above, there are environment variables to setup API KEY and API Endpoint (whatever your want). From the Lambda setup page you can setup this environment variable to let your code gather the information from AWS, in a secure way.

After you have succesfully setup your lambda with the right execution role (have a look at the documentation step, or follow the wizard to automatically create an execution role), you can test your Lambda configuring and using the test menu near to the save button in the right corner of the page. You can click on create your test (they will be available for each lambda separately), you can specify the same payload - you passed before as parameter - to the lambda as a request payload in the editor - using json format. 

#### 4/7 Build Sentiment analyzer over AWS Lambda
My Sentiment Lambda code is below:

{% highlight go %}
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
{% endhighlight %}

#### 5/7 Setup API Gateway
First, you need to create an API Endpoint. This is simple, you have to go [here](https://console.aws.amazon.com/apigateway/) and click on "Create API" button. After that, you have to create a Resource clicking on the action menu and specifying your api endpoint. Then click on "Create Resource". You can now create your "Action Method": as integration type choose "Lambda Function", then specify the region you deployed your lambda and the Lambda function (it should appear). When you will click on create, a popup will appear to warn you that the action will setup the 

#### 6/7 Fill you Elasticsearch

#### 7/7 Setup your Grafana Dashboard

### Conclusion

[^lambda]: You can find more information [here](https://aws.amazon.com/lambda/?nc1=h_ls)