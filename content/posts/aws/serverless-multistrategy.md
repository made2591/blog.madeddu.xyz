---
layout: post
title: "Augment resiliency with multistage SAM deployments"
date: 2019-10-04
categories:
- coding
- aws
- devops
tags:
- coding
- aws
- sam
- multibranch
- guide
- git
draft: true
---

### Prelude

Before time began, there was the Monolite. We know not where it comes from, only that it holds the power to create worlds and fill them with troubles. That was how our race was born. For a time, we lived in harmony, but like all great power, some wanted it for good, others for evil. And so began the war - a war that ravaged our planet until it was consumed by death, and the Monolite was lost to the far reaches of space. We scattered across the galaxy, hoping to find it and rebuild our home, searching every star, every world. And just when all hope seemed lost, message of a new discovery drew us to an unknown planet called... SAM. But we were already too late.

With all our servers gone, we hadn't the chance to be as wrong as we used to be in the past. And fate has yielded its reward: a new world full of YAML files. We live in the middle of them, hiding in plain sight... but watching over them in secret... waiting... protecting. I am a Devops, and I send this message to any surviving guy taking refuge among the denial of service: I'm here... I'm here to help you.

<div class="img_container"><img src="https://i.imgur.com/BmsJDCe.jpg" style="width: 100%; marker-top: -10px;"/></div>

### Chatper 1: just kisssss

Everyone use SAM, everyone automatically deploy things, using TDD and standard release pipelines: every one in the books are cool.

In real life, instead, testing is a dream, release is a bet, production is a beta, beta is a dev, and devs just code. Furthermore, recently I debated around this crazy idea of monorepo: cool but, ehy. No.

TODO: image

Let's analyze the problem from a "more-than-one" developer point of view: you must have a double environment, as soon as your product is a customer facing product, and you should also start worrying about develop lifecycle policy and standard to follow before actually amry a team of laptops.

But the real