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
* product\_clicked (please see note below)

A session ID is generated and stored in the browser using Shopify’s browser.sessionStorage API. This ID is included as the sessionId in all tracker events. The same value is also passed to the Query API as fh\_session\_id to enable non-cached A/B testing. The sourceId is derived from the most recent response ID returned by the Query API, and included with subsequent tracker calls.

You can enable debug mode by adding the parameter fhr\_pixel\_debug to your storefront URL. When enabled, all events are logged to the browser console, showing which events fired and what data was sent to the tracker via the Web Pixel. Console messages are prefixed with \[FHR PIXEL] for easy identification.

You can also preview events in the Shopify Admin:

* Go to Settings → Customer Events.
* Find Fredhopper Product Discovery in the list of App Pixels.
* Click … → Test. This opens a shop preview with the Pixel Helper window, where you can view events received by the tracker.


**Note on `product_clicked`**

Although the `product_clicked` event has been registered, Shopify will not fire it automatically.

To trigger this event, you need to update the Liquid code for your product grid so that it fires when a product is clicked.

Here is an example of how this can be done. Please note that it is illustrative only and may need to be adapted to match your site’s implementation.

````
<script>
  document.addEventListener('DOMContentLoaded', function() {
    document.body.addEventListener('click', function(e) {
      const target = e.target.closest('[data-product-id]');
      // Exit if click isn't on an element with data-product-id
      if (!target) return;

      const productId = target.dataset.productId;

      const urlParams = new URLSearchParams(window.location.search);
      const searchTerm = urlParams.get('q') || '';

      // Construct custom event data
      const eventData = {
        productId: productId,
        searchTerm: searchTerm
      };

      // Publish custom event to Shopify Analytics
      if (window.Shopify && window.Shopify.analytics) {
        window.Shopify.analytics.publish('product_clicked', eventData);
      } else {
        console.log('Fallback event data:', eventData);
      }
    });
  });
</script>
````