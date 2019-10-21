---
layout: post
title: "From Jekyll to Hugo, from Travis to Gitlab: a time for changes"
date: 2019-09-20
categories:
- coding
- miscellaneous
- life
tags:
- hugo
- gitlab
- aws
- golang
- holidays
- styles
draft: true
---

### Intro
In the last 50 days I had to work a lot for many... many different reasons. The main ones:

- I was accepted as a Speaker at FullStackConf19 in Turin, talking about coding in mobility. You can find the slide of my speech [here](https://fullstackconf.it/recap.html) and the material I prepared the talk in this [Github](https://github.com/made2591/fullstackconf19) repo. By the way, I was truly inspired by some of the talks during the conference, and I started brainstorming around the next post;
- I moved back to Italy and trust me -> it was a pretty complex goal to achieve, with particular regards to my car;
- I joined [Enerbrain](https://www.enerbrain.com/en/experts-in-energy-team-engineer-analysis-diagnosis-energy-audits-bms-thermal-energy-machine-learning/) and I'm really having fun with theme building smart-energy solution as a Devops Engineer;
- I joined a softball team - yes, it's mixed, but it's officially played also by men hahe. Let me say that it's not the usual kind of team, it's more a sort of "group of old friends playing drinking and..." overall having fun together;

What else... oh yes, I migrated my blog to [Hugo](https://gohugo.io) and I also migrated my release pipeline to [Gitlab](https://gitlab.com).

So yes...

<div class="img_container"><img src="https://i.imgur.com/QhY4REF.png" style="width: 100%; marker-top: -10px;"/></div>
<!-- <div class="img_container"><img src="https://i.imgur.com/IFjmF9q.jpg" style="width: 100%; marker-top: -10px;"/></div> -->

### What about Hugo

Hugo is one of the most popular open-source static site generators and the first reason I moved to it is... nothing, just curiosity but it's truly fast. Compared to Jekyll, my build reduce by a factor of ~200.

<div class="img_container"><img src="https://i.imgur.com/2ZDBKiD.png" style="width: 100%; marker-top: -10px;"/></div>

Yes, with Jekyll I was around 1 minute and 40 seconds of build, and I truly don't know why: Hugo instead is really fast.

### Step 0: install

The instructions are really simple and straightforward: just follow the setup [here](https://gohugo.io/getting-started/quick-start/) - long story short, install with `brew`

### Step 1: scaffold

Scaffold your folder structure with

```
hugo new site blog
```

### Step 2: contents

The contents are disposed inside the folder `content` - what a shame. Hugo assumes that the same structure that works to organize your source content is used to organize the rendered site, so you can easily define categories-taxonomy by just follow the folder-pattern:

```
content/<CATEGORY>/<FILE>.<FORMAT>
```

and... stop.

Furthermore, if you come like me from Jekyll you can easily sed your files as I did to replace things like:

```
...
categories: [theory, docker]
tags: [container, docker, security, informative, hacking]
...
```

with something like:

```
...
categories:
- theory
- docker
tags:
- container
- docker
- security
- informative
- hacking
...
```

in the header of your markdown posts. There's only one special file you need to know about and it's `_index.md` file: this file has a specific meaning and usage in Hugo when it comes to rendering your site homepage, section page, taxonomy list, or taxonomy terms list. These things are defined under the [list of content](https://gohugo.io/templates/lists/) section in Hugo docs.

In a few words, this file lets you define scaffolding templates to list things. The real cool thing of Hugo is that it's full of default: in fact if Hugo does not find an _index.md within the respective content section when rendering a list template, the page will be created but with no `{{.Content}}` and only the default values for .Title etc.

The default behavior is defined internally in Hugo and overwritten by themes. The whole organization of content is fully described [here](https://gohugo.io/content-management/) and it's pretty simple defining basing templates for your content.

#### A note about template

Hugo uses Go's `html/template` and `text/template` libraries as the basis for the templating: this is the kind of libraries that let you - and theme builder - build your themes using mustache notation (`{{}}`) to interpolate variables and functions.

A cool feature of Hugo is the `Data template` discussed [here](https://gohugo.io/templates/data-templates/): this template lets you define, in addition to Hugo’s built-in variables, your own custom data in templates or shortcodes that pull from both local and dynamic sources. Hugo supports loading data from YAML, JSON, and TOML files located in the data directory in the root of your Hugo project, or even functions that accept variadics arguments inside the data templates file.

The following snippet

```
<ul>
  {{ $urlPre := "https://api.github.com" }}
  {{ $gistJ := getJSON $urlPre "/users/GITHUB_USERNAME/gists" }}
  {{ range first 5 $gistJ }}
    {{ if .public }}
      <li><a href="{{ .html_url }}" target="_blank">{{ .description }}</a></li>
    {{ end }}
  {{ end }}
</ul>
```

will automatically fulfill the content of the template with the data retrieved from Github API. Simply amazing how easy it can be built by storing your content somewhere else (a markdown repository or database) or build your headless CMS!

But... let's go one step back and keep it simple.

### Step 3: run and build

You can just run

```
hugo server -D
```

or

```
hugo server --buildFuture -D
```

to locally serve fast-built articles, even with post dated in the future if required. You can just `draft` your `draft` by `draft` your header 🤔 => put *#draft* in the frontmatter)

The build process is even simpler: just run

```
hugo
```

and you're done.

#### A note about themes and archetypes

You can find the right theme for you starting from [here](https://themes.gohugo.io/). The file `config.toml` contains any references or hooks to customize your theme. My config file is pretty complex, so it doesn't worth go through it.

Another cool concept of Hugo are archetypes: archetypes are content template files in the archetypes directory of your project that contains preconfigured front matter and possibly also a content-disposition for your website’s content types. These will be used when you run `hugo new`.

The Hugo new uses the content-section to find the most suitable archetype template in your project. If your project does not contain any archetype files, it will also look into the theme.

Depending on the theme, you can leverage the power of custom archetypes that automatically scaffold your site content.

### Gitlab: a renewed release pipeline

Even my release pipeline became simpler: I can just refer to `registry.gitlab.com/pages/hugo:0.55.6` to build and deploy with cache invalidation. The key parts are:

```
stages:
  - build
  - deploy
```

that simply defines the two steps I need to run across.

```
variables:
  AWS_DEFAULT_REGION: $AWS_DEFAULT_REGION
  BUCKET_NAME: $BUCKET_NAME
  CLOUDFRONT_DIST_ID: $CLOUDFRONT_DIST_ID
  GIT_SUBMODULE_STRATEGY: recursive
```

that are injected from Gitlab and used by my `aws cli` to run the steps required.

```
build:
  image: registry.gitlab.com/pages/hugo:0.55.6
  stage: build
  script:
  - hugo
  artifacts:
    paths:
    - public
  only:
  - master
```

that define my building script using the Hugo image inside Gitlab official registry. I hope they will keep it the image up to date, but honestly... I think it could be even better referring to the official one of Hugo inside docker hub ([link](https://hub.docker.com/r/jguyomard/hugo-builder/)). And... last but not the least,

```
deploy:
  image: "python:latest"
  stage: deploy
  dependencies:
    - build
  before_script:
    - pip install awscli
  script:
    - aws configure set preview.cloudfront true
    - aws s3 sync ./public s3://$BUCKET_NAME --delete;
    - aws cloudfront create-invalidation --distribution-id $CLOUDFRONT_DIST_ID  --paths "/*";
  only:
  - master
```

my deploy script, super simple: sync built content, invalidate the cache.

### Conclusion
What can I say more... thank you Hugo! And thank you Gitlab! And thank you for reading: if you liked this post, feel free to share it!
