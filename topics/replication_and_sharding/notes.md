# Replication and Sharding

## What are Replication and Sharding?

Replication and sharding are two fundamental techniques for scaling databases and distributed systems. Replication involves creating copies of data across multiple nodes, while sharding involves partitioning data across multiple nodes.

## Replication

### 1. Types of Replication

- Master-Slave (Primary-Secondary)
- Master-Master (Multi-Master)
- Active-Active
- Circular Replication

### 2. Replication Methods

- Synchronous Replication
- Asynchronous Replication
- Semi-Synchronous Replication
- Statement-Based Replication

### 3. Replication Topologies

```
[Master] --> [Slave 1]
    |
    +--> [Slave 2]
    |
    +--> [Slave 3]
```

## Sharding

### 1. Sharding Strategies

- Range-Based Sharding
- Hash-Based Sharding
- Directory-Based Sharding
- Composite Sharding

### 2. Sharding Patterns

```
[Client] --> [Shard 1] (0-1000)
    |
    +--> [Shard 2] (1001-2000)
    |
    +--> [Shard 3] (2001-3000)
```

### 3. Shard Key Selection

- Cardinality
- Distribution
- Query patterns
- Growth patterns

## Implementation Examples

### 1. MongoDB Replication

```javascript
// Configure replica set
rs.initiate({
  _id: "rs0",
  members: [
    { _id: 0, host: "server1:27017" },
    { _id: 1, host: "server2:27017" },
    { _id: 2, host: "server3:27017" },
  ],
});
```

### 2. MySQL Sharding

```sql
-- Create sharded tables
CREATE TABLE users_0 (
    id INT PRIMARY KEY,
    name VARCHAR(100)
) PARTITION BY RANGE (id) (
    PARTITION p0 VALUES LESS THAN (1000),
    PARTITION p1 VALUES LESS THAN (2000),
    PARTITION p2 VALUES LESS THAN (3000)
);
```

## Consistency Models

### 1. Strong Consistency

- Synchronous replication
- ACID properties
- Immediate consistency
- Higher latency

### 2. Eventual Consistency

- Asynchronous replication
- BASE properties
- Delayed consistency
- Lower latency

### 3. Causal Consistency

- Partial ordering
- Dependency tracking
- Conflict resolution
- Hybrid approach

## Scaling Strategies

### 1. Horizontal Scaling

- Add more shards
- Distribute load
- Increase capacity
- Handle growth

### 2. Vertical Scaling

- Upgrade hardware
- Increase resources
- Optimize performance
- Handle bottlenecks

### 3. Hybrid Scaling

- Combination approach
- Resource optimization
- Cost efficiency
- Performance balance

## Best Practices

### 1. Design

- Choose appropriate strategy
- Plan for growth
- Consider consistency
- Handle failures

### 2. Implementation

- Monitor performance
- Handle conflicts
- Manage resources
- Backup data

### 3. Operations

- Regular maintenance
- Health checks
- Recovery procedures
- Monitoring

## Common Challenges

### 1. Data Consistency

- Replication lag
- Conflict resolution
- Data synchronization
- Consistency levels

### 2. Performance

- Query routing
- Load balancing
- Resource usage
- Network latency

### 3. Operations

- Shard management
- Replica management
- Backup strategies
- Recovery procedures

## Interview Questions

1. What are the trade-offs between different replication strategies?
2. How do you choose a sharding strategy for your data?
3. How do you handle data consistency in a distributed system?
4. What strategies do you use for scaling databases?
5. How do you handle failures in a replicated or sharded system?

## Further Reading

- [Designing Data-Intensive Applications](https://www.amazon.com/Designing-Data-Intensive-Applications-Reliable-Maintainable/dp/1449373321)
- [Database Sharding](https://www.mongodb.com/blog/post/database-sharding-explained)
- [Replication in Distributed Systems](https://www.amazon.com/Distributed-Systems-Principles-Paradigms-2nd/dp/0132392275)
- [Scaling Databases](https://www.amazon.com/Scalability-Rules-Principles-Scaling-Websites/dp/0321753887)
