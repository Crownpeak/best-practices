---
icon: cloud-arrow-up
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

# Fredhopper Data Ingestion

This chapter describes how the workflow transforms SFCC data and loads it into Fredhopper Product Discovery. The workflow uses the **Crownpeak Product Discovery n8n node** to interact with Fredhopper APIs.

## Overview

The data ingestion process consists of four main phases:

1. **Authentication** - Obtain Fredhopper API token
2. **Schema Management** - Create or update the item schema
3. **Category Tree Sync** - Create or update the category tree
4. **Catalog & Item Ingestion** - Create catalog and load product items

## Authentication

### Get Fredhopper Token

**Node:** `Get authentication token`

The Crownpeak PD node handles authentication automatically using the configured credentials. The token is obtained via the Fredhopper Authentication API and is used for all subsequent API calls.

**Resource:** `authentication`

The credentials are configured in n8n using the `crownpeakPDApi` credential type, which includes:
- API endpoint URL
- Authentication credentials (API key or OAuth)

---

## Item Schema Management

The item schema defines the structure of product data in Fredhopper, including all attributes and their types.

### Schema Structure

The workflow creates a schema with:

**Master Product Attributes:**
- All SFCC product attributes that are NOT variation attributes
- Types are mapped from SFCC types to Fredhopper types

**Nested Variant Schema:**
- A nested `variant` schema for variant-level attributes
- Includes variation attributes (e.g., `color`, `size`)
- Automatically includes: `_imageurl`, `_thumburl`, `price`

### Create Item Schema

**Node:** `Create an item schema`

**Resource:** `itemSchema`  
**Operation:** `createItemSchema`

**Parameters:**
| Parameter | Value |
|-----------|-------|
| `tenant` | From workflow configuration |
| `environment` | From workflow configuration |
| `schemaTenant` | From workflow configuration |
| `schemaEnvironment` | From workflow configuration |

**Schema Data Structure:**
```json
{
  "name": "product",
  "tenant": "my-tenant",
  "environment": "staging",
  "attributes": [
    { "name": "brand", "type": "TEXT" },
    { "name": "name", "type": "TEXT" },
    { "name": "short_description", "type": "TEXT" },
    { "name": "long_description", "type": "TEXT" },
    { "name": "ean", "type": "TEXT" },
    { "name": "upc", "type": "TEXT" }
  ],
  "nestedItemSchemas": [
    {
      "name": "variant",
      "attributes": [
        { "name": "_imageurl", "type": "LOCALIZEDTEXT" },
        { "name": "_thumburl", "type": "LOCALIZEDTEXT" },
        { "name": "price", "type": "FLOAT" },
        { "name": "color", "type": "TEXT" },
        { "name": "size", "type": "TEXT" }
      ]
    }
  ]
}
```

### Update Existing Schema

**Node:** `Item schema exists?` → `Update existing item schema`

If the schema already exists (error response contains "already exists"), the workflow updates it instead:

**Resource:** `itemSchema`  
**Operation:** `updateItemSchema`

**Additional Parameter:**
| Parameter | Value |
|-----------|-------|
| `schemaName` | From workflow configuration |

The update operation merges new attributes into the existing schema while preserving existing data.

### Error Handling

**Node:** `Error creating item schema`

If schema creation fails for reasons other than "already exists", the workflow stops with an error message. Common causes:
- Invalid attribute types
- Duplicate attribute names
- Permission issues

---

## Category Tree Management

The category tree in Fredhopper represents the navigation hierarchy from SFCC.

### Category Tree Structure

The workflow transforms SFCC categories into the Fredhopper format:

```json
{
  "tenant": "my-tenant",
  "environment": "staging",
  "name": "storefront",
  "localizedNames": [
    {
      "name": "My Store",
      "locale": "en_GB"
    }
  ],
  "children": [
    {
      "name": "mensclothing",
      "localizedNames": [
        {
          "name": "Men's Clothing",
          "locale": "en_GB"
        }
      ],
      "children": []
    },
    {
      "name": "womensclothing",
      "localizedNames": [
        {
          "name": "Women's Clothing",
          "locale": "en_GB"
        }
      ],
      "children": []
    }
  ]
}
```

> **Note:** Category IDs are sanitized by removing hyphens and underscores to comply with Fredhopper naming requirements: `String(id).replace(/[-_]/g, "").trim()`

### Create Category Tree

**Node:** `Create a category tree`

**Resource:** `categoryTree`  
**Operation:** `createCategoryTree`

**Parameters:**
| Parameter | Value |
|-----------|-------|
| `categoryTreeTenant` | From workflow configuration |
| `categoryTreeEnvironment` | From workflow configuration |

### Update Existing Category Tree

**Node:** `Category tree exists?` → `Update existing category tree`

If the category tree already exists, the workflow updates it:

**Resource:** `categoryTree`  
**Operation:** `updateCategoryTree`

**Additional Parameter:**
| Parameter | Value |
|-----------|-------|
| `categoryTreeName` | From workflow configuration |

---

## Catalog Management

Fredhopper uses catalog versioning to enable atomic updates. The workflow creates a new inactive catalog, populates it with items, and then activates it.

### List Existing Catalogs

**Node:** `List existing catalogs`

**Resource:** `catalog`  
**Operation:** (default - list)

**Parameters:**
| Parameter | Value |
|-----------|-------|
| `catalogTenant` | From workflow configuration |
| `catalogEnvironment` | From workflow configuration |

**Response:**
```json
{
  "catalogs": [
    {
      "version": 1,
      "state": "ACTIVE",
      "createdAt": "2024-01-15T10:00:00Z"
    },
    {
      "version": 2,
      "state": "INACTIVE",
      "createdAt": "2024-01-16T10:00:00Z"
    }
  ]
}
```

### Delete Old Inactive Catalog

**Node:** `Inactive catalog found?` → `Delete old inactive catalog`

If an inactive catalog already exists, it must be deleted before creating a new one:

**Resource:** `catalog`  
**Operation:** `deleteCatalog`

**Parameters:**
| Parameter | Value |
|-----------|-------|
| `catalogVersion` | Version of the inactive catalog |

### Create New Inactive Catalog

**Node:** `Create new inactive catalog`

**Resource:** `catalog`  
**Operation:** `createCatalog`

**Catalog Data:**
```json
{
  "catalogItemSchemas": [
    {
      "name": "product",
      "version": 5
    }
  ],
  "catalogCategoryTrees": [
    {
      "name": "storefront",
      "version": 3
    }
  ]
}
```

The catalog references specific versions of the item schema and category tree, ensuring consistency.

---

## Item Ingestion

The final step loads all products and variants into the newly created catalog.

### Data Transformation

**Node:** `Create/update items in Fredhopper`

The workflow transforms SFCC product data into Fredhopper item format:

#### Product (Master) Items

```json
{
  "type": "product",
  "id": "25502228M",
  "context": "global",
  "catalogVersion": 3,
  "attributes": {
    "categories": ["mensshirts"],
    "brand": "Example Brand",
    "name": "Classic Shirt",
    "short_description": "A classic cotton shirt"
  }
}
```

**Key transformations:**
- `type` is set to `"product"`
- `id` uses the SFCC product ID
- `categories` array contains sanitized category IDs (no hyphens/underscores)
- Only attributes defined in the schema are included (filtered)

#### Variant Items

```json
{
  "type": "variant",
  "id": "25502228M001",
  "parentId": "25502228M",
  "context": "global",
  "catalogVersion": 3,
  "attributes": {
    "color": "blue",
    "size": "M",
    "price": 49.99,
    "_imageurl": {
      "en_GB": "https://example.com/image.jpg"
    },
    "_thumburl": {
      "en_GB": "https://example.com/thumb.jpg"
    }
  }
}
```

**Key transformations:**
- `type` is set to `"variant"`
- `parentId` links to the master product
- `_imageurl` and `_thumburl` are localized (using `en_GB` locale)
- Image URL is extracted from the variant's image groups, matching by color

### Image URL Extraction Logic

The workflow includes custom logic to find the correct image URL for each variant based on its color:

```javascript
function getVariantImageUrl(variant) {
  const color = variant.variation_values?.color;
  if (!color || !Array.isArray(variant.image_groups)) return "";
  
  for (const group of variant.image_groups) {
    if (Array.isArray(group.variation_attributes)) {
      for (const attr of group.variation_attributes) {
        if (attr.id === "color" && 
            Array.isArray(attr.values) && 
            attr.values.some(v => v.value === color)) {
          return group.images?.[0]?.link || "";
        }
      }
    }
  }
  return variant.image_groups[0]?.images?.[0]?.link || "";
}
```

This ensures each variant displays the correct color-specific image.

### Batch Processing

**Node:** `Loop over result pages`

Products are processed in batches using n8n's Split In Batches node:
1. Product search results are paginated (50 per page)
2. Each page is processed separately
3. Products and variants are fetched and transformed
4. Items are sent to Fredhopper
5. Loop continues to the next batch

This approach:
- Reduces memory usage for large catalogs
- Provides progress visibility
- Allows partial recovery on failure

---

## Catalog Activation

### Activate the New Catalog

**Node:** `Activate inactive catalog`

After all items are ingested, the catalog is activated to make the changes live:

**Resource:** `catalog`  
**Operation:** `activateCatalog`

**Parameters:**
| Parameter | Value |
|-----------|-------|
| `catalogTenant` | From workflow configuration |
| `catalogEnvironment` | From workflow configuration |
| `catalogVersion` | Version of the newly created catalog |

**Behavior:**
- The new catalog becomes `ACTIVE`
- The previous active catalog becomes `ARCHIVED`
- Query API immediately serves data from the new catalog

---

## Complete Data Flow

```
┌────────────────────────────────────────────────────────────────────────────┐
│                    Fredhopper Data Ingestion Flow                          │
└────────────────────────────────────────────────────────────────────────────┘

1. Schema Management
   ┌─────────────────────┐
   │ Collect attributes  │
   │ (from SFCC)         │
   └──────────┬──────────┘
              │
              ▼
   ┌─────────────────────┐     ┌──────────────────────┐
   │ Get auth token      │────▶│ Create item schema   │
   └─────────────────────┘     └──────────┬───────────┘
                                          │
                               ┌──────────┴───────────┐
                               │ Schema exists?       │
                               └──────────┬───────────┘
                                    │           │
                              ┌─────┴─────┐     │
                              ▼           ▼     │
                         ┌────────┐  ┌────────┐ │
                         │ Update │  │ Error  │ │
                         │ schema │  │ (stop) │ │
                         └────┬───┘  └────────┘ │
                              │                 │
                              └────────┬────────┘
                                       │
                                       ▼
2. Category Tree
   ┌─────────────────────────┐     ┌──────────────────────────┐
   │ Create category tree    │────▶│ Category tree exists?    │
   └─────────────────────────┘     └──────────┬───────────────┘
                                              │
                                   ┌──────────┴───────────┐
                                   ▼                      ▼
                              ┌────────────┐         ┌────────┐
                              │ Update     │         │ Error  │
                              │ tree       │         │ (stop) │
                              └─────┬──────┘         └────────┘
                                    │
                                    ▼
3. Catalog Management
   ┌─────────────────────────┐     ┌──────────────────────────┐
   │ List existing catalogs  │────▶│ Inactive catalog found?  │
   └─────────────────────────┘     └──────────┬───────────────┘
                                              │
                                   ┌──────────┴───────────┐
                                   ▼                      │
                              ┌────────────────┐          │
                              │ Delete old     │          │
                              │ inactive       │          │
                              └───────┬────────┘          │
                                      │                   │
                                      └───────┬───────────┘
                                              │
                                              ▼
                              ┌────────────────────────────┐
                              │ Create new inactive catalog │
                              └──────────────┬─────────────┘
                                             │
                                             ▼
4. Item Ingestion
   ┌─────────────────────────────────────────────────────────┐
   │                    Batch Loop                           │
   │  ┌─────────────┐     ┌─────────────┐     ┌───────────┐ │
   │  │ Products    │────▶│ Transform   │────▶│ Upload to │ │
   │  │ + Variants  │     │ to FHR      │     │ Fredhopper│ │
   │  └─────────────┘     └─────────────┘     └───────────┘ │
   │                                                         │
   │  [Repeat for each batch of products]                   │
   └─────────────────────────────────────────────────────────┘
                                             │
                                             ▼
5. Activation
   ┌─────────────────────────────┐
   │ Activate inactive catalog   │
   └─────────────────────────────┘
                  │
                  ▼
   ┌─────────────────────────────┐
   │ ✓ Catalog is now ACTIVE    │
   │ ✓ Products are searchable  │
   └─────────────────────────────┘
```

---

## Fredhopper API Reference

| Operation | API Endpoint | Method |
|-----------|--------------|--------|
| Authentication | `/auth/token` | POST |
| Create Schema | `/schemas` | POST |
| Update Schema | `/schemas/{name}` | PUT |
| Create Category Tree | `/category-trees` | POST |
| Update Category Tree | `/category-trees/{name}` | PUT |
| List Catalogs | `/catalogs` | GET |
| Create Catalog | `/catalogs` | POST |
| Delete Catalog | `/catalogs/{version}` | DELETE |
| Activate Catalog | `/catalogs/{version}/activate` | POST |
| Create/Update Items | `/items` | POST |

> **Reference:** [Fredhopper Product Discovery Developer Guide](https://crownpeak.gitbook.io/product-discovery/product-discovery-developer-guide)

---

## Best Practices

### 1. Schema Versioning

- Add new attributes incrementally
- Avoid removing attributes that may contain data
- Test schema changes in staging first

### 2. Category Tree Updates

- Keep category IDs consistent between syncs
- Use meaningful, URL-safe category names
- Consider hierarchical depth for navigation UX

### 3. Catalog Management

- Always create inactive catalogs first
- Validate item counts before activation
- Keep at least one archived catalog for rollback

### 4. Item Ingestion

- Process in batches to handle large catalogs
- Monitor ingestion progress via n8n execution logs
- Validate transformed data before sending

### 5. Error Recovery

- Check Fredhopper feedback API for ingestion errors
- Implement retry logic for transient failures
- Log all API responses for debugging
