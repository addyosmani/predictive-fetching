# predictive-fetching
Improve performance by predictively fetching pages a user is likely to need

## Overview

This proposal outlines an idea for applying a data-driven approach to optimizing loading performance on the web. By building a model of pages a user is likely to visit, given an arbitrary entry-page, a solution could calculate the liklihood a user will visit a given next page or set of pages and prefetch resources for them while the user is still viewing their current page. This has the possibility of improving page-load performance for subsequent page visits as there's a strong chance a page will already be in the user's cache.

## Approach 

In order to predict the next page a user is likely to visit, a solution could use the Google Analytics API. Google Analytics session data can be used to create a model to predict the most likely page a user is going to visit next on a site. The benefit of this session data is that it can evolve over time, so that if particular navigation paths change, the predictions can stay up to date too. 

With the availability of this data, an engine could insert `<link rel="[prerender/prefetch/preload]">` tags to speed up the load time for the next page request. In some tests, such as Mark Edmondson's [Supercharging Page-Loads with R](http://code.markedmondson.me/predictClickOpenCPU/supercharge.html), this led to a 30% improvement in page load times. 

