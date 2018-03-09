# predictive-fetching
Improve performance by predictively fetching pages a user is likely to need

## Overview

This proposal outlines an idea for applying a data-driven approach to optimizing loading performance on the web. By building a model of pages a user is likely to visit, given an arbitrary entry-page, a solution could calculate the liklihood a user will visit a given next page or set of pages and prefetch resources for them while the user is still viewing their current page. This has the possibility of improving page-load performance for subsequent page visits as there's a strong chance a page will already be in the user's cache.

## Approach 

In order to predict the next page a user is likely to visit, a solution could use the [Google Analytics API](https://developers.google.com/analytics/devguides/reporting/core/v4/). Google Analytics session data can be used to create a model to predict the most likely page a user is going to visit next on a site. The benefit of this session data is that it can evolve over time, so that if particular navigation paths change, the predictions can stay up to date too. 

With the availability of this data, an engine could insert `<link rel="[prerender/prefetch/preload]">` tags to speed up the load time for the next page request. In some tests, such as Mark Edmondson's [Supercharging Page-Loads with R](http://code.markedmondson.me/predictClickOpenCPU/supercharge.html), this led to a 30% improvement in page load times. The approach Mark used in his research involved using GTM tags and machine-learning to train a model for page predictions. This is an idea Mark continued in [Machine Learning meets the Cloud - Intelligent Prefetching](https://iihnordic.com/blog/machine-learning-meets-the-cloud-intelligent-prefetching/).

While this approach is sound, the methodology used could be deemed a little complex. Another approach that could be taken (which is simpler) is attempting to get accurate prediction data from the Google Analytics API. If you ran a report for the [Page](https://developers.google.com/analytics/devguides/reporting/core/dimsmets#view=detail&group=page_tracking&jump=ga_pagepath) and [Previous Page Path](https://developers.google.com/analytics/devguides/reporting/core/dimsmets#view=detail&group=page_tracking&jump=ga_previouspagepath) dimension combined with the [Pageviews](https://developers.google.com/analytics/devguides/reporting/core/dimsmets#view=detail&group=page_tracking&jump=ga_pageviews) and [Exits](https://developers.google.com/analytics/devguides/reporting/core/dimsmets#view=detail&group=page_tracking&jump=ga_exits) metrics this should provide enough data to wire up prefetches for most popular pages.

### Machine Learning for predictive fetching?

ML could help improve the overall accuracy of a solution's predictions, but is not a necessity for an initial implementation. Predictive fetching could be accomplished by training a model on the pages users are likely to visit and improving on this model over time. 

Deep neural networks are particularly good at teasing out the complexities that may lead to a user choosing one page over another, in particular, if we wanted to attempt a version of the solution that was catered to the pages an individual user might visit vs. the pages a "general/median" user might visit next. Fixed page sequences (prev, current, next) might be the easiest to begin dealing with initially. This means building a model that is unique to your set of documents. 

Model updates tend to be done periodically, so one might setup a nightly/weekly job to refresh based on new user behaviour. This could be done in real-time, but is likely complex, so doing it periodically might be sufficient. One could imagine a generic model representing behavioural patterns for users on a site that can eitehr be driven by a trained status set, Google Analytics, or a custom description you plugin using a new layer into a router giving the site the ability to predictively fetch future pages, improving page load performance.

## Risks

### Data consumption

As with any mechanism for prefetching content ahead of time, this needs to be approached very carefully. A user on a restricted data-plan may not appreciate or benefit as much from pages being fetched ahead of time, in particular if they start to eat up their data. There are mechanisms a site/solution could take to be mindful of this concern, such as respecting the [Save-Data](https://developers.google.com/web/updates/2016/02/save-data) header. 

### Web Standards

Some of the attempts to accomplish similar proposals in the past have relied on `<link rel=prerender>`. The Chrome team is currently exploring [deprecating rel=prerender](https://groups.google.com/a/chromium.org/forum/#!topic/blink-dev/0nSxuuv9bBw) in favor of [NoStatePrefetch](https://docs.google.com/document/d/16VCYGGWau483IMSxODpg5faZny1FJ6vNK2v-BuM5EhU/edit#) - a lighter version of this mechanism that only prefetches to the HTTP cache but uses no other state of the web platform. A solution should factor in whether it will be relying on the replacement to rel=prerender or using prefetch/preload/other approaches.


## Prior work

* [Supercharging page-loads with R](http://code.markedmondson.me/predictClickOpenCPU/supercharge.html)
* [Using Google Analytics to predict clicks](https://www.noisetosignal.io/2016/11/using-google-analytics-to-predict-clicks-and-speed-up-your-website/)
* [Gatsby's Link](https://github.com/gatsbyjs/gatsby/tree/master/packages/gatsby-link) - relies on links entering in the viewport before it attempts to preload them.
* [Eve Dynamic Prerender](https://wordpress.org/plugins/eve-dynamic-prerender/)
* [InstartLogic - Multi-page Predictive Prefetching](https://www.instartlogic.com/blog/predicting-future-multi-page-predictive-prefetching)




