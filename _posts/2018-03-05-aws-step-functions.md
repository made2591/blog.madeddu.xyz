---
title: "Node.js, DynamoDB, and AWS Step Functions to collect <em>sentimented</em> movie reviews"
tags: [coding, aws, tmdb, news, step, functions, lambda, serverless, sentiment, analysis]
---

### Introduction
Recently I worked with AWS Lambda and API Gateway to extend my set of personal APIs and collect information from several sources. I wrote an article on that (if you want to [have a look](https://made2591.github.io/posts/aws-lambda)). In this article I will talk about the AWS Step Functions service that enable create finite states machines to easy coordinate the components of distributed applications and microservices using visual workflows. Why AWS Step Functions? Because they let me create a tool to gather movie titles in teather, search for reviews about each of them and make a basic sentiment analysis over the review to help me decide what's worth watching at teather and what's worth waiting for on Netflix :D
More in general, with AWS Step Functions, you can build applications made of individual components that each perform a discrete function: this lets you scale and change applications quickly. Step Functions is a reliable way to coordinate components and step through the functions of your application. They provides a graphical console to arrange and visualize the components of your application as a series of steps. This makes it simple to build and run multistep applications. Step Functions automatically triggers and tracks each step, and retries when there are errors, so your application executes in order and as expected. Step Functions logs the state of each step, so when things do go wrong, you can diagnose and debug problems quickly.

<p align="center"><img src="http://image.ibb.co/m3LxJS/aws_step_functions.png" style="width: 100%; marker-top: -10px;"/></p>

### Ingredients 
For this article, you will need the following:
- [AWS account](http://aws.amazon.com) (free tier it's ok, but API Gateway is not included);
- [Newsapi](http://newsapi.org) account to gather news from several sources (the free tier it's ok for our purpose);
- [Aylien](http://aylien.com) account to do some sentiment analysis (the free tier it's ok for our purpose);
- [AWS Lambda](https://aws.amazon.com/en/lambda/) to wrap services around the world to ghater and process data;
- [DynamoDB](https://aws.amazon.com/en/dynamodb/) to both optmize requests and persist processed data;
- [AWS Step Functions](https://aws.amazon.com/en/step-functions/), of course, to create the workflow, orchestrate lambdas and work on the data flow;

### Considerations
__First point__: as I already said in a preview post on AWS Services, there are a lot of quite simple steps. I recommend you to pay a lot of attention with AWS. You always have to know exactly what are you doing, to avoid surprise in billing in the end of the month. Fortunately, there are a lot of documentations on Amazon official site, so you only have to read them.

__Second point__: please pay attention to the number of steps and infinity loop when you are working with AWS Step Functions, Iteration and Parallel Workflow. The following workflow is prepared to be a mix of some of the features exposed by the AWS service: despite it provides some example of _how effectively you can increase the efficience of a step functions workflow_, is not intended to be efficient, nor the smartest way to implement the workflow. The first 4000 transactions from one state to another - don't be scared by the output log of the workflow! It shows quite three/four times the number of effective transactions because it show you the scheduled state, the start state, the output state, etc. After that, the cost is billed in terms of thousands, something like 0.025 dollars for 1000 thousands (more [here](https://aws.amazon.com/en/step-functions/pricing/)).

#### 1-2-3-4 Steps
The first four steps are equal to the one described [here](https://made2591.github.io/posts/aws-lambda). The only things I added is the HasElements bool `json:"has_elements"` to the NewsApiResponse Object (see next step).

#### 5 - Build MovieDB collector over AWS Lambda
Lambda currently supports different languages: C#, Java, Node.js, Python and now Go. I wrote a GoLang wrapper around the API exposed by TheMovieDB [here](http://themoviedb.org): I am working on a front-end written in Angular to show the elements gathered by this workflow, so I created a parametric AWS Lambda to wrap (almost) all the routes exposed by TheMovieDB APIs and be able to fill my front-end in the future. AWS Lambda ready GoLang file is a single ```.go``` file with a function, the ```handler``` and a ```main``` function to link the handler function to the lambda. And that's all. The only dependencies you need to install, if you want to run your lambda locally, is the ```aws-lambda-go``` sdk provided by Amazon and available on Github.

{% highlight sh %}
go get github.com/aws/aws-lambda-go/lambda
{% endhighlight %}

The entire TheMovieDB Go Wrapper ready to build for AWS Lambda is available [in this gist](https://gist.github.com/made2591/f2b8280035d0d770269f8695f54091c8.js). It follows a preview:

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

	if request.Url == "" {
		return MovieDBResponse{}, errors.New("Missing one of the required parameters: 'api_url'")
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

	...// the complete file here https://gist.github.com/made2591/f2b8280035d0d770269f8695f54091c8.js

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

GOOS=linux go build -o TMDB TMDB.go

docker run --rm -v $PWD:/var/task lambci/lambda:go1.x TMDB '{"url": "/discover/movie"}'

{% endhighlight %}

To upload your Lambda in AWS, in the creation steps specify you want to upload a Go 1.x Lambda, then zip your build (in the example, ```TMDB```)

{% highlight sh %}

zip TMDB.zip TMDB

{% endhighlight %}

and upload from an S3 bucket or manually.

<p align="center"><img src="http://image.ibb.co/mJqX9R/aws_lambda_2.png" style="width: 100%; marker-top: -10px;"/></p>

#### 6 - Build an AWS Step Functions Workflow
A finite state machine is an automata with really simple rule. Almost each states are Tasks that call AWS Lambda functions and are directly linked with one or more states. You provide an input to the workflow, the first(s) lambda are invoked, then the output of the execution is passed as the input to the next states (and eventually AWS Lambda(s) invoked by them). 

The entire workflow of a step function is described by a JSON file and can be written directly in a console available in the AWS Step Function web page. You can view a preview of the worflow in the right part of the screen while you're defining it.

<p align="center"><img src="http://image.ibb.co/dE5QNn/workflow_edit.png" style="width: 100%; marker-top: -10px;"/></p>

The final workflow for our scope will look like the image below:

<p align="center"><img src="http://image.ibb.co/nAk0Nn/workflow_empty.png" alt="perceptron" style="width: 100%; marker-top: -10px;"/></p>

The respective workflow JSON description looks like the following code:

{% highlight json %}
{
  "Comment": "A movie evaluator based on sentiment analysis.",
  "StartAt": "GetMoviesInTheather",
  "States": {
    "GetMoviesInTheather": {
      "Type": "Task",
      "Resource": "arn:aws:lambda:your_region:your_code:function:TMDB",
      "ResultPath": "$",
      "Next": "PrepareStructures"
    },
    "PrepareStructures": {
      "Type": "Task",
      "Resource": "arn:aws:lambda:your_region:your_code:function:MoviesTitleIterator",
      "ResultPath": "$",
      "Next": "MovieIterator"
    },    
    "MovieIterator": {
      "Type": "Choice",
      "Choices": [
        {
          "Variable": "$.movies.has_elements",
          "BooleanEquals": true,
          "Next": "MovieInformationExtraction"
        }
      ],
      "Default": "Done"
    },
    "MovieInformationExtraction": {
      "Type": "Parallel",
      "Branches": [
        {
          "StartAt": "PersistMovie",
          "States": {
            "PersistMovie": {
              "Type": "Task",
              "Resource": "arn:aws:lambda:your_region:your_code:function:PersistMovie",
              "End": true
            }
          }
        },
        {
          "StartAt": "BreakingNews",
          "States": {
            "BreakingNews": {
              "Type": "Task",
              "Resource": "arn:aws:lambda:your_region:your_code:function:BreakingNews",
              "End": true
            }
          }
        }
      ],
      "ResultPath": "$",
      "Next": "NewsIterator"
    },
    "NewsIterator": {
      "Type": "Choice",
      "Choices": [
        {
          "Variable": "$[1].has_elements",
          "BooleanEquals": true,
          "ResultPath": "$",
          "Next": "PrepareSentimentRequest"
        }
      ],
      "Default": "ToMovieIterator"
    },
    "ToMovieIterator": {
      "Type": "Task",
      "Resource": "arn:aws:lambda:your_region:your_code:function:ToMovieIterator",
      "Next": "MovieIterator"
    },
    "PrepareSentimentRequest": {
      "Type": "Task",
      "Resource": "arn:aws:lambda:your_region:your_code:function:PrepareSentimentRequest",
      "Next": "SentimentController"
    },
    "SentimentController": {
      "Type": "Parallel",
      "Branches": [
        {
          "StartAt": "AylienSentiment",
          "States": {
            "AylienSentiment": {
              "Type": "Task",
              "InputPath" : "$[1].aylienQuery",
              "Resource": "arn:aws:lambda:your_region:your_code:function:AylienSentiment",
              "End": true,
              "Retry": [ {
                  "ErrorEquals": [ "States.ALL" ],
                  "IntervalSeconds": 1,
                  "MaxAttempts": 2,
                  "BackoffRate": 2.0
               } ],
              "Catch": [ {
                  "ErrorEquals": [ "States.ALL" ],
                  "Next": "FallBack"
               } ]
            },
            "FallBack": {
               "Type": "Pass",
               "End": true
            }
          }
        }, 
        {
          "StartAt": "NoOp",
          "States": {
              "NoOp": {
              "Type": "Pass",
              "End": true
            }
          }
        }
      ],
      "ResultPath": "$",
      "Next": "PersistNews"
    },
    "PersistNews": {
      "Type": "Task",
      "Resource": "arn:aws:lambda:your_region:your_code:function:PersistNews",
      "Next": "NewsIterator"
    },
    "Done": {
      "Type": "Pass",
      "End": true
    }
  }
}
{% endhighlight %}

PLEASE DON'T PANIC!!!! :D

As explained in the introduction, the workflow is a collection of states linked togheter. A state is defined by different properties:

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
- Catch (Optional)[^catch], TimeoutSeconds (Optional), HeartbeatSeconds (Optional): for more details about these fields, look at the official AWS docs;

A Task state must set either the End field to true if the state ends the execution, or must provide a state in the Next field that will be run upon completion of the Task state. For instance, the state ```GetMoviesInTheather```

{% highlight json %}
{   
	...
    "GetMoviesInTheather": {
      "Type": "Task",
      "Resource": "arn:aws:lambda:your_region:your_code:function:TMDB",
      "ResultPath": "$",
      "Next": "PrepareStructures"
    },
    "PrepareStructures": {
      "Type": "Task",
      "Resource": "arn:aws:lambda:your_region:your_code:function:MoviesTitleIterator",
      "ResultPath": "$",
      "Next": "MovieIterator"
    },
    ...
}
{% endhighlight %}

calls the AWS Lambda function TMDB defined above and put the entire output in ResultPath: the ```$``` in subsequent states will refer to the output generated by the corresponding call. ```PrepareStructures``` is a state that calls a Node.js AWS Lambda Function that operate over the input to create structure ready to be parsed by the ```MovieIterator``` (a generic step for the iterator).

{% highlight js %}
exports.handler = (event, context, callback) => {

    var movies = {};
    var news = {};
    if (event instanceof Array) {
        movies = event[0].movies;
        news = event[1];
    } else {
        movies = event;
    }

    var structures = { "movies" : movies, "news" : news, "sentiment" : {} }

    // Array still has elements:
    if (movies.results.length > 0) {
        structures.movies.has_elements = true;
        console.log('Creating query news for movie: ' + structures.movies.results[0]);
        structures.movie_id = movies.results[0].id;
        structures.q = structures.movies.results[0].original_title;
        structures.sortBy = "relevancy";
        structures.pageSize = 10;
    } else {
        structures.movies.has_elements = false;
    }

    // Log array elements (for demonstration purpose)
    console.log('Array has more elements: ' + structures.movies.has_elements);
    
    callback(null, structures);
};
{% endhighlight %}

The code in the lambda simply create a dict struct if passed input is an Array - this happens after further executions. It also preparare the input to be used by a previous AWS Lambda - the ```BreakingNews``` that expected a dict of three parameters as input, to look for 10 most relevant news for the first movie of the list. To explain what happens in the next state, the ```MovieIterator```, I need to talk first about the Choice type.

##### Choice
A Choice state ("Type": "Choice") adds branching logic to a state machine. In addition to the common state fields, Choice states introduce the following additional fields:
- Choices (Required): this is an array of Choice Rules that determines which state the state machine transitions to next;
- Default (Optional, Recommended): the name of the state to transition to if none of the transitions in Choices is taken;

Choice states __do not support the End field__. In addition, they use Next only inside their Choices field. Choice type is particularly interesting when you need to process large files. Lambda functions have a couple of limitations namely memory and a 5 minute timeout. If you have some operation you need to perform on a very large dataset it may not be possible to complete this operation in a single execution of a lambda function. There are several ways to solve this problem: one of them is to use a Choice to create an Iterator pattern, loop over results provided by a first call to a AWS Lambda function - in this case ```PrepareStructures``` (that wrapps the input of the ```GetMoviesInTheather```). Let's have a look at the state below:

{% highlight json %}
{ 
	...
	"PrepareStructures": { 
		... 
		"Next": "MovieIterator"
	},
	"MovieIterator": {
      "Type": "Choice",
      "Choices": [
        {
          "Variable": "$.movies.has_elements",
          "BooleanEquals": true,
          "Next": "MovieInformationExtraction"
        }
      ],
      "Default": "Done"
    },
    ...
}
{% endhighlight %}

The state is invoked after the end of execution of the AWS Lambda function invoked by ```PrepareStructures``` (I will return later on the first input passed to the workflow to start the entire process). The state checks if the property movies of the input field has a boolean field ```has_elements``` - remember the MovieDBResponse property? - is equal to true. This implies that the input provided to the state ```MovieIterator``` has to provide always a dict object with a movies properties. I will return later on this.

Thus, the ```MovieInformationExtraction``` starts: let's introduce first the Parallel type states.

##### Parallel
The Parallel state ("Type": "Parallel") can be used to create parallel branches of execution in your state machine. In addition to the common state fields, Parallel states introduce these additional fields:
- Branches (Required): an array of objects that specify state machines to execute in parallel. Each such state machine object must have fields named States and StartAt whose meanings are exactly like those in the top level of a state machine;
- ResultPath (Optional): specifies where (in the input) to place the output of the branches. The input is then filtered as prescribed by the OutputPath field (if present) before being used as the state's output. For more information, see Input and Output Processing.
- Retry and catch are optional;

A Parallel state causes AWS Step Functions to execute each branch, starting with the state named in that branch's StartAt field, as concurrently as possible, and __wait until all branches terminate__ (reach a terminal state) before processing the Parallel state's Next field.

__NOTE__: each branch must be self-contained. A state in one branch of a Parallel state __must not have__ a Next field that targets a field outside of that branch, nor can any other state outside the branch transition into that branch.

##### Parallel State Output
A Parallel state provides each branch with a copy of its own input data (subject to modification by the InputPath field). It generates output __which is an array with one element for each branch containing the output from that branch__. There is no requirement that all elements be of the same type. The output array can be inserted into the input data (and the whole sent as the Parallel state's output) by using a ResultPath field in the usual way (see Input and Output Processing in the official AWS Docs). Let's have a look at the ```MovieInformationExtraction``` code:

{% highlight json %}
{ 
    ...
    "MovieInformationExtraction": {
      "Type": "Parallel",
      "Branches": [
        {
          "StartAt": "PersistMovie",
          "States": {
            "PersistMovie": {
              "Type": "Task",
              "Resource": "arn:aws:lambda:your_region:your_code:function:PersistMovie",
              "End": true
            }
          }
        },
        {
          "StartAt": "BreakingNews",
          "States": {
            "BreakingNews": {
              "Type": "Task",
              "Resource": "arn:aws:lambda:your_region:your_code:function:BreakingNews",
              "End": true
            }
          }
        }
      ],
      "ResultPath": "$",
      "Next": "NewsIterator"
    },
    ...
},
{% endhighlight %}

This state both call a Node.js AWS Lambda to persist the first movie in the data received from the ```MovieIterator``` step and looks for news about the movie. The input is the output provided by the ```PrepareStructures```: in fact the MovieIterator does NOT modify the input it receives. 

##### PersistMovie movie
The ```PersistMovie``` calls an AWS Lambda function written in Node.js to persit the movie to a DynamoDB table previously created - it's really simply, you can start from [here](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/SampleData.CreateTables.html) and read more about how DynamoDB works in the official AWS Docs.

{% highlight js %}
const AWS = require('aws-sdk');
AWS.config.update({ region: process.env.REGION });
const dynamodb = new AWS.DynamoDB.DocumentClient();

exports.handler = function (event, context, callback) {

    console.log("request: " + JSON.stringify(event.movies.results));

    if (event.movies.results.length > 0) {

        console.log("PUT begins");

        var insertion = event.movies.results[0];
        insertion.add_date = new Date(Date.now()).toString();
        
        let movie = {
            TableName: process.env.MOVIES_TABLE,
            Item: insertion
        };
    
        dynamodb.put(movie, (err, data) => {
            console.log("movie");
            if (err) 
                console.log("dynamodb err: ", err, err.stack);
            else {
                console.log("dynamodb data: ", data);
            }
            console.log("PUT end");
        });
    
        event.movies.results.shift();

    } else {
        event.movies.has_elements = false;
    }
    
    if (event.movies.results.length > 0) {
        event.movie_id = event.movies.results[0].id;
        event.q = event.movies.results[0].original_title;
        event.sortBy = "relevancy";
        event.pageSize = 5;
        event.movies.has_elements = true;
    } else {
        event.movies.has_elements = false;
    }

    callback(null, event);
    
};
{% endhighlight %}

The Lambda put the first movie in DynamoDB - to prevent the overload of the lambda and of DynamoDB (even if you can specify auto scaling to provision more i/o units to your table). Then, it shift() the array of result (removing the first element and prepare the query to looking for news for the next movie for the next iteration). The BreakingNews Lambda, in parallel, is looking for news for the movie inserted by the PersistMovie Lambda.

As said above, the Result of the parallel state is placed in an array of two element: so the entire return of the PersistMovie state - output of the PersistMovie Lambda - is placed in position 0 (because in the branches the state is defined first): the same for the BreakingNews state - output of the BreakingNews Lambda - that will be placed in the position 1 of the output. 

##### NewsIterator
The next state is the ```NewsIterator```: have a look at the variable is looking for. Is exactly the second element of the output array, so the ouput of the BreakingNews-Lambda state, in particular the properties has_elements (this is what I said before about the modification done to the BreakingNews Lambda defined in my previous post).

{% highlight json %}
	...
    "NewsIterator": {
      "Type": "Choice",
      "Choices": [
        {
          "Variable": "$[1].has_elements",
          "BooleanEquals": true,
          "ResultPath": "$",
          "Next": "PrepareSentimentRequest"
        }
      ],
      "Default": "ToMovieIterator"
    },
{% endhighlight %}

##### ToMovieIterator
Let's start from the Default action: if there are no news - has_elements is false, then go to ```ToMovieIterator```

{% highlight json %}
	...
    "ToMovieIterator": {
      "Type": "Task",
      "Resource": "arn:aws:lambda:your_region:your_code:function:ToMovieIterator",
      "Next": "MovieIterator"
    },
    ...
{% endhighlight %}

This state simply return the first element of the output array to preserve the structure of the data. In fact, the ToMovieIterator Lambda is a Node.js function with the following code inside.

{% highlight js %}
	exports.handler = (event, context, callback) => {
	    callback(null, event[0]);
	};
{% endhighlight %}

##### PrepareSentimentRequest
If there are News to persist and analysize, then the NewsIterators move to the ```PrepareSentimentRequest``` state.

{% highlight js %}

    "PrepareSentimentRequest": {
      "Type": "Task",
      "Resource": "arn:aws:lambda:your_region:your_code:function:PrepareSentimentRequest",
      "Next": "SentimentController"
    },
    "SentimentController": {
      "Type": "Parallel",
      "Branches": [
        {
          "StartAt": "AylienSentiment",
          "States": {
            "AylienSentiment": {
              "Type": "Task",
              "InputPath" : "$[1].aylienQuery",
              "Resource": "arn:aws:lambda:your_region:your_code:function:AylienSentiment",
              "End": true,
              "Retry": [ {
                  "ErrorEquals": [ "States.ALL" ],
                  "IntervalSeconds": 1,
                  "MaxAttempts": 2,
                  "BackoffRate": 2.0
               } ],
              "Catch": [ {
                  "ErrorEquals": [ "States.ALL" ],
                  "Next": "FallBack"
               } ]
            },
            "FallBack": {
               "Type": "Pass",
               "End": true
            }
          }
        }, 
        {
          "StartAt": "NoOp",
          "States": {
              "NoOp": {
              "Type": "Pass",
              "End": true
            }
          }
        }
      ],
      "ResultPath": "$",
      "Next": "PersistNews"
    },
{% endhighlight %}

This state first call the Lambda ```PrepareSentimentRequest``` that simply prepare the request to be done to the AylienSentiment Lambda - done by the AylienSentimentState. The ```PrepareSentimentRequest``` is Node.js lambda function

{% highlight js %}
exports.handler = (event, context, callback) => {

    event[1]["aylienQuery"] = {};
    event[1]["aylienQuery"].text = event[1].articles[0].description;
    event[1]["aylienQuery"].url  = event[1].articles[0].url;
    event[1]["aylienQuery"].mode = "document";

    callback(null, event);

};
{% endhighlight %}

##### AylienSentiment and Error Handling
Thus, the aylienQuery is prepared and the ```AylienSentiment``` can call the respective lambda. But, the input is not a dict, so throught the use of the ```InputPath``` directive, we can specify to _manipulate_ the input by projecting only what we need for the lambda, in this case ```$[1].aylienQuery```.

The AylienSentiment is the weak part of the chain: I don't know why, but sometimes the Lambda fails - I think because of the small timeout I use for my APIs and the limitation of the free tier for the service behind the Aylien API. So, I use an error catching flow that retries and eventually goes ahead if things simply doesn't work for the particular news (covering bad requests due to broken news provided by the News API, for instance).

{% highlight js %}
  "Retry": [ {
      "ErrorEquals": [ "States.ALL" ],
      "IntervalSeconds": 1,
      "MaxAttempts": 2,
      "BackoffRate": 2.0
   } ],
  "Catch": [ {
      "ErrorEquals": [ "States.ALL" ],
      "Next": "FallBack"
   } ]
{% endhighlight %}

The fields defined in the Retry properties are self-explained: with the BackoffRate, you can define the multiplier by which the retry interval increases during each attempt (2.0 by default). To learn more about error handling, have a look [here](https://docs.aws.amazon.com/step-functions/latest/dg/concepts-error-handling.html). 

I don't persist the news gathered from the movie in the parallel step - this is done by the No-OP state of type Pass that simply ignore the input and place in output what it receives - because the last state ```PersistNews```, that calls the respective Lambda function ```PersistNews```, merges together the sentiment provided by Aylien for the specific news and persist all together.

##### The final PersistNews
Have a look at the code below:

{% highlight js %}
const AWS = require('aws-sdk');
AWS.config.update({ region: process.env.REGION });
const dynamodb = new AWS.DynamoDB.DocumentClient();
const crypto = require('crypto');

function cleanEmptyField(obj) {
  for (var prop in obj) {
    if (typeof obj[prop] === 'object') {
      cleanEmptyField(obj[prop]);
    } else if(obj[prop] === '') {
      delete obj[prop];
    }
  }
  return obj;
}

exports.handler = function (event, context, callback) {

    console.log("PUT begins");
    
    var sentiment = event[0];
    event = event[1];
    var article = event[1].articles[0];
    article.id = crypto.createHash('md5').update(JSON.stringify(article)).digest("hex");
    Object.assign(article, sentiment);
    
    let news = {
        TableName: process.env.NEWS_TABLE,
        Item: cleanEmptyField(article)
    };

    article["movied_id"] = event[0].movie_id;

    console.log("news: ", article);

    dynamodb.put(news, (err, data) => {
        if (err) 
            console.log("dynamodb err: ", err, err.stack);
        else {
            console.log("dynamodb data: ", data);
        }
    });

    console.log("PUT end");

    event[1].articles.shift();
    event[1].has_elements = event[1].articles.length != 0;

    callback(null, event);
    
};

{% endhighlight %}

The Lambda clean the first news to prevent DynamoDB value checking error. Then, it merge together the sentiment provided by Aylien and create a unique identifier using the entire news. In the end, the news is persisted and the array that handles the news is shifted (the first element is removing).

#### Conclusion
The event is then returned as is to the News iterator (already disposed to handle an array because of result).

#### Amazon CloudWatch Events
To have an always updated table of Movies in Theater and News - Sentiment evalued - about them, you can use [Amazon CloudWatch Events](https://docs.aws.amazon.com/AmazonCloudWatch/latest/events/ScheduledEvents.html) and schedule the launch of the workflow, for instance once a week.

##### Improvement
The use of iterator and parallel states to perform persistence and query to other APIs is a good approach to prevent fail of lambda due to timeout errors: in my opinion, AWS Lambda should remain as simple and quick as possible, to perform request and be loaded fast. However, there are some improvement that could be done
- Setup the number of news in a parameter from the beginning;
- Clean the flow of iteration and use a parameter to setup the percentage of result to be persisted in each call of Lambda - for instance, 100 Movies, 10% => 10 Movies persisted each times: this both to reduce the number of steps and to optimize the time of executions;
- Setup better error handling and timeouts for each steps;

What will you use step functions for?

Thank you everybody for reading!

[^lambda]: You can find more information [here](https://aws.amazon.com/lambda/?nc1=h_ls);
[^catch]: I will only return on Catch for one of the last state;