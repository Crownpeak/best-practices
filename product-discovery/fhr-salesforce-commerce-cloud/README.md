<a href="http://www.crownpeak.com" target="_blank">![Crownpeak Logo](../../images/logo/crownpeak-logo.png "Crownpeak Logo")</a>

# [Fredhopper & Salesforce Commerce Cloud Reference Architecture](./README.md)

This guide provides best practices for integrating Fredhopper with Salesforce Commerce Cloud (SFCC), covering architectural principles, data ingestion, and storefront integration.

**Table of Contents**

1.  **[Introduction](./introduction/README.md)**
    * [Purpose of this Guide](./introduction/README.md#purpose-of-this-guide)
    * [Target Audience](./introduction/README.md#target-audience)
    * [Assumptions and Prerequisites](./introduction/README.md#assumptions-and-prerequisites)
2.  **[Architectural Principles](./architectural-principles/README.md)**
    * [High-Level Architecture Overview](./architectural-principles/README.md#high-level-architecture-overview-diagram)
    * [Key Components and Interactions](./architectural-principles/README.md#key-components-and-interactions)
    * [Scalability and Performance Considerations](./architectural-principles/README.md#scalability-and-performance-considerations)
    * [Security Best Practices](./architectural-principles/README.md#security-best-practices)
3.  **[Data Transformation and Ingestion](./data-transformation-and-ingestion/README.md)**
    * [Overview of Data Flow](./data-transformation-and-ingestion/README.md#overview-of-data-flow)
    * [Common Item Data Pipeline (API) Ingestion](./data-transformation-and-ingestion/README.md#common-item-data-pipeline-api-ingestion)
        * [API Overview and Authentication](./data-transformation-and-ingestion/README.md#api-overview-and-authentication)
        * [Data Mapping and Transformation (JSON Examples)](./data-transformation-and-ingestion/README.md#data-mapping-and-transformation-json-examples)
        * [Real-time vs. Batch Updates](./data-transformation-and-ingestion/README.md#real-time-vs-batch-updates)
        * [Error Handling and Monitoring](./data-transformation-and-ingestion/README.md#error-handling-and-monitoring)
        * [Code Snippets for API Calls (e.g., cURL, Python)](./data-transformation-and-ingestion/README.md#code-snippets-for-api-calls-eg-curl-python)
        * [Troubleshooting API Ingestion](./data-transformation-and-ingestion/README.md#troubleshooting-api-ingestion)
    * [Flat File Ingestion](./data-transformation-and-ingestion/README.md#flat-file-ingestion)
        * [Data Extraction from SFCC](./data-transformation-and-ingestion/README.md#data-extraction-from-sfcc)
        * [Data Transformation (Example Scripts/Snippets)](./data-transformation-and-ingestion/README.md#data-transformation-example-scriptssnippets)
        * [File Structure and Schema](./data-transformation-and-ingestion/README.md#file-structure-and-schema)
        * [Upload Process to Fredhopper](./data-transformation-and-ingestion/README.md#upload-process-to-fredhopper)
        * [Troubleshooting Flat File Ingestion](./data-transformation-and-ingestion/README.md#troubleshooting-flat-file-ingestion)
    * [Data Synchronization Strategies](./data-transformation-and-ingestion/README.md#data-synchronization-strategies)
        * [Full vs. Incremental Updates](./data-transformation-and-ingestion/README.md#full-vs-incremental-updates)
        * [Handling Deletes and Updates](./data-transformation-and-ingestion/README.md#handling-deletes-and-updates)
        * [Data Consistency and Latency](./data-transformation-and-ingestion/README.md#data-consistency-and-latency)
4.  **[Storefront Integration with Fredhopper Query API](./storefront-integration-with-fredhopper-query-api/README.md)**
    * [Query API Overview](./storefront-integration-with-fredhopper-query-api/README.md)
    * [Implementing Search and Navigation](./storefront-integration-with-fredhopper-query-api/README.md#implementing-search-and-navigation)
        * [Request Structure (JSON Examples)](./storefront-integration-with-fredhopper-query-api/README.md#request-structure-json-examples)
        * [Response Handling and Rendering](./storefront-integration-with-fredhopper-query-api/README.md#response-handling-and-rendering)
        * [Faceting and Filtering](./storefront-integration-with-fredhopper-query-api/README.md#faceting-and-filtering)
        * [Sorting and Relevance](./storefront-integration-with-fredhopper-query-api/README.md#sorting-and-relevance)
    * [Personalization and Recommendations](./storefront-integration-with-fredhopper-query-api/README.md#personalization-and-recommendations)
        * [User Context and Profiles](./storefront-integration-with-fredhopper-query-api/README.md#user-context-and-profiles)
        * [Implementing Recommendations](./storefront-integration-with-fredhopper-query-api/README.md#implementing-recommendations)
        * [A/B Testing and Optimization](./storefront-integration-with-fredhopper-query-api/README.md#ab-testing-and-optimization)
    * [Performance Optimization](./storefront-integration-with-fredhopper-query-api/README.md#performance-optimization)
        * [Caching Strategies](./storefront-integration-with-fredhopper-query-api/README.md#caching-strategies)
        * [Query Optimization](./storefront-integration-with-fredhopper-query-api/README.md#query-optimization)
        * [Load Balancing](./storefront-integration-with-fredhopper-query-api/README.md#load-balancing)
    * [Error Handling and Fallbacks](./storefront-integration-with-fredhopper-query-api/README.md#error-handling-and-fallbacks)
5.  **[Deployment and Operational Considerations](./deployment-and-operational-considerations/README.md)**
    * [Environment Setup (Development, Staging, Production)](./deployment-and-operational-considerations/README.md#environment-setup-development-staging-production)
    * [Monitoring and Logging](./deployment-and-operational-considerations/README.md#monitoring-and-logging)
    * [Disaster Recovery and Backup](./deployment-and-operational-considerations/README.md#disaster-recovery-and-backup)
    * [Performance Testing and Tuning](./deployment-and-operational-considerations/README.md#performance-testing-and-tuning)
6.  **[Case Studies and Examples](./case-studies-and-examples/README.md)**
    * [Real-world Examples of Successful Integrations](./case-studies-and-examples/README.md#real-world-examples-of-successful-integrations)
    * [Code Samples and Configuration Snippets](./case-studies-and-examples/README.md#code-samples-and-configuration-snippets)
7.  **[Troubleshooting and FAQs](./troubleshooting-and-faqs/README.md)**
    * [Common Issues and Solutions](./troubleshooting-and-faqs/README.md#common-issues-and-solutions)
    * [Frequently Asked Questions](./troubleshooting-and-faqs/README.md#frequently-asked-questions)


##  Legal Notices
All Crownpeak documentation is subject to the [MIT license](https://github.com/Crownpeak/fhr-client-proxy?tab=MIT-1-ov-file).

Copyright © 2025 Crownpeak Technology, Inc. All rights reserved. Fredhopper is a trademark of Crownpeak Technology, Inc.

## Disclaimer
This document is provided for information purposes only. Crownpeak may change the contents hereof without notice. This document is not warranted to be error-free, nor subject to any other warranties or conditions, whether expressed orally or implied in law, including implied warranties and conditions of merchantability or fitness for a particular purpose. Crownpeak specifically disclaims any liability with respect to this document and no contractual obligations are formed either directly or indirectly by this document. The technologies, functionality, services, and processes described herein are subject to change without notice.
