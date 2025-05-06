# Advanced Logging and Monitoring in Distributed Systems

## Introduction

Logging and monitoring are the eyes and ears of modern distributed systems, providing crucial visibility into application behavior, performance, and health. Without effective logging and monitoring practices, troubleshooting issues in complex systems becomes nearly impossible, and performance optimization remains guesswork.

These practices form the foundation of observability, enabling developers and operations teams to understand system behavior, detect anomalies, diagnose problems, and make data-driven decisions about system performance and reliability.

By implementing comprehensive logging and monitoring strategies, organizations can reduce mean time to detection (MTTD) and mean time to resolution (MTTR), improve service level objectives (SLOs), and ultimately deliver better user experiences.

This document explores the core concepts of logging and monitoring in modern distributed systems, examining their types, implementation approaches, practical scenarios, and industry-standard tools that transform these concepts into reality.

## What is Logging?

Logging is the practice of recording events, actions, and states that occur within a software system. These records serve as a historical account of application behavior, providing context for troubleshooting, auditing, and performance analysis.

In essence, logging creates a breadcrumb trail that developers can follow to understand what happened within a system at specific points in time.

The core functions of logging include:

- **Error Tracking**: Capturing failures, exceptions, and error conditions
- **Activity Auditing**: Recording user and system actions for security and compliance
- **Debugging Support**: Providing context for troubleshooting issues
- **Performance Insights**: Tracking execution time and resource utilization
- **Business Intelligence**: Capturing user behaviors and system usage patterns

## What is Monitoring?

Monitoring is the active observation of a system's health, performance, and behavior through the collection, aggregation, and analysis of metrics and events.

Unlike logging, which primarily records what has happened, monitoring focuses on the current state of a system and how it changes over time.

The core functions of monitoring include:

- **Health Checking**: Verifying that system components are operating correctly
- **Performance Tracking**: Measuring response times, throughput, and resource utilization
- **Capacity Planning**: Identifying trends to predict future resource needs
- **Anomaly Detection**: Identifying unusual patterns that may indicate problems
- **Alert Generation**: Notifying operators when thresholds are breached or problems occur

## Types of Logging

Different types of logging serve various purposes within a system's observability strategy.

### 1. Application Logging

Application logging captures events and states within the application code itself:

- User actions and business events
- Application flow and state transitions
- Exceptions, errors, and warning conditions
- Performance metrics for specific functions or operations
- Debug information for troubleshooting

**Example of structured application logging in Python:**

```python
import logging
import json
import time
from datetime import datetime

class StructuredLogRecord(logging.LogRecord):
    def __init__(self, *args, **kwargs):
        super().__init__(*args, **kwargs)
        self.timestamp = datetime.utcnow().isoformat() + 'Z'
        self.service = "user-service"

class StructuredLogger(logging.Logger):
    def makeRecord(self, *args, **kwargs):
        return StructuredLogRecord(*args, **kwargs)

logging.setLoggerClass(StructuredLogger)
logger = logging.getLogger("app")

def log_user_activity(user_id, action, details=None):
    """Log user activity in a structured format."""
    logger.info("User activity", extra={
        "user_id": user_id,
        "action": action,
        "details": details,
        "timestamp": time.time()
    })

# Usage example
try:
    # Some operation that might fail
    user_id = "user-123"
    log_user_activity(user_id, "login", {"source": "web", "ip": "192.168.1.1"})
    result = perform_critical_operation(user_id)
    log_user_activity(user_id, "operation_completed", {"operation_id": "op-456"})
except Exception as e:
    logger.error("Operation failed", extra={
        "user_id": user_id,
        "error": str(e),
        "stack_trace": traceback.format_exc()
    })
```

### 2. System Logging

System logging captures events from the underlying infrastructure and platforms:

- Operating system events and errors
- Service starts, stops, and restarts
- Security events (authentication, authorization)
- Resource utilization and capacity metrics
- Network activity and connectivity events

**Example of system logs in Linux (systemd journal):**

```
May 05 20:23:52 web-server-01 sshd[12345]: Accepted publickey for admin from 10.0.0.1 port 50234
May 05 20:24:15 web-server-01 kernel: [12345.678901] Out of memory: Kill process 23456 (java) score 251 or sacrifice child
May 05 20:24:15 web-server-01 kernel: [12345.678902] Killed process 23456 (java) total-vm:8192kB, anon-rss:6144kB, file-rss:0kB
May 05 20:25:03 web-server-01 nginx[3456]: 10.0.0.2 - - [05/May/2025:20:25:03 +0000] "GET /api/users HTTP/1.1" 200 1234 "-" "Mozilla/5.0 ..."
```

### 3. Access Logging

Access logging records interactions between users or systems and services:

- HTTP requests and responses in web servers
- API calls and their outcomes
- Database queries and transaction details
- Authentication and authorization attempts
- Resource access patterns and usage

**Example of NGINX access log configuration:**

```nginx
http {
    log_format detailed_log '$remote_addr - $remote_user [$time_local] '
                           '"$request" $status $body_bytes_sent '
                           '"$http_referer" "$http_user_agent" '
                           '$request_time $upstream_response_time $pipe '
                           '$connection $connection_requests';

    access_log /var/log/nginx/access.log detailed_log;

    server {
        listen 80;
        server_name example.com;

        # More specific logging for critical paths
        location /api/payments {
            access_log /var/log/nginx/payments.log detailed_log;
            # ...
        }

        # ...
    }
}
```

### 4. Audit Logging

Audit logging focuses on recording security-relevant events for compliance and forensic analysis:

- User authentication events (success and failures)
- Authorization decisions
- Data access and modification events
- Administrative actions and configuration changes
- Security policy violations

**Example of audit logging in a web application:**

```python
import logging
from functools import wraps
from flask import Flask, request, g

audit_logger = logging.getLogger("audit")

def audit_log(event_type):
    """Decorator to audit log function calls."""
    def decorator(f):
        @wraps(f)
        def wrapped(*args, **kwargs):
            user_id = g.user.id if hasattr(g, 'user') else 'anonymous'

            # Log before action
            audit_logger.info("Audit event", extra={
                "event_type": event_type,
                "user_id": user_id,
                "action": "attempt",
                "resource": request.path,
                "client_ip": request.remote_addr
            })

            try:
                result = f(*args, **kwargs)

                # Log successful action
                audit_logger.info("Audit event", extra={
                    "event_type": event_type,
                    "user_id": user_id,
                    "action": "success",
                    "resource": request.path,
                    "client_ip": request.remote_addr,
                    "details": {"status_code": getattr(result, "status_code", 200)}
                })

                return result
            except Exception as e:
                # Log failed action
                audit_logger.info("Audit event", extra={
                    "event_type": event_type,
                    "user_id": user_id,
                    "action": "failure",
                    "resource": request.path,
                    "client_ip": request.remote_addr,
                    "details": {"error": str(e)}
                })
                raise

        return wrapped
    return decorator

app = Flask(__name__)

@app.route('/admin/users/<user_id>', methods=['DELETE'])
@audit_log("user_deletion")
def delete_user(user_id):
    # User deletion logic here
    return {"status": "success"}, 200
```

### 5. Transaction Logging

Transaction logging records business-level operations and their outcomes:

- Financial transactions and their status
- Order processing events
- User registration and profile updates
- Content creation and modification
- Integration events between systems

This type of logging is particularly important for business intelligence, auditing, and reconstructing user journeys.

## Types of Monitoring

Monitoring comes in various forms, each serving different observability needs.

### 1. Infrastructure Monitoring

Infrastructure monitoring focuses on the health and performance of physical or virtual hardware and systems:

- CPU, memory, disk, and network utilization
- Server availability and uptime
- Virtual machine and container metrics
- Cloud resource utilization and costs
- Hardware health (temperatures, fan speeds, etc.)

**Real-world example**: Using AWS CloudWatch to monitor EC2 instance metrics:

```python
import boto3
import datetime

cloudwatch = boto3.client('cloudwatch')

# Get CPU utilization for an EC2 instance
response = cloudwatch.get_metric_statistics(
    Namespace='AWS/EC2',
    MetricName='CPUUtilization',
    Dimensions=[
        {
            'Name': 'InstanceId',
            'Value': 'i-1234567890abcdef0'
        },
    ],
    StartTime=datetime.datetime.utcnow() - datetime.timedelta(days=1),
    EndTime=datetime.datetime.utcnow(),
    Period=300,  # 5-minute intervals
    Statistics=['Average', 'Maximum']
)

for datapoint in response['Datapoints']:
    print(f"Time: {datapoint['Timestamp']}, Avg CPU: {datapoint['Average']}%, Max CPU: {datapoint['Maximum']}%")
```

### 2. Application Performance Monitoring (APM)

APM tracks the performance and behavior of application code:

- Request latency and throughput
- Error rates and exceptions
- Database query performance
- External service dependencies
- Code execution hotspots

**Example of APM with Python and OpenTelemetry:**

```python
from opentelemetry import trace
from opentelemetry.sdk.trace import TracerProvider
from opentelemetry.sdk.trace.export import BatchSpanProcessor
from opentelemetry.exporter.otlp.proto.grpc.trace_exporter import OTLPSpanExporter
from opentelemetry.instrumentation.flask import FlaskInstrumentor
from flask import Flask

# Set up OpenTelemetry
trace.set_tracer_provider(TracerProvider())
otlp_exporter = OTLPSpanExporter(endpoint="localhost:4317")
span_processor = BatchSpanProcessor(otlp_exporter)
trace.get_tracer_provider().add_span_processor(span_processor)

# Create a tracer
tracer = trace.get_tracer(__name__)

# Initialize Flask app
app = Flask(__name__)
FlaskInstrumentor().instrument_app(app)

@app.route('/api/products/<product_id>')
def get_product(product_id):
    with tracer.start_as_current_span("fetch_product_details") as span:
        span.set_attribute("product.id", product_id)

        # Simulate database query
        with tracer.start_as_current_span("database_query"):
            product = query_database(product_id)

        # Simulate external API call for inventory
        with tracer.start_as_current_span("inventory_api_call"):
            inventory = check_inventory(product_id)

        # Combine results
        result = {**product, "inventory": inventory}
        return result

if __name__ == '__main__':
    app.run(debug=True)
```

### 3. Real User Monitoring (RUM)

RUM captures the actual user experience with an application:

- Page load times and rendering performance
- User interactions and journeys
- JavaScript errors and exceptions
- Network request performance
- User demographics and device information

**Example of RUM using JavaScript:**

```javascript
// Simple RUM implementation
class SimpleRUM {
  constructor(endpoint) {
    this.endpoint = endpoint;
    this.data = {
      pageLoadTime: 0,
      resourceLoadTimes: [],
      userInteractions: [],
      errors: [],
    };

    this.init();
  }

  init() {
    // Measure page load time
    window.addEventListener("load", () => {
      const navigationTiming = performance.getEntriesByType("navigation")[0];
      this.data.pageLoadTime =
        navigationTiming.loadEventEnd - navigationTiming.startTime;

      // Send initial page load data
      this.send("page_load", {
        url: window.location.href,
        loadTime: this.data.pageLoadTime,
        timestamp: new Date().toISOString(),
      });
    });

    // Track resource timings
    setInterval(() => {
      const resources = performance.getEntriesByType("resource");
      resources.forEach((resource) => {
        if (!this.data.resourceLoadTimes.includes(resource.name)) {
          this.data.resourceLoadTimes.push(resource.name);
          this.send("resource_load", {
            url: resource.name,
            duration: resource.duration,
            type: resource.initiatorType,
            timestamp: new Date().toISOString(),
          });
        }
      });
    }, 3000);

    // Track user interactions
    document.addEventListener("click", (event) => {
      const element = event.target;
      this.send("user_interaction", {
        type: "click",
        element: element.tagName,
        id: element.id || null,
        class: element.className || null,
        timestamp: new Date().toISOString(),
      });
    });

    // Track errors
    window.addEventListener("error", (errorEvent) => {
      this.send("error", {
        message: errorEvent.message,
        source: errorEvent.filename,
        line: errorEvent.lineno,
        column: errorEvent.colno,
        timestamp: new Date().toISOString(),
      });
    });
  }

  send(eventType, data) {
    // Send data to collection endpoint
    fetch(this.endpoint, {
      method: "POST",
      headers: {
        "Content-Type": "application/json",
      },
      body: JSON.stringify({
        eventType,
        data,
        sessionId: this.getSessionId(),
        userId: this.getUserId(),
      }),
      // Use keepalive to ensure the request completes even if the page is unloading
      keepalive: true,
    }).catch((error) => console.error("RUM data send failed:", error));
  }

  getSessionId() {
    // Generate or retrieve session ID from localStorage
    if (!localStorage.getItem("rum_session_id")) {
      localStorage.setItem("rum_session_id", "session_" + Date.now());
    }
    return localStorage.getItem("rum_session_id");
  }

  getUserId() {
    // In a real app, this would be the authenticated user ID
    return "anonymous";
  }
}

// Initialize RUM
const rum = new SimpleRUM("https://analytics-api.example.com/rum");
```

### 4. Synthetic Monitoring

Synthetic monitoring uses scripted interactions to simulate user behavior:

- Scheduled checks of critical user paths
- API endpoint availability and performance
- Multi-step business transaction validation
- Global performance from different geographic locations
- Competitor benchmarking

**Example of synthetic monitoring using Python and Selenium:**

```python
from selenium import webdriver
from selenium.webdriver.common.by import By
from selenium.webdriver.chrome.options import Options
import time
import json
import requests

def run_synthetic_test():
    # Set up headless Chrome
    chrome_options = Options()
    chrome_options.add_argument("--headless")
    chrome_options.add_argument("--no-sandbox")
    chrome_options.add_argument("--disable-dev-shm-usage")

    driver = webdriver.Chrome(options=chrome_options)

    results = {
        "test_name": "login_flow",
        "timestamp": time.time(),
        "steps": [],
        "overall_success": True,
        "total_duration_ms": 0
    }

    start_time = time.time()

    try:
        # Step 1: Load homepage
        step_start = time.time()
        driver.get("https://example.com")
        step_duration = (time.time() - step_start) * 1000  # ms

        results["steps"].append({
            "name": "load_homepage",
            "duration_ms": step_duration,
            "success": True
        })

        # Step 2: Click login button
        step_start = time.time()
        login_button = driver.find_element(By.ID, "login-button")
        login_button.click()
        time.sleep(1)  # Wait for page transition
        step_duration = (time.time() - step_start) * 1000  # ms

        results["steps"].append({
            "name": "navigate_to_login",
            "duration_ms": step_duration,
            "success": True
        })

        # Step 3: Fill in login form
        step_start = time.time()
        driver.find_element(By.ID, "username").send_keys("test@example.com")
        driver.find_element(By.ID, "password").send_keys("test_password")
        driver.find_element(By.ID, "login-submit").click()
        time.sleep(2)  # Wait for login processing
        step_duration = (time.time() - step_start) * 1000  # ms

        # Check if login was successful (dashboard element exists)
        dashboard_element = driver.find_element(By.ID, "user-dashboard")
        login_success = dashboard_element.is_displayed()

        results["steps"].append({
            "name": "perform_login",
            "duration_ms": step_duration,
            "success": login_success
        })

        if not login_success:
            results["overall_success"] = False

    except Exception as e:
        results["overall_success"] = False
        results["error"] = str(e)
    finally:
        driver.quit()
        results["total_duration_ms"] = (time.time() - start_time) * 1000  # ms

    # Send results to monitoring system
    requests.post(
        "https://monitoring.example.com/synthetic/results",
        json=results
    )

    return results

if __name__ == "__main__":
    result = run_synthetic_test()
    print(json.dumps(result, indent=2))
```

## Logging Best Practices

Effective logging requires deliberate design and implementation choices.

### 1. Use Structured Logging

Structured logging formats (JSON, ECS) provide consistent, machine-parsable records:

```python
# Unstructured log (difficult to parse)
logger.info(f"User {user_id} purchased item {item_id} for ${price} using {payment_method}")

# Structured log (easy to parse and analyze)
logger.info("Purchase completed", extra={
    "user_id": user_id,
    "item_id": item_id,
    "price": price,
    "payment_method": payment_method,
    "transaction_id": transaction_id
})
```

### 2. Define and Use Log Levels Consistently

Each log level should have a clear purpose and usage pattern:

- **ERROR**: System errors requiring immediate attention
- **WARN**: Potential issues that don't prevent operation
- **INFO**: Normal but significant events
- **DEBUG**: Detailed information useful for troubleshooting
- **TRACE**: Very detailed debugging (method entry/exit, variable values)

### 3. Include Contextual Information

Enrich logs with context to aid troubleshooting:

- Unique request/transaction IDs for tracing requests across services
- User IDs or session information (carefully handle PII)
- Relevant business context (order ID, product ID, etc.)
- Environmental information (server name, deployment version)

### 4. Implement Sampling Strategies

For high-volume systems, consider sampling to reduce log volume:

- Keep 100% of error and warning logs
- Sample informational logs (e.g., 10% of normal traffic)
- Use adaptive sampling based on system conditions
- Increase sampling during troubleshooting or for specific users

### 5. Handle Sensitive Information

Protect privacy and security in logs:

- Never log credentials, tokens, or keys
- Mask or hash personally identifiable information (PII)
- Consider regulatory requirements (GDPR, HIPAA, etc.)
- Implement log access controls and auditing

## Monitoring Best Practices

Effective monitoring requires thoughtful instrumentation and analysis.

### 1. Define Meaningful Metrics

Focus on metrics that drive business decisions:

- **The Four Golden Signals**:
  - Latency (how long it takes to serve a request)
  - Traffic (load on the system)
  - Errors (rate of failed requests)
  - Saturation (how "full" the service is)
- **USE Method**:
  - Utilization (percent time the resource is busy)
  - Saturation (amount of work resource has to do)
  - Errors (count of error events)

### 2. Implement Effective Alerting

Design alerts that minimize noise and maximize actionability:

- Alert on symptoms, not causes
- Define clear thresholds based on historical data
- Implement alert severity levels
- Avoid alert fatigue with thoughtful aggregation
- Include runbooks or troubleshooting guidance

### 3. Use Visualization Effectively

Design dashboards that provide immediate insights:

- Organize metrics by service or user journey
- Show correlated metrics together
- Include both real-time and historical views
- Highlight anomalies and trends
- Make dashboards accessible to all stakeholders

### 4. Implement Distributed Tracing

Connect requests across multiple services:

- Use correlation IDs to track requests
- Measure latency between services
- Visualize request flows and bottlenecks
- Identify cross-service dependencies

### 5. Automate Response to Common Issues

Reduce mean time to resolution with automation:

- Auto-scaling based on load metrics
- Self-healing systems (restart failed services)
- Automated rollbacks for problematic deployments
- Circuit breakers for failing dependencies

## Key Tools and Technologies

Several tools dominate the logging and monitoring landscape:

### Prometheus

Prometheus is an open-source monitoring system with a dimensional data model, flexible query language, and alert management:

```yaml
# prometheus.yml configuration
global:
  scrape_interval: 15s

scrape_configs:
  - job_name: "web_servers"
    static_configs:
      - targets: ["web-server-1:9090", "web-server-2:9090"]

  - job_name: "api_servers"
    static_configs:
      - targets: ["api-server-1:9090", "api-server-2:9090"]
```

**Example of exposing metrics in Python:**

```python
from prometheus_client import start_http_server, Counter, Histogram
import random
import time

# Create metrics
REQUEST_COUNT = Counter('app_requests_total', 'Total app HTTP requests', ['method', 'endpoint', 'status'])
REQUEST_LATENCY = Histogram('app_request_latency_seconds', 'Request latency', ['endpoint'])

# Start metrics server
start_http_server(9090)

# Simulate HTTP requests
def simulate_requests():
    endpoints = ['/api/users', '/api/products', '/api/orders']
    methods = ['GET', 'POST', 'PUT', 'DELETE']
    statuses = ['200', '400', '500']

    while True:
        endpoint = random.choice(endpoints)
        method = random.choice(methods)
        status = random.choice(statuses)
        latency = random.uniform(0.01, 10.0)

        # Record metrics
        REQUEST_COUNT.labels(method=method, endpoint=endpoint, status=status).inc()

        # Track latency
        with REQUEST_LATENCY.labels(endpoint=endpoint).time():
            # Simulating actual work
            time.sleep(latency)

        time.sleep(1)

if __name__ == '__main__':
    simulate_requests()
```

### Grafana

Grafana provides visualization and dashboarding for metrics data:

```
+---------------------------------+
|  Service Health Dashboard       |
+---------------------------------+
|                                 |
|  +---------+     +---------+   |
|  | CPU     |     | Memory  |   |
|  | Usage   |     | Usage   |   |
|  |         |     |         |   |
|  +---------+     +---------+   |
|                                 |
|  +---------+     +---------+   |
|  | Request |     | Error   |   |
|  | Rate    |     | Rate    |   |
|  |         |     |         |   |
|  +---------+     +---------+   |
|                                 |
|  +-------------------------+   |
|  | Service Latency Over Time|   |
|  |                         |   |
|  |                         |   |
|  +-------------------------+   |
+---------------------------------+
```

Grafana connects to various data sources (Prometheus, CloudWatch, etc.) and allows creating dashboards with panels for different metrics and logs.

### AWS CloudWatch

CloudWatch is AWS's monitoring and observability service:

```python
import boto3
import datetime

# Create CloudWatch client
cloudwatch = boto3.client('cloudwatch')

# Create a custom metric
response = cloudwatch.put_metric_data(
    Namespace='MyApplication',
    MetricData=[
        {
            'MetricName': 'UserRegistrations',
            'Dimensions': [
                {
                    'Name': 'Environment',
                    'Value': 'Production'
                },
            ],
            'Value': 1.0,
            'Unit': 'Count'
        },
    ]
)

# Create an alarm on the metric
response = cloudwatch.put_metric_alarm(
    AlarmName='HighUserRegistrationRate',
    ComparisonOperator='GreaterThanThreshold',
    EvaluationPeriods=1,
    MetricName='UserRegistrations',
    Namespace='MyApplication',
    Period=60,
    Statistic='Sum',
    Threshold=100.0,
    ActionsEnabled=True,
    AlarmDescription='Alarm when user registrations exceed 100 per minute',
    Dimensions=[
        {
            'Name': 'Environment',
            'Value': 'Production'
        },
    ],
    AlarmActions=[
        'arn:aws:sns:us-east-1:123456789012:AlertNotifications',
    ]
)
```

### ELK Stack (Elasticsearch, Logstash, Kibana)

The ELK stack provides log collection, processing, and visualization:

```yaml
# Logstash configuration
input {
file {
path => "/var/log/application/*.log"
type => "application"
}
}

filter {
if [type] == "application" {
json {
source => "message"
}
date {
match => [ "timestamp", "ISO8601" ]
}
}
}

output {
elasticsearch {
hosts => ["elasticsearch:9200"]
index => "application-logs-%{+YYYY.MM.dd}"
}
}
```

## Implementation Examples

### Setting Up Prometheus and Grafana

```yaml
# docker-compose.yml for Prometheus and Grafana
version: "3"
services:
  prometheus:
    image: prom/prometheus
    volumes:
      - ./prometheus.yml:/etc/prometheus/prometheus.yml
    ports:
      - "9090:9090"
    command:
      - "--config.file=/etc/prometheus/prometheus.yml"
      - "--storage.tsdb.path=/prometheus"
      - "--web.console.libraries=/usr/share/prometheus/console_libraries"
      - "--web.console.templates=/usr/share/prometheus/consoles"

  grafana:
    image: grafana/grafana
    depends_on:
      - prometheus
    ports:
      - "3000:3000"
    volumes:
      - grafana-storage:/var/lib/grafana
    environment:
      - GF_SECURITY_ADMIN_PASSWORD=secure_password
      - GF_USERS_ALLOW_SIGN_UP=false

volumes:
  grafana-storage:
```

### Python Flask Application with Logging and Monitoring

```python
import time
import logging
import json
from flask import Flask, request, g
from logging.handlers import RotatingFileHandler
import uuid
from prometheus_client import Counter, Histogram, start_http_server

# Set up logging
log_handler = RotatingFileHandler('application.log', maxBytes=10000000, backupCount=5)
log_handler.setFormatter(logging.Formatter(
    '%(asctime)s %(levelname)s: %(message)s [in %(pathname)s:%(lineno)d]'
))
logger = logging.getLogger('app')
logger.setLevel(logging.INFO)
logger.addHandler(log_handler)

# Set up Prometheus metrics
REQUEST_COUNT = Counter('app_request_count', 'Application Request Count', ['method', 'endpoint', 'status'])
REQUEST_LATENCY = Histogram('app_request_latency_seconds', 'Application Request Latency', ['endpoint'])
USER_LOGINS = Counter('app_user_logins', 'Number of user logins')

# Start Prometheus metrics server on port 9090
start_http_server(9090)

app = Flask(__name__)

@app.before_request
def before_request():
    g.request_id = str(uuid.uuid4())
    g.start_time = time.time()
    logger.info('Request started', extra={
        'request_id': g.request_id,
        'method': request.method,
        'path': request.path,
        'remote_addr': request.remote_addr,
        'user_agent': request.user_agent.string
    })

@app.after_request
def after_request(response):
    response_time = time.time() - g.start_time
    status_code = response.status_code

    # Update Prometheus metrics
    REQUEST_COUNT.labels(
        method=request.method,
        endpoint=request.path,
        status=status_code
    ).inc()
    REQUEST_LATENCY.labels(endpoint=request.path).observe(response_time)

    # Log the response
    logger.info('Request finished', extra={
        'request_id': g.request_id,
        'method': request.method,
        'path': request.path,
        'status': status_code,
        'response_time': response_time
    })

    return response

@app.route('/api/users/login', methods=['POST'])
def login():
    # Logging before processing
    logger.info('Login attempt', extra={
        'request_id': g.request_id,
        'username': request.json.get('username')
    })

    # Process login (simulated)
    time.sleep(0.1)  # Simulate processing time
    success = request.json.get('username') == 'test' and request.json.get('password') == 'password'

    if success:
        # Increment our login counter
        USER_LOGINS.inc()

        logger.info('Login successful', extra={
            'request_id': g.request_id,
            'username': request.json.get('username')
        })
        return {'status': 'success', 'message': 'Login successful'}, 200
    else:
        logger.warning('Login failed', extra={
            'request_id': g.request_id,
            'username': request.json.get('username'),
            'reason': 'Invalid credentials'
        })
        return {'status': 'error', 'message': 'Invalid credentials'}, 401

@app.route('/api/products', methods=['GET'])
def get_products():
    logger.info('Fetching products', extra={
        'request_id': g.request_id,
        'filters': request.args.to_dict()
    })

    # Simulate database query
    time.sleep(0.2)

    # Simulate occasional errors
    if random.random() < 0.05:  # 5% chance of error
        logger.error('Database connection error', extra={
            'request_id': g.request_id,
            'error': 'Connection timeout'
        })
        return {'status': 'error', 'message': 'Database error'}, 500

    products = [
        {'id': 1, 'name': 'Product A', 'price': 19.99},
        {'id': 2, 'name': 'Product B', 'price': 29.99}
    ]

    logger.info('Products fetched successfully', extra={
        'request_id': g.request_id,
        'count': len(products)
    })

    return {'products': products}, 200

if __name__ == '__main__':
    logger.info('Application starting')
    app.run(debug=True, host='0.0.0.0', port=5000)
```

### AWS CloudWatch Integration Example

```python
import boto3
import datetime
import time
import logging
import json
from flask import Flask, request, g
import uuid

# Set up CloudWatch client
cloudwatch = boto3.client('cloudwatch')
logs_client = boto3.client('logs')
log_group_name = '/app/production'

# Create log group if it doesn't exist
try:
    logs_client.create_log_group(logGroupName=log_group_name)
except logs_client.exceptions.ResourceAlreadyExistsException:
    pass

# Set up logging
logger = logging.getLogger('app')
logger.setLevel(logging.INFO)

app = Flask(__name__)

@app.before_request
def before_request():
    g.request_id = str(uuid.uuid4())
    g.start_time = time.time()

@app.after_request
def after_request(response):
    response_time = time.time() - g.start_time
    status_code = response.status_code

    # Send custom metric to CloudWatch
    cloudwatch.put_metric_data(
        Namespace='AppMetrics',
        MetricData=[
            {
                'MetricName': 'ResponseTime',
                'Dimensions': [
                    {
                        'Name': 'Endpoint',
                        'Value': request.path
                    },
                ],
                'Value': response_time,
                'Unit': 'Seconds'
            },
            {
                'MetricName': 'RequestCount',
                'Dimensions': [
                    {
                        'Name': 'Endpoint',
                        'Value': request.path
                    },
                    {
                        'Name': 'StatusCode',
                        'Value': str(status_code)
                    }
                ],
                'Value': 1,
                'Unit': 'Count'
            }
        ]
    )

    # Send log to CloudWatch Logs
    log_stream_name = datetime.datetime.now().strftime('%Y-%m-%d')

    try:
        logs_client.create_log_stream(
            logGroupName=log_group_name,
            logStreamName=log_stream_name
        )
    except logs_client.exceptions.ResourceAlreadyExistsException:
        pass

    log_event = {
        'request_id': g.request_id,
        'method': request.method,
        'path': request.path,
        'status': status_code,
        'response_time': response_time,
        'remote_addr': request.remote_addr,
        'user_agent': request.user_agent.string
    }

    try:
        response = logs_client.describe_log_streams(
            logGroupName=log_group_name,
            logStreamNamePrefix=log_stream_name,
            limit=1
        )

        sequence_token = response['logStreams'][0].get('uploadSequenceToken')

        kwargs = {
            'logGroupName': log_group_name,
            'logStreamName': log_stream_name,
            'logEvents': [
                {
                    'timestamp': int(time.time() * 1000),
                    'message': json.dumps(log_event)
                }
            ]
        }

        if sequence_token:
            kwargs['sequenceToken'] = sequence_token

        logs_client.put_log_events(**kwargs)

    except Exception as e:
        print(f"Failed to send logs to CloudWatch: {e}")

    return response

@app.route('/api/health', methods=['GET'])
def health_check():
    return {'status': 'healthy'}, 200

@app.route('/api/users', methods=['GET'])
def get_users():
    # Simulate database query
    time.sleep(0.1)

    users = [
        {'id': 1, 'name': 'John Doe'},
        {'id': 2, 'name': 'Jane Smith'}
    ]

    return {'users': users}, 200

if __name__ == '__main__':
    app.run(debug=True, host='0.0.0.0', port=5000)
```

## Logging and Monitoring Architecture

A comprehensive logging and monitoring architecture typically includes several components working together:

```
+------------------+                 +------------------+
| Application      |                 | Infrastructure   |
| - Request logs   |                 | - CPU/Memory     |
| - Error logs     |                 | - Disk/Network   |
| - Audit logs     |                 | - Host metrics   |
+--------+---------+                 +--------+---------+
         |                                    |
         v                                    v
+------------------+                 +------------------+
| Log Collectors   |                 | Metrics Agents   |
| - Fluentd/Fluent |                 | - Prometheus     |
|   Bit            |                 |   Node Exporter  |
| - Logstash       |                 | - StatsD         |
| - AWS CloudWatch |                 | - Telegraf       |
|   Agent          |                 +--------+---------+
+--------+---------+                          |
         |                                    |
         v                                    v
+------------------+                 +------------------+
| Log Storage      |                 | Metrics Storage  |
| - Elasticsearch  |                 | - Prometheus     |
| - CloudWatch Logs|                 | - InfluxDB       |
| - Splunk         |                 | - AWS CloudWatch |
+--------+---------+                 +--------+---------+
         |                                    |
         v                                    v
+------------------+                 +------------------+
| Log Analysis     |                 | Dashboards       |
| - Kibana         |                 | - Grafana        |
| - CloudWatch     |                 | - CloudWatch     |
|   Insights       |                 |   Dashboards     |
| - Splunk Search  |                 | - DataDog        |
+--------+---------+                 +--------+---------+
         |                                    |
         v                                    v
+------------------------------------------+
|              Alert Manager               |
| - Prometheus Alertmanager                |
| - PagerDuty                              |
| - OpsGenie                               |
| - CloudWatch Alarms                      |
+------------------------------------------+
```

This architecture enables:

1. Collecting logs and metrics from all system components
2. Storing data in specialized storage systems
3. Analyzing logs and visualizing metrics
4. Generating alerts when issues are detected

## Advanced Topics

### 1. Distributed Tracing

Distributed tracing tracks requests across multiple services:

```
Service A                   Service B                   Service C
+----------+                +----------+                +----------+
|          |---request----->|          |---request----->|          |
|          |                |          |                |          |
|  Span A  |                |  Span B  |                |  Span C  |
|          |<--response-----|          |<--response-----|          |
+----------+                +----------+                +----------+
|<----------------- Complete Transaction ----------------->|
```

**Example using OpenTelemetry:**

```python
from opentelemetry import trace
from opentelemetry.sdk.trace import TracerProvider
from opentelemetry.sdk.trace.export import BatchSpanProcessor
from opentelemetry.exporter.jaeger.thrift import JaegerExporter
from opentelemetry.instrumentation.requests import RequestsInstrumentor
from opentelemetry.instrumentation.flask import FlaskInstrumentor
import flask
import requests

# Set up OpenTelemetry
trace.set_tracer_provider(TracerProvider())
jaeger_exporter = JaegerExporter(
    agent_host_name="jaeger",
    agent_port=6831,
)
trace.get_tracer_provider().add_span_processor(BatchSpanProcessor(jaeger_exporter))

# Create a tracer
tracer = trace.get_tracer(__name__)

# Instrument libraries
RequestsInstrumentor().instrument()

# Create Flask app
app = flask.Flask(__name__)
FlaskInstrumentor().instrument_app(app)

@app.route("/")
def index():
    with tracer.start_as_current_span("frontend_request"):
        # Make request to backend service
        response = requests.get("http://backend-service:5000/data")
        return f"Frontend received: {response.text}"

if __name__ == "__main__":
    app.run(host="0.0.0.0", port=5000)
```

### 2. Anomaly Detection

Machine learning can detect unusual patterns in metrics:

```python
import pandas as pd
import numpy as np
from sklearn.ensemble import IsolationForest
import matplotlib.pyplot as plt

# Load metrics data (could be from Prometheus, CloudWatch, etc.)
df = pd.read_csv('response_times.csv')

# Prepare data for anomaly detection
X = df[['response_time']].values

# Train isolation forest model
model = IsolationForest(contamination=0.05)  # Expect 5% anomalies
model.fit(X)

# Predict anomalies
df['anomaly'] = model.predict(X)
df['anomaly'] = df['anomaly'].map({1: 0, -1: 1})  # Map to 0: normal, 1: anomaly

# Visualize results
plt.figure(figsize=(12, 6))
plt.plot(df['timestamp'], df['response_time'], 'b.', label='Normal')
plt.plot(df[df['anomaly'] == 1]['timestamp'], df[df['anomaly'] == 1]['response_time'], 'r.', label='Anomaly')
plt.xlabel('Time')
plt.ylabel('Response Time (ms)')
plt.title('Response Time Anomalies')
plt.legend()
plt.savefig('anomalies.png')

# Alert on anomalies
anomalies = df[df['anomaly'] == 1]
if not anomalies.empty:
    print(f"Found {len(anomalies)} anomalies")
    # Send alert to alert manager
```

### 3. Service Level Objectives (SLOs)

SLOs define reliability targets:

```python
from prometheus_client import start_http_server, Counter, Histogram, Gauge
import random
import time

# Start Prometheus metrics server
start_http_server(9090)

# Define metrics
REQUEST_COUNT = Counter('requests_total', 'Total requests', ['status'])
REQUEST_LATENCY = Histogram('request_latency_seconds', 'Request latency',
                           buckets=[0.005, 0.01, 0.025, 0.05, 0.075, 0.1, 0.25, 0.5, 0.75, 1.0, 2.5, 5.0, 7.5, 10.0])

# SLO metrics
ERROR_BUDGET_REMAINING = Gauge('error_budget_remaining_ratio', 'Remaining error budget as a ratio')

# SLO configuration
SLO_TARGET = 0.999  # 99.9% availability
ERROR_BUDGET = 1 - SLO_TARGET  # 0.1% error budget
WINDOW_SECONDS = 30 * 24 * 60 * 60  # 30 days in seconds

# Tracking state
errors = 0
total = 0
start_time = time.time()

while True:
    # Simulate a request
    latency = max(0.001, random.normalvariate(0.05, 0.01))  # Normal distribution around 50ms
    is_error = random.random() < 0.005  # 0.5% error rate

    # Record metrics
    if is_error:
        REQUEST_COUNT.labels(status='error').inc()
        errors += 1
    else:
        REQUEST_COUNT.labels(status='success').inc()
    total += 1

    # Record latency
    REQUEST_LATENCY.observe(latency)

    # Calculate error budget remaining
    elapsed = time.time() - start_time
    if elapsed > WINDOW_SECONDS:
        # Reset if outside window
        errors = 0
        total = 0
        start_time = time.time()

    # Calculate error ratio and budget remaining
    error_ratio = errors / total if total > 0 else 0
    budget_used_ratio = error_ratio / ERROR_BUDGET
    budget_remaining_ratio = max(0, 1 - budget_used_ratio)

    # Update gauge
    ERROR_BUDGET_REMAINING.set(budget_remaining_ratio)

    # Sleep for random interval
    time.sleep(random.uniform(0.01, 0.1))
```

## Conclusion

Logging and monitoring are foundational practices for building reliable, observable distributed systems. They provide the visibility needed to understand system behavior, troubleshoot issues, and make data-driven decisions about performance and reliability.

By implementing structured logging, collecting meaningful metrics, visualizing data effectively, and setting up appropriate alerts, developers can create systems that are not only functional but also observable and maintainable over time.

As distributed systems continue to grow in complexity, these practices become increasingly critical for maintaining reliability and performance. Mastering the tools and techniques of logging and monitoring is an essential skill for any software engineer working with modern application architectures.

## Further Reading

- [Google's Site Reliability Engineering Book](https://sre.google/sre-book/monitoring-distributed-systems/)
- [Prometheus Documentation](https://prometheus.io/docs/introduction/overview/)
- [Grafana Getting Started Guide](https://grafana.com/docs/grafana/latest/getting-started/)
- [AWS CloudWatch Documentation](https://docs.aws.amazon.com/AmazonCloudWatch/latest/monitoring/WhatIsCloudWatch.html)
- [Distributed Systems Observability (Book by Cindy Sridharan)](https://www.oreilly.com/library/view/distributed-systems-observability/9781492033431/)
- [OpenTelemetry Documentation](https://opentelemetry.io/docs/)
- [The Art of Monitoring (Book by James Turnbull)](https://www.artofmonitoring.com/)
