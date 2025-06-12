---
layout:
  title:
    visible: false
  description:
    visible: false
  tableOfContents:
    visible: true
  outline:
    visible: true
  pagination:
    visible: true
---

# [Fredhopper & Salesforce Commerce Cloud Reference Architecture](./)

This guide provides best practices for integrating Fredhopper with Salesforce Commerce Cloud (SFCC), covering architectural principles, data ingestion, and storefront integration.

**Table of Contents**

1. [**Introduction**](introduction/)
   * [Purpose of this Guide](introduction/#purpose-of-this-guide)
   * [Target Audience](introduction/#target-audience)
   * [Assumptions and Prerequisites](introduction/#assumptions-and-prerequisites)
2. [**Architectural Principles**](architectural-principles/)
   * [High-Level Architecture Overview](architectural-principles/#high-level-architecture-overview)
   * [Scalability and Performance Considerations](architectural-principles/#scalability-and-performance-considerations)
   * [Security Best Practices](architectural-principles/#security-best-practices)
3. [**Data Transformation and Ingestion**](data-transformation-and-ingestion/)
   * [Overview of Data Flow](data-transformation-and-ingestion/#overview-of-data-flow)
   * [Items API Ingestion](data-transformation-and-ingestion/#items-api-ingestion)
   * [Flat File Ingestion](data-transformation-and-ingestion/#flat-file-ingestion)
   * [Data Synchronization Strategies](data-transformation-and-ingestion/#data-synchronization-strategies)
4. [**Storefront Integration**](storefront-integration/)
   * [Query API Overview](storefront-integration/)
   * [Implementing Search and Navigation](storefront-integration/#implementing-search-and-navigation)
   * [Personalization and Recommendations](storefront-integration/#personalization-and-recommendations)
   * [Performance Optimization](storefront-integration/#performance-optimization)
   * [Error Handling and Fallbacks](storefront-integration/#error-handling-and-fallbacks)
5. [**Deployment and Operational Considerations**](deployment-and-operational-considerations/)
   * [Environment Setup (Development, Staging, Production)](deployment-and-operational-considerations/#environment-setup-development-staging-production)
   * [Monitoring and Logging](deployment-and-operational-considerations/#monitoring-and-logging)
   * [Disaster Recovery and Backup](deployment-and-operational-considerations/#disaster-recovery-and-backup)
   * [Performance Testing and Tuning](deployment-and-operational-considerations/#performance-testing-and-tuning)
6. [**Troubleshooting and FAQs**](troubleshooting-and-faqs/)
   * [Common Issues and Solutions](troubleshooting-and-faqs/#common-issues-and-solutions)
   * [Frequently Asked Questions](troubleshooting-and-faqs/#frequently-asked-questions)

### Legal Notices

All Crownpeak documentation is subject to the [MIT license](https://github.com/Crownpeak/fhr-client-proxy?tab=MIT-1-ov-file).

Copyright © 2025 Crownpeak Technology, Inc. All rights reserved. Fredhopper is a trademark of Crownpeak Technology, Inc.

### Disclaimer

This document is provided for information purposes only. Crownpeak may change the contents hereof without notice. This document is not warranted to be error-free, nor subject to any other warranties or conditions, whether expressed orally or implied in law, including implied warranties and conditions of merchantability or fitness for a particular purpose. Crownpeak specifically disclaims any liability with respect to this document and no contractual obligations are formed either directly or indirectly by this document. The technologies, functionality, services, and processes described herein are subject to change without notice.
