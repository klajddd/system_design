# Load Balancers

## What are Load Balancers?

Load balancers distribute incoming network traffic across multiple servers to ensure no single server becomes overwhelmed. They improve application availability, scalability, and reliability by distributing the workload efficiently.

## Types of Load Balancers

### 1. Application Load Balancer (Layer 7)

- Content-based routing
- HTTP/HTTPS traffic
- Advanced routing rules
- SSL termination

### 2. Network Load Balancer (Layer 4)

- TCP/UDP traffic
- High performance
- Static IP addresses
- Connection-based load balancing

### 3. Classic Load Balancer

- EC2-Classic instances
- Basic load balancing
- TCP/SSL/TLS protocols
- Health checks

## Load Balancing Algorithms

### 1. Round Robin

- Distributes requests sequentially
- Simple implementation
- Equal distribution
- No server awareness

### 2. Least Connections

- Tracks active connections
- Sends to least busy server
- Good for long-lived connections
- Dynamic load distribution

### 3. IP Hash

- Client IP-based routing
- Session persistence
- Predictable routing
- Cache-friendly

### 4. Weighted Round Robin

- Server capacity-based distribution
- Custom weights per server
- Resource-aware
- Manual configuration

## Health Checks

### 1. Types

- HTTP health checks
- TCP health checks
- Custom health checks
- SSL certificate checks

### 2. Configuration

```yaml
health_check:
  protocol: HTTP
  port: 80
  path: /health
  interval: 30s
  timeout: 5s
  healthy_threshold: 2
  unhealthy_threshold: 3
```

## Session Persistence

### 1. Methods

- Cookie-based
- IP-based
- Application-based
- Custom headers

### 2. Implementation

```nginx
upstream backend {
    ip_hash;  # Session persistence
    server backend1.example.com;
    server backend2.example.com;
}
```

## High Availability

### 1. Active-Active

```
[Client] --> [Load Balancer 1] --> [Server 1]
                    |                    |
                    +--> [Server 2]      |
                    |                    |
[Client] --> [Load Balancer 2] --> [Server 3]
```

### 2. Active-Passive

```
[Client] --> [Active LB] --> [Server 1]
                    |            |
                    +--> [Server 2]  [Passive LB]
```

## Best Practices

### 1. Configuration

- Proper timeout settings
- Connection limits
- SSL/TLS configuration
- Health check parameters

### 2. Security

- DDoS protection
- SSL/TLS termination
- Access control
- Security groups

### 3. Monitoring

- Performance metrics
- Error rates
- Health check status
- Resource usage

## Common Challenges

### 1. Session Management

- Session persistence
- State synchronization
- Cache consistency
- Cookie handling

### 2. SSL/TLS

- Certificate management
- SSL termination
- Performance impact
- Security configuration

### 3. Scaling

- Dynamic scaling
- Capacity planning
- Cost optimization
- Maintenance

## Implementation Examples

### 1. Nginx Configuration

```nginx
upstream backend {
    server backend1.example.com weight=5;
    server backend2.example.com weight=5;
    server backend3.example.com backup;
}

server {
    listen 80;
    location / {
        proxy_pass http://backend;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
    }
}
```

### 2. HAProxy Configuration

```haproxy
frontend http-in
    bind *:80
    default_backend servers

backend servers
    balance roundrobin
    server server1 10.0.0.1:80 check
    server server2 10.0.0.2:80 check
    server server3 10.0.0.3:80 check backup
```

## Interview Questions

1. What are the different types of load balancers and when would you use each?
2. How do you handle session persistence in a load-balanced environment?
3. What strategies do you use for health checking?
4. How do you ensure high availability of the load balancer itself?
5. What are the trade-offs between different load balancing algorithms?

## Further Reading

- [Nginx Load Balancing](https://nginx.org/en/docs/http/load_balancing.html)
- [HAProxy Documentation](https://www.haproxy.org/#docs)
- [AWS Elastic Load Balancing](https://aws.amazon.com/elasticloadbalancing/)
- [Load Balancing in Distributed Systems](https://www.nginx.com/resources/glossary/load-balancing/)
