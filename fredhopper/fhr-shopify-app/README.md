---
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

# Shopify Fredhopper Product Discovery App

### Introduction

The Fredhopper Product Discovery App integrates Crownpeak’s Fredhopper into Shopify, bringing AI-powered search, merchandising, and recommendations directly into your storefront.

It enables faster time-to-market with a ready-made connector, simplified catalog syncs and schema management, global multi-language/multi-region support with Shopify Markets, and secure frontend integration with the Fredhopper Query API via the Shopify App Proxy.

### Key Capabilities

#### Data Integration

* Full & incremental product syncs from Shopify to Fredhopper
* Queue-based bulk sync: all Shopify product IDs are written into a shared processing queue and streamed to an inactive Fredhopper catalog via a self-re-enqueueing worker, with automatic or manual catalog activation.
* Queue-based streaming updates: Shopify webhook events (product create, update, delete) are added to the same processing queue and streamed to the active Fredhopper catalog in near real-time, with a catch-up mechanism that ensures no updates are lost during a running bulk sync.
* Smart scheduling with ETA-based optimal start times, weekday selection, and configurable safety rules.
* Management of schema creation, catalog management, and queued bulk sync through the app.

#### Frontend Integration

* JavaScript SDK exposes Fredhopper’s power directly to developers inside Shopify themes
* Out of the box Theme App Blocks for rendering dynamic PLP/search pages using the Fredhopper Query API
* Uses the Shopify App Proxy to ensure secure, scoped API requests
* Web Pixel integration to capture shopper behavior, powering A/B testing, insights, and AI scores.
