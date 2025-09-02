---
icon: database
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

# Data Transformation and Ingestion

### Overview of Data Flow

This section delves into the crucial aspect of data transformation and ingestion, which forms the backbone of the Fredhopper and Salesforce Commerce Cloud (SFCC) integration. Accurate and efficient data flow is essential for ensuring that Fredhopper's search, merchandising, and navigation capabilities are powered by up-to-date and relevant product information directly from either SFCC or from the source systems that provide data to SFCC.

We will explore the two primary methods of data ingestion: Flat File Integration using the Services API and API Integration using the Items API. Understanding the nuances of each method, including data extraction, transformation, and synchronization strategies, is vital for a successful integration. We'll outline the general data flow, highlighting the key stages and considerations in moving data from SFCC to Fredhopper.

### Items API Ingestion

The Item API provides a flexible method for ingesting data into Fredhopper. It allows you to send data updates to Fredhopper using API calls.

**API Overview and Authentication**

* **RESTful API:** Fredhopper provides a RESTful API for data ingestion.
* **Authentication:** Either using a Public Key or Password Authentication using OAUTH2.

**Data Mapping and Transformation (JSON Examples)**

* **JSON Format:** Send data updates in JSON format.
* **Data Mapping:** Map SFCC data fields to the corresponding Fredhopper attributes in the JSON payload.
*   **Example JSON Payload:**

    ```json
    [
      {
        "id": "A0001",
        "catalogVersion": 1,
        "type": "product",
        "attributes": {
          "title": "My item",
          "description": "This is my item.",
          "categories": ["category_id_1", "category_id_2"]
        }
      }
    ]
    ```

**Incremental vs. Batch Updates**

* **Incremental Updates:** Send data updates immediately as data changes occur in SFCC to ensure you are working with the most current data.
* **Batch Updates:** Group multiple data updates into a single API call for full catalog syncs.

**Error Handling and Monitoring**

* **API Error Codes:** Handle API error codes and implement retry mechanisms.
* **Logging:** Log API requests and responses for monitoring and troubleshooting.

**Troubleshooting API Ingestion**

* **Authentication Errors:** Verify API keys and tokens.
* **Data Format Errors:** Check the JSON payload for errors.
* **API Rate Limits:** Monitor API rate limits and implement retry mechanisms.
* **Feedback API:** Check the status of previous actions.

> For more details on using the [Items API](https://crownpeak.gitbook.io/product-discovery/product-discovery-developer-guide/item-catalog-management/what-is-the-items-api), see the documentation.

### Flat File Ingestion

Flat file ingestion is a traditional method of transferring data from SFCC to Fredhopper. It involves exporting data from SFCC into flat files (e.g., CSV, JSON), transforming them, and then uploading them to Fredhopper for indexing using the Services API.

**Data Extraction from SFCC**

* **Pipelines and Data Feeds:** SFCC provides Pipelines and Data Feeds to extract data from the platform. These tools allow you to define the data to be exported and the format of the output file.
* **Custom Scripts:** For more complex data extraction requirements, you can use custom scripts (e.g., SFCC Script API) to retrieve and format data.
* **Scheduled Exports:** Schedule regular data exports to keep Fredhopper data synchronized with SFCC.

**Data Transformation**

* **Scripting Languages:** Use scripting languages like Python, Node, or shell scripting to transform the extracted data into the format required by Fredhopper.
* **Data Mapping:** Map SFCC data fields to the corresponding Fredhopper attributes.
* **Data Cleansing:** Clean and validate the data to ensure accuracy and consistency.

**File Structure and Schema**

* **Fredhopper Schema:** Define the schema for your Fredhopper data, including the attributes and data types.
* **File Format:** Ensure the flat files are in the correct format (e.g., CSV, JSON), adhere to the Fredhopper schema, and match the required syntax as per the Fredhopper documentation.

**Upload Process to Fredhopper**

* **Services API Endpoint:** Use the Services API to submit the flat files.
* **Command-Line Tools:** Use command-line tools or APIs to automate the upload process.

**Troubleshooting Flat File Ingestion**

* **File Format Errors:** Check for errors in the file format and schema.
* **Data Mapping Issues:** Verify that the data is correctly mapped to Fredhopper attributes.
* **Upload Failures:** Investigate upload failures and check Fredhopper logs for errors.

> For more details on using [Flat File Ingestion](https://crownpeak.gitbook.io/product-discovery/fredhopper-integration-guide/fredhopper-integration-guide-1/data-integration), see the documentation.

### Data Synchronization Strategies

Maintaining data consistency between SFCC and Fredhopper is crucial for accurate search and navigation.

**Full vs. Incremental Updates**

* **Full Updates:** Replace and re-index all data in Fredhopper.
* **Incremental Updates:** Update only the changed data (add new, remove, or update existing).

Use incremental updates as much as possible to minimize processing time; however, full catalog updates are required at regular intervals to ensure data consistency and optimization.

**Handling Deletes and Updates**

* **Deletes:** Implement mechanisms to delete products from Fredhopper.
* **Updates:** Ensure that product updates are reflected in Fredhopper in a timely manner.

**Data Consistency and Latency**

* **Data Consistency:** Implement strategies to ensure data consistency between SFCC and Fredhopper.
* **Latency:** Minimize latency between data updates in SFCC and their reflection in Fredhopper.
