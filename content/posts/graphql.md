---
layout: post
title: "GraphQL is the new black"
date: 2019-03-05
polly: https://madeddu.xyz/mp3/graphql.mp3
categories:
- coding
- api
- golang
tags:
- coding
- golang
- api
- graphql
- guide
---

### Prelude
GraphQL is an open-source data query and manipulation language for APIs and a runtime for fulfilling queries with existing data. GraphQL was developed internally by Facebook in 2012 before being publicly released in 2015 because - believe me or not - it was quite tricky for them dealing with their schema. It allows clients to define the structure of the data required, and exactly the same structure of the data is returned from the server, therefore preventing excessively large amounts of data from being returned. Useless going into much more details without putting hands on - let me just say one thing more I wanted to have a look at it: it's the first time a REST alternative appeared on the market. Because REST is cool, ok but... well, I'm honest I don't know. Barney Stinson would say *new is always better*. I say let's see.

<div class="img_container"><img src="https://i.imgur.com/juXPUvq.gif"  style="width: 100%; marker-top: -10px;"/></div>

### GraphQL: concepts
A GraphQL service is created by defining types and fields on those types, then providing functions for each field on each type.

The Schema Definition Language (SDL)
GraphQL has its own type system that's used to define the schema of an API. The syntax for writing schemas is called Schema Definition Language (SDL).

