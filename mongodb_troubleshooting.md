# MongoDB Troubleshooting Guide

Quick reference for diagnosing and resolving database performance issues.

## Current Operations

### List long-running operations

```javascript
db.currentOp(true)
```

**What to look for:**

- `secs_running`: How long the operation has been executing. High values (>30s) indicate potential problems.
- `op`: Operation type (`query`, `insert`, `update`, `delete`, `command`, etc.)
- `ns`: Namespace (database.collection being accessed)
- `locks`: Lock types held (`W` = write lock, `R` = read lock)
- `waitingForLock`: If `true`, the operation is waiting for a lock (contention issue)

**Output logic:** Operations running for >30s with high `secs_running` values are candidates for killing if they're blocking other work.

### Filter operations running longer than X seconds

```javascript
db.currentOp({
  secs_running: { $gte: 60 }
})
```

**What to look for:**

- `client`: Client IP address making the request
- `command` or `query`: The actual query being executed
- `secs_running`: Total elapsed time
- `locks` and `waitingForLock`: Indicates if the operation is blocked

**Output logic:** Returns only operations exceeding your threshold. Focus on queries with high `secs_running` and `waitingForLock: true` — these are blocking other operations.

**Compact summary (mongosh):** Maps `inprog` to a small object per operation. `opid` is the value passed to `db.killOp()`. Modern servers usually populate `command`; older or rare shapes may only have `query`, so the mapped `command` field uses `op.command ?? op.query`.

```javascript
db.currentOp({
  secs_running: { $gte: 60 }
}).inprog.map((op) => ({
  secs_running: op.secs_running,
  query: op.query,
  operation: op.op,
  opid: op.opid,
  command: op.command ?? op.query,
  collection: op.ns ? op.ns.split(".")[1] : null,
}))
```

**Replace `60` with your desired threshold in seconds.**

### Kill a specific operation

```javascript
db.killOp(opId)
```

**Usage:** Replace `opId` with the operation ID from `currentOp()` output (look for `opid` field).

**Note:** Use cautiously. Killing operations may leave data in an inconsistent state. Best used for queries, not writes in progress.

## Index Analysis

### Find queries without indexes

```javascript
db.collection('your_collection').aggregate([
  { $indexStats: {} }
]).pretty()
```

**What to look for:**

- `name`: Index name
- `accesses.ops`: How many times the index was used. Low values = unused index
- `accesses.since`: When the index was last accessed

**Output logic:** Look for indexes with `ops: 0` or very old `since` timestamps — these are candidates for deletion. Collections with 0 results may not have any indexes at all.

**Replace `your_collection` with the actual collection name.**

### List all indexes on a collection

```javascript
db.your_collection.getIndexes()
```

**What to look for:**

- `key`: The fields indexed (e.g., `{ "email": 1 }` for ascending order)
- `unique`: Whether the index enforces uniqueness
- `sparse`: Whether null values are indexed
- `background`: If `true`, the index was built without blocking writes

**Replace `your_collection` with the actual collection name.**

### Check index size

```javascript
db.your_collection.stats().indexSizes
```

**What to look for:**

- Size of each index in bytes
- Large indexes consume memory and slow down writes

**Output logic:** Compare index sizes with `avgObjSize`. Indexes significantly larger than data often indicate unnecessary compound indexes or indexes on text fields that should be trimmed.

**Replace `your_collection` with the actual collection name.**

### Find candidate fields for indexing

```javascript
db.your_collection.find().explain("executionStats")
```

**What to look for:**

- `executionStages.stage`: If it says `COLLSCAN`, the query scanned the entire collection without an index
- `executionStats.totalDocsExamined`: Total documents scanned
- `executionStats.totalKeysExamined`: Total index keys examined
- `executionStats.nReturned`: Actual results returned
- **Efficiency ratio:** `totalKeysExamined / nReturned` should be close to 1. High ratios mean many documents were examined to find results.

**Output logic:** High `totalDocsExamined` vs `nReturned` or `COLLSCAN` stages indicate missing indexes on the fields in your query filter.

**Replace `your_collection` with the actual collection name. Include a `find()` filter to analyze specific queries: `db.your_collection.find({status: "active"}).explain("executionStats")`**

## Index Creation Status

### List indexes being built

```javascript
db.currentOp(true).inprog.filter(op => op.op === 'command' && op.command.createIndexes)
```

**What to look for:**

- `msg`: Shows "Index Build" with progress percentage (on MongoDB 4.2+)
- `progress`: Index build completion percentage
- `locks`: Lock level being held

**Output logic:** Empty result means no indexes are currently being built. Non-empty results show ongoing index builds that may be consuming resources.

### Check background index build progress

```javascript
db.admin.system.indexes.find().pretty()
```

**What to look for:**

- `v`: Index version (should be 2 for modern MongoDB)
- Creation status in the document

**Note:** This is database-specific. Better alternatives: Check MongoDB 4.2+ with `db.currentOp()` or monitor logs for `"Index Build"` messages.

### Monitor index creation with logging

```javascript
db.setLogLevel(1, "index")
```

## Resource Monitoring

### Check database stats for size and operation count

```javascript
db.stats()
```

**What to look for:**

- `db`: Database name
- `collections`: Number of collections
- `dataSize`: Raw data size (documents only)
- `indexSize`: Combined size of all indexes
- `storageSize`: Actual disk usage (includes fragmentation)
- `fileSize`: Physical file size on disk

**Output logic:** If `storageSize >> dataSize`, fragmentation is high — consider running `db.repairDatabase()` (requires downtime). High `indexSize` relative to `dataSize` suggests too many or overly large indexes.

### Get collection size details

```javascript
db.your_collection.stats()
```

**What to look for:**

- `count`: Number of documents in the collection
- `size`: Raw data size
- `avgObjSize`: Average document size in bytes
- `storageSize`: Allocated space (includes free space)
- `totalIndexSize`: Total size of all indexes on this collection

**Output logic:** Compare `storageSize` to `size`. High fragmentation (much larger storageSize) impacts memory efficiency. High `totalIndexSize` relative to `size` indicates expensive indexing.

**Replace `your_collection` with the actual collection name.**

### Monitor memory usage

```javascript
db.serverStatus().mem
```

**What to look for:**

- `resident`: Memory currently in RAM (MB) — this is what matters for performance
- `virtual`: Total virtual memory allocated (MB)
- `mapped` (pre-4.2): Amount of data mapped into memory

**Output logic:** If `resident` approaches available system memory, MongoDB may face memory pressure. High `resident` with slow queries can indicate missing indexes causing full collection scans. Compare against system available memory.

**Alert threshold:** When `resident` > 80% of available RAM, investigate slow queries and consider adding indexes or removing unused indexes.

### Check replication lag (if replica set)

```javascript
rs.status()
```

**What to look for:**

- `members[].state`: 1 = primary, 2 = secondary, 0 = down
- `members[].optimeDate`: Timestamp of last operation applied
- `members[].syncSourceHost`: Which node the secondary is syncing from
- `members[].health`: 1 = healthy, 0 = down

**Output logic:** Compare `optimeDate` across members. Lagging secondaries (older `optimeDate`) can't handle reads properly. Lag indicates write volume exceeding replication network speed or secondary resource constraints.

**Alert threshold:** Lag >30 seconds indicates replication is falling behind write volume.

## Connection & Lock Info

### List all client connections

```javascript
db.currentOp().client
```

**What to look for:**

- `host`: IP:port of connecting client
- `desc`: Client description (connection string or driver info)

**Output logic:** Shows what clients are connected. High connection count from a single client may indicate connection pooling issues or misconfigured applications.

### Get database lock status

```javascript
db.serverStatus().locks
```

**What to look for:**

- `Global`, `Database`, `Collection` lock objects
- `acquisitionCount`: How many times the lock was acquired
- `waitCount`: How many times an operation waited for the lock
- `timeAcquiringMicros`: Total time spent waiting for the lock

**Output logic:** High `waitCount` relative to `acquisitionCount` indicates lock contention. Very high `timeAcquiringMicros` means operations are frequently blocked waiting for locks — a major performance issue.

**Alert threshold:** If `waitCount > acquisitionCount * 0.1`, investigate concurrent write patterns or consider sharding.

## Quick Diagnostics

### Run a full server diagnostic

```javascript
db.serverStatus()
```

**Key fields to analyze:**

- `mem`: Memory usage (see Memory Monitoring section)
- `connections`: Current and available connections
- `opcounters`: Operation counts (insert, query, update, delete, command)
- `locks`: Lock statistics (see Lock Info section)
- `storageEngine`: Which engine is in use
- `uptime`: How long the server has been running

**Output logic:** A comprehensive health snapshot. Use this after high CPU/memory alerts to correlate memory usage, connection count, and operation volume. Spikes in specific opcounters may indicate problematic traffic patterns.

### Identify slow queries (if profiling enabled)

```javascript
db.system.profile.find().limit(5).sort({ ts: -1 }).pretty()
```

**What to look for:**

- `millis`: Execution time in milliseconds — queries >100ms warrant investigation
- `nscanned` vs `nreturned`: If much higher than returned documents, the query lacks an index
- `execStats.executionStages.stage`: Look for `COLLSCAN` stages
- `command`: The actual query being executed
- `ts`: Timestamp of execution

**Output logic:** Sort by milliseconds to find slowest queries. High `nscanned` values or `COLLSCAN` stages indicate missing indexes.

**Enable profiling first:**

```javascript
db.setProfilingLevel(1)  // Log slow queries (default: >100ms)
db.setProfilingLevel(1, { slowms: 50 })  // Log queries slower than 50ms
```

**Note:** Profiling adds overhead. Disable when done: `db.setProfilingLevel(0)`

---

**Tip:** Always test performance queries on a secondary replica (if available) to avoid impacting production traffic.