# Detailed Notes on Caching

## What is Caching?

Caching is a technique used to store frequently accessed data in a temporary storage area (cache) to improve data retrieval performance. Instead of repeatedly accessing the original data source, which might be slower, the system can quickly retrieve data from the cache.

## Types of Caching

### 1. In-Memory Caching

- Stores data in RAM for fastest access
- Examples: Redis, Memcached
- Best for: Frequently accessed data, session storage
- Limitations: Memory constraints, data persistence

### 2. Distributed Caching

- Spreads cache across multiple servers
- Examples: Redis Cluster, Hazelcast
- Best for: Large-scale applications, high availability
- Benefits: Scalability, fault tolerance

### 3. CDN Caching

- Caches static content at edge locations
- Examples: Cloudflare, Akamai
- Best for: Static assets, media files
- Benefits: Reduced latency, bandwidth savings

### 4. Browser Caching

- Stores resources locally in the browser
- Types: Memory cache, disk cache
- Best for: Static resources, images, CSS, JavaScript
- Control: HTTP cache headers

## Caching Strategies

### Cache-Aside (Lazy Loading)

1. Application checks cache for data
2. If cache miss, fetches from database
3. Updates cache with new data
4. Returns data to application

#### Implementation Example

```python
def get_data(key):
    # Try to get from cache
    data = cache.get(key)
    if data is None:
        # Cache miss - get from database
        data = database.get(key)
        # Update cache
        cache.set(key, data)
    return data
```

#### Diagram

```
[Client] --> [Check Cache] --> [Cache Hit] --> [Return Data]
                |
                v
            [Cache Miss] --> [Database] --> [Update Cache] --> [Return Data]
```

### Write-Through

1. Write to cache and database simultaneously
2. Ensures cache consistency
3. Higher write latency
4. Good for write-heavy applications

#### Implementation Example

```python
def write_data(key, value):
    # Write to database
    database.set(key, value)
    # Write to cache
    cache.set(key, value)
```

#### Diagram

```
[Client] --> [Write Request] --> [Database] --> [Cache] --> [Success Response]
```

### Write-Back (Write-Behind)

1. Write to cache first
2. Asynchronously update database
3. Better write performance
4. Risk of data loss on cache failure

#### Implementation Example

```python
def write_data(key, value):
    # Write to cache immediately
    cache.set(key, value)
    # Queue for database update
    async def update_db():
        await database.set(key, value)
    asyncio.create_task(update_db())
```

#### Diagram

```
[Client] --> [Write to Cache] --> [Success Response]
                |
                v
            [Async Queue] --> [Database Update]
```

## Cache Invalidation

### Time-Based Expiration

- TTL (Time To Live) for cache entries
- Simple to implement
- May serve stale data

#### Implementation Example

```python
def set_with_ttl(key, value, ttl_seconds):
    cache.set(key, value, ex=ttl_seconds)
```

### Event-Based Invalidation

- Invalidate on data changes
- More complex to implement
- Better consistency

#### Implementation Example

```python
def update_data(key, value):
    # Update database
    database.set(key, value)
    # Invalidate cache
    cache.delete(key)
```

### Version-Based Invalidation

- Use version numbers for cache keys
- Update version on data changes
- Good for static content

#### Implementation Example

```python
def get_versioned_data(key, version):
    cache_key = f"{key}:v{version}"
    return cache.get(cache_key)
```

## Common Challenges

### Cache Stampede

- Multiple requests miss cache simultaneously
- All try to update cache
- Solution: Use locks or request coalescing

#### Implementation Example

```python
def get_data_with_lock(key):
    # Try to get from cache
    data = cache.get(key)
    if data is None:
        # Acquire lock
        with cache.lock(f"lock:{key}", timeout=10):
            # Double-check cache
            data = cache.get(key)
            if data is None:
                data = database.get(key)
                cache.set(key, data)
    return data
```

### Cache Penetration

- Requests for non-existent data
- Wastes cache space
- Solution: Cache null values

#### Implementation Example

```python
def get_data_with_null_cache(key):
    data = cache.get(key)
    if data is None:
        data = database.get(key)
        # Cache null values too
        cache.set(key, data if data is not None else "NULL")
    return None if data == "NULL" else data
```

### Cache Avalanche

- Many cache entries expire simultaneously
- Sudden database load
- Solution: Randomize expiration times

#### Implementation Example

```python
def set_with_random_ttl(key, value, base_ttl):
    # Add random jitter to TTL
    jitter = random.randint(-300, 300)  # ±5 minutes
    cache.set(key, value, ex=base_ttl + jitter)
```

## Best Practices

1. Choose the right caching strategy
2. Implement proper cache invalidation
3. Monitor cache performance
4. Use appropriate cache size
5. Consider cache consistency requirements
6. Implement fallback mechanisms
7. Use cache warming for critical data

## Interview Questions

1. How would you design a caching system for a high-traffic website?
2. What caching strategy would you use for a read-heavy vs. write-heavy application?
3. How do you handle cache invalidation in a distributed system?
4. What are the trade-offs between different caching strategies?
5. How would you prevent cache stampede?

## Further Reading

- [Redis Documentation](https://redis.io/documentation)
- [Memcached Documentation](https://memcached.org/documentation)
- [System Design Interview](https://www.amazon.com/System-Design-Interview-Insiders-Guide/dp/1736049119)
