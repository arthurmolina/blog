---
title: "OpenLayers and Google Maps vs. Internet Explorer"
lang: en
last_modified_at: 2012-03-19T16:00:00-03:00
categories:
  - articles
tags:
  - code
  - openlayers
  - php
show_overlay_excerpt: false
header:
  image: /assets/images/2012/openlayers.jpg
  overlay_image: /assets/images/2012/openlayers.jpg
  overlay_filter: 0.15
  show_overlay_excerpt: false
---

I was developing my website with maps using [OpenLayers](http://www.openlayers.org/) as a framework and, obviously, testing and improving everything in Firefox. The reason is very simple: Firebug. This tool greatly improves the life of a front-end developer! I think that later I will do an evaluation of browser debuggers. But that's for another post.

Even though Firefox already has a debugger built in and they all work with the same message warning command (console.log and others), I can't seem to let go of Firebug. This also has its downside because when I need to test my application in another browser I get lost. Even Internet Explorer has already incorporated its debugger tool, and there is no way to include a Firebug Lite like Chrome (or is there?). So I have to use it anyway. Despite having very interesting functions, such as the possibility of testing the page on previous versions, overall I find it very weak...

But back to the subject: OpenLayers. I'm using several Basemaps (which are the maps that are underneath the layers you're going to work on. It's only allowed to show one basemap at a time, after all, they take up all the space and it wouldn't even be possible to see more than one at a time, unless unless we used transparency (I will test this possibility later)…

But here's the problem. I'm using Google Maps, Google Satellite, Google Hybrid, Google Terrain and OpenStreetMap as a base, the latter being the default layer. In Firefox, they worked fine. When I tested the application in other browsers, they opened in OpenStreetMap (by default) and it also seemed to work.

One fine day (later it wasn't so beautiful) I'm going to do a demonstration of the application and the machine I was going to show didn't have Firefox, just IE 9 (at least it was 9!) and the other base layers (those from Google ) simply did not appear. After the embarrassment I experienced in that demonstration, I went looking for the reason for this and couldn't find anything on the subject. I updated the OpenLayers version, at the time, to 2.11 and nothing. I looked inside the OpenLayers.js, inside the stylesheets and nothing… I gave up after searching so much for the reason.

When I was at the end of the development of this project, about to go into production, I committed myself once again to solving this bug. Version 2.12 of OpenLayers had just come out of the oven and, thinking that this bug would be resolved, I updated once again. And once again, nothing.

Another good time of research when I found the solution on a [Drupal forum](https://www.drupal.org/node/1364304): to fix the problem I just needed to replace the blank.gif file that was defective and prevented Google images from being displayed.

Just replacing this file in the OpenLayers images made Google Maps appear in IE like magic.

But there are some questions:

- What was in the previous blank.gif that prevented it from being viewed?

- Why did this problem only occur in IE?

- How could I waste so much time on something so foolish?