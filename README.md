# predictive-fetching
Improve performance by predictively fetching pages a user is likely to need

## Overview

This proposal outlines an idea for applying a data-driven approach to optimizing loading performance on the web. By building a model of pages a user is likely to visit, given an arbitrary entry-page, a solution could calculate the likelihood a user will visit a given next page or set of pages and prefetch resources for them while the user is still viewing their current page. This has the possibility of improving page-load performance for subsequent page visits as there's a strong chance a page will already be in the user's cache.

## Approach 

In order to predict the next page a user is likely to visit, a solution could use the [Google Analytics API](https://developers.google.com/analytics/devguides/reporting/core/v4/). Google Analytics session data can be used to create a model to predict the most likely page a user is going to visit next on a site. The benefit of this session data is that it can evolve over time, so that if particular navigation paths change, the predictions can stay up to date too. 

With the availability of this data, an engine could insert `<link rel="[prerender/prefetch/preload]">` tags to speed up the load time for the next page request. In some tests, such as Mark Edmondson's [Supercharging Page-Loads with R](http://code.markedmondson.me/predictClickOpenCPU/supercharge.html), this led to a 30% improvement in page load times. The approach Mark used in his research involved using GTM tags and machine-learning to train a model for page predictions. This is an idea Mark continued in [Machine Learning meets the Cloud - Intelligent Prefetching](https://iihnordic.com/blog/machine-learning-meets-the-cloud-intelligent-prefetching/).

While this approach is sound, the methodology used could be deemed a little complex. Another approach that could be taken (which is simpler) is attempting to get accurate prediction data from the Google Analytics API. If you ran a report for the [Page](https://developers.google.com/analytics/devguides/reporting/core/dimsmets#view=detail&group=page_tracking&jump=ga_pagepath) and [Previous Page Path](https://developers.google.com/analytics/devguides/reporting/core/dimsmets#view=detail&group=page_tracking&jump=ga_previouspagepath) dimension combined with the [Pageviews](https://developers.google.com/analytics/devguides/reporting/core/dimsmets#view=detail&group=page_tracking&jump=ga_pageviews) and [Exits](https://developers.google.com/analytics/devguides/reporting/core/dimsmets#view=detail&group=page_tracking&jump=ga_exits) metrics this should provide enough data to wire up prefetches for most popular pages.

### Machine Learning for predictive fetching

ML could help improve the overall accuracy of a solution's predictions, but is not a necessity for an initial implementation. Predictive fetching could be accomplished by training a model on the pages users are likely to visit and improving on this model over time. 

Deep neural networks are particularly good at teasing out the complexities that may lead to a user choosing one page over another, in particular, if we wanted to attempt a version of the solution that was catered to the pages an individual user might visit vs. the pages a "general/median" user might visit next. Fixed page sequences (prev, current, next) might be the easiest to begin dealing with initially. This means building a model that is unique to your set of documents. 

Model updates tend to be done periodically, so one might setup a nightly/weekly job to refresh based on new user behaviour. This could be done in real-time, but is likely complex, so doing it periodically might be sufficient. One could imagine a generic model representing behavioural patterns for users on a site that can eitehr be driven by a trained status set, Google Analytics, or a custom description you plugin using a new layer into a router giving the site the ability to predictively fetch future pages, improving page load performance.

## Risks

### Data consumption

As with any mechanism for prefetching content ahead of time, this needs to be approached very carefully. A user on a restricted data-plan may not appreciate or benefit as much from pages being fetched ahead of time, in particular if they start to eat up their data. There are mechanisms a site/solution could take to be mindful of this concern, such as respecting the [Save-Data](https://developers.google.com/web/updates/2016/02/save-data) header. 

### Web Standards

#### Future of rel=prerender
Some of the attempts to accomplish similar proposals in the past have relied on `<link rel=prerender>`. The Chrome team is currently exploring [deprecating rel=prerender](https://groups.google.com/a/chromium.org/forum/#!topic/blink-dev/0nSxuuv9bBw) in favor of [NoStatePrefetch](https://docs.google.com/document/d/16VCYGGWau483IMSxODpg5faZny1FJ6vNK2v-BuM5EhU/edit#) - a lighter version of this mechanism that only prefetches to the HTTP cache but uses no other state of the web platform. A solution should factor in whether it will be relying on the replacement to rel=prerender or using prefetch/preload/other approaches.

There are two key differences between NoStatePrefetch and Prefetch

1. nostate-prefetch is a mechanism, and `<link rel=prefetch>` is an API. The nostate-prefetch can be requested by other entry points: omnibox prediction, custom tabs, `<link rel=prerender>`.

2. The implementation is different: `<link rel=prefetch>` prefetches one resource, but nostate-prefetch on top of that runs the preload scanner on the resource (in a fresh new renderer), discovers subresources and prefetches them as well (without recursing into preload scanner).

## Methods of predictively fetching content

### Speculative prefetch on page load

Speculative prefetch can prefetch pages likely be navigated to on page load. This assumes the existence of knowledge about the probability a page will need a certain next page or set of pages, or a training model that can provide a data-driven approach to determining such probabilities.

Prefetching on page load can be accomplished in a number of ways, from deferring to the UA to decide when to prefetch resources (e.g at low priority with `<link rel=prefetch>`), during page idle time (via `requestIdleCallback()`) or at some other interval. No further interaction is required by the user. 

### Speculative prefetch when links come into the viewport

A page could speculatively begin prefetching content when links in the page are visible in the viewport, signifying that the user may have a higher chance of wanting to click on them.

This is an approach used by Gatsby (which uses React and React Router). Their specific implementation is as follows:

* In browsers that support IntersectionObserver, whenever a `<Link>` component becomes invisible, the link "votes" for the page linked to to be prefetched votes are worth slightly less points each time so links at the top of the page are prioritized over ones lower down 
* e.g. the top nav if a page is linked to multiple times, its vote count goes higher the prefetcher takes the top page and starts prefetching resources. 
* It's restricted to prefetching one page at a time so as to reduce contention over bandwidth with on page stuff (not a problem on fast networks. If a user visits a page and its resources haven't been fully downloaded, prefetching stops until the page is loaded to ensure the user waits as little time as possible.

### Speculative prefetch on user interaction

A page could begin speculatively prefetching resources when a user indicates they are interested in some content. This can take many forms, including when a user chooses to hover over a link or some portion of UI that would navigate them to a separate page. The browser could begin fetching content for the link as soon as there was a clear indication of interest. This is an approach taken by JavaScript libraries such as [InstantClick](http://instantclick.io/).

## Prior work

* [Supercharging page-loads with R](http://code.markedmondson.me/predictClickOpenCPU/supercharge.html)
* [Using Google Analytics to predict clicks](https://www.noisetosignal.io/2016/11/using-google-analytics-to-predict-clicks-and-speed-up-your-website/)
* [Gatsby's Link](https://github.com/gatsbyjs/gatsby/tree/master/packages/gatsby-link) - relies on links entering in the viewport before it attempts to preload them.
* [Eve Dynamic Prerender](https://wordpress.org/plugins/eve-dynamic-prerender/)
* [InstartLogic - Multi-page Predictive Prefetching](https://www.instartlogic.com/blog/predicting-future-multi-page-predictive-prefetching)
* [Sirko Engine](https://github.com/sirko-io/engine) - relies on Service Worker




