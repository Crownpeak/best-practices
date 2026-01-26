---
icon: cloud-arrow-down
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

# SFCC Data Extraction

This chapter details how the workflow extracts data from Salesforce Commerce Cloud (SFCC) using the Open Commerce API (OCAPI). The workflow uses both the **Data API** and **Shop API** to retrieve categories, product attributes, products, and variants.

## Authentication

### OAuth2 Token Retrieval

Before making any API calls, the workflow obtains an OAuth2 access token using the client credentials flow.

**Node:** `Get auth token`

**Endpoint:**
```
POST https://account.demandware.com/dw/oauth2/access_token
```

**Request:**
| Parameter | Value |
|-----------|-------|
| Method | `POST` |
| Content-Type | `multipart/form-data` |
| Authorization | Basic Auth (Client ID : Client Secret) |
| Body | `grant_type=client_credentials` |

**Response:**
```json
{
  "access_token": "eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCJ9...",
  "token_type": "Bearer",
  "expires_in": 1800
}
```

> **Reference:** [SFCC OAuth 2.0 Authentication](https://developer.salesforce.com/docs/commerce/b2c-commerce/references/b2c-commerce-ocapi/b2c-api-doc.html)

The access token is automatically used for subsequent API calls via the HTTP Bearer Auth credential.

---

## Category Extraction

The workflow extracts categories in two steps: first retrieving the category list, then fetching details for each category.

### Step 1: Get All Categories from Catalog

**Node:** `Get all categories from catalog`

**API:** Data API

**Endpoint:**
```
GET https://{baseUrl}/s/-/dw/data/v{apiVersion}/catalogs/{catalog}/categories
```

**Parameters:**
| Parameter | Description | Example |
|-----------|-------------|---------|
| `baseUrl` | SFCC instance hostname | `abcd-001.dx.commercecloud.salesforce.com` |
| `apiVersion` | OCAPI version | `24_1` |
| `catalog` | Catalog ID | `storefront-catalog-en` |
| `count` | Results per page | `50` |

**Pagination:**
The workflow uses n8n's built-in pagination to automatically fetch all pages:
- **Mode:** Response contains next URL
- **Next URL:** `$response.body.next`
- **Complete when:** `$response.body.next == undefined`
- **Request interval:** 500ms (to respect rate limits)

**Response Structure:**
```json
{
  "count": 50,
  "data": [
    {
      "id": "mens-clothing",
      "name": "Men's Clothing",
      "parent_category_id": "root",
      "online": true
    },
    {
      "id": "womens-clothing",
      "name": "Women's Clothing",
      "parent_category_id": "root",
      "online": true
    }
  ],
  "next": "https://...?start=50&count=50",
  "total": 125
}
```

> **Reference:** [Categories Resource (Data API)](https://developer.salesforce.com/docs/commerce/b2c-commerce/references/b2c-commerce-ocapi/b2c-api-doc.html)

### Step 2: Get Category Details

**Node:** `Get category details`

**API:** Shop API

**Endpoint:**
```
GET https://{baseUrl}/s/{siteId}/dw/shop/v{apiVersion}/categories/{categoryId}
```

**Parameters:**
| Parameter | Description | Example |
|-----------|-------------|---------|
| `baseUrl` | SFCC instance hostname | `abcd-001.dx.commercecloud.salesforce.com` |
| `siteId` | Site ID | `RefArch` |
| `apiVersion` | OCAPI version | `24_1` |
| `categoryId` | Category ID | `mens-clothing` |
| `client_id` | OCAPI client ID | (from configuration) |

**Response Structure:**
```json
{
  "id": "mens-clothing",
  "name": "Men's Clothing",
  "description": "All men's clothing items",
  "page_title": "Shop Men's Clothing",
  "c_showInMenu": true,
  "parent_category_id": "root",
  "online": true
}
```

> **Reference:** [Categories Resource (Shop API)](https://developer.salesforce.com/docs/commerce/b2c-commerce/references/b2c-commerce-ocapi/b2c-api-doc.html)

### Category Processing

**Node:** `Split out categories` → `Aggregate category details`

The workflow:
1. Splits the paginated results into individual category items
2. Fetches details for each category in parallel
3. Aggregates all category details into a single array

This aggregated data is used later to:
- Create the Fredhopper category tree
- Map products to categories

---

## Product Attribute Extraction

The workflow extracts product attribute definitions to dynamically build the Fredhopper item schema.

### Get Product Attribute Definitions

**Node:** `Get product attributes from Salesforce`

**API:** Data API

**Endpoint:**
```
POST https://{baseUrl}/s/-/dw/data/v{apiVersion}/system_object_definitions/Product/attribute_definition_search
```

**Parameters:**
| Parameter | Description | Example |
|-----------|-------------|---------|
| `baseUrl` | SFCC instance hostname | `abcd-001.dx.commercecloud.salesforce.com` |
| `apiVersion` | OCAPI version | `24_1` |

**Request Body:**
```json
{
  "count": 200,
  "start": 0,
  "select": "(**)",
  "query": {
    "match_all_query": {}
  }
}
```

**Response Structure:**
```json
{
  "count": 200,
  "hits": [
    {
      "id": "brand",
      "display_name": { "default": "Brand" },
      "value_type": "string",
      "searchable": true,
      "site_specific": false
    },
    {
      "id": "color",
      "display_name": { "default": "Color" },
      "value_type": "string",
      "searchable": true,
      "site_specific": false
    },
    {
      "id": "price",
      "display_name": { "default": "Price" },
      "value_type": "double",
      "searchable": false,
      "site_specific": true
    }
  ],
  "total": 85
}
```

> **Reference:** [System Object Definitions Resource](https://developer.salesforce.com/docs/commerce/b2c-commerce/references/b2c-commerce-ocapi/b2c-api-doc.html)

### Attribute Type Mapping

**Node:** `Collect attributes`

The workflow transforms SFCC attribute types to Fredhopper types:

| SFCC Type | Fredhopper Type |
|-----------|-----------------|
| `boolean` | `BOOLEAN` |
| `datetime` | `TIMESTAMP` |
| `image` | `TEXT` |
| `html` | `TEXT` |
| `quantity` | `INTEGER` |
| `double` | `FLOAT` |
| `string` | `TEXT` |

### Master vs. Variant Attribute Separation

The workflow separates attributes based on the `variationAttributes` configuration:

**Master (Product) Attributes:**
- All attributes NOT in the `variationAttributes` array
- These are set at the product level

**Variant Attributes:**
- Attributes listed in `variationAttributes` (e.g., `color`, `size`)
- Plus automatically added: `_imageurl`, `_thumburl`, `price`
- These are set at the variant level

---

## Product Extraction

The workflow extracts all products and their variants from the SFCC catalog.

### Get All Products (Product Search)

**Node:** `Get all products`

**API:** Shop API

**Endpoint:**
```
GET https://{baseUrl}/s/{siteId}/dw/shop/v{apiVersion}/product_search
```

**Parameters:**
| Parameter | Description | Value |
|-----------|-------------|-------|
| `count` | Results per page | `50` |
| `refine_1` | Category refinement | `cgid=root` |
| `refine_2` | Product type refinement | `htype=master` |
| `client_id` | OCAPI client ID | (from configuration) |

**Query String:**
```
?count=50&refine_1=cgid=root&refine_2=htype=master&client_id={clientId}
```

**Pagination:**
Same as category extraction:
- Follows `next` URL in response
- Completes when `next` is undefined
- 500ms interval between requests

**Response Structure:**
```json
{
  "count": 50,
  "hits": [
    {
      "product_id": "25502228M",
      "product_name": "Classic Shirt",
      "product_type": {
        "master": true
      },
      "represented_product": {
        "id": "25502228M",
        "link": "..."
      }
    }
  ],
  "next": "https://...?start=50&count=50",
  "total": 450
}
```

> **Reference:** [Product Search Resource](https://developer.salesforce.com/docs/commerce/b2c-commerce/references/b2c-commerce-ocapi/b2c-api-doc.html)

### Get Product Details

**Node:** `Get product details`

**API:** Shop API

**Endpoint:**
```
GET https://{baseUrl}/s/{siteId}/dw/shop/v{apiVersion}/products/{productId}
```

**Parameters:**
| Parameter | Description | Value |
|-----------|-------------|-------|
| `productId` | Master product ID | `25502228M` |
| `expand` | Additional data to include | `availability,bundled_products,links,options,variations` |
| `currency` | Currency for prices | `EUR` |
| `client_id` | OCAPI client ID | (from configuration) |

**Query String:**
```
?expand=availability,bundled_products,links,options,variations&currency=EUR&client_id={clientId}
```

**Response Structure:**
```json
{
  "id": "25502228M",
  "type": {
    "master": true
  },
  "name": "Classic Shirt",
  "brand": "Example Brand",
  "primary_category_id": "mens-shirts",
  "short_description": "A classic cotton shirt",
  "long_description": "<p>Full description...</p>",
  "price": 49.99,
  "price_max": 59.99,
  "orderable": true,
  "variants": [
    {
      "product_id": "25502228M001",
      "variation_values": {
        "color": "blue",
        "size": "M"
      }
    },
    {
      "product_id": "25502228M002",
      "variation_values": {
        "color": "blue",
        "size": "L"
      }
    }
  ],
  "variation_attributes": [
    {
      "id": "color",
      "name": "Color",
      "values": [
        { "name": "Blue", "value": "blue" }
      ]
    },
    {
      "id": "size",
      "name": "Size",
      "values": [
        { "name": "Medium", "value": "M" },
        { "name": "Large", "value": "L" }
      ]
    }
  ]
}
```

> **Reference:** [Products Resource](https://developer.salesforce.com/docs/commerce/b2c-commerce/references/b2c-commerce-ocapi/b2c-api-doc.html)

### Get Variant Details

**Node:** `Get variant details`

**API:** Shop API

**Endpoint:**
```
GET https://{baseUrl}/s/{siteId}/dw/shop/v{apiVersion}/products/{variantId}
```

**Parameters:**
| Parameter | Description | Value |
|-----------|-------------|-------|
| `variantId` | Variant product ID | `25502228M001` |
| `expand` | Additional data to include | `prices,images,variations` |
| `currency` | Currency for prices | `EUR` |
| `client_id` | OCAPI client ID | (from configuration) |

**Response Structure:**
```json
{
  "id": "25502228M001",
  "type": {
    "variant": true
  },
  "name": "Classic Shirt - Blue, M",
  "price": 49.99,
  "orderable": true,
  "master": {
    "master_id": "25502228M",
    "link": "..."
  },
  "variation_values": {
    "color": "blue",
    "size": "M"
  },
  "image_groups": [
    {
      "view_type": "large",
      "images": [
        {
          "link": "https://...image.jpg",
          "title": "Classic Shirt Blue"
        }
      ],
      "variation_attributes": [
        {
          "id": "color",
          "values": [{ "value": "blue" }]
        }
      ]
    }
  ]
}
```

---

## Data Flow Summary

```
┌────────────────────────────────────────────────────────────────────────────┐
│                        SFCC Data Extraction Flow                           │
└────────────────────────────────────────────────────────────────────────────┘

1. Authentication
   ┌─────────────────┐
   │ Get auth token  │──────────────────────────────────────────────────────┐
   └─────────────────┘                                                      │
          │                                                                 │
          ▼                                                                 │
2. Categories                                                               │
   ┌─────────────────────────┐     ┌──────────────────┐     ┌─────────────┐│
   │ Get all categories      │────▶│ Split categories │────▶│Get details  ││
   │ (Data API - paginated)  │     └──────────────────┘     └─────────────┘│
   └─────────────────────────┘                                      │      │
                                                                    │      │
          ┌─────────────────────────────────────────────────────────┘      │
          ▼                                                                 │
   ┌────────────────────────┐                                              │
   │ Aggregate categories   │──────────────────────────────────────────────┤
   └────────────────────────┘                                              │
          │                                                                 │
          ▼                                                                 │
3. Attributes                                                               │
   ┌────────────────────────────┐     ┌───────────────────┐               │
   │ Get product attributes     │────▶│ Collect attributes│               │
   │ (Data API)                 │     │ (transform types) │               │
   └────────────────────────────┘     └───────────────────┘               │
                                              │                            │
                                              ▼                            │
                                    ┌─────────────────────┐               │
                                    │ To Fredhopper       │───────────────┤
                                    │ (Schema creation)   │               │
                                    └─────────────────────┘               │
                                                                           │
4. Products & Variants                                                      │
   ┌─────────────────────────┐     ┌──────────────────┐     ┌─────────────┐│
   │ Get all products        │────▶│ Loop over pages  │────▶│Get details  ││
   │ (Shop API - paginated)  │     └──────────────────┘     └─────────────┘│
   └─────────────────────────┘                                      │      │
                                                                    │      │
   ┌────────────────────────────────────────────────────────────────┘      │
   │                                                                        │
   ▼                           ▼                                           │
   ┌─────────────────────┐     ┌─────────────────────┐                     │
   │ Aggregate products  │     │ Split variants      │                     │
   └─────────────────────┘     └─────────────────────┘                     │
          │                           │                                     │
          │                           ▼                                     │
          │                    ┌─────────────────────┐                     │
          │                    │ Get variant details │                     │
          │                    └─────────────────────┘                     │
          │                           │                                     │
          │                           ▼                                     │
          │                    ┌─────────────────────┐                     │
          │                    │ Aggregate variants  │                     │
          │                    └─────────────────────┘                     │
          │                           │                                     │
          ▼                           ▼                                     │
   ┌─────────────────────────────────────────────┐                        │
   │                    Merge                     │                        │
   │         (products + variants)                │                        │
   └─────────────────────────────────────────────┘                        │
                          │                                                │
                          ▼                                                │
                 ┌─────────────────────┐                                  │
                 │ To Fredhopper       │──────────────────────────────────┘
                 │ (Item ingestion)    │
                 └─────────────────────┘
```

## Rate Limiting Considerations

The SFCC OCAPI has rate limits that vary by instance type:

| Instance Type | Rate Limit |
|---------------|------------|
| Sandbox | Lower limits |
| Development | Moderate limits |
| Staging/Production | Higher limits |

The workflow includes a 500ms delay between paginated requests to respect these limits. If you encounter rate limiting errors:

1. Increase the request interval in pagination settings
2. Reduce the `count` parameter for smaller batches
3. Schedule the workflow during off-peak hours
4. Contact Salesforce support for rate limit increases

## Error Handling

Common SFCC API errors and solutions:

| Error Code | Meaning | Solution |
|------------|---------|----------|
| `401` | Unauthorized | Check OAuth credentials |
| `403` | Forbidden | Verify OCAPI permissions |
| `404` | Not Found | Check catalog/site/product IDs |
| `429` | Rate Limited | Increase request intervals |
| `500` | Server Error | Retry or contact SFCC support |
