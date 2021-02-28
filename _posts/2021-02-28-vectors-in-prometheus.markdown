---
published: true
layout: post
title: "Prometheus: instant and range vectors"
date: 2021-02-28 20:00:00 +0100
categories: databases
excerpt: >-
  Prometheus offers a number of useful functions to work with time series data. While it gives a great understanding of application, it offers statistic data instead of exact one. Let's check why and how to get the maximum out of it.
---

{%
    include image-ref.html
    file='20210228/cover.jpg'
    authorUrl='https://unsplash.com/@mbaumi'
    authorName='Mika Baumeister'
    siteUrl='https://unsplash.com/photos/Wpnoqo2plFA'
    siteName='Unsplash'
%}

<span>Photo by <a href="https://unsplash.com/@mbaumi?utm_source=unsplash&amp;utm_medium=referral&amp;utm_content=creditCopyText">Mika Baumeister</a> on <a href="https://unsplash.com/s/photos/numbers?utm_source=unsplash&amp;utm_medium=referral&amp;utm_content=creditCopyText">Unsplash</a></span>

Prometheus is a great tool for monitoring and statistics. It has a good performance, can handle and aggregate a huge amount of data. To be able to effectively use this tool we need to understand different types of vectors.

### Instant vectors

Let's check the definition of an instant vector from [prometheus documentation](https://prometheus.io/docs/prometheus/latest/querying/basics/):

> **Instant vector** - a set of time series containing a single sample for each time series, all sharing the same timestamp.

To understand what is it about, we need to draw some metrics. Let's take `logback_events_total` which is counting the number of logs produced by the application. That metric is added by default by Actuator in Spring Boot application. The application can log errors, infos, etc. Because of that counter has an additional dimension `level`. It represents the severity of a log message. It means that metric `logback_events_total` has a **set of time series**, where each represents a log severity.

{%
    include image.html
    file='20210228/01-instant-vector.png'
    alt='Instant vector in prometheus'
%}

In the picture, we see our set of 5-time series where for every timestamp each time series has **a single value**. That data can be easily transformed into a graph representation. Only instant vectors can be transformed into a graph. But aggregation and display of instant vectors can be misleading.

For example, let’s keep the only infos. We rarely interesting in the absolute values of counters, we rather would like to understand the number of occurrences over some time. Let’s take infos from our logs and calculate the increase between the values.

{%
    include image.html
    file='20210228/02-increase-1.png'
    alt='Increase values'
%}

But because of a scale, our visualization tool can display only even timestamps.

{%
    include image.html
    file='20210228/03-increase-2.png'
    alt='Increase values'
%}

Having only values for even timestamps would be completely misleading. We need to find a way to be able to influence the aggregation values.

### Range vectors

To provide a possibility for users to control the aggregations, prometheus has introduced range vectors. Let's check the definition of an instant vector from [prometheus documentation](https://prometheus.io/docs/prometheus/latest/querying/basics/):

> **Range vector** - a set of time series containing a range of data points over time for each time series.

Let’s visualize an example of logs. To simplify let’s take the only infos. In the end, a set can contain only one element. To make a range vector out of instant, we simply need to add a time range.

> `logback_events_total{level="info"}[3m]`

The result would be:

{%
    include image.html
    file='20210228/04-range-vector.png'
    alt='Increase values'
%}

We see that time series has a range of data points for every timestamp. It simply assigns all values from the `(timestamp - time range, timestamp]` range to that timestamp. With a time range in the brackets, you can configure how many data points you want to include in the range. You cannot display it on the graph, but you can transform it back to instant vector through aggregation functions like `delta`, `deriv`, `rate`.

Increasing of time range smooth the graph. For example, the same time series with time range 1 minute:

{%
    include image.html
    file='20210228/05-graph1.png'
    alt='Increase values'
%}

and 2 minutes:

{%
    include image.html
    file='20210228/06-graph2.png'
    alt='Increase values'
%}

You can see both graphs has the same trends, second is just smoothed.

> Prometheus does not provide exact numbers, it provides an overview and an understanding of what is happening in the application.

Read more about  ([prometheus functions](https://prometheus.io/docs/prometheus/latest/querying/functions/)).
