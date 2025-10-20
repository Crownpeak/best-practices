---
icon: code
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

# JavaScript SDK

## Overview

The Fredhopper SDK is designed as a lean API client for interacting with the Fredhopper Query API, through the Shopify App Proxy, focused on making it easy to query Fredhopper endpoints while letting developers build their own UI and rendering logic. It handles authentication, request formatting, and provides utilities for common Fredhopper operations.

> To enable the [SDK Theme Embed](../user-guide/#enabling-the-javascript-sdk) provided with the Shopify App, see the documentation

> For more details on using the [Fredhopper Query Language](https://crownpeak.gitbook.io/product-discovery/building-the-front-end-experience-with-fhr/fredhopper-query-language), used by both the SDK and the Query API, see the documentation.

## Quick Start

### Basic Setup

The SDK is automatically initialized when the script loads. Credentials are managed through the app and exposed via `window.fredhopper.credentials`.

```javascript
// SDK is available globally
const results = await fredhopper.getSearchWithOptions("shoes");
const collection = await fredhopper.getCollectionWithOptions({category: "catalog01_12345"});
```

### Development Mode

Add `?fhr_debug=true` to your URL to enable detailed console logging in the browser console.

## API Reference

### Core Methods

#### `getSearchWithOptions(query, options)`

Stateless method to retrieve search results from Fredhopper.

```javascript
const results = await fredhopper.getSearchWithOptions("shoes", {
  page: 1,
  limit: 24,
  sort: "price-asc",
  filters: {
    brand: ["nike", "adidas"],
    price: "10<price<100",
  },
  catalog: "catalog01",
  locale: "en_GB",
  category: "catalog01_12345",
  facetMultiMap: {
    brand: true, // Multi-select
    price: false, // Single select
  },
});
```

**Parameters:**

* `query` (string, *required*): The search term to query
* `options` (object, optional): Configuration object

**Options Properties:**

* `page` (number, optional): Page number. Default: `1`
* `limit` (number, optional): Results per page. Default: `24`
* `sort` (string, optional): Sort option. Default: `"relevance"`
  * Available values: `"relevance"`, `"price-asc"`, `"price-desc"`, or custom sort field
* `filters` (object, optional): Filter object. Default: `{}`
  * Single value: `{ brand: "nike" }` or `{ brand: ["nike"] }`
  * Multiple values: `{ brand: ["nike", "adidas"] }`
  * Range values: `{ price: "10<price<100" }`
* `catalog` (string, optional): Catalog identifier. Default: `"catalog01"`
* `locale` (string, optional): Locale in Fredhopper format (e.g., `"en_GB"`). Default: auto-detected with `utils.detectLocale()`
* `category` (string, optional): Category identifier. Default: `''`
* `facetMultiMap` (object, optional): Defines which facets allow multiple selections. Default: `{}`
  * Format: `{ facetName: true/false }`
  * `true` = multi-select, `false` = single-select

**Returns:** `Promise<any>` - Search results from Fredhopper API

**Important Note:** This method automatically constructs the two parameters required by the Fredhopper API:
- `fh_view: 'search'` - Specifies this is a search query
- `fh_location` - Built from your catalog, locale, search term, category, and filters

You do not need to manually construct these parameters.

#### `getCollectionWithOptions(options)`

Stateless method to retrieve collection/category results from Fredhopper.

```javascript
const results = await fredhopper.getCollectionWithOptions({
  page: 1,
  limit: 36,
  sort: "price-desc",
  filters: {
    size: ["8", "9", "10"],
    color: ["red"],
  },
  catalog: "catalog01",
  locale: "fr_FR",
  category: "catalog01_789",
  facetMultiMap: {
    size: true,
    color: false,
  },
});
```

**Parameters:**

* `options` (object, optional): Configuration object

**Options Properties:**

* `page` (number, optional): Page number. Default: `1`
* `limit` (number, optional): Results per page. Default: `24`
* `sort` (string, optional): Sort option. Default: `"relevance"`
  * Available values: `"relevance"`, `"price-asc"`, `"price-desc"`, or custom sort field
* `filters` (object, optional): Filter object. Default: `{}`
* `catalog` (string, optional): Catalog identifier. Default: `"catalog01"`
* `locale` (string, optional): Locale in Fredhopper format. Default: auto-detected with `utils.detectLocale()`
* `category` (string, optional): Category identifier. Default: `''`
  * **Without a category identifier, this returns **all products** from the catalog. It is recommended to always provide a category for collection queries.**
* `facetMultiMap` (object, optional): Defines which facets allow multiple selections. Default: `{}`
  * Format: `{ facetName: true/false }`
  * `true` = multi-select, `false` = single-select

*Returns:* `Promise<any>` - Collection results from Fredhopper API

**Important Note:** This method automatically constructs the parameters required by the Fredhopper API:
- `fh_view: 'lister'` - Specifies this is a collection query.
- `fh_location` - Built from your catalog, locale, search term, category, and filters

You do not need to manually construct these parameters.

### Raw API Access

#### `rawRequest(params)`

Direct API request with full control over all Fredhopper parameters:

```javascript
const results = await fredhopper.rawRequest({
  fh_location: "//catalog01/en_GB/categories<{shoes}/brand=nike",
  fh_view: "lister",
  fh_view_size: 48,
  fh_start_index: 0,
  fh_sort_by: "-price",
  market: "GB",
  custom_param: "value",
});
```
**Parameters:**

* `params` (object, required): Direct Fredhopper API parameters

**Required Parameters:**

* `fh_location` (string): Fredhopper location string
* `fh_view` (string): Fredhopper view type

**Optional Parameters:**

* `fh_view_size` (number): Number of results
* `fh_start_index` (number): Starting index for pagination
* `fh_sort_by` (string): Sort field
* `fh_session` (string): Session identifier
* `market` (string): Market code

**Returns:** `Promise<any>` - Raw API response

**Important Note:** Credentials are automatically added to all requests.

## Utilities

### Location Building

#### `utils.buildLocation(options)`

Build Fredhopper location strings programmatically.

```javascript
const location = fredhopper.utils.buildLocation({
  catalog: "catalog01",
  locale: "en_GB",
  searchTerm: "shoes",
  category: "catalog01_12345",
  filters: {
    brand: ["nike"],
    price: "50<price<200",
  },
  pageType: "search",
  facetMultiMap: {
    brand: true,
  },
});

// Result: "//catalog01/en_GB/categories<{catalog01_12345}/$s=shoes/brand>{nike}/50<price<200"
```
**Parameters:**

`options` (object, optional): Location building configuration

**Options Properties:**

* `catalog` (string, optional): Catalog identifier. Default: `"catalog01"`
* `locale` (string, optional): Locale. Default: auto-detected using `utils.detectLocale()`
* `searchTerm` (string, optional): Search term. Default: `''`
* `category` (string, optional): Category identifier. Default: `''`
* `filters` (object, optional): Filter object. Default: `{}`
* `pageType` (string, optional): `"search"` or `"collection"`. Default: `"collection"`
* `facetMultiMap` (object, optional): Defines which facets allow multiple selections. Default: `{}`
* Format: `{ facetName: true/false }`
* `true` = multi-select, `false` = single-select

**Returns:** `string` - Formatted Fredhopper location string

### Encoding Utilities

#### `utils.encodeFHUnicodeValue(value)`

Encode filter values for Fredhopper Unicode format.

```javascript
const encoded = fredhopper.utils.encodeFHUnicodeValue("special chars!");

// Result: "special\\u0020chars\\u0021"
```

**Parameters:**

* `value` (string, required): Value to encode

**Returns:** `string` - Encoded value

#### `utils.formatSearchTerm(term)`

Format search terms for Fredhopper by converting special characters to Unicode escape sequences.

```javascript
const formatted = fredhopper.utils.formatSearchTerm("running shoes");

// Result: "running\\u0020shoes"
```

**Parameters:**

* `term` (string, required): Search term to format

**Returns:** `string` - Formatted search term

### Other Utilities

#### `utils.debounce(func, delay)`

Debounce function for optimizing search inputs and preventing excessive API calls.

```javascript
const debouncedSearch = fredhopper.utils.debounce(async (term) => {
  if (term.length > 2) {
    const results = await fredhopper.getSearchWithOptions(term);
    updateSearchResults(results);
  }
}, 300);
```

**Parameters:**

* `func` (function, required): Function to debounce
* `wait` (number, required): Delay in milliseconds

**Returns:** `function` - Debounced function

#### `utils.detectLocale()`

Auto-detect Shopify locale and convert it to Fredhopper format.

```javascript
const locale = fredhopper.utils.detectLocale();

// Result: "en_GB" (based on window.Shopify.locale/country)
```

**Parameters**: None

**Returns:** `string` - Fredhopper locale string

#### `utils.formatPrice(price, currency)`

Format prices using Intl.NumberFormat.

```javascript
const formatted = fredhopper.utils.formatPrice(29.99, "USD");

// Result: "$29.99"
```

**Parameters:**

* `price` (number, optional): Price to format. Default: `0`
* `currency` (string, optional): Currency code. Default: `"USD"`

**Returns:** `string - Formatted price string

## Configuration & Initialization

### Basic Initialization

Initialize or reconfigure the SDK. The SDK auto-initializes on load, but you can reconfigure it.

```javascript
await fredhopper.init({
  debug: true,
  development: false,
});
```
**Parameters:**

* `options` (object, optional): Configuration object

**Options Properties:**

* `debug` (boolean, optional): Enable debug logging
* `development` (boolean, optional): Enable development mode

**Returns:** `Promise<void>`

### Event Handling

#### `onReady(callback)`

Execute callback when SDK is fully initialized.

```javascript
fredhopper.onReady(() => {
  console.log("SDK ready!");
  
  // Perform initial search/collection load
});
```
**Parameters:**

* callback (function, required): Function to execute when ready

**Returns:** `void`

#### Event System

Subscribe to SDK events.

```javascript
fredhopper.events.on("filtersChanged", (filters) => {
  console.log("Filters updated:", filters);
});

fredhopper.events.emit("customEvent", data);
```

**Parameters:**

* `event` (string, required): Event name
* `callback` (function, required): Event handler function

**Returns:** `void`

## Examples

### Basic Product Search

```javascript
async function searchProducts(term) {
  try {
    const results = await fredhopper.getSearchWithOptions(term, {
      limit: 24,
      sort: "relevance",
    });

    displayProducts(results);
  } catch (error) {
    console.error("Search failed:", error);
  }
}
```

### Advanced Collection with Filters

```javascript
async function loadFilteredCollection(categoryId, userFilters = {}) {
  const results = await fredhopper.getCollectionWithOptions({
    category: categoryId,
    page: 1,
    limit: 36,
    sort: "price-asc",
    filters: userFilters,
    locale: fredhopper.utils.detectLocale(),
    facetMultiMap: {
      brand: true,
      size: true,
      color: true
    }
  });

  return results;
}

// Usage
const products = await loadFilteredCollection("catalog01_12345", {
  brand: ["nike", "adidas"],
  size: ["8", "9", "10"],
  price: "20<price<150"
});
```

### Debounced Search Input

```javascript
const searchInput = document.getElementById("search");
const debouncedSearch = fredhopper.utils.debounce(async (term) => {
  if (term.length > 2) {
    const results = await fredhopper.getSearchWithOptions(term, {limit: 10});
    updateSearchResults(results);
  }
}, 300);

searchInput.addEventListener("input", (e) => {
  debouncedSearch(e.target.value);
});
```

## Error Handling

The SDK throws errors for failed requests. Always wrap API calls in try-catch blocks:

```javascript
try {
  const results = await fredhopper.getSearchWithOptions("shoes");
  // Handle successful response
} catch (error) {
  if (error.message.includes("API request failed")) {
    // Handle API errors
    console.error("Fredhopper API error:", error);
  } else {
    // Handle other errors
    console.error("Unexpected error:", error);
  }
}
```

## Best Practices

### 1. Use Stateless Methods

Use `getSearchWithOptions()` and `getCollectionWithOptions()` for cleaner, more predictable code.

### 2. Handle Localization

Use `utils.detectLocale()` to automatically handle different markets:

```javascript
const results = await fredhopper.getSearchWithOptions("shoes", {
  locale: fredhopper.utils.detectLocale(), // Auto-detects from Shopify
  catalog: "catalog01",
});
```

### 3. Debounce User Input

Always debounce search inputs to avoid excessive API calls:

```javascript
const debouncedSearch = fredhopper.utils.debounce(searchFunction, 300);
```

### 4. Use Location Builder for Complex Queries

For complex filtering scenarios, use the location builder utility:

```javascript
const location = fredhopper.utils.buildLocation({
  // ... complex filter logic
});

const results = await fredhopper.rawRequest({
  fh_location: location,
  fh_view: "search",
});
```

### 5. Handle Loading States

Track loading states in your UI:

```javascript
// Example: Using CSS classes for loading state
const container = document.querySelector('.products-container');
container.classList.add('is-loading');
try {
  const results = await fredhopper.getSearchWithOptions(term);
  displayResults(results);
} finally {
  container.classList.remove('is-loading');
}
```

## Troubleshooting

### Debug Mode

Enable debug mode to see detailed request/response logging:

```
https://yourstore.com/collections/shoes?fhr_debug=true
```

### Common Issues

1. **Credentials Missing**: Ensure the app is properly installed and credentials are set
2. **CORS Errors**: The SDK uses Shopify proxies to avoid CORS issues
3. **Invalid Location Strings**: Use the location builder utility to ensure proper formatting
4. **Rate Limiting**: Implement debouncing for user input

## TypeScript Support

While the SDK is written in JavaScript, it includes JSDoc type annotations for better IDE support. For full TypeScript support, you can create type definitions:

```typescript
declare global {
  interface Window {
    fredhopper: {
      getSearchWithOptions(
        query: string,
        options: AdvancedSearchOptions
      ): Promise<any>;
      getCollectionWithOptions(
        options: AdvancedCollectionOptions
      ): Promise<any>;
      rawRequest(params: Record<string, any>): Promise<any>;
      utils: {
        buildLocation(options: LocationOptions): string;
        encodeFHUnicodeValue(value: string): string;
        formatSearchTerm(term: string): string;
        debounce(func: Function, delay: number): Function;
        detectLocale(): string;
        formatPrice(price: number, currency: string): string;
      };
    };
  }
}
```

## Support

For issues specific to this SDK implementation, check:

1. Console logs with `?fhr_debug=true`
2. Network tab for API request/response details
3. Verify credentials are properly set in the app

For Fredhopper API documentation and support, consult the official Fredhopper documentation.
