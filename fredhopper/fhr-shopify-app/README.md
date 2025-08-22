# Fredhopper Product Discovery Shopify App User Guide

## Introduction

The Fredhopper Product Discovery App integrates Crownpeak’s Fredhopper into Shopify, bringing AI-powered search, merchandising, and personalized recommendations directly into your storefront. 

It enables faster time-to-market with a ready-made connector, simplified catalog syncs and schema management, global multi-language/multi-region support with Shopify Markets, and secure frontend integration with the Fredhopper Query API via the Shopify App Proxy.

### Key Capabilities

#### Data Integration 
* Full & incremental product syncs from Shopify to Fredhopper
* Bulk ingestion via Shopify Bulk Query API, broken into batches for reliability.
* Product update streaming using Shopify webhooks
* Management of schema creation, catalog management, and batch ingestion through the app.

#### Frontend Integration
* JavaScript SDK exposes Fredhopper’s power directly to developers inside Shopify themes
* Out of the box Theme App Blocks for rendering dynamic PLP/search pages using the Fredhopper Query API
* Uses the Shopify App Proxy to ensure secure, scoped API requests
* Web Pixel integration to capture shopper behavior, powering A/B testing, insights, and AI scores.

## Installing the app

To install the app inside your Shopify store, visit the App Store and type Fredhopper, or use the following URL: https://apps.shopify.com/fredhopper-product-discovery 

Click the install button and when prompted give the app the necessary permission.

The app is now installed and ready to be configured.

Reach out to your CSM, Technical Consultant or onboarding team so they configure the app for you so it can communicate with your Fredhopper instance.

Once the app has been configured, you can start to use the features of the app.

### Installing the Web Pixel

The Fredhopper tracking pixel extension allows click data to be sent to Fredhopper’s tracker to power features such A/B testing, insights and AI scores.

To install the pixel, click the Install Pixel Extention button on the Settings page of the app.

![Shopify App Settings Page](../../images/shopify/settings.png "Shopify App Settings Page")

You can remove the Web Pixel at anytime by clicking the Remove Pixel Extension button.

### Enabling the JavaScript SDK

The JavaScript SDK allows developers to integrate Fredhopper into your front-end  themes and blocks. The SDK adds an object to the window that you can use to make calls to the Fredhopper Query API via the Shopify App Proxy, adding the necessary filters to each call, and receiving the results to render on the page.

You can enable the SDK via the Settings page of the app by clicking on the Enable Fredhopper SDK button. Or by visiting theme editor within the Shopify Admin, selecting the App Embeds panel and toggling the Fredhopper SDK Loader.

![Shopify App SDK Loader](../../images/shopify/sdk.png "Shopify App SDK Loader")

For more information on using the SDK, please view the [SDK Documentation](./sdk/README.md)

## Using the Schema Manager

The Schema Manager allows you to create a schema that tells Fredhopper the shape and types of data that the app will be sending from Shopify to Fredhopper. The allows Fredhopper to have an understanding of the data and defines how it can be used in Fredhopper when creating facets, rankings and other rules.

The schema manager displays the data fields within Shopify and allows them to be mapped to types in Fredhopper. You can add, remove or adjust these.

![Shopify App Schema Manager Page](../../images/shopify/schema1.png "Shopify App Schema Manager Page")

In addition to the basic product fields, the app also supports most metafield formats within Shopify with the exception of reference type metafields. Any reference type metafields included in the schema will only send the object reference to Fredhopper.

Once created, the schema will be saving in Fredhopper and used by the Catalog Manager and the Batch Manager to transform the data from Shopify in to a format that Fredhopper understands as part of the catalog sync.

You can create new Schema’s if the data structure has been changed in Shopify, for example a new metafield has been added, or you can use the edit button to use a previous schema version as a template for a new schema.

Once the schema has been created, you can use the catalog manager to run a full catalog sync using that schema or you can wait for the next scheduled catalog sync to happen.

If you have created a new metafield and it is not being displayed with the Schema Manager, please use the Create Staging Catalog option within the Batch Manager to run a sync to ensure that the app has the latest set of data from Shopify. This will sync the data ready for sending data to Fredhopper but not activate the catalog inside Fredhopper. See the Batch Manager section for more details. Once this process is complete, the new fields will show in the Schema Manager and you can create a new schema to tell Fredhopper to use these fields.

## Using the Catalog Manager

The Catalog Manager shows you the details of the current active catalog in Fredhopper and allows you to manually sync data to Fredhopper.

To sync all of the product data and create a new catalog in Fredhopper, you can first select the Schema to used, the latest version of the schema will be selected be default, and click the Create New Catalog button.

![Shopify App Catalog Manager Page](../../images/shopify/catalog.png "Shopify App Catalog Manager Page")

The will start the process of batching the data from Shopify, running the transformation and post processing jobs and send the data to Fredhopper.

Once the batch processing has completed, Fredhopper will then index all of the data for use within your frontend storefront.

Creating a manual catalog sync is useful if you have made changes to the schema and/or data structure in Shopify and don’t want to wait for the next scheduled catalog sync before the changes are sent to Fredhopper.

Any none structural changes to products, such as updating a product’s title, will be automatically sent to Fredhopper when the product has been changed to ensure changes throughout the day are applied to Fredhopper.

## Using the Batch Manager

The Batch Manager allows you to view the progress of any running batch jobs and the status a previous jobs. 

You can also create a create a staging catalog from within the Batch Manager. 

When you create a staging catalog it will prepare a batch and process and transform the data from Shopify in the same way that the Catalog Manager does but it will not activate the data in Fredhopper until later activated via the Catalog Manager.

![Shopify App Batch Manager Page](../../images/shopify/batch.png "Shopify App Batch Manager Page")

This is useful to ensure that the app has the latest set of data from Shopify or to prepare a data sync that you want to activate in Fredhopper at a certain point.

Once a staged batch is ready, you will see following displayed within the Catalog Manager.

You can click the Promote button to activate the catalog within Fredhopper which will immediately start the indexing process inside Fredhopper.

Alternately, you can abandon the staged catalog by selecting the new New Catalog radio button and clicking the Create New Catalog button

## Setting a Schedule

Using the Batch Manger, you can also set a scheduled to run a full catalog sync at a specific time that suits you.

This will perform a full catalog sync in the same way as the manual catalog sync works but at a specific point in time.

![Shopify App Schedule Manager Page](../../images/shopify/schedule.png "Shopify App Schedule Manager Page")

We recommend that you do a nightly sync of all your product data, with incremental updates running throughout the day as products are updated.

## Theme App Blocks

In additional to the SDK, the app also provides two basic Theme App Blocks with fixed layouts, one for collection pages and one for search, that can be used out-of-box within your shop’s theme. These blocks use the SDK to retrieve data from the Fredhopper Query API and render the results, including facets, within a listing format.

![Shopify App Theme App Block](../../images/shopify/themeappblock.png "Shopify App Theme App Block")

For more information on using the SDK, please view the [SDK Documentation](./sdk/README.md)
