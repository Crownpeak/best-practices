---
icon: workflow
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

# n8n Workflow: SFCC to Fredhopper Integration

## Introduction

This guide provides comprehensive documentation for the **["Fredhopper n8n Workflow for Salesforce Commerce Cloud"](https://github.com/Crownpeak/n8n-workflow-fredhopper-sfcc)** n8n workflow. This workflow automates the complete process of extracting product data from Salesforce Commerce Cloud (SFCC) and ingesting it into Fredhopper Product Discovery.

### Purpose of this Guide

This documentation enables developers, solution architects, and system integrators to:

* Understand the complete data flow from SFCC to Fredhopper
* Configure and customize the workflow for their specific requirements
* Troubleshoot common issues during data synchronization
* Extend the workflow for additional use cases

### What is n8n?

[n8n](https://n8n.io/) is a free and open-source workflow automation tool that allows you to connect different services and automate tasks. It provides a visual interface for building complex workflows without extensive coding. This workflow leverages n8n's capabilities alongside the **Crownpeak Product Discovery n8n node** to seamlessly integrate SFCC and Fredhopper.

## Workflow Overview

The workflow performs a complete catalog synchronization from Salesforce Commerce Cloud to Fredhopper, including:

1. **Authentication** with both SFCC and Fredhopper APIs
2. **Category extraction** from the SFCC catalog
3. **Schema management** in Fredhopper based on SFCC product attributes
4. **Category tree synchronization** to Fredhopper
5. **Catalog versioning** with inactive catalog creation
6. **Product and variant extraction** from SFCC
7. **Item ingestion** into Fredhopper
8. **Catalog activation** to make changes live

### High-Level Architecture

```
┌─────────────────────────────────────────────────────────────────────────────┐
│                           n8n Workflow Engine                                │
├─────────────────────────────────────────────────────────────────────────────┤
│                                                                              │
│  ┌──────────────┐    ┌──────────────┐    ┌──────────────┐    ┌────────────┐ │
│  │ Configuration│───▶│  Extract     │───▶│  Transform   │───▶│   Load     │ │
│  │              │    │  (SFCC)      │    │              │    │(Fredhopper)│ │
│  └──────────────┘    └──────────────┘    └──────────────┘    └────────────┘ │
│         │                   │                   │                   │        │
│         ▼                   ▼                   ▼                   ▼        │
│  • SFCC Credentials  • Categories        • Attribute      • Item Schema     │
│  • FHR Credentials   • Product Attrs     • Mapping        • Category Tree   │
│  • Catalog Settings  • Products          • Type Convert   • Catalog         │
│                      • Variants                           • Items           │
│                                                                              │
└─────────────────────────────────────────────────────────────────────────────┘
```

### Workflow Stages

| Stage | Description | SFCC API Used | Fredhopper API Used |
|-------|-------------|---------------|---------------------|
| 1. Configuration | Set workflow parameters | - | - |
| 2. Authentication | Obtain OAuth tokens | Account Manager OAuth2 | Authentication API |
| 3. Category Extraction | Fetch catalog categories | Data API, Shop API | - |
| 4. Schema Creation | Create/update item schema | Data API | Item Schema API |
| 5. Category Tree | Sync category hierarchy | - | Category Tree API |
| 6. Catalog Management | Create inactive catalog | - | Catalog API |
| 7. Product Extraction | Fetch products & variants | Shop API | - |
| 8. Item Ingestion | Upload items to Fredhopper | - | Items API |
| 9. Catalog Activation | Activate new catalog | - | Catalog API |

## Prerequisites

### Required Access

* **Salesforce Commerce Cloud:**
  * OCAPI access with Data API and Shop API permissions
  * Client credentials for OAuth2 authentication
  * Access to the catalog and product data

* **Fredhopper Product Discovery:**
  * API credentials (tenant ID, environment)
  * Permissions to manage schemas, category trees, and catalogs

* **n8n:**
  * n8n instance (self-hosted or cloud)
  * [Crownpeak Product Discovery n8n node](https://www.npmjs.com/package/n8n-nodes-crownpeak-pd) installed

### Required Credentials in n8n

| Credential Type | Purpose |
|-----------------|---------|
| HTTP Basic Auth | SFCC OAuth2 client credentials |
| HTTP Bearer Auth | SFCC API access token (auto-managed) |
| Crownpeak PD API | Fredhopper authentication |

## Getting Started

### 1. Import the Workflow

1. Download the workflow JSON file
2. In n8n, navigate to **Workflows** → **Import from File**
3. Select the JSON file and import

### 2. Configure Credentials

Set up the required credentials in n8n:

1. **SFCC OAuth2 Credentials**: Configure HTTP Basic Auth with your SFCC client ID and secret
2. **Fredhopper Credentials**: Configure the Crownpeak PD API credentials

### 3. Update Workflow Configuration

Open the "Workflow configuration" node and update the parameters as described in the [Configuration chapter](configuration/README.md).

### 4. Execute the Workflow

Click "Execute Workflow" to run the full synchronization, or set up a scheduled trigger for automated runs.

## Chapter Overview

This guide is organized into the following chapters:

* **[Configuration](configuration/README.md)** - Detailed configuration parameters and setup
* **[SFCC Data Extraction](sfcc-data-extraction/README.md)** - How the workflow extracts data from SFCC
* **[Fredhopper Data Ingestion](fredhopper-data-ingestion/README.md)** - How data is transformed and loaded into Fredhopper
* **[Troubleshooting](troubleshooting/README.md)** - Common issues and solutions


