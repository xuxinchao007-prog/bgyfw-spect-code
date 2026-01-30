---
name: redis-patterns
description: Redis data structure patterns for caching, rate limiting, pub/sub, and performance optimization. Based on official Redis documentation.
---

# Redis Patterns

Quick reference for Redis best practices. For detailed guidance, refer to official Redis documentation at https://redis.io/docs/

## When to Activate

- Implementing caching strategies
- Building rate limiters
- Using pub/sub messaging
- Optimizing memory usage
- Designing data structures
- Implementing sessions and locks

## Quick Reference

### Data Structure Selection

| Use Case | Data Type | Command | Example |
|----------|-----------|---------|---------|
| Simple key-value | String | `SET`, `GET` | `SET user:123 "John"` |
| Object fields | Hash | `HSET`, `HGET` | `HSET user:123 name "John"` |
| Ordered list | List | `LPUSH`, `LRANGE` | `LPUSH queue:tasks "task1"` |
| Unique members | Set | `SADD`, `SMEMBERS` | `SADD tags:article1 "redis"` |
| Scored items | Sorted Set | `ZADD`, `ZRANGE` | `ZADD leaderboard 1000 "player1"` |
| Streams | Stream | `XADD`, `XRANGE` | `XADD events * sensor 25` |
| Geospatial | Geospatial | `GEOADD`, `GEORADIUS` | `GEOADD locations -122.4 37.8 "SF"` |

### Common Patterns

**Caching:**
```redis
# Simple cache with TTL
SET cache:user:123 "serialized_data" EX 3600  # 1 hour
GET cache:user:123

# Cache-aside pattern
# 1. Check cache
GET cache:user:123
# 2. If miss, fetch from DB
# 3. Store in cache with TTL
SET cache:user:123 "data" EX 3600

# Write-through: Update cache on DB write
SET cache:user:123 "new_data" EX 3600
```

**Rate Limiting:**
```redis
# Fixed window rate limiter
# INCR key, if > limit then reject
INCR rate:limit:user:123:2024011510
EXPIRE rate:limit:user:123:2024011510 60
# Check if > 100 (limit)

# Token bucket (Lua script)
local key = KEYS[1]
local limit = tonumber(ARGV[1])
local rate = tonumber(ARGV[2])
local current = redis.call('GET', key) or 0
if current + 1 <= limit then
  redis.call('INCR', key)
  redis.call('EXPIRE', key, rate)
  return 1
else
  return 0
end
```

**Distributed Lock:**
```redis
# Acquire lock with NX (only if not exists)
SET lock:resource:123 "unique_token" NX PX 30000

# Release lock (Lua script for atomicity)
if redis.call("GET", KEYS[1]) == ARGV[1] then
  return redis.call("DEL", KEYS[1])
else
  return 0
end

# Redlock algorithm (more robust)
# Use multiple independent Redis instances
```

**Leader Election:**
```redis
# Compete for leadership
SET election:leader "node1" NX PX 10000

# Check if leader
GET election:leader

# Heartbeat to maintain leadership
SET election:leader "node1" PX 10000
```

**Pub/Sub Messaging:**
```redis
# Subscribe to channel
SUBSCRIBE channel:events

# Publish to channel
PUBLISH channel:events "message_data"

# Pattern matching
PSUBSCRIBE channel:events:*
```

**Counter:**
```redis
# Simple counter
INCR counter:page:views
GET counter:page:views

# Atomic increment by
INCRBY counter:likes 5
DECR counter:available_slots
```

**Session Storage:**
```redis
# Store session data with TTL
HSET session:abc123 user_id 123 login_at 1705315200
EXPIRE session:abc123 3600

# Retrieve session
HGETALL session:abc123

# Update session TTL on activity
EXPIRE session:abc123 3600
```

**Time Series (Sorted Set):**
```redis
# Add events with timestamp as score
ZADD events:temperature 1705315200 "sensor1:25.5"
ZADD events:temperature 1705315260 "sensor1:25.8"

# Get last 10 events
ZREVRANGE events:temperature 0 9 WITHSCORES

# Get events in time range
ZRANGEBYSCORE events:temperature 1705315200 1705315800

# Remove old events
ZREMRANGEBYSCORE events:temperature 0 1705315200
```

### Memory Optimization

**Memory-Efficient Data Types:**
```redis
# Use hashes for objects instead of strings
# Good:
HSET user:123 name "John" email "john@example.com"

# Avoid (more memory):
SET user:123:name "John"
SET user:123:email "john@example.com"

# Use compressed lists/sets for small collections
# Redis automatically uses ziplist/ziplist for small collections
```

**Key Expiration:**
```redis
# Set TTL on keys
SET key "value" EX 3600    # Seconds
SET key "value" PX 3600000 # Milliseconds

# Update TTL
EXPIRE key 3600
PEXPIRE key 3600000

# Check TTL
TTL key     # Time to live in seconds
PTTL key    # Time to live in milliseconds
```

**Key Naming:**
```redis
# Good: Hierarchical, descriptive
user:123:profile
session:abc123:data
cache:api:v1:user:456

# Bad: Flat, no context
user123
abc123data
```

**Eviction Policy:**
```conf
# redis.conf

# allkeys-lfu: Evict least frequently used keys
# allkeys-lru: Evict least recently used keys
# volatile-lru: Evict LRU among keys with TTL set
# allkeys-random: Evict random keys
maxmemory 2gb
maxmemory-policy allkeys-lru
```

### Performance Monitoring

**Info Commands:**
```redis
# General info
INFO

# Memory stats
INFO memory

# CPU stats
INFO stats

# Replication
INFO replication

# Persistence
INFO persistence
```

**Slow Log:**
```redis
# Configure slow log
CONFIG SET slowlog-log-slower-than 10000  # microseconds
CONFIG SET slowlog-max-len 128

# View slow queries
SLOWLOG GET 10

# Get slow log length
SLOWLOG LEN
```

**Client Stats:**
```redis
# List connected clients
CLIENT LIST

# Kill client
CLIENT KILL id 123

# Get client info
CLIENT GETNAME
CLIENT SETNAME "my-app"
```

**Latency Monitor:**
```redis
# Enable latency monitor
CONFIG SET latency-monitor-threshold 100

# Check latency spikes
LATENCY LATEST

# Get latency history
LATENCY HISTORY command
LATENCY DOCTOR
```

### Anti-Pattern Detection

**Large Keys:**
```redis
# Find large keys (use redis-cli)
redis-cli --bigkeys

# Or use MEMORY command
MEMORY USAGE key

# Issue: >1MB strings, >10k elements in lists/sets
# Solution: Split into smaller keys
```

**Blocking Operations:**
```redis
# Avoid blocking commands on production
KEYS *           # Use SCAN instead
FLUSHALL         # Don't use in production
DEL huge_key     # Use UNLINK instead (async)

# Use non-blocking alternatives
SCAN 0 MATCH user:* COUNT 100
UNLINK key1 key2  # Async delete
```

**O(N) Operations:**
```redis
# Avoid large operations
SMEMBERS huge_set     # Use SSCAN instead
LRANGE list 0 -1      # Use LSCAN instead
HGETALL huge_hash     # Use HSCAN instead

# Use cursor-based scanning
SSCAN huge_set 0 COUNT 100
```

**Memory Leaks:**
```redis
# Check for keys without TTL
redis-cli --scan --pattern "cache:*" | xargs redis-cli PTTL
# -1 means no expiration (potential leak)

# Set TTL on all cache keys
for key in $(redis-cli --scan --pattern "cache:*"); do
  redis-cli EXPIRE $key 3600
done
```

**N+1 Queries:**
```redis
# Bad: Multiple round trips
GET user:123:name
GET user:123:email
GET user:123:phone

# Good: Single round trip (pipeline or hash)
HGETALL user:123
# Or use MGET for multiple keys
MGET user:123:name user:123:email user:123:phone
```

### Configuration Template

```conf
# redis.conf

# Network
bind 127.0.0.1
protected-mode yes
port 6379
tcp-backlog 511
timeout 0

# Memory
maxmemory 2gb
maxmemory-policy allkeys-lru
maxmemory-samples 5

# Persistence
save 900 1
save 300 10
save 60 10000
stop-writes-on-bgsave-error yes
rdbcompression yes
rdbchecksum yes

# Replication
replica-serve-stale-data yes
replica-read-only yes
repl-diskless-sync no

# Security
# requirepass yourpassword
# rename-command FLUSHALL ""
# rename-command CONFIG "CONFIG_b835c3f8"

# Slow log
slowlog-log-slower-than 10000
slowlog-max-len 128

# Client
maxclients 10000

# Advanced
hash-max-ziplist-entries 512
hash-max-ziplist-value 64
list-max-ziplist-size -2
set-max-intset-entries 512
zset-max-ziplist-entries 128
zset-max-ziplist-value 64
```

### Security Best Practices

**Authentication:**
```conf
# redis.conf
requirepass your_strong_password
```

```redis
# Authenticate
AUTH your_strong_password

# ACL (Redis 6.0+)
ACL SETUSER john on >password ~* +@all
ACL SETUSER readonly on >password ~* +@read
ACL SAVE
```

**Network Security:**
```conf
# Bind to specific interface
bind 127.0.0.1

# Disable dangerous commands
rename-command FLUSHDB ""
rename-command FLUSHALL ""
rename-command CONFIG "CONFIG_b835c3f8"
rename-command DEBUG ""

# Use TLS (Redis 6.0+)
tls-port 6379
tls-cert-file /path/to/redis.crt
tls-key-file /path/to/redis.key
tls-ca-cert-file /path/to/ca.crt
```

**Access Control (ACL):**
```redis
# Create user with specific permissions
ACL SETUSER app_user on >app_password ~app:* +@all -@dangerous

# Create read-only user
ACL SETUSER reader on >reader_pass ~* +@read

# List users
ACL USERS

# Get user info
ACL GETUSER app_user
```

**Data Security:**
```redis
# Enable encryption at rest (via file system)
# Use encrypted volumes or filesystem encryption

# Secure backups
# Encrypt RDB files when copying
# Use secure backup protocols

# Disable sensitive info in INFO
# ACL can restrict CONFIG command access
```

## Related

- Official Redis Documentation: https://redis.io/docs/
- Redis University: https://university.redis.com/
- Redis Commands: https://redis.io/commands/
- Skill: `backend-patterns` - API and backend patterns
- Skill: `postgres-patterns` - PostgreSQL patterns

---

*Based on [Redis Documentation](https://redis.io/docs/)*
