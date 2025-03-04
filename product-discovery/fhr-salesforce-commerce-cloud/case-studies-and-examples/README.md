<a href="http://www.crownpeak.com" target="_blank">![Crownpeak Logo](../../../images/logo/crownpeak-logo.png "Crownpeak Logo")</a>

## [Fredhopper & Salesforce Commerce Cloud Reference Architecture](../README.md)

# Case Studies and Examples
This section showcases real-world examples and practical code samples to demonstrate the successful integration of Fredhopper with Salesforce Commerce Cloud (SFCC). These examples provide valuable insights and practical guidance for implementing the best practices outlined in this guide.

## Real-world Examples of Successful Integrations

* **Case Study 1: Enhanced Product Discovery for a Global Apparel Retailer**
  * A large apparel retailer implemented Fredhopper to improve product discoverability across its global e-commerce sites.
  * By leveraging Fredhopper's advanced search and navigation features, the retailer saw a significant increase in conversion rates and customer satisfaction.
  * They used the API ingestion method to keep product data synchronized in near real time.
  * They tuned relevancy to promote new products and trending items.
* **Case Study 2: Personalized Recommendations for an Electronics Marketplace**
  * An electronics marketplace implemented Fredhopper's recommendation engine to deliver personalized product recommendations to its customers.
  * By analyzing user browsing and purchase history, the marketplace was able to provide highly relevant recommendations, resulting in increased average order value.
  * They utilized customer data platforms to send user profile data to Fredhopper.
* **Case Study 3: Streamlined Data Ingestion for a Multi-Brand E-commerce Platform**
  * A multi-brand e-commerce platform utilized flat file ingestion to efficiently transfer product data from SFCC to Fredhopper.
  * They automated the data extraction and transformation processes, reducing manual effort and ensuring data consistency.
  * They created a data dictionary to ensure all brand data was mapped correctly.

## Code Samples and Configuration Snippets

* **Example 1: Node for Query API Integration**

```node
const axios = require('axios');

async function searchProducts(apiUrl, query) {
  const requestBody = {
    query: query,
    start: 0,
    rows: 20,
  };

  try {
    const response = await axios.post(apiUrl, requestBody, {
      headers: {
        'Content-Type': 'application/json',
      },
    });

    console.log(response.data); // Process and display the search results
  } catch (error) {
    console.error('Error fetching search results:', error);
  }
}

const apiUrl = 'YOUR_FREDHOPPER_QUERY_API_ENDPOINT';
searchProducts(apiUrl, 'laptop');
```

* **Example 2: JavaScript Code for Query API Integration**

```javascript
async function searchProducts(query) {
    const apiUrl = "YOUR_FREDHOPPER_QUERY_API_ENDPOINT";
    const requestBody = {
        query: query,
        start: 0,
        rows: 20
    };

    try {
        const response = await fetch(apiUrl, {
            method: "POST",
            headers: {
                "Content-Type": "application/json"
            },
            body: JSON.stringify(requestBody)
        });

        const data = await response.json();
        // Process and display the search results
        console.log(data);
    } catch (error) {
        console.error("Error fetching search results:", error);
    }
}

searchProducts("laptop");

```
These examples demonstrate how to apply the principles and techniques discussed in this guide. By leveraging these real-world scenarios and code snippets, you can accelerate your Fredhopper and SFCC integration and deliver a superior e-commerce experience.



|                                                                                                                   |                                                                             |
|-------------------------------------------------------------------------------------------------------------------|-----------------------------------------------------------------------------|
| **Previous: [Deployment and Operational Considerations](../deployment-and-operational-considerations/README.md)** | **Next: [Troubleshooting and FAQs](../troubleshooting-and-faqs/README.md)** |