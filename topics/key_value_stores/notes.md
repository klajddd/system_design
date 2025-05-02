# Key-Value Stores

## What are Key-Value Stores?

Key-value stores are NoSQL databases that store data as a collection of key-value pairs, where each key is unique and maps to a value. They are designed for high performance, scalability, and flexibility in data storage.

## Types of Key-Value Stores

### 1. In-Memory Stores

- Redis
- Memcached
- Hazelcast
- Apache Ignite

### 2. Disk-Based Stores

- RocksDB
- LevelDB
- Berkeley DB
- LMDB

### 3. Distributed Stores

- DynamoDB
- Riak
- Voldemort
- etcd

## Key Features

### 1. Data Model

```python
# Simple key-value pair
{
    "user:123": {
        "name": "John Doe",
        "email": "john@example.com"
    }
}

# Complex value
{
    "session:abc": {
        "user_id": 123,
        "created_at": "2024-03-20T10:00:00Z",
        "expires_at": "2024-03-20T11:00:00Z"
    }
}
```

### 2. Operations

- GET
- PUT
- DELETE
- SCAN
- BATCH operations

### 3. Data Types

- Strings
- Lists
- Sets
- Hashes
- Sorted Sets

## Implementation Examples

### 1. Redis

```python
import redis

# Connect to Redis
r = redis.Redis(host='localhost', port=6379, db=0)

# Basic operations
r.set('user:123', 'John Doe')
r.get('user:123')

# Complex operations
r.hset('user:123', mapping={
    'name': 'John Doe',
    'email': 'john@example.com'
})
r.hgetall('user:123')
```

### 2. DynamoDB

```python
import boto3

# Connect to DynamoDB
dynamodb = boto3.resource('dynamodb')
table = dynamodb.Table('Users')

# Basic operations
table.put_item(
    Item={
        'user_id': '123',
        'name': 'John Doe',
        'email': 'john@example.com'
    }
)

table.get_item(
    Key={
        'user_id': '123'
    }
)
```

## Use Cases

### 1. Caching

- Session storage
- Page caching
- API response caching
- Database query caching

### 2. Session Management

- User sessions
- Shopping carts
- Game state
- Real-time data

### 3. Real-time Analytics

- Counters
- Rate limiting
- Metrics collection
- Event tracking

## Scaling Strategies

### 1. Sharding

- Key-based sharding
- Range-based sharding
- Consistent hashing
- Dynamic sharding

### 2. Replication

- Master-Slave
- Multi-Master
- Active-Active
- Geographic replication

### 3. Partitioning

- Hash partitioning
- Range partitioning
- Composite partitioning
- Dynamic partitioning

## Performance Optimization

### 1. Memory Management

- Eviction policies
- Memory limits
- Cache warming
- Memory fragmentation

### 2. Batch Operations

```python
# Redis pipeline
with r.pipeline() as pipe:
    for i in range(100):
        pipe.set(f'key:{i}', f'value:{i}')
    pipe.execute()

# DynamoDB batch write
with table.batch_writer() as batch:
    for i in range(100):
        batch.put_item(
            Item={
                'id': str(i),
                'data': f'value:{i}'
            }
        )
```

### 3. Compression

- Data compression
- Key compression
- Value compression
- Network compression

## Best Practices

### 1. Key Design

- Meaningful prefixes
- Consistent patterns
- Size optimization
- Namespace separation

### 2. Value Design

- Serialization format
- Size limits
- Compression
- Schema evolution

### 3. Operations

- Batch processing
- Connection pooling
- Error handling
- Retry strategies

## Common Challenges

### 1. Performance

- Memory usage
- Network latency
- Disk I/O
- Connection limits

### 2. Consistency

- Eventual consistency
- Strong consistency
- Conflict resolution
- Data synchronization

### 3. Maintenance

- Backup strategies
- Monitoring
- Capacity planning
- Version upgrades

## Interview Questions

1. What are the advantages and disadvantages of key-value stores?
2. How do you handle data consistency in a distributed key-value store?
3. What strategies do you use for scaling key-value stores?
4. How do you choose between different key-value store implementations?
5. What are the common use cases for key-value stores?

## Further Reading

- [Redis Documentation](https://redis.io/documentation)
- [DynamoDB Best Practices](https://docs.aws.amazon.com/amazondynamodb/latest/developerguide/best-practices.html)
- [Designing Data-Intensive Applications](https://www.amazon.com/Designing-Data-Intensive-Applications-Reliable-Maintainable/dp/1449373321)
- [NoSQL Distilled](https://www.amazon.com/NoSQL-Distilled-Emerging-Polyglot-Persistence/dp/0321826620)
