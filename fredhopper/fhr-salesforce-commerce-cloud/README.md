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
    * [Scalability and Performance Considerations](./architectural-principles/README.md#scalability-and-performance-considerations)
    * [Security Best Practices](./architectural-principles/README.md#security-best-practices)
3.  **[Data Transformation and Ingestion](./data-transformation-and-ingestion/README.md)**
    * [Overview of Data Flow](./data-transformation-and-ingestion/README.md#overview-of-data-flow)
    * [Items API Ingestion](./data-transformation-and-ingestion/README.md#items-api-ingestion)
    * [Flat File Ingestion](./data-transformation-and-ingestion/README.md#flat-file-ingestion)
    * [Data Synchronization Strategies](./data-transformation-and-ingestion/README.md#data-synchronization-strategies)
4.  **[Storefront Integration with Fredhopper Query API](./storefront-integration-with-fredhopper-query-api/README.md)**
    * [Query API Overview](./storefront-integration-with-fredhopper-query-api/README.md)
    * [Implementing Search and Navigation](./storefront-integration-with-fredhopper-query-api/README.md#implementing-search-and-navigation)
    * [Personalization and Recommendations](./storefront-integration-with-fredhopper-query-api/README.md#personalization-and-recommendations)
    * [Performance Optimization](./storefront-integration-with-fredhopper-query-api/README.md#performance-optimization)
    * [Error Handling and Fallbacks](./storefront-integration-with-fredhopper-query-api/README.md#error-handling-and-fallbacks)
5.  **[Deployment and Operational Considerations](./deployment-and-operational-considerations/README.md)**
    * [Environment Setup (Development, Staging, Production)](./deployment-and-operational-considerations/README.md#environment-setup-development-staging-production)
    * [Monitoring and Logging](./deployment-and-operational-considerations/README.md#monitoring-and-logging)
    * [Disaster Recovery and Backup](./deployment-and-operational-considerations/README.md#disaster-recovery-and-backup)
    * [Performance Testing and Tuning](./deployment-and-operational-considerations/README.md#performance-testing-and-tuning)
6.  **[Troubleshooting and FAQs](./troubleshooting-and-faqs/README.md)**
    * [Common Issues and Solutions](./troubleshooting-and-faqs/README.md#common-issues-and-solutions)
    * [Frequently Asked Questions](./troubleshooting-and-faqs/README.md#frequently-asked-questions)


##  Legal Notices
All Crownpeak documentation is subject to the [MIT license](https://github.com/Crownpeak/fhr-client-proxy?tab=MIT-1-ov-file).

Copyright © 2025 Crownpeak Technology, Inc. All rights reserved. Fredhopper is a trademark of Crownpeak Technology, Inc.

## Disclaimer
This document is provided for information purposes only. Crownpeak may change the contents hereof without notice. This document is not warranted to be error-free, nor subject to any other warranties or conditions, whether expressed orally or implied in law, including implied warranties and conditions of merchantability or fitness for a particular purpose. Crownpeak specifically disclaims any liability with respect to this document and no contractual obligations are formed either directly or indirectly by this document. The technologies, functionality, services, and processes described herein are subject to change without notice.
