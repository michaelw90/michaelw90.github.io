---
layout: post
title:  "Hitting 100% in Lighthouse Accessibility Tests"
author: michael
categories: [ development ]
image: assets/images/posts/100-percent-in-lighthouse-accessibility.jpg
thumb: assets/images/posts/100-percent-in-lighthouse-accessibility-thumbnail.jpg
comments: false
featured: false
excerpt: Sometimes, hitting that final few percentage can be tricky, here's the tools I used to hit 100%.
---

#### Preface

Lighthouse is a tool that Google created, and subsequently integrated into the Google Chrome developer tools. It runs an audit of a website, and returns a ranking out of 100 for 4 categories - Performance, Accessibility, Best Practices and SEO. When building personal sites, I like to make sure that they're as close to 100/100/100/100 in all of the categories as possible. Depending on the recommendations for Performance improvements, I'll happily forgo a few percentage points on that.

##### Performance is not Quite 100

Before I jump into how I hit 100 on the Accessibility side of things, I'd like to point out that currently (depending on how you run the test) the site is sitting at 97 or 96 in Performance, with a 100 point allocation for the three other categories.

The main complaint that is returned in the Performance category is the fact that there is no Cache Policy on the assets (images/CSS) that are used within the site. This is, unfortunately, a negative side effect of hosting directly with GitHub Pages, as they have a maximum cache lifetime of just 10 minutes, and it's not configurable to anything else on othe assets.

In this scenario the effort involved (both in setup, and longer term content management) is too high to warrant implementation at this point in time as it'll require setting up a CDN, or similar for serving the content. This conflicts with the goal for this site, which was to serve everything (at low cost) straight from GitHub pages instead.

So, in this case, I can accept the slight decrease in points on Performance.

##### Accessibility to 100!

One of the key things that people forget when it comes to Accessibility is colour differentiation for colourblind users, or users with bad eyesight. As someone that wears glasses I've seen this in extreme cases during presentations etc, but not really experienced it on day-to-day device usage, so I was a bit unaware of it.

Running the Lighthouse test for the site returned the helpful error message of:
>"Background and foreground colors do not have a sufficient contrast ratio."

On the surface, it makes sense, it's clear, it's pretty helpful for identifying the issue. But I had absolutely no idea how to fix it, and short of changing all the colours to 100% black I was confused.

After some research, I came across [a very handy article](https://web.dev/color-contrast/) by the Google team themselves on [web.dev](https://web.dev) which outlines an [amazing feature that's built into Google Chrome's Developer Tools](https://web.dev/color-contrast/#how-to-ensure-text-has-sufficient-color-contrast) that will tell you for each image, whether it's got a sufficient contrast ratio. Using this, I was able to toggle some colours in the colour picker, while looking at the UI to ensure it wasn't going too monotone and pick colours that were 100% accessible.

##### It's Never Been This Easy

I think it's worth pointing out that given all of the tools that are available to developers to better run audits, and get helpful information about how to solve the issues, there's very little reason for websites being poor from an Accessibility perspective.

Understandably some of the contrast decisions, and similar, come in an earlier design and branding stage so might be more ingrained, but there are tools out there. 

The internet is supposed to be open and accessible to all, and using these tools developers are helping push forward for a more open, accessibile web.