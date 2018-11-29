---
layout: post
title:  "Using iPhoneInstallOverlay plugin in your progressive web-app to show add to home screen banner in iPhone Safari"
date:  2016-07-31 00:34:00
intro: "A JavaScript plugin which shows a message to install the web application on iPhone devices(on Safari browser). This will be useful if you are building a progressive web application. If you don't know what progressive web applications are, head over to Progressive web apps blog by Google here: https://developers.google.com/web/progressive-web-apps/"
categories: front-end
---

### What is it?

A JavaScript plugin which shows a message to install the web application on iPhone devices(on Safari browser). This will be useful if you are building a progressive web application. If you don't know what progressive web applications are, head over to [Progressive web apps blog by Google](https://developers.google.com/web/progressive-web-apps/)

<br/>

### Why is it required

Though service workers are not yet supported on Safari, we can still achieve a lot using HTML5 application cache and localStorage. And Safari does support launching your application in a full screen mode using just a meta-tag. (Refer [Safari meta-tags](https://developer.apple.com/library/iad/documentation/AppleApplications/Reference/SafariHTMLRef/Articles/MetaTags.html) to know more).

This is a feature chrome has recently released where it checks that if your web app is using service workers and shows a banner to install the application to the home screen. [Official blog by Google](https://developers.google.com/web/updates/2015/03/increasing-engagement-with-app-install-banners-in-chrome-for-android?hl=en)

#### I believe that this is a really cool feature to have and I thought of bringing the same feature to the iOS devices.

<br/>

### Using the plugin

The plugin is available as a bower component and an NPM module. You can install it in an application by either

```javascript
bower install iphone-install-overlay
```
or

```javascript
npm install iphone-install-overlay
```

Using the plugin is pretty straight forward. The plugins exposes a global variable called `iPhoneInstallOverlay`.

Then you need to call the init method and pass the options.

```javascript
iPhoneInstallOverlay.init({
  blurElement: '.view-container',
  appIconURL: './assets/images/login/app-icon.png',
  spritesURL: './assets/images/mobile-sprite.png',
  appName: 'Awesome Progressive app'
});
```
<br/>

| Option   |      Required      |||Explanation    |
|----------|:------------------:|||:--------------|
| blurElement | YES ||| The element on which `blur` class will be applied. (The div containing the whole view of the application) |
| appIconURL |    NO   |||   URL to the  application icon.  |
| spritesURL | NO ||| URL to the sprites containing png icons. Its there in `dist/mobile-sprites.png` |
| appName | NO |||  The application name (Your App Name by default) |
| showOnReload | NO ||| Show overlay on reload or show only for the first time |
| previewText | NO ||| Text to show to skip the overlay (Preview in browser by default) |

<br/>

You can also control the visibility of the overlay after initialising the plugin using the init() method using

```javascript
iPhoneInstallOverlay.showOverlay();
```
and

```javascript
iPhoneInstallOverlay.hideOverlay();
```
<br/>

### Screenshots and Demo
Working demo of the plugin: [http://rahulgaba.com/iPhone-install-overlay](http://rahulgaba.com/iPhone-install-overlay)

![App overlay](/static/img/iphone_overlay/dummy_overlay.gif "App overlay"){:class="post-img"}
![Installed app](/static/img/iphone_overlay/dummy_app.png "Installed app"){:class="post-img"}
![Full GIF](/static/img/iphone_overlay/app.gif "Full GIF"){:class="post-img"}
{: style="text-align: center;"}
<br/>

#### Please go ahead and use this plugin in your next progressive web app and feel free to raise [issues/pull-requests](https://github.com/rahulgaba16/iPhone-install-overlay)
