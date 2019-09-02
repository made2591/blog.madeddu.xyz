---
layout: post
title: "Golang, Docker and multistage build"
date: 2017-12-18
categories:
- coding
- golang
- docker
tags:
- golang
- docker
- goa
- api
- rest
---

### Recipe for a good meal
A few months ago I started working with Golang to a _proof-of-concept_ project using the amazing [goa](https://goa.design/)
package (thank you [atosatto](https://github.com/atosatto) for your advise): I omit the praise, I would only say that - _imho_ - Golang it's a very funny language to use for many reasons (stay tuned, I will write about Golang and my favourite packages). However, in this post I want to share a little piece of my experience about this project: the main ingredients for this recipe are Golang (in particular the goa package) and Docker - with some piece of Angular 4, nginx, and minor stuff. I excluded the storage part so the two elements of the _boilerplate_ I will talk about live in separate container - you can of course orchestrate with k8... ok, whatever you want :P
- api-container:
	- [goa.design], a Golang package to build restful api (just a few notes),
- web-container:
	- [Angular 4], the popular framework to build cool frontend (just a few notes),
	- [nginx], a simple container webserver to handle request request from fe,

If you want to find out how to start building with these tools, this is the right place for you!

<div class="img_container"><img src="https://i.imgur.com/npPEvD5.png" style="width: 70%; marker-top: -10px;"/></div>

#### goa.design
goa is a _holistic approach for building microservices_ in Golang (cit). I don't want to go through all the features of the packages, you can read a _quite_ complete [overview](https://goa.design/design/overview/) of the project in the official site. What goa actually offers is a good starting point if you want to _design_ your microservices without losing time in the construction of the outline logic and focusing mainly on the definition of resources, actions and media types. Look at some piece of code from the [goa-cellar](https://github.com/goadesign/goa-cellar) example:

{{< highlight golang >}}
package design

import (
	. "github.com/goadesign/goa/design"
	. "github.com/goadesign/goa/design/apidsl"
)

// This is the cellar application API design used by goa to generate
// the application code, client, tests, documentation etc.
var _ = API("MyPorh", func() {
	Title("The virtual wine cellar")
	Description("A basic example of a CRUD API implemented with goa")
	Contact(func() {
		Name("goa team")
		Email("admin@goa.design")
		URL("http://goa.design")
	})
	License(func() {
		Name("MIT")
		URL("https://github.com/goadesign/goa/blob/master/LICENSE")
	})
	Docs(func() {
		Description("goa guide")
		URL("http://goa.design/getting-started.html")
	})
	Host("localhost:8081")
	Scheme("http")
	BasePath("/cellar")

	Origin("http://swagger.goa.design", func() {
		Methods("GET", "POST", "PUT", "PATCH", "DELETE")
		MaxAge(600)
		Credentials()
	})

	ResponseTemplate(Created, func(pattern string) {
		Description("Resource created")
		Status(201)
		Headers(func() {
			Header("Location", String, "href to created resource", func() {
				Pattern(pattern)
			})
		})
	})
})
{{< / highlight >}}

The goa _design language_ consists of _functions_ defined to describe _definitions_. The goa design language root definition is the API definition. The goa API design language is a ```DSL``` _implemented_ in Go and is not Go: it makes use of _anonymous_ functions to describe the various definitions. In the example above the API function accepts the name of the API as first argument and an anonymous function as second argument. This anonymous function defines additional properties of the API: the pattern (name + DSL) is used by many other DSL functions. Let's move one step forward, introducing ```Resource```.

{{< highlight golang >}}
var _ = Resource("account", func() {

	DefaultMedia(Account)
	BasePath("/accounts")

	Action("list", func() {
		Routing(
			GET(""),
		)
		Description("Retrieve all accounts.")
		Response(OK, CollectionOf(Account))
	})

	Action("show", func() {
		Routing(
			GET("/:accountID"),
		)
		Description("Retrieve account with given id. IDs 1 and 2 pre-exist in the system.")
		Params(func() {
			Param("accountID", Integer, "Account ID", func() {
				Minimum(1)
			})
		})
		Response(OK)
		Response(NotFound)
		Response(BadRequest, ErrorMedia)
	})

	Action("create", func() {
		Routing(
			POST(""),
		)
		Description("Create new account")
		Payload(func() {
			Member("name")
			Required("name")
		})
		Response(Created, "/accounts/[0-9]+")
		Response(BadRequest, ErrorMedia)
	})

	Action("update", func() {
		Routing(
			PUT("/:accountID"),
		)
		Description("Change account name")
		Params(func() {
			Param("accountID", Integer, "Account ID")
		})
		Payload(func() {
			Member("name")
			Required("name")
		})
		Response(NoContent)
		Response(NotFound)
		Response(BadRequest, ErrorMedia)
	})

	Action("delete", func() {
		Routing(
			DELETE("/:accountID"),
		)
		Params(func() {
			Param("accountID", Integer, "Account ID")
		})
		Response(NoContent)
		Response(NotFound)
		Response(BadRequest, ErrorMedia)
	})
})
{{< / highlight >}}

The ```Resource``` function defines a set of API endpoints. Each actual endpoint is described using the ```Action``` function: inside a resource function, it is possible to define any arbitrary number of actions. Apart from the root, API definition the goa API design language also makes it possible to describe the actual endpoints together with details on the shape of the requests and responses: below, the _MediaType_ definition.

{{< highlight golang >}}
// Account is the account resource media type.
var Account = MediaType("application/vnd.account+json", func() {
	Description("A tenant account")
	Attributes(func() {
		Attribute("id", Integer, "ID of account", func() {
			Example(1)
		})
		Attribute("href", String, "API href of account", func() {
			Example("/accounts/1")
		})
		Attribute("name", String, "Name of account", func() {
			Example("test")
		})
		Attribute("created_at", DateTime, "Date of creation")
		Attribute("created_by", String, "Email of account owner", func() {
			Format("email")
			Example("me@goa.design")
		})

		Required("id", "href", "name", "created_at", "created_by")
	})

	View("default", func() {
		Attribute("id")
		Attribute("href")
		Attribute("name")
		Attribute("created_at")
		Attribute("created_by")
	})

	View("tiny", func() {
		Description("tiny is the view used to list accounts")
		Attribute("id")
		Attribute("href")
		Attribute("name")
	})

	View("link", func() {
		Attribute("id")
		Attribute("href")
	})
})
{{< / highlight >}}

The goa design language ```MediaType``` function describes media types which represent the shape of response bodies.
You can define a ```MediaType``` as a ```Type``` (I didn't talk about that): there are similarities in features, for instance the definition of Collections of a particular MediaType (CollectionOf) and Type (ArrayOf), the function used to define ```Attributes```. There are two properties unique to media types: first, the ```views``` describe different _serialization_ of the same media type. You can create _short_ representation of a resource in listing requests, a more detailed one in requests that return a single resource. Second, the ```links``` that represents related media types that should be rendered embedded in the response[^views].

After the creation of the design[^goa] file inside a _design_ package, the only thing you have to do is _bootstrapping_ your design (using the cli ```goagen```[^goagen])

{{< highlight bash >}}

goagen bootstrap -d myprojects/myapi/design

{{< / highlight >}}

and, if there's no error, you will be able to start from the _hooks_ already written for each action you defined over your resources. Cool: let's talk about how to create the image!

#### Facing the build
After (ok, even before) the creation of the API, you have to deal with Docker to _encapsulate_ the binary. Actually, it is very common to have one Dockerfile to use for development (which contained everything needed to build your application), and a slimmed-down one to use for production, which only contained your application and exactly what was needed to run it. Golang is perfect to create handful microservice, because with a simple [busybox](https://hub.docker.com/_/busybox/)[^busybox] you can run the compiled binary and you've done. But...maintaining two Dockerfiles is not ideal. The question is: _how_ and _where_ do you build the binary from your API, mainting at the same time the image as much as possible _small_? I faced with this problem for my project, I don't know how to call it - let me say - it's _a build dilemma_. Multi-stage builds simplify this situation a lot.

#### Multistage build
Since version 17.05, Docker supports [multistage](https://docs.docker.com/engine/userguide/eng-image/multistage-build/): with multi-stage builds, you use multiple ```FROM``` statements in your Dockerfile. Each ```FROM``` instruction can use a different base, and each of them begins a new stage of the build. You can selectively copy artifacts from one stage to another, leaving behind everything you don't want in the final image (cit.). Look at my Dockerfile to multistage-build my Golang API microservice:

{{< highlight docker >}}

### STAGE 1: Build ###

# The builder node
FROM golang:latest as builder

# create working directory
WORKDIR /go/src/github.com/made2591/myproject/api

# copy the content
COPY . .

# install dependencies
RUN go get ./...

# build binary
RUN CGO_ENABLED=0 GOOS=linux go build -a -installsuffix cgo -o myproject .


### STAGE 2: Setup ###

# The runner node
FROM alpine:latest as runner

# setup env
RUN apk --no-cache add ca-certificates
WORKDIR /root/

# copy the binary from previous stage
COPY --from=builder /go/src/github.com/made2591/myproject/api .

# execute
CMD ["./myproject"]

{{< / highlight >}}

What happens? With the statement ```as``` in the first stage of the build we create a sort of _label_, then with the option ```--from``` of the ```COPY``` command, Docker copies just the built artifact from the previous stage into this new stage. The Go SDK and any intermediate artifacts are left behind, and not saved in the final image.

__NOTE__: I don't use to _bootstrap_ my goa APIs inside the build stage, because first) if you play a little bit with goa you find out that bootstrap process is done one time only to help you create procject structure - they have to work on incremental design because the cli doesn't support creation for new hooks if the core file are already modified - and second) you have to build your core file in any case. My advise is to create a dev env as much as possible confortable to work with goa. After that, use the dockerfile to create your multistage build!

#### Angular 4 - another scenario
Needless to say, the entire Internet is full of _boilerplates_ to start working with Angular \* or any other _\*.js_ you like. I have been able to use angular cli, I honestly think that the project is really interesting but that it changes too frequently and it is often difficult to manage dependencies with npm. This is to say: in six years as a computer, I understand that you can build your frontend as you like, in any case in a very short time will not work anyway any more, and you can not do anything about that. In any case, why am I talking about Angular 4? To share with you my Dockerfile to multistage-build my Angular 4 frontend!

{{< highlight docker >}}

### STAGE 1: Build ###

# We label our stage as 'builder'
FROM node:8-alpine as builder

COPY package.json package-lock.json ./

RUN npm set progress=false && npm config set depth 0 && npm cache clean --force

## Storing node modules on a separate layer will prevent unnecessary npm installs at each build
RUN npm i && mkdir /ng-app && cp -R ./node_modules ./ng-app

WORKDIR /ng-app

COPY . .

## Build the angular app in production mode and store the artifacts in dist folder
RUN $(npm bin)/ng build --prod --build-optimizer


### STAGE 2: Setup ###

FROM nginx:1.13.3-alpine

## Copy our default nginx config
COPY nginx/default.conf /etc/nginx/conf.d/

## Remove default nginx website
RUN rm -rf /usr/share/nginx/html/*

## From 'builder' stage copy over the artifacts in dist folder to default nginx public folder
COPY --from=builder /ng-app/dist /usr/share/nginx/html

CMD ["nginx", "-g", "daemon off;"]

{{< / highlight >}}

This also provide you a way to serve the application you build with docker, without having to deal with the blackhole npm node_modules dependencies etc.

Thank you everybody for reading!

[^goa]: you can of course split info across multiple file belonging the same _design_ package.
[^views]: ```Views``` may then use the special ```Links``` function to render all the links: if you don't define a link and the resource has a parent, then the compiler will arise an error during building.
[^goagen]: the goagen is a tool the generate various artifacts from a goa design package - more info [here](https://goa.design/implement/goagen/)
[^busybox]: if you want the SSL/TLS support, prefer the [busybox-curl](https://hub.docker.com/r/odise/busybox-curl/) image, even maybe it is good for a dev env, but I would prefer an alpine for production.
