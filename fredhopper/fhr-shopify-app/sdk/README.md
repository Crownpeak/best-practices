# JavaScript SDK Documentation

## Overview

The Fredhopper SDK is designed as a lean API client for interacting with the Fredhopper Query API, through the Shopify App Proxy, focused on making it easy to query Fredhopper endpoints while letting developers build their own UI and rendering logic. It handles authentication, request formatting, and provides utilities for common Fredhopper operations.

> To enable the [SDK Theme Embed](../user-guide/README.md#enabling-the-javascript-sdk) provided with the Shopify App, see the documentation

> For more details on using the [Fredhopper Query Language](https://crownpeak.gitbook.io/product-discovery/building-the-front-end-experience-with-fhr/fredhopper-query-language), used by both the SDK and the Query API, see the documentation.

## Quick Start

### Basic Setup

The SDK is automatically initialized when the script loads. Credentials are managed through the app and exposed via `window.fredhopper.credentials`.

```javascript
// SDK is available globally
const results = await fredhopper.getSearch("shoes");
const collection = await fredhopper.getCollection("sneakers");
```

### Development Mode

Add `?fhr_debug=true` to enable detailed console logging.

## API Reference

### Core Methods

#### `getSearch(query, options, location)`

Retrieve search results from Fredhopper.

```javascript
const results = await fredhopper.getSearch(
  "running shoes",
  {
    page: 1,
    limit: 24,
    sort: "price-asc",
  },
  "fh_location=//catalog01/en_GB"
);
```

**Parameters:**

- `query` (string): Search term
- `options` (object, optional): Search options
  - `page` (number): Page number (default: 1)
  - `limit` (number): Results per page (default: 20)
  - `sort` (string): Sort option ('relevance', 'price-asc', 'price-desc')
- `location` (string, optional): Fredhopper location string

#### `getCollection(handle, options, location)`

Retrieve collection/category results from Fredhopper.

```javascript
const results = await fredhopper.getCollection(
  "sneakers",
  {
    page: 1,
    limit: 24,
    sort: "relevance",
  },
  "fh_location=//catalog01/en_GB/categories<{catalog01_123456}"
);
```

**Parameters:**

- `handle` (string): Collection handle
- `options` (object, optional): Collection options (same as getSearch)
- `location` (string, optional): Fredhopper location string

### Stateless Methods (Recommended)

For cleaner, more explicit code, use the stateless methods that accept all parameters:

#### `getSearchWithOptions(query, options)`

Stateless search method with explicit parameters.

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

#### `getCollectionWithOptions(handle, options)`

Stateless collection method with explicit parameters.

```javascript
const results = await fredhopper.getCollectionWithOptions("sneakers", {
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

### Raw API Access

For maximum control over Fredhopper parameters:

#### `rawRequest(params)`

Direct API request with custom Fredhopper parameters.

```javascript
const results = await fredhopper.rawRequest({
  fh_location: "//catalog01/en_GB/categories<{shoes}/brand=nike",
  fh_view: "lister",
  fh_view_size: 48,
  fh_start_index: 0,
  fh_sort_by: "-price",
  custom_param: "value",
});
```

**Note:** Credentials are automatically added to all requests.

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
  pageType: "search", // or 'collection'
  facetMultiMap: {
    brand: true,
  },
});

// Result: "//catalog01/en_GB/categories<{catalog01_12345}/$s=shoes/brand>{nike}/50<price<200"
```

### Encoding Utilities

#### `utils.encodeFHUnicodeValue(value)`

Encode values for Fredhopper Unicode format.

```javascript
const encoded = fredhopper.utils.encodeFHUnicodeValue("special chars!");
// Result: "special\\u0020chars\\u0021"
```

#### `utils.formatSearchTerm(term)`

Format search terms for Fredhopper.

```javascript
const formatted = fredhopper.utils.formatSearchTerm("running shoes");
// Result: "running\\u0020shoes"
```

### Other Utilities

#### `utils.debounce(func, delay)`

Debounce function for search inputs.

```javascript
const debouncedSearch = fredhopper.utils.debounce((term) => {
  fredhopper.getSearch(term);
}, 300);
```

#### `utils.detectLocale()`

Auto-detect Shopify locale and convert to Fredhopper format.

```javascript
const locale = fredhopper.utils.detectLocale();
// Result: "en_GB" (based on window.Shopify.locale/country)
```

#### `utils.formatPrice(price, currency)`

Format prices using Intl.NumberFormat.

```javascript
const formatted = fredhopper.utils.formatPrice(29.99, "USD");
// Result: "$29.99"
```

## Configuration & Initialization

### Basic Initialization

```javascript
await fredhopper.init({
  debug: true,
  development: false,
});
```

### Event Handling

#### `onReady(callback)`

Execute callback when SDK is initialized.

```javascript
fredhopper.onReady(() => {
  console.log("SDK ready!");
  // Perform initial search/collection load
});
```

#### Event System

Subscribe to SDK events.

```javascript
fredhopper.events.on("filtersChanged", (filters) => {
  console.log("Filters updated:", filters);
});

fredhopper.events.emit("customEvent", data);
```

## Examples

### Basic Product Search

```javascript
async function searchProducts(term) {
  try {
    const results = await fredhopper.getSearch(term, {
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
async function loadCollection(handle, userFilters = {}) {
  const results = await fredhopper.getCollectionWithOptions(handle, {
    page: 1,
    limit: 36,
    filters: userFilters,
    catalog: "catalog01",
    locale: fredhopper.utils.detectLocale(),
    facetMultiMap: {
      brand: true,
      size: true,
      color: false,
    },
  });

  return results;
}

// Usage
const products = await loadCollection("sneakers", {
  brand: ["nike", "adidas"],
  size: ["8", "9"],
});
```

### Custom Location Building

```javascript
function buildCustomLocation(searchTerm, filters) {
  return fredhopper.utils.buildLocation({
    catalog: "catalog01",
    locale: "en_GB",
    searchTerm: searchTerm,
    filters: filters,
    pageType: "search",
    facetMultiMap: {
      brand: true,
      category: false,
    },
  });
}

const location = buildCustomLocation("shoes", {
  brand: ["nike"],
  price: "20<price<100",
});

const results = await fredhopper.getSearch("shoes", {}, location);
```

### Debounced Search Input

```javascript
const searchInput = document.getElementById("search");
const debouncedSearch = fredhopper.utils.debounce(async (term) => {
  if (term.length > 2) {
    const results = await fredhopper.getSearch(term);
    updateSearchResults(results);
  }
}, 300);

searchInput.addEventListener("input", (e) => {
  debouncedSearch(e.target.value);
});
```

## Migration Guide

### From Legacy Methods

If you're using deprecated methods, here's how to migrate:

#### ❌ Old Way

```javascript
// Deprecated - uses internal state
fredhopper.setFilters({ brand: "nike" });
const results = await fredhopper.search("shoes");
const state = fredhopper.getState(); // Deprecated
```

#### ✅ New Way

```javascript
// Recommended - stateless and explicit
const results = await fredhopper.getSearchWithOptions("shoes", {
  filters: { brand: ["nike"] },
  page: 1,
  limit: 24,
});
```

#### ❌ Old Method Names

```javascript
await fredhopper.searchWithOptions(term, options); // Deprecated
```

#### ✅ New Method Names

```javascript
await fredhopper.getSearchWithOptions(term, options); // Consistent naming
```

## Error Handling

The SDK throws errors for failed requests. Always wrap calls in try-catch:

```javascript
try {
  const results = await fredhopper.getSearch("shoes");
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

Prefer `getSearchWithOptions()` and `getCollectionWithOptions()` for cleaner, more predictable code.

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
setLoading(true);
try {
  const results = await fredhopper.getSearch(term);
  displayResults(results);
} finally {
  setLoading(false);
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
      getSearch(
        query: string,
        options?: SearchOptions,
        location?: string
      ): Promise<any>;
      getCollection(
        handle: string,
        options?: CollectionOptions,
        location?: string
      ): Promise<any>;
      getSearchWithOptions(
        query: string,
        options: AdvancedSearchOptions
      ): Promise<any>;
      getCollectionWithOptions(
        handle: string,
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
