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
