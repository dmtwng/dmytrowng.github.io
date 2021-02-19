---
published: true
layout: post
title: "How does application monitoring work"
date: 2021-02-19 20:00:00 +0100
categories: databases
excerpt: >-
  Having multiple services running, developers should be aware what is happening there as well as what was happening yesterday and a week ago. Application monitoring is aiming to provide a solution. Let's check the main parts of it and how they are connected.
---

{%
    include image-ref.html
    file='20210219/cover.jpg'
    authorUrl='https://unsplash.com/@srd844'
    authorName='Stephen Dawson'
    siteUrl='https://unsplash.com/photos/qwtCeJ5cLYs'
    siteName='Unsplash'
%}

If you don't know what is happening in your application, you cannot know if everything is all right. Today we are gonna have a look at application monitoring.

### Parts of the application monitoring

{%
    include image.html
    file='20210219/01-system-diagram.png'
    alt='Monitoring System'
%}

To have reliable and accessible information about your services, you need following parts in your system:
 * Application. Yes, you need an application. And not only for doing a job, but the application is also a part of a monitoring system.
 * Data storage to store all monitoring data.
 * Visualization tool to have a convenient way to build dashboards.

### Application

Apart from doing the business job, an application has a small responsibility in the monitoring system. The application should  **know and be able to expose the own state and health at a current time**. It is not much, right? But without that part monitoring cannot happen.

All information about application health can be split into two main parts:
 * **System metrics**. Includes generic measures like memory usage, CPU utilization, network calls, etc. That information usually available out the box in most libraries and frameworks. For example, if you have a Spring Boot application with Actuator enabled, you have all these metrics available right away. Some systems like DataDog offers agents which can be attached to the process and collect system metrics.
 * **Application metrics**. Includes application-specific metrics which can be collected by the application itself. For example, your application is processing orders in the online shop. Probably you would like to know the number of orders that have been processed. Or you would like to have a breakdown of all orders by the delivery type. In that case, the application has to count specific events by itself. Many libraries simplify metrics calculation and a convenient way to expose it. Spring Boot uses Micrometer for that purpose.

 So the application can tell you the current state and how many orders it has been processed from the startup. But it cannot tell you how many orders have been processed over the last hour or yesterday evening. It is possible to implement something like that in the application. But it would be too much. In the end, the application should process orders. To save historical information we can use external storage.

### Storage

To have a history of the state of the application we need to store the data somewhere. For that purpose usually, time-series databases used like Prometheus or InfluxDB. For every metric, we will store a value every time interval.

{%
    include image.html
    file='20210219/02-tsdb.png'
    alt='Time Series Data Base'
%}

Of course, TSDB is a bit more complex than what you can see on the drawing. But let's start with it. In our example, we can see that at 6:00 our application processed 100 orders, and at 7:00 it was already 110. It means, during that hour 10 orders have been processed. From 7:00 till 8:00 was the busiest time, 50 orders. Based on the information storage can also provide aggregations, for instance, the delta function.

{%
    include image.html
    file='20210219/03-tsdb-delta.png'
    alt='Time Series Data Base'
%}

Of course, having state stored every hour won't give you good input. Usually, we can store the data every 15 seconds or depends on the system even more often. But the question is still open, how the data end up in the storage? Application is still responsible for exposing metrics, right? So we need one more service which will call the application every 15 seconds (scrape metrics) and put data in the storage. That is a preferable way, however, some databases support push of metrics from the application.

Also, it does not necessarily have to be a TSDB. Time series also can be stored in other types of databases. For that purpose often ClickHouse and Cassandra are used.

Nice, we have historical data safe in the storage and we can aggregate something out of it. Now would be nice to visualize it.

### Visualization

The leading project in visualizing time series data at a time is Grafana. It allows visualizing data from different data sources. Grafana has very flexible possibilities to customize and adjust dashboards and graphs.

However, some tools like DataDog and Instana provide a complete package for monitoring, including storage and visualization.

Having described the system will provide an overview of the state of the application.
