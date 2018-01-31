---
title: "Buffalo: a GoLang open web framework"
tags: [coding, golang, docker, buffalo, api, rest, web]
---

### Introduction
A few months ago I used Golang to create a _proof-of-concept_ project using the amazing [goa](https://goa.design/)
package: it is really handful to create restful api - look at my boilerplate post [here](https://made2591.github.io/goa-docker-multistage), but if you want to dive into a full-stack GoLang experience you really should try [Buffalo](https://gobuffalo.io)[^br]. As the team says, buffalo provide _a Go web development eco-system, designed to make the life of a Go web developer easier_. In this post I want to share a little piece of my experience about this project: there are almost no requirements, except you should already know Golang, Docker, Postgres. And Markdown :D

<p align="center"><img src="https://gobuffalo.io/assets/images/buffalo.svg" style="width: 70%; marker-top: -10px;"/></p>

If you want to find out how to start building with Buffalo, keep reading!

#### Step 1: Install Buffalo
To install all required packages locally in your dev machine open a shell and simply run:

    go get -u -v github.com/gobuffalo/buffalo/buffalo

#### Step 2: Generate a new project
To generate a new project go under ```$GOPATH/src/github.com/$USER/``` and simply run:

    buffalo new explorer

That will generate a new Buffalo application called coke ready to be customized.

#### Step 3: Understanding a Buffalo project
In the Buffalo project root you just created (), you should see at least these directories and files:
- d <span style="color:#A04279; font-family:Courier; font-size:1.1em;">actions/</span>: this directory handles the Controller part of the MVC pattern. It contains the handlers for URLs, plus the ```app.go``` and ```render.go``` files to, respectively, setup your app, your routes and the template engine(s).
- d <span style="color:#a2a122; font-family:Courier; font-size:1.1em;">assets/</span>: 
- d <span style="color:#A04279; font-family:Courier; font-size:1.1em;">grifts/</span>: 
- d <span style="color:#A04279; font-family:Courier; font-size:1.1em;">locales/</span>: 
- d <span style="color:#A04279; font-family:Courier; font-size:1.1em;">models/</span>: 
- d <span style="color:#A04279; font-family:Courier; font-size:1.1em;">public/</span>: 
- d <span style="color:#A04279; font-family:Courier; font-size:1.1em;">templates/</span>: 
- d <span style="color:#A04279; font-family:Courier; font-size:1.1em;">tmp/</span>: 
- f <span style="color:#A04279; font-family:Courier; font-size:1.1em;">database.yml</span>: 
- f <span style="color:#A04279; font-family:Courier; font-size:1.1em;">main.go</span>: 



Thank you everybody for reading!

[^br]: Github official [repository](https://github.com/gobuffalo/buffalo)