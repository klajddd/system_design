# Rate Limiting in Distributed Systems

---

Rate limiting is a critical technique for maintaining the stability and fairness of distributed systems, controlling the frequency of requests a client can make to prevent abuse and ensure resource availability. By throttling excessive traffic, rate limiting protects services from overload and enhances user experience. This chapter, designed for advanced computer science students, explores rate limiting mechanisms, algorithms, practical scenarios, and industry-standard tools.

## What is Rate Limiting?

---

Rate limiting involves restricting the number of requests a client, such as a user or application, can make to a system within a specific time frame. This mechanism safeguards APIs, web servers, and databases from being overwhelmed by excessive or malicious traffic, ensuring fair resource allocation and system reliability. Rate limiting can be applied at various levels, such as per user, per IP, or globally across a service. In practice, tools like **NGINX**, **Redis**, and **AWS API Gateway** implement rate limiting to manage traffic effectively in distributed environments.

**Diagram: Rate Limiting Architecture**  
![Rate Limiting Architecture](rate_limiting_architecture.png)  
*Description*: A client box sends requests to a rate limiter box, which checks a counter or token store (e.g., Redis). If allowed, the request proceeds to an application server; if blocked, a rejection response is returned. Arrows indicate request flow, with a dotted line for rejected requests. The diagram uses a clean, flowchart style with labeled components.

## Rate Limiting Algorithms

---

Rate limiting relies on algorithms to track and enforce request quotas. This section examines three common algorithms, each illustrated with Python pseudocode to clarify their implementation.

### Fixed Window

The fixed window algorithm counts requests within a fixed time window (e.g., 60 seconds). If the count exceeds the limit, additional requests are blocked until the window resets. While simple, it can allow bursty traffic at window boundaries.

```python
# === Fixed Window Rate Limiter ===
from time import time

class FixedWindowLimiter:
    def __init__(self, limit, window_seconds):
        self.limit = limit  # Max requests per window
        self.window_seconds = window_seconds  # Window duration
        self.counts = {}  # Track client counts
        self.windows = {}  # Track window start times

    def allow_request(self, client_id):
        current_time = int(time())
        window_start = current_time - (current_time % self.window_seconds)
        
        # Reset count if new window
        if self.windows.get(client_id, 0) != window_start:
            self.windows[client_id] = window_start
            self.counts[client_id] = 0
        
        # Check and update count
        if self.counts[client_id] < self.limit:
            self.counts[client_id] += 1
            return True
        return False

# Simulate requests (limit: 3 requests per 60 seconds)
limiter = FixedWindowLimiter(3, 60)
for _ in range(5):
    print(f"Request allowed: {limiter.allow_request('client1')}")
# Output: True, True, True, False, False
# === End Fixed Window ===
```

This approach is used in **NGINX** with its `limit_req` directive for straightforward rate control.

### Sliding Window

The sliding window algorithm tracks requests over a continuous time window, using timestamps to avoid boundary issues. It offers smoother throttling but requires more memory to store request timestamps.

```python
# === Sliding Window Rate Limiter ===
from time import time
from collections import deque

class SlidingWindowLimiter:
    def __init__(self, limit, window_seconds):
        self.limit = limit  # Max requests per window
        self.window_seconds = window_seconds  # Window duration
        self.requests = {}  # Track client request timestamps

    def allow_request(self, client_id):
        current_time = time()
        self.requests.setdefault(client_id, deque())
        
        # Remove outdated timestamps
        while self.requests[client_id] and self.requests[client_id][0] <= current_time - self.window_seconds:
            self.requests[client_id].popleft()
        
        # Check and add request
        if len(self.requests[client_id]) < self.limit:
            self.requests[client_id].append(current_time)
            return True
        return False

# Simulate requests (limit: 3 requests per 60 seconds)
limiter = SlidingWindowLimiter(3, 60)
for _ in range(5):
    print(f"Request allowed: {limiter.allow_request(client1)}")
# Output: True, True, True, False, False
# === End Sliding Window ===
```

**Redis** is often used to store timestamps for sliding window implementations in distributed systems.

### Token Bucket

The token bucket algorithm maintains a bucket of tokens, refilled at a constant rate. Each request consumes a token, and requests are blocked if no tokens are available. This approach handles bursts effectively.

```python
# === Token Bucket Rate Limiter ===
from time import time

class TokenBucketLimiter:
    def __init__(self, capacity, rate):
        self.capacity = capacity  # Max tokens in bucket
        self.rate = rate  # Tokens added per second
        self.tokens = {}  # Track tokens per client
        self.last_refill = {}  # Track last refill time

    def allow_request(self, client_id):
        current_time = time()
        self.tokens.setdefault(client_id, self.capacity)
        self.last_refill.setdefault(client_id, current_time)
        
        # Refill tokens
        elapsed = current_time - self.last_refill[client_id]
        self.tokens[client_id] = min(self.capacity, self.tokens[client_id] + elapsed * self.rate)
        self.last_refill[client_id] = current_time
        
        # Check and consume token
        if self.tokens[client_id] >= 1:
            self.tokens[client_id] -= 1
            return True
        return False

# Simulate requests (capacity: 3 tokens, rate: 1 token/second)
limiter = TokenBucketLimiter(3, 1)
for _ in range(5):
    print(f"Request allowed: {limiter.allow_request('client1')}")
# Output: True, True, True, False, False
# === End Token Bucket ===
```

**AWS API Gateway** uses token bucket for API rate limiting, balancing burst and sustained traffic.

**Diagram: Rate Limiting Algorithms**  
![Rate Limiting Algorithms](rate_limiting_algorithms.png)  
*Description*: Three rows, each showing a client sending requests to a rate limiter. For fixed window, a timeline shows a fixed 60-second window with 3 allowed requests. For sliding window, a timeline tracks requests within a moving 60-second window. For token bucket, a bucket graphic shows tokens consumed and refilled. Arrows indicate allowed/blocked requests.

## Key Scenarios

---

Rate limiting shines in several critical scenarios. First, it prevents API abuse by restricting malicious or overly aggressive clients, such as bots attempting to scrape data. For instance, **AWS API Gateway** enforces per-client limits to protect backend services. Second, rate limiting ensures fair resource allocation in multi-tenant systems, preventing any single user from monopolizing resources. **Redis** is often used to maintain shared counters across distributed nodes. Third, handling burst traffic is crucial for systems with sporadic spikes, where the token bucket algorithm excels by allowing controlled bursts. **NGINX** can throttle bursts using its rate limiting module, ensuring system stability.

## Real-World Tools

---

Several tools power rate limiting in production systems. **NGINX** uses its `limit_req` module to implement fixed window rate limiting, ideal for web servers. **Redis**, with its fast in-memory storage, supports sliding window and token bucket implementations by storing counters or timestamps. **AWS API Gateway** provides built-in rate limiting with token bucket algorithms, integrating seamlessly with cloud-based APIs.

## Conclusion

---

Rate limiting is essential for protecting distributed systems, using algorithms like fixed window, sliding window, and token bucket to control request rates. These mechanisms prevent abuse, ensure fairness, and handle bursts, with tools like **NGINX**, **Redis**, and **AWS API Gateway** driving real-world implementations. Students should experiment with these algorithms and tools to master rate limiting in practice.

*Word Count*: ~460 words (excluding code and image descriptions).