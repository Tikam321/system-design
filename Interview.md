# Device Statistics Reporting API – Enterprise Device Management System

## Overview

I worked on an **Enterprise Device Management System** designed to provide Super Administrators with visibility into hardware usage across the organization.

One of the key features I implemented was a **Device Statistics Reporting API** that allows administrators to generate reports containing:

- Device model
- OS version
- Last login timestamp
- Associated users
- Organization / Department filters

The API supports filtering data by:
- Organization
- Department
- Individual user

---

# Technical Challenge

The required data was distributed across multiple database tables:

- Users
- Organizations
- Devices
- User-Device Mapping
- Connection Logs

The dataset could easily contain **millions of records**.

### Problems with a Traditional HTTP Response

If we returned all records in a single response:

- HTTP requests would **timeout**
- The server would consume **large memory**
- Loading the full dataset could **crash the application**

---

# Solution

To address this, I implemented **Server-Sent Events (SSE)** using Spring's `SseEmitter`.

Instead of returning a massive JSON response, the API **streams data incrementally** to the client.

### Key Idea

1. Process the request **asynchronously**
2. Fetch records in **small paginated batches**
3. Stream each batch to the client

Each batch contains **100 records**.

This approach allows the client to start processing data immediately while the server continues sending results.

---

# Backend Implementation

## Streaming API

Used **Spring Boot `SseEmitter`** to stream results from the server to the client.

```java
SseEmitter emitter = new SseEmitter();

@Async
public void streamDeviceStats(SseEmitter emitter) {
    while(hasMoreData) {
        List<DeviceStats> batch = fetchNextBatch();
        emitter.send(batch);
    }
    emitter.complete();
}
```


🗣️ Interview Answer (Clear Story)

I worked on an Enterprise Device Management System where Super Administrators needed visibility into hardware usage across the organization.

One of the major features I built was a Device Statistics Reporting API which allowed admins to download reports containing device models, OS versions, last login timestamps, and user associations. The reports could be filtered by organization, department, or individual users.

⚠️ Technical Challenge

The main challenge was that the required data was spread across multiple tables such as Users, Organizations, Devices, User-Device mappings, and Connection Logs.

When admins generated a report, the dataset could easily contain millions of records.

Returning everything as a single HTTP response caused request timeouts and high memory consumption, which could potentially crash the server.

🛠️ My Solution

To solve this, I implemented Server-Sent Events (SSE) using Spring’s SseEmitter.

Instead of returning a massive JSON payload, I designed the API to stream the results incrementally.

The backend processes the data asynchronously and queries the database in paginated batches of 100 records, streaming each batch to the client as it becomes available.

⚙️ Backend Optimization

To efficiently fetch the data, I used QueryDSL to build complex dynamic queries across multiple tables.

I also used projections so that only the required columns were fetched instead of loading entire entities.

Additionally, I carefully handled LEFT JOINs so that the report included all registered devices, even those that were currently offline, while determining their last active status using connection logs.

🚀 Impact

This approach significantly improved the system:

Prevented HTTP timeouts

Reduced server memory usage

Allowed clients to start processing data immediately

Enabled the system to handle millions of records efficiently
