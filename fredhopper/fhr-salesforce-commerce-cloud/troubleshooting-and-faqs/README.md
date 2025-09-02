---
icon: seal-question
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

# Troubleshooting and FAQs

This section addresses common issues and frequently asked questions related to integrating Fredhopper with Salesforce Commerce Cloud (SFCC). It provides practical solutions and guidance for resolving potential problems.

### Common Issues and Solutions

* **Data Ingestion Issues:**
  * **Issue:** Flat file upload fails.
    * **Solution:** Verify file format, schema, and upload permissions. Check Fredhopper logs for error messages.
  * **Issue:** API ingestion fails with authentication errors.
    * **Solution:** Double-check API keys and tokens. Ensure they have the correct permissions.
  * **Issue:** Data synchronization delays.
    * **Solution:** Monitor data ingestion processes and API response times. Optimize data transformation scripts and API query parameters.
* **Query API Issues:**
  * **Issue:** Search results are inaccurate or irrelevant.
    * **Solution:** Fine-tune Fredhopper relevance settings. Verify data mapping and indexing.
  * **Issue:** API response times are slow.
    * **Solution:** Implement caching strategies. Optimize API queries. Monitor server performance and network latency.
  * **Issue:** Facets are not displayed correctly.
    * **Solution:** Verify facet configurations in Fredhopper. Ensure facet data is correctly mapped and returned by the API.
* **Storefront Integration Issues:**
  * **Issue:** JavaScript errors occur during API integration.
    * **Solution:** Debug JavaScript code. Verify API endpoint and request parameters.
  * **Issue:** Search results are not rendered correctly.
    * **Solution:** Check HTML and CSS code. Verify data parsing and rendering logic.
  * **Issue:** Recommendation widgets are not displaying.
    * **Solution:** Verify that the user profile data is being correctly sent to Fredhopper. Check the recommendation configuration in Fredhopper.

### Frequently Asked Questions

* **Q:** What are the best practices for data synchronization between SFCC and Fredhopper?
  * **A:** Use incremental updates for updates throughout the day and conduct regular scheduled full catalog syncs to ensure data consistency & optimization. Implement robust error handling and monitoring. Optimize data transformation scripts.
* **Q:** How can I improve the performance of the Fredhopper Query API?
  * **A:** Implement caching strategies, optimize API queries, and monitor server performance. Use a CDN.
* **Q:** How do I troubleshoot data mapping issues?
  * **A:** Carefully review your data mapping configurations in both SFCC and Fredhopper. Use logging and debugging tools to trace data flow and identify discrepancies.
* **Q:** How do I handle API rate limits?
  * **A:** Implement retry mechanisms with exponential backoff. Monitor API rate limits and optimize API calls to minimize requests.
* **Q:** What are the benefits of using the Common Item Data Pipeline (API) over flat file ingestion?
  * **A:** The API provides real-time updates, increased flexibility, and improved error handling.
* **Q:** How do I implement personalized recommendations?
  * **A:** Use Fredhopper's recommendation features. Pass user context and profile data to the API. Implement A/B testing to optimize recommendation strategies.
* **Q:** How do I test the Fredhopper integration?
  * **A:** Use staging environments, conduct performance testing, and implement user acceptance testing (UAT).
* **Q:** What is the best way to handle deletes from SFCC, inside of Fredhopper?
  * **A:** Ensure that the delete event is registered in your data pipeline, and that a delete request is sent to Items API.
* **Q:** What version of OCAPI should I use?
  * **A:** Always use the latest stable version of OCAPI.
