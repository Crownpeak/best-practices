---
icon: sitemap
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

# Architecture

This document describes the technical architecture of the Fredhopper Shopify App, focusing on how product data flows from Shopify to Fredhopper through queue-based processing.

## System Overview

The app runs on the [Gadget](https://gadget.dev) platform and uses a **queue-based architecture** for all data synchronization between Shopify and Fredhopper. There are two primary data flows:

* **Bulk Sync** — a full catalog rebuild that processes all products from Shopify into a new inactive Fredhopper catalog, then activates it.
* **Streaming Updates** — real-time product changes via Shopify webhooks, queued and streamed to the active Fredhopper catalog.

Both flows share the same `productStreamingQueue` model, distinguished by the `isBulkItem` flag.

```mermaid
graph TB
    subgraph Shopify
        S_API[Shopify Admin API]
        S_WH[Shopify Webhooks]
    end

    subgraph App ["Fredhopper Shopify App (Gadget)"]
        SM[Sync Manager UI]
        SCH[Smart Schedule Monitor]
        INIT[initializeQueuedBulk]
        POP[populateQueuedBulkBackground]
        PROC_B[processQueuedBulk]
        PROC_S[processProductStreamingQueue]
        DISP_S[dispatchProductStreamingQueue]
        DISP_B[dispatchQueuedBulk]
        ACT[triggerBulkActivation]
        REPLAY[replayCatchupQueue]
        FLUSH[flushInactiveQueueBeforeActivation]
        STREAM[streamChunkToFredhopperBackground]
        PSQ[(productStreamingQueue)]
    end

    subgraph Fredhopper ["Fredhopper (Crownpeak)"]
        CAT_A[Active Catalog]
        CAT_I[Inactive Catalog]
        ITEMS_API[Items API]
    end

    SM -->|Start Bulk Sync| INIT
    SCH -->|Scheduled Start| INIT
    INIT --> POP
    INIT --> PROC_B
    POP -->|Fetch all product IDs| S_API
    POP -->|Write bulk items| PSQ
    PROC_B -->|Read bulk items| PSQ
    PROC_B -->|Fetch product data| S_API
    PROC_B --> STREAM
    STREAM -->|Stream to inactive| ITEMS_API
    ITEMS_API --> CAT_I
    PROC_B -->|All done + autoActivate| ACT
    ACT --> FLUSH
    FLUSH -->|Stream catch-up items| ITEMS_API
    ACT -->|POST /catalogs/activate| ITEMS_API
    ACT --> REPLAY
    REPLAY -->|Replay to active| ITEMS_API
    ITEMS_API --> CAT_A

    S_WH -->|Product create/update/delete| PSQ
    DISP_S -->|Every minute| PROC_S
    PROC_S -->|Read streaming items| PSQ
    PROC_S -->|Fetch product data| S_API
    PROC_S --> STREAM
    STREAM -->|Stream to active| ITEMS_API

    DISP_B -->|Every minute| PROC_B

    style PSQ fill:#f9f,stroke:#333,stroke-width:2px
    style CAT_A fill:#9f9,stroke:#333,stroke-width:2px
    style CAT_I fill:#ff9,stroke:#333,stroke-width:2px
```

## Bulk Sync Flow

A bulk sync is the process of rebuilding an entire Fredhopper catalog from scratch. It is triggered either manually from the Sync Manager or automatically via the Smart Schedule Monitor.

### Bulk Sync Lifecycle

```mermaid
stateDiagram-v2
    [*] --> populating : initializeQueuedBulk
    populating --> processing : All product IDs queued
    populating --> failed : Error fetching products
    populating --> cancelled : User cancels

    processing --> ready_for_activation : All items processed\nautoActivate = off
    processing --> activating : triggerBulkActivation\nautoActivate = on
    processing --> failed : Critical error
    processing --> cancelled : User cancels

    ready_for_activation --> activating : User clicks Activate
    ready_for_activation --> cancelled : User cancels

    activating --> catching_up : Activation request sent
    activating --> failed : Activation API error

    catching_up --> completed : All catch-up items replayed
    catching_up --> failed : Replay error

    failed --> processing : retryQueuedBulk
    cancelled --> processing : retryQueuedBulk

    [*] --> skipped : Schedule skipped\n(process already running)
```

### Bulk Sync Step-by-Step

```mermaid
sequenceDiagram
    actor User
    participant SM as Sync Manager
    participant INIT as initializeQueuedBulk
    participant FH as Fredhopper API
    participant POP as populateQueuedBulk-<br/>Background
    participant Q as productStreamingQueue
    participant PROC as processQueuedBulk
    participant STREAM as streamChunkTo-<br/>FredhopperBackground
    participant ACT as triggerBulkActivation

    User->>SM: Start Bulk Sync
    SM->>INIT: shopId, schemaId, autoActivate
    INIT->>FH: Create inactive catalog
    FH-->>INIT: catalogVersion
    INIT->>INIT: Create bulkQueueProcess (populating)
    INIT-->>POP: Enqueue (high priority)
    INIT-->>PROC: Enqueue (5s delay)

    POP->>POP: Fetch all Shopify product IDs (paginated)
    loop For each page of products
        POP->>Q: Write product IDs (isBulkItem=true)
    end
    POP->>POP: Set status → processing

    loop Until queue empty or 10min elapsed
        PROC->>Q: Fetch batch (250 items)
        PROC->>PROC: Fetch products from Shopify (40/batch)
        PROC->>PROC: Transform to Fredhopper format
        PROC->>STREAM: Enqueue chunks (max 1MB each)
        STREAM->>FH: POST items to inactive catalog
    end

    alt 10min elapsed, items remaining
        PROC-->>PROC: Re-enqueue self (1s delay)
    else All items processed
        alt autoActivate = true
            PROC-->>ACT: Enqueue triggerBulkActivation
        else autoActivate = false
            PROC->>PROC: Set status → ready_for_activation
        end
    end
```

### Self-Re-Enqueueing Pattern

The bulk processor uses a **self-re-enqueueing pattern** to work around the 15-minute execution timeout of the Gadget platform. After processing items for 10 minutes, the processor saves a checkpoint and enqueues itself with a 1-second delay. This creates a continuous processing chain that runs until every item is processed.

```mermaid
graph LR
    A[processQueuedBulk<br/>Run 1: Items 1-250] -->|10min checkpoint| B[processQueuedBulk<br/>Run 2: Items 251-500]
    B -->|10min checkpoint| C[processQueuedBulk<br/>Run 3: Items 501-750]
    C -->|Queue empty| D[triggerBulkActivation<br/>or ready_for_activation]
```

## Streaming Updates Flow

Streaming updates handle real-time product changes from Shopify. When a product is created, updated, or deleted, a Shopify webhook triggers the creation of a queue item.

```mermaid
sequenceDiagram
    participant Shopify
    participant WH as Webhook Handler<br/>(streamProductUpdate)
    participant Q as productStreamingQueue
    participant DISP as dispatchProduct-<br/>StreamingQueue
    participant PROC as processProduct-<br/>StreamingQueue
    participant FH as Fredhopper API

    Shopify->>WH: Product created/updated/deleted
    WH->>Q: Create queue item<br/>(isBulkItem=false)

    Note over WH,Q: If bulk sync is active,<br/>also write catch-up entry

    DISP->>DISP: Cron: every minute
    DISP->>Q: Check for pending items
    DISP-->>PROC: Enqueue processor

    loop Until queue empty or 10min elapsed
        PROC->>Q: Fetch batch (250 items)
        PROC->>PROC: Group by shop
        PROC->>PROC: Fetch product data from Shopify
        PROC->>PROC: Transform with active schema
        PROC->>FH: Stream to active catalog
    end
```

### Dispatcher-Worker Pattern

Both bulk and streaming flows use a **dispatcher-worker pattern**:

* **Dispatcher** (lightweight, 15s timeout): Runs on a cron schedule (every minute), checks for pending work, and enqueues the worker.
* **Worker** (heavyweight, 15min timeout): Processes items in batches, with self-re-enqueueing for long-running operations.

Named queues with `maxConcurrency: 1` ensure that only one worker instance runs per shop at a time, preventing duplicate processing.

```mermaid
graph LR
    subgraph "Every Minute (Cron)"
        D1[dispatchProductStreamingQueue]
        D2[dispatchQueuedBulk]
    end

    subgraph "Workers (maxConcurrency: 1)"
        W1[processProductStreamingQueue<br/>Queue: product-streaming-queue]
        W2[processQueuedBulk<br/>Queue: bulk-shopId]
    end

    D1 -->|Enqueue if pending items| W1
    D2 -->|Enqueue if active process| W2
```

## Catch-Up Mechanism

The catch-up mechanism ensures that no product updates are lost when a bulk sync is running. It uses a **dual-write strategy** for webhook events.

```mermaid
sequenceDiagram
    participant Shopify
    participant WH as streamProductUpdate
    participant Q as productStreamingQueue
    participant PROC_S as processProduct-<br/>StreamingQueue
    participant ACT as triggerBulk-<br/>Activation
    participant FLUSH as flushInactiveQueue-<br/>BeforeActivation
    participant REPLAY as replayCatchupQueue
    participant FH_A as Fredhopper<br/>Active Catalog
    participant FH_I as Fredhopper<br/>Inactive Catalog

    Note over Shopify,FH_I: During Bulk Sync (processing/activating)

    Shopify->>WH: Product updated
    WH->>Q: Entry 1: Active queue<br/>(isBulkItem=false, catchupPhase=none)
    WH->>Q: Entry 2: Catch-up queue<br/>(catchupPhase=pending_catchup,<br/>targetCatalogVersion=inactive)

    PROC_S->>Q: Process active entries
    PROC_S->>FH_A: Stream to active catalog

    Note over ACT,FH_I: When bulk processing completes

    ACT->>ACT: 10s grace period
    loop Until no more catch-up items
        ACT->>FLUSH: Process pending_catchup items
        FLUSH->>Q: Fetch catch-up items
        FLUSH->>FH_I: Stream to inactive catalog
    end
    ACT->>FH_I: POST /catalogs/activate
    Note over FH_I,FH_A: Inactive becomes Active

    REPLAY->>Q: Fetch remaining catch-up items
    REPLAY->>FH_A: Replay to now-active catalog
    REPLAY->>REPLAY: Set status → completed
```

### Catch-Up Phases

| Phase | Description |
|-------|-------------|
| `none` | Normal streaming item, not related to bulk sync |
| `pending_catchup` | Webhook event received during bulk sync, waiting to be flushed/replayed |
| `replayed` | Catch-up item has been successfully replayed after catalog activation |

## Queue Item Lifecycle

All items in the `productStreamingQueue` follow a standardized lifecycle with automated cleanup.

```mermaid
stateDiagram-v2
    [*] --> pending : Webhook or Bulk populate

    pending --> waiting_for_schema : Schema not synced yet
    waiting_for_schema --> pending : Schema becomes available

    pending --> processing : Worker picks up item
    processing --> completed : Successfully streamed
    processing --> failed : Error after 3 retries
    processing --> pending : Retry (retryCount < 3)

    completed --> deletable : After 1 hour (cleanup job)
    deletable --> [*] : After 24 hours (physically deleted)

    pending --> cancelled : Bulk sync cancelled
    processing --> cancelled : Bulk sync cancelled
```

### Cleanup Process

A hourly cron job (`dispatchCompletedQueueCleanup`) triggers the cleanup worker:

1. **Phase 1**: Completed items older than 1 hour are marked as `deletable`.
2. **Phase 2**: Deletable items older than 24 hours are physically removed from the database.

This two-phase approach keeps recent items available for debugging while preventing unbounded queue growth.

## Smart Scheduling

The Smart Schedule Monitor runs every minute and evaluates configured schedules to determine if a bulk sync should be started.

```mermaid
flowchart TD
    CRON[Cron: Every Minute] --> CHECK{Schedule\nenabled?}
    CHECK -->|No| SKIP_DISABLED[Skip]
    CHECK -->|Yes| WEEKDAY{Today is a\nselected weekday?}
    WEEKDAY -->|No| SKIP_DAY[Skip]
    WEEKDAY -->|Yes| INTERVAL{Interval since\nlast run met?}
    INTERVAL -->|No| SKIP_INT[Skip]
    INTERVAL -->|Yes| TIMING{Smart timing:\noptimal start\ntime reached?}
    TIMING -->|No| SKIP_TIME[Skip: Not yet time]
    TIMING -->|Yes| RUNNING{Bulk process\nalready running?}
    RUNNING -->|Yes, skipIfRunning=true| SKIPPED[Create skipped\nrecord + notify]
    RUNNING -->|No| THRESHOLD{Recent failures\nexceed threshold?}
    THRESHOLD -->|Yes| PAUSE[Pause scheduling]
    THRESHOLD -->|No| START[Start bulk sync\ninitializeQueuedBulk]
    START --> UPDATE[Update schedule:\nlastRun, nextRun]
    START -->|Error| ERROR{errorCount\n>= maxRetries?}
    ERROR -->|Yes| DISABLE[Disable schedule]
    ERROR -->|No| INCREMENT[Increment errorCount]
```

### Optimal Start Time Calculation

When a **desired activation time** is configured, the scheduler uses historical data to calculate when the bulk sync should start so that it completes on time.

```mermaid
graph LR
    subgraph "Historical Data"
        H1[Run 1: 45min]
        H2[Run 2: 50min]
        H3[Run 3: 42min]
        H4[Run 4: 48min]
        H5[Run 5: 55min]
    end

    subgraph "Fibonacci-Weighted Average"
        W[Weights: 8, 5, 3, 2, 1<br/>Newest first]
    end

    subgraph "Calculation"
        AVG[Average ms/product]
        COUNT[Current product count]
        EST[Estimated duration]
        BUF[Buffer: max 10%, 15min]
        OPT[Optimal start time]
    end

    H1 & H2 & H3 & H4 & H5 --> W --> AVG
    AVG --> EST
    COUNT --> EST
    EST --> BUF --> OPT

    subgraph "Result"
        DES[Desired: 06:00 AM]
        OPT2["Start: 04:52 AM<br/>(53min + 15min buffer)"]
    end

    OPT --> OPT2
```

**Confidence levels** based on available history:

| Level | Runs | Description |
|-------|------|-------------|
| **High** | 4+ | Reliable prediction based on sufficient data points |
| **Medium** | 2–3 | Reasonable estimate that may vary |
| **Low** | 0–1 | Uses default of 500ms/product, recommend monitoring |

## Data Model

### Core Models

```mermaid
erDiagram
    shopifyShop ||--o{ bulkQueueProcess : "has many"
    shopifyShop ||--o{ productStreamingQueue : "has many"
    shopifyShop ||--o{ categoryStreamingQueue : "has many"
    shopifyShop ||--o{ jobSchedule : "has many"
    bulkQueueProcess ||--o{ queueStatsSnapshot : "tracked by"

    bulkQueueProcess {
        enum status "pending|populating|processing|ready_for_activation|activating|catching_up|completed|failed|cancelled|skipped"
        boolean autoActivate "default: true"
        string targetCatalogVersion
        string targetSchemaId
        number totalItemsQueued
        number itemsProcessed
        number itemsFailed
        datetime startedAt
        datetime completedAt
        datetime activatedAt
        boolean wasScheduled
        string skippedReason
    }

    productStreamingQueue {
        enum status "pending|waiting_for_schema|processing|completed|deletable|failed|cancelled"
        string itemId "Shopify Product ID"
        enum operation "CREATE|UPDATE|DELETE"
        boolean isBulkItem "default: false"
        string bulkProcessId
        string targetCatalogVersion
        enum catchupPhase "none|pending_catchup|replayed"
        number retryCount "default: 0"
        number maxRetries "default: 3"
    }

    categoryStreamingQueue {
        enum status "pending|processing|completed|deletable|failed|cancelled"
        string itemId "Collection ID"
        enum operation "CREATE|UPDATE|DELETE"
        json collectionData
    }

    jobSchedule {
        enum batchType "bulk_sync"
        boolean enabled
        number intervalValue
        enum intervalUnit "day|week"
        json selectedWeekdays
        string desiredActivationTime "HH:mm"
        string latestStartTime "HH:mm"
        string timezone
        boolean skipIfRunning "default: true"
        number maxFailedItemsThreshold "default: 100"
        number errorCount
        number maxRetries "default: 5"
    }

    queueStatsSnapshot {
        number bulkPending
        number bulkProcessing
        number bulkCompleted
        number bulkFailed
        number streamingPending
        number streamingProcessing
        number streamingCompleted
        number streamingFailed
        number totalQueueSize
    }
```

## Configuration

Key processing parameters are centrally configured and can be overridden via environment variables:

| Parameter | Default | Description |
|-----------|---------|-------------|
| `MAX_PAYLOAD_SIZE_BYTES` | 1 MB | Maximum payload size per Fredhopper API request |
| `FREDHOPPER_CHUNK_SIZE` | 1000 | Maximum items per Fredhopper API request |
| `SHOPIFY_BATCH_SIZE` | 250 | Shopify GraphQL query batch size |
| `QUEUE_MAX_CONCURRENCY` | 50 | Maximum parallel streams per tenant |
| `QUEUE_RETRY_COUNT` | 3 | Retry attempts per queue item |
| `QUEUE_BACKOFF_FACTOR` | 2 | Exponential backoff multiplier for retries |
| `MAX_ITEMS_PER_RUN` | 250 | Items processed per streaming run |
| `STREAMING_QUEUE_GLOBALLY_ENABLED` | true | Global on/off switch for streaming processing |
