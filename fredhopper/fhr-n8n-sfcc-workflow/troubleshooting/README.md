---
icon: wrench
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

# Troubleshooting

This chapter provides solutions for common issues encountered when running the n8n workflow for SFCC to Fredhopper integration.

## Common Issues

### Authentication Errors

#### SFCC OAuth2 Token Failure

**Symptoms:**
- Node "Get auth token" fails
- Error: `401 Unauthorized` or `invalid_client`

**Causes & Solutions:**

| Cause | Solution |
|-------|----------|
| Invalid client credentials | Verify Client ID and Secret in Account Manager |
| Expired credentials | Regenerate credentials in Account Manager |
| Wrong credential type | Ensure HTTP Basic Auth is configured, not Bearer |
| Organization access | Verify client has access to your organization |

**Verification Steps:**
1. Test credentials manually with curl:
   ```bash
   curl -X POST https://account.demandware.com/dw/oauth2/access_token \
     -u "CLIENT_ID:CLIENT_SECRET" \
     -d "grant_type=client_credentials"
   ```
2. Check Account Manager for client status
3. Verify organization membership

---

#### Fredhopper Authentication Failure

**Symptoms:**
- Node "Get authentication token" fails
- Error: `401` or `Invalid credentials`

**Causes & Solutions:**

| Cause | Solution |
|-------|----------|
| Invalid API credentials | Check Fredhopper credentials in n8n |
| Wrong tenant/environment | Verify tenant ID and environment |
| Expired API key | Request new credentials from Crownpeak |

---

### SFCC API Errors

#### 403 Forbidden on API Calls

**Symptoms:**
- Category or product API calls fail with `403`
- Error: `Access denied` or `Insufficient permissions`

**Causes & Solutions:**

1. **Missing OCAPI Configuration:**
   - Go to Business Manager → Administration → Site Development → Open Commerce API Settings
   - Add configuration for your client ID

2. **Shop API Permissions:**
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

3. **Data API Permissions:**
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

---

#### 404 Not Found

**Symptoms:**
- API calls return `404`
- Error: `Category not found` or `Product not found`

**Causes & Solutions:**

| Resource | Possible Cause | Solution |
|----------|----------------|----------|
| Catalog | Invalid catalog ID | Verify catalog exists in Business Manager |
| Category | Category deleted/renamed | Check catalog category list |
| Product | Product not online | Ensure products are orderable/online |
| Site | Invalid site ID | Verify site ID in configuration |

---

#### 429 Rate Limited

**Symptoms:**
- API calls sporadically fail
- Error: `429 Too Many Requests`

**Solutions:**

1. **Increase pagination interval:**
   - In HTTP Request nodes, change `requestInterval` from `500` to `1000` or higher

2. **Reduce batch size:**
   - Change `count` parameter from `50` to `25`

3. **Schedule off-peak:**
   - Run the workflow during low-traffic hours

4. **Request limit increase:**
   - Contact Salesforce support for higher rate limits

---

### Fredhopper API Errors

#### Schema Already Exists

**Symptoms:**
- "Create an item schema" fails
- Error contains: `already exists`

**Expected Behavior:**
This is handled automatically by the workflow. The "Item schema exists?" node routes to "Update existing item schema".

**If still failing:**
- Check if the error message differs from expected
- Verify schema name matches configuration
- Manually delete schema in Fredhopper if corrupted

---

#### Category Tree Already Exists

**Symptoms:**
- "Create a category tree" fails
- Error contains: `The category tree already exists`

**Expected Behavior:**
Handled by the workflow routing to "Update existing category tree".

**If still failing:**
- Verify category tree name in configuration
- Check for naming conflicts in Fredhopper

---

#### Invalid Attribute Type

**Symptoms:**
- Schema creation/update fails
- Error: `Invalid attribute type`

**Causes & Solutions:**

| SFCC Type | Should Map To | Common Issue |
|-----------|---------------|--------------|
| `string` | `TEXT` | Mapping incorrect |
| `enum-of-string` | `TEXT` | Unsupported type |
| `set-of-string` | (not supported) | Needs custom handling |
| `int` | `INTEGER` | Check for decimals |

**Custom Type Handling:**
If SFCC uses types not in the default mapping, modify the "Collect attributes" node:

```javascript
const typeMapping = {
  "boolean": "BOOLEAN",
  "datetime": "TIMESTAMP",
  "image": "TEXT",
  "html": "TEXT",
  "quantity": "INTEGER",
  "double": "FLOAT",
  "string": "TEXT",
  // Add custom mappings:
  "enum-of-string": "TEXT",
  "int": "INTEGER"
};
```

---

#### Catalog Activation Failure

**Symptoms:**
- "Activate inactive catalog" fails
- Error: `Cannot activate catalog` or `Validation failed`

**Causes & Solutions:**

| Cause | Solution |
|-------|----------|
| No items in catalog | Verify item ingestion succeeded |
| Invalid item references | Check category IDs in items match tree |
| Schema version mismatch | Ensure catalog references correct schema version |
| Missing required attributes | Add missing attributes to items |

**Debugging:**
1. Check catalog state in Fredhopper dashboard
2. Use Fredhopper Feedback API to view ingestion errors
3. Verify item counts match expected

---

### Data Transformation Issues

#### Missing Products in Fredhopper

**Symptoms:**
- Product count in Fredhopper lower than SFCC
- Some products not searchable

**Causes & Solutions:**

1. **Products not master type:**
   - Workflow filters for `htype=master`
   - Variants without masters won't be included

2. **Products not in root category:**
   - Workflow uses `cgid=root` refinement
   - Ensure products have category assignments

3. **Products not online:**
   - Check product online/orderable status in SFCC

4. **Transformation errors:**
   - Check n8n execution logs for failed items
   - Verify attribute values are valid

---

#### Incorrect Category Assignments

**Symptoms:**
- Products show in wrong categories
- Category navigation doesn't work

**Causes & Solutions:**

1. **Category ID sanitization:**
   - IDs like `mens-clothing` become `mensclothing`
   - Ensure consistency between tree and item categories

2. **Primary category mismatch:**
   - Workflow uses `primary_category_id`
   - Secondary categories are not included

3. **Category not in tree:**
   - Ensure all categories exist in category tree
   - Check for orphan categories

---

#### Missing Images

**Symptoms:**
- Products show without images
- Wrong images for variants

**Causes & Solutions:**

1. **Image group structure:**
   - Verify SFCC image groups have `variation_attributes`
   - Check image group `view_type` matches expectations

2. **Color matching:**
   - Image extraction matches by `color` attribute
   - Verify color values are consistent

3. **Missing fallback:**
   - If no color match, uses first image in first group
   - Ensure at least one image exists

**Custom Image Logic:**
Modify the `getVariantImageUrl` function if your image structure differs:

```javascript
function getVariantImageUrl(variant) {
  // Custom logic for your image structure
  if (variant.image?.link) {
    return variant.image.link;
  }
  // ... fallback logic
}
```

---

## Workflow Debugging

### Enable Detailed Logging

1. In n8n Settings, enable **Save Execution Progress**
2. Set log level to **Debug**
3. Check execution history for detailed step output

### Test Individual Nodes

1. Click on any node
2. Select "Execute Node" to run just that step
3. Check input/output data in the panel

### Common Debug Points

| Issue Area | Node to Check | What to Verify |
|------------|---------------|----------------|
| Auth | Get auth token | `access_token` in output |
| Categories | Get all categories | Category count, IDs |
| Attributes | Collect attributes | `attributes` and `nestedAttributes` arrays |
| Products | Get all products | Product count, IDs |
| Items | Create/update items | JSON payload structure |

---

## Performance Optimization

### Large Catalogs

**For catalogs with 10,000+ products:**

1. **Increase batch processing:**
   - Adjust Loop node batch size based on memory

2. **Use incremental sync:**
   - Modify workflow to only sync changed products
   - Track last sync timestamp

3. **Parallel processing:**
   - Split into multiple workflows by category
   - Merge results at the end

### Memory Issues

**If n8n runs out of memory:**

1. Reduce `count` parameter in API calls
2. Process in smaller batches
3. Increase n8n memory allocation
4. Use n8n's streaming mode for large datasets

---

## Frequently Asked Questions

### Q: How often should I run the sync?

**A:** Depends on your catalog change frequency:
- **Real-time changes:** Use incremental sync with webhooks
- **Daily updates:** Schedule workflow once per day
- **Occasional updates:** Run manually when needed

### Q: Can I sync multiple sites/catalogs?

**A:** Yes, options include:
1. Duplicate workflow with different configurations
2. Use workflow variables for site-specific settings
3. Create a parent workflow that calls the sync workflow for each site

### Q: How do I handle localization?

**A:** The current workflow uses `en_GB` locale. To support multiple locales:
1. Fetch localized category names from SFCC
2. Modify category tree structure for multiple `localizedNames`
3. Add localized attributes to item schema

### Q: What happens if the workflow fails mid-execution?

**A:** The catalog remains inactive until activation succeeds. You can:
1. Fix the issue and re-run
2. Delete the incomplete inactive catalog
3. Previous active catalog remains unaffected

### Q: How do I add custom attributes?

**A:** Modify the "Collect attributes" node:
1. Add attributes to the `attributes` or `nestedAttributes` array
2. Ensure the attribute exists in the item data
3. Re-run the workflow to update schema and items

---

## Support Resources

- **SFCC Documentation:** [OCAPI Reference](https://developer.salesforce.com/docs/commerce/b2c-commerce/references/b2c-commerce-ocapi/b2c-api-doc.html)
- **Fredhopper Documentation:** [Product Discovery Developer Guide](https://crownpeak.gitbook.io/product-discovery/product-discovery-developer-guide)
- **n8n Documentation:** [n8n Docs](https://docs.n8n.io/)
- **Crownpeak PD Node:** [npm Package](https://www.npmjs.com/package/n8n-nodes-crownpeak-pd)
