# Relational Databases

## What are Relational Databases?

Relational databases store data in tables with rows and columns, where relationships between data are maintained through keys. They follow ACID properties and use SQL for data manipulation and querying.

## Key Concepts

### 1. ACID Properties

- Atomicity
- Consistency
- Isolation
- Durability

### 2. Database Normalization

- First Normal Form (1NF)
- Second Normal Form (2NF)
- Third Normal Form (3NF)
- Boyce-Codd Normal Form (BCNF)

### 3. Relationships

- One-to-One
- One-to-Many
- Many-to-Many
- Self-referencing

## Database Design

### 1. Schema Design

```sql
CREATE TABLE users (
    id INT PRIMARY KEY,
    username VARCHAR(50) UNIQUE,
    email VARCHAR(100) UNIQUE,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE TABLE orders (
    id INT PRIMARY KEY,
    user_id INT,
    order_date TIMESTAMP,
    FOREIGN KEY (user_id) REFERENCES users(id)
);
```

### 2. Indexing

- Primary Index
- Secondary Index
- Composite Index
- Partial Index

### 3. Constraints

- Primary Key
- Foreign Key
- Unique
- Check
- Not Null

## Query Optimization

### 1. Index Usage

```sql
-- Create index
CREATE INDEX idx_user_email ON users(email);

-- Use index in query
SELECT * FROM users WHERE email = 'user@example.com';
```

### 2. Query Planning

- Execution plan
- Cost estimation
- Index selection
- Join optimization

### 3. Performance Tuning

- Query rewriting
- Index optimization
- Statistics updates
- Configuration tuning

## Transactions

### 1. Transaction Management

```sql
BEGIN TRANSACTION;
    UPDATE accounts SET balance = balance - 100 WHERE id = 1;
    UPDATE accounts SET balance = balance + 100 WHERE id = 2;
COMMIT;
```

### 2. Isolation Levels

- Read Uncommitted
- Read Committed
- Repeatable Read
- Serializable

### 3. Concurrency Control

- Locking
- MVCC (Multi-Version Concurrency Control)
- Deadlock prevention
- Timeout handling

## Scaling Strategies

### 1. Vertical Scaling

- Hardware upgrades
- Resource allocation
- Configuration tuning
- Performance optimization

### 2. Horizontal Scaling

- Sharding
- Replication
- Read replicas
- Load balancing

### 3. Partitioning

- Range partitioning
- Hash partitioning
- List partitioning
- Composite partitioning

## High Availability

### 1. Replication

- Master-Slave
- Master-Master
- Multi-Master
- Circular replication

### 2. Failover

- Automatic failover
- Manual failover
- Failover testing
- Recovery procedures

### 3. Backup Strategies

- Full backup
- Incremental backup
- Point-in-time recovery
- Backup verification

## Best Practices

### 1. Design

- Proper normalization
- Index strategy
- Constraint usage
- Naming conventions

### 2. Performance

- Query optimization
- Index maintenance
- Statistics updates
- Configuration tuning

### 3. Security

- Access control
- Encryption
- Audit logging
- Regular updates

## Common Challenges

### 1. Performance

- Slow queries
- Index overhead
- Lock contention
- Resource exhaustion

### 2. Scalability

- Data growth
- Connection limits
- Storage limits
- Performance degradation

### 3. Maintenance

- Backup management
- Index maintenance
- Statistics updates
- Version upgrades

## Interview Questions

1. What are the ACID properties and why are they important?
2. How do you handle database scaling?
3. What strategies do you use for query optimization?
4. How do you ensure high availability?
5. What are the trade-offs between different isolation levels?

## Further Reading

- [Database Design](https://www.amazon.com/Database-Design-Mere-Mortals-Hands/dp/0321888383)
- [SQL Performance Explained](https://use-the-index-luke.com/)
- [High Performance MySQL](https://www.amazon.com/High-Performance-MySQL-Optimization-Replication/dp/1449314287)
- [Database Internals](https://www.amazon.com/Database-Internals-Deep-Distributed-Systems/dp/1492040347)
