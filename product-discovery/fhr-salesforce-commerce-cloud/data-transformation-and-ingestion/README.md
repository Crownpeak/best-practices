<a href="http://www.crownpeak.com" target="_blank">![Crownpeak Logo](../../../images/logo/crownpeak-logo.png "Crownpeak Logo")</a>

## [Fredhopper & Salesforce Commerce Cloud Reference Architecture Guide](../README.md)

# Data Transformation and Ingestion

## Overview of Data Flow
This section delves into the crucial aspect of data transformation and ingestion, which forms the backbone of the Fredhopper and Salesforce Commerce Cloud (SFCC) integration. Accurate and efficient data flow is essential for ensuring that Fredhopper's search and navigation capabilities are powered by up-to-date and relevant product information from SFCC.

We will explore the two primary methods of data ingestion: Flat File and the Common Item Data Pipeline (API). Understanding the nuances of each method, including data extraction, transformation, and synchronization strategies, is vital for a successful integration. We'll outline the general data flow, highlighting the key stages and considerations involved in moving data from SFCC to Fredhopper.

## Common Item Data Pipeline (API) Ingestion

The Common Item Data Pipeline (API) provides a more real-time and flexible method for ingesting data into Fredhopper. It allows you to send data updates directly to Fredhopper using API calls.

### API Overview and Authentication
* **RESTful API:** Fredhopper provides a RESTful API for data ingestion.
* **Authentication:** Use API keys or tokens for authentication.

### Data Mapping and Transformation (JSON Examples)
* **JSON Format:** Send data updates in JSON format.
* **Data Mapping:** Map SFCC data fields to the corresponding Fredhopper attributes in the JSON payload.
* **Example JSON Payload:**

```json
{
  "items": [
    {
      "id": "12345",
      "name": "Product Name",
      "price": 99.99,
      "category": "Electronics"
    }
  ]
}
```

### Real-time vs. Batch Updates
* **Real-time Updates:** Send data updates immediately as changes occur in SFCC.
* **Batch Updates:** Group multiple data updates into a single API call for efficiency.

### Error Handling and Monitoring
* **API Error Codes:** Handle API error codes and implement retry mechanisms.
* **Logging:** Log API requests and responses for monitoring and troubleshooting.

### Code Snippets for API Calls (e.g., cURL, Node)
* **cURL Example:**
```bash
curl -X POST -H "Content-Type: application/json" -H "Authorization: Bearer YOUR_API_KEY" -d '{"items": [{"id": "12345", "name": "Product Name", "price": 99.99, "category": "Electronics"}]}' "YOUR_FREDHOPPER_API_ENDPOINT"
```

* **Node Example:**
```node
const axios = require('axios');

async function ingestData(apiKey, apiEndpoint, data) {
    try {
        const response = await axios.post(apiEndpoint, data, {
            headers: {
                'Content-Type': 'application/json',
                Authorization: `Bearer ${apiKey}`,
            },
        });

        if (response.status === 200) {
            console.log('Data ingested successfully');
        } else {
            console.error(`Error ingesting data: ${response.status}, ${response.data}`);
        }
    } catch (error) {
        console.error('Error ingesting data:', error);
    }
}

const apiKey = 'YOUR_API_KEY';
const apiEndpoint = 'YOUR_FREDHOPPER_API_ENDPOINT';
const data = {
    items: [
        { id: '12345', name: 'Product Name', price: 99.99, category: 'Electronics' },
    ],
};

ingestData(apiKey, apiEndpoint, data);
```

### Troubleshooting API Ingestion
* **Authentication Errors:** Verify API keys and tokens.
* **Data Format Errors:** Check the JSON payload for errors.
* **API Rate Limits:** Monitor API rate limits and implement retry mechanisms.

## Flat File Ingestion
Flat file ingestion is a traditional method of transferring data from SFCC to Fredhopper. It involves exporting data from SFCC into flat files (e.g., CSV, XML) and then uploading these files to Fredhopper for indexing.

### Data Extraction from SFCC
* **Pipelines and Data Feeds:** SFCC provides Pipelines and Data Feeds to extract data from the platform. These tools allow you to define the data to be exported and the format of the output file.
* **Custom Scripts:** For more complex data extraction requirements, you can use custom scripts (e.g., SFCC Script API) to retrieve and format data.
* **Scheduled Exports:** Schedule regular data exports to keep Fredhopper data synchronized with SFCC.

### Data Transformation (Example Scripts/Snippets)
* **Scripting Languages:** Use scripting languages like Python, Node or shell scripting to transform the extracted data into the format required by Fredhopper.
* **Data Mapping:** Map SFCC data fields to the corresponding Fredhopper attributes.
* **Data Cleansing:** Clean and validate the data to ensure accuracy and consistency.
* **Node Example:**
```node
const fs = require('fs');
const csv = require('csv-parser');
const createCsvWriter = require('csv-writer').createObjectCsvWriter;

async function transformData(inputFile, outputFile) {
    const results = [];

    fs.createReadStream(inputFile)
        .pipe(csv())
        .on('data', (data) => {
            results.push({
                id: data.product_id,
                name: data.product_name,
                price: data.product_price,
                category: data.product_category,
            });
        })
        .on('end', async () => {
            const csvWriter = createCsvWriter({
                path: outputFile,
                header: [
                    { id: 'id', title: 'id' },
                    { id: 'name', title: 'name' },
                    { id: 'price', title: 'price' },
                    { id: 'category', title: 'category' },
                ],
            });

            await csvWriter.writeRecords(results);
            console.log('Data transformation complete.');
        });
}

transformData('input.csv', 'output.csv');
```

### File Structure and Schema
* **Fredhopper Schema:** Define the schema for your Fredhopper data, including the attributes and data types.
* **File Format:** Ensure the flat files are in the correct format (e.g., CSV, XML) and adhere to the Fredhopper schema.

### Upload Process to Fredhopper
* **Fredhopper Admin Interface:** Use the Fredhopper admin interface to upload the flat files.
* **Command-Line Tools:** Use command-line tools or APIs to automate the upload process.

### Troubleshooting Flat File Ingestion
* **File Format Errors:** Check for errors in the file format and schema.
* **Data Mapping Issues:** Verify that the data is correctly mapped to Fredhopper attributes.
* **Upload Failures:** Investigate upload failures and check Fredhopper logs for errors.


## Data Synchronization Strategies
Maintaining data consistency between SFCC and Fredhopper is crucial for accurate search and navigation.

### Full vs. Incremental Updates
* **Full Updates:** Re-index all data in Fredhopper.
* **Incremental Updates:** Update only the changed data.
* Use incremental updates as much as possible to minimize processing time.

### Handling Deletes and Updates
* **Deletes:** Implement mechanisms to delete removed products from Fredhopper.
* **Updates:** Ensure that product updates are reflected in Fredhopper in a timely manner.

### Data Consistency and Latency
* **Data Consistency:** Implement strategies to ensure data consistency between SFCC and Fredhopper.
* **Latency:** Minimize latency between data updates in SFCC and their reflection in Fredhopper.

|                                                                                 |                                                                                                                             |
|---------------------------------------------------------------------------------|-----------------------------------------------------------------------------------------------------------------------------|
| **Previous: [Architectural Principles](../architectural-principles/README.md)** | **Next: [Storefront Integration with Fredhopper Query API](../storefront-integration-with-fredhopper-query-api/README.md)** |