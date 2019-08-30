---
layout: post
title: "Smart SPA Shower at home"
categories: [coding, js, miscellaneous, life]
tags: [coding, js, smarthome, life, shower, spa]
---

### Preamble
I recently bought 4 small smart bulbs - the latest one you most probably decide to buy for your smart home 😂😂 I think it's useless talk about what you can do: I will only focus on the important things.

- They DON'T need an hub;
- They support Alexa;
- They support Google Assistant;
- They support IFTTT;
- There is an app, called Smart Life ([iOS](https://itunes.apple.com/us/app/smart-life-smart-living/id1115101477?mt=8), [Android](https://play.google.com/store/apps/details?id=com.tuya.smartlife&hl=it))

But most important you can build your small SPA in your bathroom. If you are interested, go ahead!

<div class="img_container"><img src="https://i.imgur.com/URCh7Qx.jpg" style="width: 100%; marker-top: -10px;"/></div>

And no, I will not transform your bathroom in the one shown in the picture: and no, that is not my bathroom unfortunately XD

### What you need
Before going ahead with this, this is what you need:

- [Smart Alexa Lamp, Maxcio Wifi Smart Lamp, 7W E27 RGB + W Multicoloured and Dimmable Light, Remote Control via App, Compatible with Amazon Alexa and Google Home](https://www.amazon.de/gp/product/B07JMR3ZZ9/ref=ppx_yo_dt_b_asin_title_o00__o00_s00?ie=UTF8&psc=1) or any other product that use [Tuya](http://tuya.com/) cloud network to communicate. This is a required condition to work with [tuyapi](https://github.com/codetheweb/tuyapi), a library for communicating with devices that use the Tuya cloud network. These devices are branded under many different names, but if your device works with the TuyaSmart app or port 6668 is open on your device chances are this library will work.
- A Google Home (mini or not) - or my [Gist](https://gist.github.com/made2591/bca41ce13cced70bcb4c1712801726e3) with the colors mapped already for you (it should work, hopefully), otherwise you should reverse them manually by changing the color of the bulb with the Google Home app, running the server, getting the status of the lamp and save the data retrieved;
- A Node.js server - who doens't have one today?! just grub a raspberry or you can even use your laptop, it will definetly not be a 24/7 shower;

### Setup your bulb
Buy your bulb - pay attention to choose the correct socket for your lamp! - and setup your bulb with Smart Life. Then, sync your device with google home by asking him to "sync device". I suggest to not setup rooms inside the app to avoid collision with rooms you can easily and more efficiently setup in your Google home app.

### The ID, KEY and IP
Before setup your node server, you need to discover some hidden information about your bulb. To retrieve needed ID and KEY of your bulb, just follow the instruction given [there](https://github.com/codetheweb/tuyapi/blob/master/docs/SETUP.md). Everywhere it is suggested to fix an IP to your bulb.

### How to discover the colors
I retrieved them by looking at the change of status after every click over the colors available in the Google Home application
<div class="img_container"><img src="https://i.imgur.com/azcDhAQ.jpg" style="width: 30%; marker-top: -10px;"/></div>

The White color looks like that:

{% highlight json %}
{
    "1":true,
    "2":"white",
    "3":255,
    "4":255,
    "5":"fffafa000005ff",
    "6":"00ff0000000000",
    "7":"ffff500100ff00",
    "8":"ffff8003ff000000ff000000ff000000000000000000",
    "9":"ffff5001ff0000",
    "10":"ffff0505ff000000ff00ffff00ff00ff0000ff000000"
}
{% endhighlight %}

The Color looks like that:

{% highlight json %}
{
    "1":true,
    "2":"colour",
    "3":255,
    "4":255,
    "5":"f8f8ff00f007ff",
    "6":"00ff0000000000",
    "7":"ffff500100ff00",
    "8":"ffff8003ff000000ff000000ff000000000000000000",
    "9":"ffff5001ff0000",
    "10":"ffff0505ff000000ff00ffff00ff00ff0000ff000000"
},
{% endhighlight %}

If you need to discover yours, or want to experiment with voltage etc, you can do it by changing the [JSON](https://gist.github.com/made2591/bca41ce13cced70bcb4c1712801726e3) just in case.

### Wrap everything in a loop
And now my stupid and bad written code to give your bathroom some colors in a loop of 3 seconds over the colors file provided above ([Gist](https://gist.github.com/made2591/bca41ce13cced70bcb4c1712801726e3) just in case)


{% highlight javascript %}
const TuyAPI = require('tuyapi');
var sleep = require('sleep');

const device = new TuyAPI({
  id: "<YOUR_BULB_ID>",
  key: "<YOUR_BULB_KEY>",
  ip: "<YOUR_IP_KEY>",
  persistentConnection: true});

device.on('connected',() => {
  console.log('Connected to device.');
});

device.on('disconnected',() => {
  console.log('Disconnected from device.');
});

device.on('data', data => {
  console.log('Data from device:', data);
});

device.on('error',(err) => {
  console.log('Error: ' + err);
});

device.connect();

var fs = require('fs');
var colours = JSON.parse(fs.readFileSync('./colours.json', 'utf8'));

function applyColour(info) {
  return device.set({
    multiple: true,
    data: {
      '1': true,
      '2': info["2"],
      '5': info["5"]
      }
    }
  )
}

const funcs = colours.map(info => () => applyColour(info))

const promiseSerial = funcs =>
  funcs.reduce((promise, func) =>
    promise.then(result => func().then(Array.prototype.concat.bind(result), sleep.sleep(3))),
    Promise.resolve([]))

promiseSerial(funcs)
  .then(console.log.bind(console))
  .catch(console.error.bind(console))

setTimeout(() => { device.disconnect(); }, 1000000);
{% endhighlight %}

### One more thing
To have a more relaxing experience you can even listen to [The Relaxing Sounds of Swedish Nature](spotify:artist:3yQUKaHkSwdGxlk8LxN5iu) with your Google Home or any other device you want!

Thank you everybody for reading and have a good shower!!
