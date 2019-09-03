---
layout: post
title: "Time for a change - Part I"
date: 2019-09-02
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
In the last two months, many things happened to my life, so this is the reason I wasn't able to dedicate a lot of time to my blog. What can I say... I've been in South Africa, and I literaly lived experiences I will never forget and cannot being described. I visited the Blyde Canyon and the Rain Forest. I made many safari and I saw elephants in the morning washing and drinking from the river, giraffes, then we saw the warthogs - by the way, they are the same as Pumba in the Lion King by Disney, the lion, which was about to attack a buffalo, just next to our jeep and the white rhino, the hyena and the leopard. I ate in places with monkeys running free, I slept in a lodge - which was a castle so much more confortable then my apartment - I had showers with the windows on the savannah, and nothing else around. ... and then I went to Cape Town, I bought a beautiful engraved ostrich egg, I visited the island of the seals by boat. I stopped at many panoramic points on the coast, and I saw a breeding of ostriches, then at the tip of Good Hope, we took the funicular up to the lighthouse, we saw where the Atlantic and Indian oceans meet. I saw the south africans penguins on the beach, I stopped inside false bay and after that I saw the botanical garden under the mountains, with lots of flowers and directly connected to the forest. It was an amazing trip, but it is not the only thing it happened to me. In fact, I also decided to leave Germany...

<div class="img_container"><img src="https://i.imgur.com/IFjmF9q.jpg" style="width: 100%; marker-top: -10px;"/></div>

####
I'm planning to write about my experience soon, but today I wanna talk about some techinical changes I wanted to move with
After all the amazing experience I had in this country, and literally at the opposite side of the world, I also decided to make some changes to my infrastructure. So...

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