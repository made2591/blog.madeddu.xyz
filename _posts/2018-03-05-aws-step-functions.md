---
title: "AWS Step Functions to collect sentimented movie review"
tags: [coding, aws, tmdb, news, step, functions, lambda, serverless, sentiment, analysis]
---

### Introduction
Recently I worked with AWS Lambda and API Gateway to extend my set of personal APIs and collect information from several sources. I wrote an article on that (if you want to [have a look](TODO)). In this article I will talk about the AWS Step Functions service that enable create finite states machines to easy coordinate the components of distributed applications and microservices using visual workflows. Why AWS Step Functions? Because they let me create a tool to gather movie titles in teather, search for reviews about each of them and make a basic sentiment analysis over the review to help me decide what's worth watching at teather and what's worth waiting for on Netflix :D
More in general, with AWS Step Functions, you can build applications made of individual components that each perform a discrete function: this lets you scale and change applications quickly. Step Functions is a reliable way to coordinate components and step through the functions of your application. They provides a graphical console to arrange and visualize the components of your application as a series of steps. This makes it simple to build and run multistep applications. Step Functions automatically triggers and tracks each step, and retries when there are errors, so your application executes in order and as expected. Step Functions logs the state of each step, so when things do go wrong, you can diagnose and debug problems quickly.

<p align="center"><img src="https://images.everyeye.it/img-notizie/tramite-gli-steam-awards-giocatori-chiedono-arrivo-half-life-3-v3-278335-1280x720.jpg" style="width: 100%; marker-top: -10px;"/></p>

### Ingredients 
For this article, you will need the following:
- An [AWS account](http://aws.amazon.com) (free tier it's ok, but API Gateway is not included);
- Python or Bash to perform queries;
- A [Newsapi](http://newsapi.org) account to gather news from several sources (the free tier it's ok for our purpose);
- A [Aylien](http://aylien.com) account to do some sentiment analysis (the free tier it's ok for our purpose);
- [AWS Lambda](TODO) to wrap services around the world to ghater and process data;
- [DynamoDB](TODO) to both optmize requests and persist processed data;
- [AWS Step Functions](TODO), of course, to create the workflow, orchestrate lambdas and work on the data flow;

### Recipe
As I already said in a preview post on AWS Services, there are a lot of quite simple steps. I recommend you to pay a lot of attention with AWS. You always have to know exactly what are you doing, to avoid surprise in billing in the end of the month. Fortunately, there are a lot of documentations on Amazon official site, so you only have to read them.

#### 1-2-3-4 Steps
The first four steps are equal to the one described [here]().

#### 5 - Build MovieDB collector over AWS Lambda
Lambda currently supports different languages: C#, Java, Node.js, Python and now Go. I wrote a GoLang wrapper around the API exposed by TheMovieDB [here](TODO): I am working on a front-end for this workflow on Angular, so I created a parametric to wrap almost all the routes exposed by TheMovieDB APIs and be able to fill my front-end in the future. AWS Lambda ready GoLang file is a single ```.go``` file with a function, the ```handler``` and a ```main``` function to link the handler function to the lambda. And that's all. The only dependencies you need to install, if you want to run your lambda locally, is the ```aws-lambda-go``` sdk provided by Amazon and available on Github.

{% highlight sh %}
go get github.com/aws/aws-lambda-go/lambda
{% endhighlight %}

{% highlight go %}
package main

import (
	"os"
	"fmt"
	"errors"
	"strings"
	"net/http"
	"encoding/json"
	"github.com/aws/aws-lambda-go/lambda"
)

var (
	API_KEY      = os.Getenv("API_KEY")
	API_BASE_URL = os.Getenv("API_BASE_URL")
	AWS_API_KEY  = os.Getenv("AWS_API_KEY")
	ErrorBackend = errors.New("Something went wrong")
)

type Request struct {
	Url 					string		`json:"api_url"`
	AwsApiGatewayKey 		string		`json:"aws_api_gateway_key"`

	ExternalID 				*string		`json:"external_id"`
	ExternalSource 			*string		`json:"external_source"`

	Query 					*string		`json:"query"`

	ApiKey 					*string 	`json:"api_key"`
	Language 				*string 	`json:"language"`
	Region 					*string 	`json:"region"`
	SortBy 					*string 	`json:"sort_by"`
	CertificationCountry 	*string 	`json:"certification_country"`
	Certification 			*string 	`json:"certification"`
	CertificationLTE 		*string 	`json:"certification.lte"`
	IncludeAdult			*string 	`json:"include_adult"`
	IncludeVideo 			*string 	`json:"include_video"`
	Page 					*string 	`json:"page"`
	PrimaryReleaseYear 		*string 	`json:"primary_release_year"`
	PrimaryReleaseDateGTE 	*string 	`json:"primary_release_date.gte"`
	PrimaryReleaseDateLTE	*string 	`json:"primary_release_date.lte"`
	ReleaseDateGTE 			*string 	`json:"release_date.gte"`
	ReleaseDateLTE 			*string 	`json:"release_date.lte"`
	VoteCountGTE 			*string 	`json:"vote_count.gte"`
	VoteCountLTE 			*string 	`json:"vote_count.lte"`
	VoteAverageGTE 			*string 	`json:"vote_average.gte"`
	VoteAverageLTE 			*string 	`json:"vote_average.lte"`
	WithCast 				*string 	`json:"with_cast"`
	WithCrew 				*string 	`json:"with_crew"`
	WithCompanies 			*string 	`json:"with_companies"`
	WithGenres 				*string 	`json:"with_genres"`
	WithKeywords 			*string 	`json:"with_keywords"`
	WithPeople 				*string 	`json:"with_people"`
	Year 					*string 	`json:"year"`
	WithoutGenres 			*string 	`json:"without_genres"`
	WithRuntimeGTE 			*string 	`json:"with_runtime.gte"`
	WithRuntimeLTE 			*string 	`json:"with_runtime.lte"`
	WithReleaseType 		*string 	`json:"with_release_type"`
	WithOriginalLanguage 	*string 	`json:"with_original_language"`
	WithoutKeywords 		*string 	`json:"without_keywords"`
}

type MovieDBResponse struct {
	Page 					int 	`json:"page"`
	Results 				[]Movie `json:"results"`
	TotalResults			int 	`json:"total_results"`
	TotalPages 				int 	`json:"total_pages"`
	HasElements 			bool    `json:"has_elements"`
}

type Movie struct {
	Cover 					string 	`json:"poster_path"`
	PosterPath 				string 	`json:"poster_path"`
	Adult 					bool 	`json:"adult"`
	Overview 				string 	`json:"overview"`
	ReleaseDate 			string 	`json:"release_date"`
	GenreIDs 				[]int 	`json:"genre_ids"`
	ID 						int 	`json:"id"`
	OriginalTitle 			string 	`json:"original_title"`
	OriginalLanguage 		string 	`json:"original_language"`
	Title 					string 	`json:"title"`
	BackdropPath 			string 	`json:"backdrop_path"`
	Popularity 				float32 `json:"popularity"`
	VoteCount 				int 	`json:"vote_count"`
	Video 					bool 	`json:"video"`
	VoteAverage 			float32 `json:"vote_average"`
}

func Handler(request Request) (MovieDBResponse, error) {

	if request.Url == "" || request.AwsApiGatewayKey == "" || strings.Compare(request.AwsApiGatewayKey, AWS_API_KEY) != 0 {
		return MovieDBResponse{}, errors.New("Missing one or two required parameters: 'api_url, aws_api_gateway_key'")
	}

	url := fmt.Sprintf("%s%s?api_key=%s", API_BASE_URL, request.Url, API_KEY)

	client := &http.Client{}

	req, err := http.NewRequest("GET", url, nil)
	if err != nil {
		return MovieDBResponse{}, ErrorBackend
	}

	if request.CertificationCountry != nil {
		q := req.URL.Query()
		q.Add("certification_country", *request.CertificationCountry)
		req.URL.RawQuery = q.Encode()
	}

	if request.VoteAverageLTE != nil {
		q := req.URL.Query()
		q.Add("vote_average.lte", *request.VoteAverageLTE)
		req.URL.RawQuery = q.Encode()
	}

	if request.WithoutGenres != nil {
		q := req.URL.Query()
		q.Add("without_genres", *request.WithoutGenres)
		req.URL.RawQuery = q.Encode()
	}

	if request.WithCast != nil {
		q := req.URL.Query()
		q.Add("with_cast", *request.WithCast)
		req.URL.RawQuery = q.Encode()
	}

	if request.Language != nil {
		q := req.URL.Query()
		q.Add("language", *request.Language)
		req.URL.RawQuery = q.Encode()
	}

	if request.WithRuntimeLTE != nil {
		q := req.URL.Query()
		q.Add("with_runtime.lte", *request.WithRuntimeLTE)
		req.URL.RawQuery = q.Encode()
	}

	if request.Region != nil {
		q := req.URL.Query()
		q.Add("region", *request.Region)
		req.URL.RawQuery = q.Encode()
	}

	if request.WithCrew != nil {
		q := req.URL.Query()
		q.Add("with_crew", *request.WithCrew)
		req.URL.RawQuery = q.Encode()
	}

	if request.WithReleaseType != nil {
		q := req.URL.Query()
		q.Add("with_release_type", *request.WithReleaseType)
		req.URL.RawQuery = q.Encode()
	}

	if request.IncludeAdult != nil {
		q := req.URL.Query()
		q.Add("include_adult", *request.IncludeAdult)
		req.URL.RawQuery = q.Encode()
	}

	if request.CertificationLTE != nil {
		q := req.URL.Query()
		q.Add("certification.lte", *request.CertificationLTE)
		req.URL.RawQuery = q.Encode()
	}

	if request.WithKeywords != nil {
		q := req.URL.Query()
		q.Add("with_keywords", *request.WithKeywords)
		req.URL.RawQuery = q.Encode()
	}

	if request.IncludeVideo != nil {
		q := req.URL.Query()
		q.Add("include_video", *request.IncludeVideo)
		req.URL.RawQuery = q.Encode()
	}

	if request.PrimaryReleaseDateGTE != nil {
		q := req.URL.Query()
		q.Add("primary_release_date.gte", *request.PrimaryReleaseDateGTE)
		req.URL.RawQuery = q.Encode()
	}

	if request.WithoutKeywords != nil {
		q := req.URL.Query()
		q.Add("without_keywords", *request.WithoutKeywords)
		req.URL.RawQuery = q.Encode()
	}

	if request.Page != nil {
		q := req.URL.Query()
		q.Add("page", *request.Page)
		req.URL.RawQuery = q.Encode()
	}

	if request.VoteAverageGTE != nil {
		q := req.URL.Query()
		q.Add("vote_average.gte", *request.VoteAverageGTE)
		req.URL.RawQuery = q.Encode()
	}

	if request.ReleaseDateLTE != nil {
		q := req.URL.Query()
		q.Add("release_date.lte", *request.ReleaseDateLTE)
		req.URL.RawQuery = q.Encode()
	}

	if request.ApiKey != nil {
		q := req.URL.Query()
		q.Add("api_key", *request.ApiKey)
		req.URL.RawQuery = q.Encode()
	}

	if request.WithRuntimeGTE != nil {
		q := req.URL.Query()
		q.Add("with_runtime.gte", *request.WithRuntimeGTE)
		req.URL.RawQuery = q.Encode()
	}

	if request.WithGenres != nil {
		q := req.URL.Query()
		q.Add("with_genres", *request.WithGenres)
		req.URL.RawQuery = q.Encode()
	}

	if request.Certification != nil {
		q := req.URL.Query()
		q.Add("certification", *request.Certification)
		req.URL.RawQuery = q.Encode()
	}

	if request.ReleaseDateGTE != nil {
		q := req.URL.Query()
		q.Add("release_date.gte", *request.ReleaseDateGTE)
		req.URL.RawQuery = q.Encode()
	}

	if request.VoteCountLTE != nil {
		q := req.URL.Query()
		q.Add("vote_count.lte", *request.VoteCountLTE)
		req.URL.RawQuery = q.Encode()
	}

	if request.WithOriginalLanguage != nil {
		q := req.URL.Query()
		q.Add("with_original_language", *request.WithOriginalLanguage)
		req.URL.RawQuery = q.Encode()
	}

	if request.PrimaryReleaseYear != nil {
		q := req.URL.Query()
		q.Add("primary_release_year", *request.PrimaryReleaseYear)
		req.URL.RawQuery = q.Encode()
	}

	if request.WithCompanies != nil {
		q := req.URL.Query()
		q.Add("with_companies", *request.WithCompanies)
		req.URL.RawQuery = q.Encode()
	}

	if request.WithPeople != nil {
		q := req.URL.Query()
		q.Add("with_people", *request.WithPeople)
		req.URL.RawQuery = q.Encode()
	}

	if request.ExternalID != nil {
		q := req.URL.Query()
		q.Add("external_id", *request.ExternalID)
		req.URL.RawQuery = q.Encode()
	}

	if request.SortBy != nil {
		q := req.URL.Query()
		q.Add("sort_by", *request.SortBy)
		req.URL.RawQuery = q.Encode()
	}

	if request.Year != nil {
		q := req.URL.Query()
		q.Add("year", *request.Year)
		req.URL.RawQuery = q.Encode()
	}

	if request.ExternalSource != nil {
		q := req.URL.Query()
		q.Add("external_source", *request.ExternalSource)
		req.URL.RawQuery = q.Encode()
	}

	if request.Query != nil {
		q := req.URL.Query()
		q.Add("query", *request.Query)
		req.URL.RawQuery = q.Encode()
	}

	if request.PrimaryReleaseDateLTE != nil {
		q := req.URL.Query()
		q.Add("primary_release_date.lte", *request.PrimaryReleaseDateLTE)
		req.URL.RawQuery = q.Encode()
	}

	if request.VoteCountGTE != nil {
		q := req.URL.Query()
		q.Add("vote_count.gte", *request.VoteCountGTE)
		req.URL.RawQuery = q.Encode()
	}

	resp, err := client.Do(req)
	if err != nil {
		return MovieDBResponse{}, ErrorBackend
	}
	defer resp.Body.Close()

	var data MovieDBResponse
	if err := json.NewDecoder(resp.Body).Decode(&data); err != nil {
		return MovieDBResponse{}, ErrorBackend
	}
	data.HasElements = false;
	if len(data.Results) > 0 {
		data.HasElements = true;
	}

	return data, nil
}

func main() {
	lambda.Start(Handler)
}
{% endhighlight %}

__NOTE__: the ```HasElements``` property is an important field of the MovieDBResponse because it will let us implement an iteration in the AWS Step Function workflow.

I also found [this](https://github.com/lambci/docker-lambda) beautiful docker image that let you test your lambda (and support also GoLang) with a single docker run. You can pass the parameters as a string (payload requests), as shown in the method above: of course, you first have to compile your lambda for linux.

{% highlight sh %}

GOOS=linux go build -o MyCompiledLambda MyLambda.go

docker run --rm -v $PWD:/var/task lambci/lambda:go1.x MyCompiledLambda '{"parameter": "value"}'

{% endhighlight %}

To upload your Lambda in AWS, in the creation steps specify you want to upload a Go 1.x Lambda, then zip your build (in the example, ```MyCompiledLambda```)

{% highlight sh %}

zip MyCompiledLambda.zip MyCompiledLambda

{% endhighlight %}

and upload from an S3 bucket or manually.

<p align="center"><img src="http://image.ibb.co/mJqX9R/aws_lambda_2.png" style="width: 100%; marker-top: -10px;"/></p>

#### 6 - Build an AWS Step Functions Workflow
A finite state machine is an automata with really simple rule. Each state is equivalent to a AWS Lambda invocation and its directly linked with one or more states. You provide an input to the workflow, the first(s) lambda are invoked, then the output is the input of the next lambda(s). For my application, I created the following workflow - name of states define what is done by the corresponding lambda function.

<p align="center"><img src="TODO" style="width: 100%; marker-top: -10px;"/></p>

The entire workflow of a step function is described by a JSON file and can be written directly in a console available in the AWS Step Function web page. A workflow looks like the following code:

{% highlight json %}
{
  "Comment": "A Jarvis movie evaluator based on sentiment analysis from API",
  "StartAt": "GetMoviesInTheather",
  "States": {
	"GetMoviesInTheather": {
	  "Type": "Task",
	  "Resource": "arn_of_my_TMDB_function",
	  "ResultPath": "$",
	  "Next": "MoviesTitleIterator"
	},
	"MoviesTitleIterator" : { ... }
 "Done": {
      "Type": "Pass",
      "End": true
    }
  }
}	
{% endhighlight %}

As explained in the introduction, the workflow is a collection of states linked togheter. A state is defined by different properties

##### Common State Fields
- __Type (Required)__: the states could be defined of different types. The most important keyword-types for state for this how-to are three: ```Task```, ```Parallel```, ```Choice```.
- __Next__: the name of the next state that will be run when the current state finishes. Some state types, such as Choice, allow multiple transition states.
- __End__: designates this state as a terminal state (it ends the execution) if set to true. There can be any number of terminal states per state machine. Only one of Next or End can be used in a state. Some state types, such as Choice, do not support or use the End field.
- __Comment (Optional)__: holds a human-readable description of the state.
- __InputPath (Optional)__: a path that selects a portion of the state's input to be passed to the state's task for processing. If omitted, it has the value $ which designates the entire input. For more information, see Input and Output Processing).
- __OutputPath (Optional)__: a path that selects a portion of the state's input to be passed to the state's output. If omitted, it has the value $ which designates the entire input. For more information, see Input and Output Processing.

##### Task
A Task state ("Type": "Task") represents a single unit of work performed by a state machine. In addition to the common state fields, Task states have the following fields:
- Resource (Required): a URI, especially an Amazon Resource Name (ARN) that uniquely identifies the specific task to execute;
- ResultPath (Optional): specifies where (in the input) to place the results of executing the task specified in Resource. The input is then filtered as prescribed by the OutputPath field (if present) before being used as the state's output. For more information, see path;
- Retry (Optional): an array of objects, called Retriers, that define a retry policy in case the state encounters runtime errors. For more information, see Retrying After an Error;
- Catch (Optional), TimeoutSeconds (Optional), HeartbeatSeconds (Optional): for more details about these fields, look at the official AWS docs;

A Task state must set either the End field to true if the state ends the execution, or must provide a state in the Next field that will be run upon completion of the Task state. For instance, the state

{% highlight json %}
{ ...
	"GetMoviesInTheather": {
	  "Type": "Task",
	  "Resource": "arn_of_my_TMDB_function",
	  "ResultPath": "$",
	  "Next": "MoviesTitleIterator"
	},
}
{% endhighlight %}

calls the AWS Lambda function TMDB defined above and put the entire output in ResultPath: the ```$``` in subsequent states will refer to the output generated by the corresponding call.

##### Choice
A Choice state ("Type": "Choice") adds branching logic to a state machine. In addition to the common state fields, Choice states introduce the following additional fields:
- Choices (Required): this is an array of Choice Rules that determines which state the state machine transitions to next;
- Default (Optional, Recommended): the name of the state to transition to if none of the transitions in Choices is taken;

Choice states __do not support the End field__. In addition, they use Next only inside their Choices field. You can use a Choice to create an Iterator pattern, loop over results provided by a first call to a AWS Lambda function - in this case ```GetMoviesInTheather```. Let's have a look at the state below:

{% highlight json %}
{ ...
	"GetMoviesInTheather": { 
		... 
		"Next": "MoviesTitleIterator"
	},
    "MoviesTitleIterator": {
      "Type": "Choice",
      "Choices": [
        {
          "Variable": "$.has_elements",
          "BooleanEquals": true,
          "Next": "MovieInformationExtraction"
        }
      ],
      "Default": "Done"
    },
}
{% endhighlight %}

The state is invoked after the end of execution of the AWS Lambda function invoked by ```GetMoviesInTheather``` (I will return later on the first input passed to the workflow to start the entire process). The state checks if the input field ```has_elements``` - remember the MovieDBResponse property? - is equal to true. Thus, the ```MovieInformationExtraction``` starts: let's introduce first the Parallel type states.

##### Parallel
The Parallel state ("Type": "Parallel") can be used to create parallel branches of execution in your state machine. In addition to the common state fields, Parallel states introduce these additional fields:
- Branches (Required): an array of objects that specify state machines to execute in parallel. Each such state machine object must have fields named States and StartAt whose meanings are exactly like those in the top level of a state machine;
- ResultPath (Optional): specifies where (in the input) to place the output of the branches. The input is then filtered as prescribed by the OutputPath field (if present) before being used as the state's output. For more information, see Input and Output Processing.
- Retry and catch are optional;

A Parallel state causes AWS Step Functions to execute each branch, starting with the state named in that branch's StartAt field, as concurrently as possible, and __wait until all branches terminate__ (reach a terminal state) before processing the Parallel state's Next field.

__NOTE__: each branch must be self-contained. A state in one branch of a Parallel state __must not have__ a Next field that targets a field outside of that branch, nor can any other state outside the branch transition into that branch.

##### Parallel State Output
A Parallel state provides each branch with a copy of its own input data (subject to modification by the InputPath field). It generates output which is an array with one element for each branch containing the output from that branch. There is no requirement that all elements be of the same type. The output array can be inserted into the input data (and the whole sent as the Parallel state's output) by using a ResultPath field in the usual way (see Input and Output Processing). Let's have a look at the ```MovieInformationExtraction``` code:

{% highlight json %}
{ ...
	"MovieInformationExtraction": {
	  "Type": "Parallel",
	  "Branches": [
	    {
	      "StartAt": "PersistMovie",
	      "States": {
	        "PersistMovie": {
	          "Type": "Task",
	          "Resource": "aws_lambda_function_PersistMovie",
	          "End": true
	        }
	      }
	    },
	    {
	      "StartAt": "BreakingNews",
	      "States": {
	        "BreakingNews": {
	          "Type": "Task",
	          "Resource": "aws_lambda_function_BreakingNews",
	          "End": true
	        }
	      }
	    }
	  ],
	  "Next": "SentimentController"
	}
},
{% endhighlight %}

This state both call a Node.js AWS Lambda to persist data received from the above function 

<p align="center"><img src="http://image.ibb.co/ebriG6/grafana_sentiment.png" style="width: 100%; marker-top: -10px;"/></p>

Thank you everybody for reading!

[^lambda]: You can find more information [here](https://aws.amazon.com/lambda/?nc1=h_ls)