<a href="http://www.crownpeak.com" target="_blank">![Crownpeak Logo](../../../images/logo/crownpeak-logo.png "Crownpeak Logo")</a>

## [Fredhopper & Salesforce Commerce Cloud Reference Architecture](../README.md)

# Introduction

## Purpose of this Guide
This guide aims to provide developers and architects with a comprehensive set of best practices, architectural patterns, and practical implementation details for successfully integrating Fredhopper with Salesforce Commerce Cloud (SFCC). It serves as a practical resource, drawing from real-world experiences, to streamline data ingestion, optimize storefront integration through the Query API, and ensure scalable, high-performing e-commerce search and navigation. By providing clear guidance and troubleshooting tips, this guide empowers teams to effectively leverage the combined capabilities of Fredhopper and SFCC to deliver enhanced customer experiences.

## Target Audience

- **E-commerce Developers:**
  - Those responsible for implementing and maintaining the storefront integration between SFCC and Fredhopper.
  - Developers working on data integrations.
  - Front-end developers implementing search and navigation functionality.
- **Solution Architects:**
  - Individuals designing the overall architecture of the e-commerce platform.
  - Architects responsible for ensuring scalability, performance, and security of the integration.
  - Those making decisions about data flow and system integration.
- **System Integrators:**
  - Teams implementing and deploying the Fredhopper and SFCC integration.
  - Those responsible for configuring and troubleshooting the integration.
- **Technical Leads:**
  - Individuals overseeing the technical aspects of the e-commerce platform.
  - Those responsible for guiding development teams and ensuring best practices are followed.
- **SFCC Administrators:**
  - Those who are responsible for the data exports out of SFCC, and need to understand how that data will be used within Fredhopper.
- **Fredhopper Administrators:**
  - Those who are responsible for the data imports into Fredhopper, and need to understand how that data is being provided from SFCC.

## Assumptions and Prerequisites
This guide assumes that readers have a foundational understanding of the following:

- **Salesforce Commerce Cloud (SFCC):**
  - Basic knowledge of SFCC's architecture, data model, and development environment.
  - Familiarity with SFCC's OCAPI and Pipelines.
  - Understanding of SFCC's data export capabilities.
- **Fredhopper:**
  - Basic understanding of Fredhopper's indexing, querying, and configuration concepts.
  - Knowledge of Fredhopper's Query API and data ingestion methods (Flat File and Common Item Data Pipeline).
  - Access to a Fredhopper environment.
- **Web Development:**
  - Proficiency in web development concepts (HTML, CSS, JavaScript).
  - Familiarity with RESTful APIs and JSON data format.
- **Data Transformation:**
  - Basic understanding of data transformation concepts and scripting languages

**Prerequisites:**

- Access to a Salesforce Commerce Cloud (SFCC) instance with relevant data.
- Access to a Fredhopper instance.
- Appropriate developer tools and environments (e.g., IDE, command-line tools).
- API credentials for both SFCC and Fredhopper, when the API ingestion method is being used.


|                                                              |   |
|--------------------------------------------------------------|---|
| **Previous: [Table of Contents](../README.md)** | **Next: [Architectural Principles](../architectural-principles/README.md)** |

