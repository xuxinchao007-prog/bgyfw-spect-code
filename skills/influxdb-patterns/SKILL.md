---
name: influxdb-patterns
description: InfluxDB time-series database patterns for data organization, Flux queries, schema design, and performance. Based on official InfluxDB documentation.
---

# InfluxDB Patterns

Quick reference for InfluxDB 2.x+ best practices. For detailed guidance, refer to official InfluxDB documentation at https://docs.influxdata.com/

## When to Activate

- Designing time-series data schemas
- Writing Flux queries for analytics
- Optimizing query and storage performance
- Setting up data retention and downsampling
- Implementing security and authentication
- Planning cardinality management

## Quick Reference

### Data Organization

| Component | Description | Example |
|-----------|-------------|---------|
| **Bucket** | Container for time-series data | `sensors/autogen` |
| **Measurement** | Grouping of related data | `temperature`, `humidity` |
| **Tag** | Indexed metadata (strings) | `location="room1"`, `sensor="A01"` |
| **Field** | Actual data (typed values) | `value=25.5`, `status="ok"` |
| **Timestamp** | Time associated with data | `2024-01-15T10:00:00Z` |

**Key Principles:**
- Tags are indexed: Use for filtering and grouping
- Fields are not indexed: Use for actual measurements
- Low tag cardinality (<10,000 values per tag key)
- Fields are required; tags are optional

### Data Type Reference

| Use Case | InfluxDB Type | Notes |
|----------|---------------|-------|
| Measurements | Float | Default, most common |
| Counts/IDs | Integer | 64-bit signed |
| Status/Flags | String | Use in fields, not tags |
| Boolean values | Boolean | `true` or `false` |
| Timestamps | Timestamp | RFC3339 or epoch nanoseconds |

### Common Patterns

**Line Protocol (Data写入):**
```
# Syntax: measurement,tag_set field_set timestamp
temperature,location=room1,sensor=A01 value=25.5 1705315200000000000
humidity,location=room1,sensor=A02 value=65.2,status="normal" 1705315200000000000

# Batch writes (separate with \n)
temperature,location=room1 value=25.5 1705315200000000000
temperature,location=room2 value=24.8 1705315200000000000
```

**Flux Query Basics:**
```flux
// Basic query structure
from(bucket: "sensors/autogen")
  |> range(start: -1h)
  |> filter(fn: (r) => r._measurement == "temperature")
  |> filter(fn: (r) => r.location == "room1")
  |> aggregateWindow(every: 5m, fn: mean)

// Key operators:
// from(): Select bucket
// range(): Time range
// filter(): Filter data
// aggregateWindow(): Time-based aggregation
```

**Tag Cardinality Management:**
```flux
// Check tag cardinality
from(bucket: "sensors/autogen")
  |> range(start: -24h)
  |> group(columns: ["_measurement"])
  |> distinct(column: "location")
  |> count()

// High cardinality warning signs:
// - >10,000 unique values per tag key
// - High memory usage
// - Slow queries
```

**Downsampling / Retention:**
```flux
// Create task for downsampling
option task = {
  name: "downsample_raw_data",
  every: 1h,
  offset: 10m
}

from(bucket: "raw/autogen")
  |> range(start: -1h)
  |> aggregateWindow(every: 5m, fn: mean, createEmpty: false)
  |> to(bucket: "downsampled_5m", org: "my-org")

// Set retention policy via UI or API
// raw: 7 days retention
// downsampled_5m: 1 year retention
```

**Aggregate Functions:**
```flux
// Common aggregations
from(bucket: "metrics/autogen")
  |> range(start: -1h)
  |> filter(fn: (r) => r._measurement == "cpu")
  |> aggregateWindow(every: 5m, fn: mean, createEmpty: false)    // Average
  // |> aggregateWindow(every: 5m, fn: max)                       // Maximum
  // |> aggregateWindow(every: 5m, fn: min)                       // Minimum
  // |> aggregateWindow(every: 5m, fn: sum)                       // Sum
  // |> aggregateWindow(every: 5m, fn: count)                     // Count
  // |> aggregateWindow(every: 5m, fn: median)                    // Median

// Yield results
  |> yield(name: "aggregated")
```

**Time-based Joins:**
```flux
// Join two measurements
temp = from(bucket: "sensors/autogen")
  |> range(start: -1h)
  |> filter(fn: (r) => r._measurement == "temperature")
  |> aggregateWindow(every: 1m, fn: mean)

humidity = from(bucket: "sensors/autogen")
  |> range(start: -1h)
  |> filter(fn: (r) => r._measurement == "humidity")
  |> aggregateWindow(every: 1m, fn: mean)

join(tables: {temp: temp, humidity: humidity}, on: ["_time", "_start", "_stop"])
  |> yield(name: "combined")
```

**Pivot for Wide Format:**
```flux
// Transform to wide format
from(bucket: "metrics/autogen")
  |> range(start: -1h)
  |> filter(fn: (r) => r._measurement == "system")
  |> pivot(rowKey:["_time"], columnKey: ["_field"], valueColumn: "_value")
  |> yield(name: "wide")
```

### Performance Monitoring

**Query Profiling:**
```flux
// Check query execution time
// (Available in InfluxDB UI or via API)

// Optimize queries:
// 1. Use time bounds in range()
// 2. Filter on tags before fields
// 3. Use aggregateWindow for large datasets
// 4. Limit results with limit() or last()
```

**Storage Statistics:**
```flux
// Check bucket sizes
// (Use InfluxDB UI: Data > Buckets)

// Via API:
influx bucket list --org my-org
influx bucket --name sensors --org my-org
```

**Query Optimization:**
```flux
// Good: Filter on tags (indexed)
from(bucket: "sensors/autogen")
  |> range(start: -1h)
  |> filter(fn: (r) => r._measurement == "temperature")
  |> filter(fn: (r) => r.location == "room1")  // Tag filter

// Bad: Filter on fields (not indexed)
from(bucket: "sensors/autogen")
  |> range(start: -1h)
  |> filter(fn: (r) => r._value > 25.0)  // Field filter (slower)
```

### Anti-Pattern Detection

**High Cardinality Tags:**
```flux
// Find high cardinality tags
from(bucket: "sensors/autogen")
  |> range(start: -24h)
  |> group(columns: ["_measurement"])
  |> group(columns: ["_field"])
  |> keys()
  |> keep(columns: ["_value"])

// Manual check:
// - Does tag have user IDs or unique identifiers?
// - Does tag have timestamps or dates?
// - Does tag have unbounded growth?

// Solution: Move to field, use different schema
```

**Too Many Fields:**
```flux
// Check field count
from(bucket: "sensors/autogen")
  |> range(start: -1h)
  |> keep(columns: ["_field"])
  |> distinct(column: "_field")

// Issue: >100 fields in single measurement
// Solution: Split into multiple measurements
```

**Missing Indexes:**
```
// Tags are automatically indexed
// Ensure frequently filtered columns are tags, not fields

// Good:
temperature,location=room1,sensor=A01 value=25.5

// Bad for filtering:
temperature value=25.5,location="room1"  // location should be tag
```

**Large Time Ranges:**
```flux
// Bad: Querying all data
from(bucket: "sensors/autogen")
  |> range(start: 0)  // All data!

// Good: Limit time range
from(bucket: "sensors/autogen")
  |> range(start: -24h)  // Last 24 hours
```

### Configuration Template

```toml
# influxdb.conf (InfluxDB 2.x)

# Data settings
[data]
  dir = "/var/lib/influxdb"
  engine-path = "/var/lib/influxdb/engine"
  query-timeout = "0s"
  max-series-per-database = 10000000
  max-values-per-tag = 100000

# Coordinator
[coordinator]
  write-timeout = "10s"
  max-concurrent-queries = 0

# Retention
[retention]
  check-interval = "30m"

# Logging
[logging]
  format = "auto"
  level = "info"

# Monitor
[monitor]
  store-enabled = true
  store-database = "_internal"
```

### Security Best Practices

**Authentication:**
```bash
# Create user via CLI
influx user create --name admin --org my-org --password pass123

# Create authentication token
influx auth create --org my-org --write-bucket sensors/autogen

# List tokens
influx auth list
```

**Authorization:**
```bash
# Token-based authentication
# 1. All-Access Token
# 2. Read-Write Token (bucket-specific)
# 3. Read-Only Token

# Create read-only token
influx auth create \
  --org my-org \
  --read-bucket sensors/autogen \
  --description "Read-only sensor data"
```

**Network Security:**
```toml
# influxdb.conf

[http]
  auth-enabled = true
  https-enabled = true
  https-certificate = "/path/to/cert.pem"
  https-private-key = "/path/to/key.pem"

# Bind to specific interface
bind-address = "127.0.0.1:8086"
```

**Data Security:**
```bash
# Enable encryption at rest
# (Configure in your storage backend)

# Backup and restore
influx backup /path/to/backup --bucket sensors/autogen
influx restore /path/to/backup --bucket sensors/autogen

# Set retention policies
influx bucket update --name sensors --retention 7d
```

## Related

- Official InfluxDB Documentation: https://docs.influxdata.com/
- InfluxDB University: https://influxdata.com/university/
- Flux Documentation: https://docs.influxdata.com/flux/latest/
- Skill: `clickhouse-io` - ClickHouse analytics patterns
- Skill: `postgres-patterns` - PostgreSQL patterns

---

*Based on [InfluxDB Documentation](https://docs.influxdata.com/)*
