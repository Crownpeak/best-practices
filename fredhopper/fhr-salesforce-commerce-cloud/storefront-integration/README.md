---
layout:
  title:
    visible: false
  description:
    visible: false
  tableOfContents:
    visible: true
  outline:
    visible: true
  pagination:
    visible: true
---

# Storefront Integration

[![Crownpeak Logo](../../../images/logo/crownpeak-logo.png)](http://www.crownpeak.com)

### [Fredhopper & Salesforce Commerce Cloud Reference Architecture](../)

## Storefront Integration

This section focuses on integrating the Fredhopper Query API into your storefront to provide enhanced search, navigation, and personalization capabilities. We'll cover the essential aspects of implementing the API, from constructing queries to handling responses and optimizing performance.

### Query API Overview

The Fredhopper Query API is a RESTful interface that allows your storefront to communicate with Fredhopper and retrieve search results, facets, and campaigns .

### Implementing Search and Navigation

#### Request Structure

The Query API uses the Fredhopper Query Language and query string arguments to request the relevant results.

> Please see the following page for a detailed description of the [Fredhopper Query Language.](https://crownpeak.gitbook.io/product-discovery/fredhopper-integration-guide/fredhopper-integration-guide-1/front-end-integration/fredhopper-query-language)

#### Response Handling and Rendering

The API returns an XML (or JSON if content-type “application/json” is requested) response containing search results, facets, and other relevant data. Your storefront application needs to parse this response and render the results appropriately.

Example response:

```json
{
  "info": {
    "lang": "en",
    "country": "GB",
    "locale": { ... },
    "current-universe": "uk-shop",
    "view": "search",
    "mode": "user",
    "query": "//uk-shop/en_GB/$s=shirt",
    "path": "/fredhopper/query",
    "type": "search",
    "query-string-httpencoded": "fh_location=%2f%2fuk-shop%2fen_GB%2f%24s%3dshirt",
    "ranges": { ... },
    "url": "/fredhopper/query.fh?fh_location=%2f%2fuk-shop%2fen_GB%2f%24s%3dshirt",
    "source-xml": "/fredhopper/query?fh_location=%2f%2fuk-shop%2fen_GB%2f%24s%3dshirt",
    "server": { ... }
  },
  "searchterms": {
    "term": {
      "value": "shirt",
      "profile": ""
    }
  },
  "searchpass": "EXACT 2020",
  "universes": {
    "universe": [
      {
        "link": { ... },
        "facetmap": [ ... ],
        "breadcrumbs": { ... },
        "items-section": {
          "results": { ... },
          "heading": { ... },
          "items": {
            "item": [ ... ]
          }
        },
        "themes": [ ... ],
        "display-fields": { ... },
        "attribute-types": { ... },
        "name": "uk-shop",
        "type": "selected"
      },
      {
        "link": {
          "name": "uk-shoprelated",
          "url-params": "fh_location=%2f%2fuk-shoprelated%2fen_GB"
        },
        "facetmap": [],
        "themes": [],
        "name": "uk-shoprelated",
        "type": "deselected"
      }
    ]
  },
  "footer": {},
  "qid": "...",
  "rid": "..."
}
```

Your storefront should:

* Parse the JSON or XML response.
* Render the results as product listings.
* Display facets for filtering.
* Implement pagination.
* Apply redirects
* Render the campaigns (text, image and items)

#### Faceting and Filtering

Faceting allows users to refine their search results by applying filters based on product attributes. Implement facet display and filtering logic on your storefront.

#### Sorting and Relevance

Implement sorting options (e.g., by price, relevance, popularity) and ensure that the search results are ranked according to relevance. Fine tune relevancy settings within Fredhopper.

> For more details on using the [Query API](https://crownpeak.gitbook.io/product-discovery/fredhopper-integration-guide/fredhopper-integration-guide-1/front-end-integration), see the documentation.

### Personalization and Recommendations

#### User Context and Profiles

Utilize user context (e.g., browsing history, purchase history) to personalize search results and recommendations.

#### Implementing Recommendations

Use Fredhopper's recommendation features to display personalized product recommendations. Construct API calls to retrieve recommendations based on user context.

#### A/B Testing and Optimization

Implement A/B testing to evaluate the effectiveness of different search and recommendation strategies. Use analytics to track user behaviour and optimise conversion.

### Performance Optimization

#### Caching Strategies

* **CDN Caching:** Cache static assets and API responses on a CDN where possible.
* **Browser Caching:** Utilize browser caching to store frequently accessed data.
* **Server-Side Caching:** Implement server-side caching strategies to reduce the load on the Fredhopper Query API, where applicable/suitable.

#### Query Optimization

* Optimize query parameters to minimize response time.
* Use appropriate filters and sorting criteria.
* Retrieve only the needed data by maintaining the proper display fields.

#### Load Balancing

* Distribute API requests across multiple Fredhopper servers.
* Use load balancers to ensure high availability.

### Error Handling and Fallbacks

#### Error Handling

* Implement error handling for API requests and responses.
* Display informative error messages to users.
* Log API errors for troubleshooting.

#### Fallbacks

* Implement fallback mechanisms for every third-party data provider.
* Consider using cached data or a fallback search engine.
* Display a user-friendly message when the search system is unavailable.

|                                                                                              |                                                                                                          |
| -------------------------------------------------------------------------------------------- | -------------------------------------------------------------------------------------------------------- |
| **Previous:** [**Data Transformation and Ingestion**](../data-transformation-and-ingestion/) | **Next:** [**Deployment and Operational Considerations**](../deployment-and-operational-considerations/) |
