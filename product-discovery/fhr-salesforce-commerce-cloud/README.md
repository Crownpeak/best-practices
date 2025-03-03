<a href="http://www.crownpeak.com" target="_blank">![Crownpeak Logo](../../images/logo/crownpeak-logo.png "Crownpeak Logo")</a>

# [Fredhopper & Salesforce Commerce Cloud Reference Architecture Guide](./README.md)

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
3.  **Data Transformation and Ingestion**
    * Overview of Data Flow
    * Flat File Ingestion
        * Data Extraction from SFCC
        * Data Transformation (Example Scripts/Snippets)
        * File Structure and Schema
        * Upload Process to Fredhopper
        * Troubleshooting Flat File Ingestion
    * Common Item Data Pipeline (API) Ingestion
        * API Overview and Authentication
        * Data Mapping and Transformation (JSON Examples)
        * Real-time vs. Batch Updates
        * Error Handling and Monitoring
        * Code Snippets for API Calls (e.g., cURL, Python)
        * Troubleshooting API Ingestion
    * Data Synchronization Strategies
        * Full vs. Incremental Updates
        * Handling Deletes and Updates
        * Data Consistency and Latency
4.  **Storefront Integration with Fredhopper Query API**
    * Query API Overview
    * Implementing Search and Navigation
        * Request Structure (JSON Examples)
        * Response Handling and Rendering
        * Faceting and Filtering
        * Sorting and Relevance
    * Personalization and Recommendations
        * User Context and Profiles
        * Implementing Recommendations
        * A/B Testing and Optimization
    * Performance Optimization
        * Caching Strategies
        * Query Optimization
        * Load Balancing
    * Error Handling and Fallbacks
5.  **Deployment and Operational Considerations**
    * Environment Setup (Development, Staging, Production)
    * Monitoring and Logging
    * Disaster Recovery and Backup
    * Performance Testing and Tuning
6.  **Case Studies and Examples**
    * Real-world Examples of Successful Integrations
    * Code Samples and Configuration Snippets
7.  **Troubleshooting and FAQs**
    * Common Issues and Solutions
    * Frequently Asked Questions
8.  **Contributing**
    * How to Contribute to this Guide
    * License Information

##  Legal Notices
All Crownpeak documentation is subject to the [MIT license](https://github.com/Crownpeak/fhr-client-proxy?tab=MIT-1-ov-file).

Copyright © 2025 Crownpeak Technology, Inc. All rights reserved. Fredhopper is a trademark of Crownpeak Technology, Inc.

## Disclaimer
This document is provided for information purposes only. Crownpeak may change the contents hereof without notice. This document is not warranted to be error-free, nor subject to any other warranties or conditions, whether expressed orally or implied in law, including implied warranties and conditions of merchantability or fitness for a particular purpose. Crownpeak specifically disclaims any liability with respect to this document and no contractual obligations are formed either directly or indirectly by this document. The technologies, functionality, services, and processes described herein are subject to change without notice.
