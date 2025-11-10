---
icon: ranking-star
layout:
  width: default
  title:
    visible: true
  description:
    visible: false
  tableOfContents:
    visible: true
  outline:
    visible: true
  pagination:
    visible: true
  metadata:
    visible: true
---

# Web-Pixel Integration

The Fredhopper tracker is integrated into Shopify via the Web Pixel API and records the following events:

* page\_viewed
* product\_viewed
* product\_added\_to\_cart
* product\_removed\_from\_cart
* cart\_viewed
* search\_submitted
* checkout\_completed

A session ID is generated and stored in the browser using Shopify’s browser.sessionStorage API. This ID is included as the sessionId in all tracker events. The same value is also passed to the Query API as fh\_session\_id to enable non-cached A/B testing. The sourceId is derived from the most recent response ID returned by the Query API, and included with subsequent tracker calls.

You can enable debug mode by adding the parameter fhr\_pixel\_debug to your storefront URL. When enabled, all events are logged to the browser console, showing which events fired and what data was sent to the tracker via the Web Pixel. Console messages are prefixed with \[FHR PIXEL] for easy identification.

You can also preview events in the Shopify Admin:

* Go to Settings → Customer Events.
* Find Fredhopper Product Discovery in the list of App Pixels.
* Click … → Test. This opens a shop preview with the Pixel Helper window, where you can view events received by the tracker.

