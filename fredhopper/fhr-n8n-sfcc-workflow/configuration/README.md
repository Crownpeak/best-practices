---
icon: gear
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

# Configuration

The workflow requires configuration of both Salesforce Commerce Cloud and Fredhopper parameters. All configuration is centralized in the **"Workflow configuration"** node at the beginning of the workflow.

## Workflow Configuration Node

The configuration node uses n8n's Set node to define all required parameters. These values are referenced throughout the workflow using expressions like `$('Workflow configuration').item.json.salesforce.baseUrl`.

## Salesforce Commerce Cloud Parameters

### `salesforce.baseUrl`

**Type:** `string`  
**Required:** Yes  
**Example:** `abcd-001.dx.commercecloud.salesforce.com`

The base URL of your SFCC instance. This is typically in the format `{realm}-{instance}.dx.commercecloud.salesforce.com` for sandbox environments or your custom domain for production.

**How to find:**
1. Log into Business Manager
2. Check the URL in your browser
3. Use only the hostname without `https://` or paths

---

### `salesforce.apiVersion`

**Type:** `string`  
**Required:** Yes  
**Example:** `24_1`

The OCAPI version to use. This determines which API features are available. Use a version that is supported by your SFCC instance.

**Available versions:** Check the [SFCC OCAPI documentation](https://developer.salesforce.com/docs/commerce/b2c-commerce/references/b2c-commerce-ocapi/b2c-api-doc.html) for supported versions.

> **Note:** Using an older API version may limit available features. Using a version newer than your instance supports will result in errors.

---

### `salesforce.clientId`

**Type:** `string`  
**Required:** Yes  
**Example:** `aaaaaaaaaaaaaaaaaaaaaaaaaaaaaa`

The OCAPI client ID used for Shop API requests. This client must be configured in Business Manager with appropriate permissions.

**How to configure:**
1. In Business Manager, go to **Administration** → **Site Development** → **Open Commerce API Settings**
2. Add or update the Shop API configuration for your client ID
3. Ensure the client has permissions for the required resources

**Required Shop API permissions:**
```json
{
  "client_id": "your_client_id",
  "resources": [
    {
      "resource_id": "/categories/*",
      "methods": ["get"],
      "read_attributes": "(**)"
    },
    {
      "resource_id": "/product_search",
      "methods": ["get"],
      "read_attributes": "(**)"
    },
    {
      "resource_id": "/products/*",
      "methods": ["get"],
      "read_attributes": "(**)"
    }
  ]
}
```

---

### `salesforce.siteId`

**Type:** `string`  
**Required:** Yes  
**Example:** `RefArch`

The SFCC site ID to use for Shop API requests. This determines which site's catalog and products are exported.

**How to find:**
1. In Business Manager, go to **Administration** → **Sites** → **Manage Sites**
2. The Site ID is displayed in the list

---

### `salesforce.catalog`

**Type:** `string`  
**Required:** Yes  
**Example:** `storefront-catalog-en`

The catalog ID to export categories from. This is used for the Data API categories endpoint.

**How to find:**
1. In Business Manager, go to **Merchant Tools** → **Products and Catalogs** → **Catalogs**
2. The Catalog ID is displayed in the list

---

### `salesforce.variationAttributes`

**Type:** `array`  
**Required:** Yes  
**Default:** `["color", "size"]`  
**Example:** `["color", "size", "width"]`

An array of attribute IDs that should be treated as variant-level attributes rather than product-level attributes. These attributes will be mapped to the nested item schema in Fredhopper.

**Common variation attributes:**
* `color` - Product color variations
* `size` - Size variations (S, M, L, XL, etc.)
* `width` - Width variations (for shoes, etc.)
* `length` - Length variations

> **Important:** These must match the variation attribute IDs defined in your SFCC product model. The workflow converts these to snake_case for Fredhopper compatibility.

## Fredhopper Parameters

### `fredhopper.tenantId`

**Type:** `string`  
**Required:** Yes  
**Example:** `my-tenant`

Your Fredhopper tenant identifier. This is provided by Crownpeak when your Fredhopper environment is provisioned.

---

### `fredhopper.environment`

**Type:** `string`  
**Required:** Yes  
**Example:** `staging` or `production`

The Fredhopper environment to use. Typically `staging` for testing and `production` for live data.

---

### `fredhopper.itemSchema`

**Type:** `string`  
**Required:** Yes  
**Default:** `product`  
**Example:** `product`

The name of the item schema to create or update in Fredhopper. This schema defines the structure of your product data.

> **Note:** If the schema already exists, the workflow will update it with any new attributes found in SFCC.

---

### `fredhopper.categoryTree`

**Type:** `string`  
**Required:** Yes  
**Default:** `catalog01`  
**Example:** `main-catalog`

The name of the category tree in Fredhopper. This tree will be populated with categories from your SFCC catalog.

---

### `fredhopper.storeName`

**Type:** `string`  
**Required:** Yes  
**Example:** `My Online Store`

A human-readable name for your store. This is used as the localized name for the root of the category tree.

## Credentials Configuration

### SFCC OAuth2 Credentials

The workflow uses OAuth2 client credentials flow to authenticate with SFCC. Configure HTTP Basic Auth credentials in n8n:

| Field | Value |
|-------|-------|
| User | Your SFCC API Client ID |
| Password | Your SFCC API Client Secret |

These credentials are used by the **"Get auth token"** node to obtain an access token from `https://account.demandware.com/dw/oauth2/access_token`.

**Creating API Client Credentials in SFCC:**
1. Go to **Account Manager** (account.demandware.com)
2. Navigate to **API Client** section
3. Create a new API Client or use an existing one
4. Note the Client ID and Client Secret
5. Ensure the client has access to your organization and required roles

### SFCC Data API Permissions

For the Data API endpoints used (categories and attribute definitions), configure OCAPI in Business Manager:

1. Go to **Administration** → **Site Development** → **Open Commerce API Settings**
2. Select **Data** API type
3. Add configuration for your client:

```json
{
  "client_id": "your_client_id",
  "resources": [
    {
      "resource_id": "/catalogs/*/categories",
      "methods": ["get"],
      "read_attributes": "(**)"
    },
    {
      "resource_id": "/system_object_definitions/Product/attribute_definition_search",
      "methods": ["post"],
      "read_attributes": "(**)"
    }
  ]
}
```

### Fredhopper API Credentials

Configure the Crownpeak PD API credentials in n8n with your Fredhopper account details. The credential type is `crownpeakPDApi`.

## Example Configuration

Here is a complete example configuration:

```json
{
  "salesforce": {
    "baseUrl": "abcd-001.dx.commercecloud.salesforce.com",
    "apiVersion": "24_1",
    "clientId": "aaaaaaaaaaaaaaaaaaaaaaaaaaaaaa",
    "siteId": "RefArch",
    "catalog": "storefront-catalog-en",
    "variationAttributes": ["color", "size"]
  },
  "fredhopper": {
    "tenantId": "my-tenant",
    "environment": "staging",
    "itemSchema": "product",
    "categoryTree": "storefront",
    "storeName": "My Store"
  }
}
```

## Environment-Specific Configuration

For different environments (development, staging, production), consider:

1. **Using n8n variables:** Store environment-specific values in n8n variables
2. **Separate workflows:** Clone the workflow for each environment
3. **Conditional logic:** Add IF nodes to switch configurations based on environment

### Recommended Approach for Multiple Environments

```
┌─────────────────┐     ┌─────────────────┐     ┌─────────────────┐
│   Development   │     │     Staging     │     │   Production    │
├─────────────────┤     ├─────────────────┤     ├─────────────────┤
│ SFCC Sandbox    │     │ SFCC Staging    │     │ SFCC Production │
│ FHR Staging     │     │ FHR Staging     │     │ FHR Production  │
└─────────────────┘     └─────────────────┘     └─────────────────┘
```

## Validation Checklist

Before running the workflow, verify:

- [ ] SFCC base URL is correct and accessible
- [ ] SFCC API version is supported
- [ ] SFCC client ID has required OCAPI permissions
- [ ] SFCC site ID exists and is assigned to the catalog
- [ ] SFCC catalog ID exists and contains categories/products
- [ ] Fredhopper tenant ID and environment are correct
- [ ] All n8n credentials are configured and tested
- [ ] Variation attributes match your SFCC product model
