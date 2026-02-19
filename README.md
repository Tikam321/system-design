# Dropbox System Design – HLD (Interview Revision Notes)
---
# 1. Requirements

## Functional Requirements (FR)

1. Upload files from device
2. Download files on any device
3. Sync changes across devices
4. Version tracking
5. File deduplication
6. Resume interrupted uploads
7. Multi-device consistency
---
## Non-Functional Requirements (NFR)

* High availability
* Strong consistency for metadata
* Low latency sync
* High durability
* Scalability
* Bandwidth efficiency
* Security

---

# 2. Entity Design (Database Schema)

## FileMetadata Table

| Column          | Description                  |
| --------------- | ---------------------------- |
| FileID          | Unique file id               |
| NamespaceID     | User or folder id            |
| RelativePath    | File path                    |
| BlockList       | Ordered list of block hashes |
| LatestJournalID | Pointer to latest version    |

---

## Journal Table

| Column           | Description           |
| ---------------- | --------------------- |
| JournalID        | Version id            |
| FileID           | Reference to file     |
| Timestamp        | Change time           |
| ChangeType       | Add / Modify / Delete |
| ChangedBlockList | Changed blocks        |

---

## Users & Permissions

| Column      | Description          |
| ----------- | -------------------- |
| UserID      | User id              |
| NamespaceID | Namespace            |
| Permissions | Read / Write / Admin |

---

## Relationships

* FileMetadata → Journal = One to Many
* User → FileMetadata = One to Many

---

# 3. API Design

## Metadata Service

### Save Metadata

POST /save_metadata

```json
{
  "filename": "file.txt",
  "path": "/user/file.txt",
  "block_hashes": ["h1","h2"]
}
```

---

### Get Metadata

GET /get_metadata?path=/user/file.txt

Response:

```json
{
  "filename": "file.txt",
  "block_hashes": ["h1","h2"]
}
```

---

## Block Service

### Upload Block

POST /upload/{hash}

Binary block data

---

### Download Block

GET /download/{hash}

Binary block data

---

## Notification Service

GET /long_poll

Returns change event when file updates.

---

# 4. System Design & Data Flow


## Upload Flow
<img width="1880" height="1060" alt="image" src="https://gist.github.com/user-attachments/assets/8e99686b-b14f-4ff2-bfe6-16eddb556f89" />



1. Client chunks file into 4MB blocks
2. Hash each block using SHA-256
3. Upload blocks to Block Server
4. Send metadata to Metadata Server
5. Metadata stored in DB and cache
6. Notification sent to other devices

---

## Download Flow
<img width="2110" height="1060" alt="image" src="https://gist.github.com/user-attachments/assets/d0879ba5-5454-4304-8af1-6239323581d2" />


1. Client receives notification
2. Requests metadata
3. Fetches blocks from Block Server
4. Reconstructs file

---

## Sync After Offline

1. Client sends last timestamp
2. Server queries Journal table
3. Returns changed metadata
4. Client downloads only required blocks

---

# 5. Deep Dive

## Chunking vs Whole File

Chosen: Chunking
Reason: Partial upload, resume, dedup, parallelism
Trade-off: More metadata

---

## Deduplication

Hash-based block storage avoids duplicate storage.

Trade-off: Hash computation cost

---

## Delta Sync

Uses rolling hash and variable chunking.

Benefit: Only changed bytes transferred
Trade-off: CPU overhead

---

## Metadata DB

Chosen: RDBMS
Reason: Strong consistency and transactions
Trade-off: Less horizontal scalability

---

## Notification Strategy

Chosen: Long Polling

Why not WebSocket:

* Overkill
* Stateful complexity

Why not SSE:

* Browser limitations

Trade-off: Slight latency

---

## Consistency Model

Metadata → Strong consistency
Blocks → Eventual consistency

---

## Storage Backend

AWS S3 initially → Magic Pocket

Reason:

* Cost control
* Performance

Trade-off:

* Operational complexity

---

## Failure Handling

* Chunk retry
* Resume upload
* Metadata version rollback
* Cache fallback

---

## Security

* Block encryption
* Hash verification
* Permission isolation
* Namespace separation

---

# Interview Closing Summary

Dropbox uses chunk-based storage with hash-based deduplication, RDBMS-backed metadata with journal versioning, delta sync for bandwidth efficiency, and long polling for device notifications to ensure reliable multi-device synchronization.

---

# Trade-offs to Mention

* Chunking vs full file
* RDBMS vs NoSQL
* Long poll vs WebSocket
* Fixed chunk vs delta sync
* Cloud storage vs own storage
