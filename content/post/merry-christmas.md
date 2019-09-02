---
layout: post
title: "Best wishes for Christmas holidays!"
date: 2017-12-24
categories:
- coding
- miscellaneous
- life
tags:
- coding
- holidays
- styles
- algorithm
- christmas
---

### Merry Christmas by nerds
Here we are!! Christmas holidays are coming and it's time to collect nerd greetings for nerd friends! In this article I have collected some of my favorites: starting from css to the most unhealthy c code in the world, I hope you enjoy these repositories / snippets / gist / sketch! Best wishes for happy holidays!

#### CSS
While I was looking for some cool css stuff to share with you, I found this fantastic gist that show the kind of magic a web expert can do with a bunch of css lines. Riddle: what do you see if you use __IE__ as a browser? The answer after the preview!

<div class="img_container"><img src="https://i.imgur.com/nNZpWOJ.png" style="width: 100%; marker-top: -10px;"/></div>

The code to reproduce this nice Santa Claus is available in [this](https://gist.github.com/narendrashetty/61dc353c42aedc23473c) gist: it's a really simple gist, with one html and one css file!
Answer to the riddle: a Halloween pumpkin! 😂😂😂😂

#### Javascript
[anvaka](https://github.com/anvaka) created the fantastic tree in the figure below using only 11 lines of javascript codes! Have a look at his [repo](https://github.com/anvaka/atree)

<div class="img_container"><img src="https://i.imgur.com/tDTI2T4.gif" style="width: 100%; marker-top: -10px;"/></div>

For the most curious, the lines used to build the spiral are the ones below:

{{< highlight javascript >}}
return function(i) {
  var zoff = i * Math.sin(i),
    z = dz / (dz - sign * zoff * zScale),
    x = getX(i, z, sign),
    y = getY(i * yLocalScale, z);

  if (zoff + sign * Math.PI / 4 < 0) {
    switchColor(foreground);
  } else {
    switchColor(background);
  }
  ctx.moveTo(x, y);
  ctx.lineTo(getX(i + 0.03, z, sign), getY((i + 0.01) * yLocalScale, z));
};
{{< / highlight >}}


#### The DOM Trick
[hakimel](https://github.com/hakimel) shared a tree created using _DOM_ elements: you definetly have a look at the live [demo](https://lab.hakim.se/domtree/), it's amazing! The repo is available [here](https://github.com/hakimel/DOM-Tree).

#### Best whishes
[soyuka](https://github.com/soyuka) thinks that instead of offering a gift to everyone, it's fun to offer to some random member of your family (or friends). Have a look to its [repo](https://github.com/soyuka/noel)

#### ASCII Art in cli
In a contest organised by polish nerd forum during Christmas in 2016, [repo](https://github.com/plkpiotr) wrote an application that creates Christmas card based on ASCII Art!

<div class="img_container"><img src="https://i.imgur.com/jabP4rw.jpg" style="width: 100%; marker-top: -10px;"/></div>

Original repo [here](https://github.com/plkpiotr/christmas-tree)

#### Sublime Text Christmas Theme
If you want to customize your Sublime Text editor (vintage version, but ehy, you know), you can follow [zntfdr](https://github.com/zntfdr) instructions ⛄:
- Locate your Sublime Text Packages folder by using the menu item Preferences -> Browse Packages...
- Download (Right click, save as) and put the .tmTheme file into a new folder named Christmas - Color Theme
- Move the new folder into Sublime Text's Packages directory
- Activate the Christmas theme in Preferences -> Color Scheme...
- Enjoy! 🎁

<div class="img_container"><img src="https://i.imgur.com/fffoFbs.png" style="width: 100%; marker-top: -10px;"/></div>

The repo is available [here](https://github.com/zntfdr/Christmas)

Thank you everybody for reading!