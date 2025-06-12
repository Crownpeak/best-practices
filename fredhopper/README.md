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

# Salesforce Commerce Cloud Reference Architecture

This guide provides best practices for integrating Fredhopper with Salesforce Commerce Cloud (SFCC), covering architectural principles, data ingestion, and storefront integration.

**Table of Contents**

1. [**Introduction**](fhr-salesforce-commerce-cloud/introduction/)
   * [Purpose of this Guide](fhr-salesforce-commerce-cloud/introduction/#purpose-of-this-guide)
   * [Target Audience](fhr-salesforce-commerce-cloud/introduction/#target-audience)
   * [Assumptions and Prerequisites](fhr-salesforce-commerce-cloud/introduction/#assumptions-and-prerequisites)
2. [**Architectural Principles**](fhr-salesforce-commerce-cloud/architectural-principles/)
   * [High-Level Architecture Overview](fhr-salesforce-commerce-cloud/architectural-principles/#high-level-architecture-overview)
   * [Scalability and Performance Considerations](fhr-salesforce-commerce-cloud/architectural-principles/#scalability-and-performance-considerations)
   * [Security Best Practices](fhr-salesforce-commerce-cloud/architectural-principles/#security-best-practices)
3. [**Data Transformation and Ingestion**](fhr-salesforce-commerce-cloud/data-transformation-and-ingestion/)
   * [Overview of Data Flow](fhr-salesforce-commerce-cloud/data-transformation-and-ingestion/#overview-of-data-flow)
   * [Items API Ingestion](fhr-salesforce-commerce-cloud/data-transformation-and-ingestion/#items-api-ingestion)
   * [Flat File Ingestion](fhr-salesforce-commerce-cloud/data-transformation-and-ingestion/#flat-file-ingestion)
   * [Data Synchronization Strategies](fhr-salesforce-commerce-cloud/data-transformation-and-ingestion/#data-synchronization-strategies)
4. [**Storefront Integration**](fhr-salesforce-commerce-cloud/storefront-integration/)
   * [Query API Overview](fhr-salesforce-commerce-cloud/storefront-integration/)
   * [Implementing Search and Navigation](fhr-salesforce-commerce-cloud/storefront-integration/#implementing-search-and-navigation)
   * [Personalization and Recommendations](fhr-salesforce-commerce-cloud/storefront-integration/#personalization-and-recommendations)
   * [Performance Optimization](fhr-salesforce-commerce-cloud/storefront-integration/#performance-optimization)
   * [Error Handling and Fallbacks](fhr-salesforce-commerce-cloud/storefront-integration/#error-handling-and-fallbacks)
5. [**Deployment and Operational Considerations**](fhr-salesforce-commerce-cloud/deployment-and-operational-considerations/)
   * [Environment Setup (Development, Staging, Production)](fhr-salesforce-commerce-cloud/deployment-and-operational-considerations/#environment-setup-development-staging-production)
   * [Monitoring and Logging](fhr-salesforce-commerce-cloud/deployment-and-operational-considerations/#monitoring-and-logging)
   * [Disaster Recovery and Backup](fhr-salesforce-commerce-cloud/deployment-and-operational-considerations/#disaster-recovery-and-backup)
   * [Performance Testing and Tuning](fhr-salesforce-commerce-cloud/deployment-and-operational-considerations/#performance-testing-and-tuning)
6. [**Troubleshooting and FAQs**](fhr-salesforce-commerce-cloud/troubleshooting-and-faqs/)
   * [Common Issues and Solutions](fhr-salesforce-commerce-cloud/troubleshooting-and-faqs/#common-issues-and-solutions)
   * [Frequently Asked Questions](fhr-salesforce-commerce-cloud/troubleshooting-and-faqs/#frequently-asked-questions)

### Legal Notices

All Crownpeak documentation is subject to the [MIT license](https://github.com/Crownpeak/fhr-client-proxy?tab=MIT-1-ov-file).

Copyright © 2025 Crownpeak Technology, Inc. All rights reserved. Fredhopper is a trademark of Crownpeak Technology, Inc.

### Disclaimer

This document is provided for information purposes only. Crownpeak may change the contents hereof without notice. This document is not warranted to be error-free, nor subject to any other warranties or conditions, whether expressed orally or implied in law, including implied warranties and conditions of merchantability or fitness for a particular purpose. Crownpeak specifically disclaims any liability with respect to this document and no contractual obligations are formed either directly or indirectly by this document. The technologies, functionality, services, and processes described herein are subject to change without notice.
