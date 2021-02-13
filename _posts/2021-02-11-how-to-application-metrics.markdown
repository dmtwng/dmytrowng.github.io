---
published: false
layout: post
title: "How does application monitoring work"
date: 2021-01-24 20:00:00 +0100
categories: databases
excerpt: >-
  Having multiple services running, developers should have possibility to not only to see what is happening in
  which service right now, but also check what was happening yesterday or even last week. Lets check what
  do we need to have an application monitoring up and running.
---

{%
    include image-ref.html
    file='20210211/cover.jpg'
    authorUrl='https://unsplash.com/@srd844'
    authorName='Stephen Dawson'
    siteUrl='https://unsplash.com/photos/qwtCeJ5cLYs'
    siteName='Unsplash'
%}

If you don't know what is happening in your application, you cannot know if everything is fine with it. Or, what is more important, what was happening with it last week. Today we are gonna talk about classic approach to the monitoring. It helps developers to get an understanding of what is or what was happening with the service.

### The Service

First of all, a service should know what is happening with it. A service should record everything what is happening with it. It could be a number of messages processed, threads are working on the processing, time spent, etc.
