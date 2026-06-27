# 22 Monitoring and Logging

This guide covers system monitoring, application performance monitoring (APM), structured logging (Winston), centralized logging aggregations, alerting rules, and error tracking platforms (Sentry).

## Monitoring vs Logging

### Definition
Monitoring is the automated collection and analysis of quantitative system metrics (CPU usage, memory footprint, request latency, error rates). Logging is the recording of discrete, qualitative application events containing contextual metadata (e.g. "User ID 101 purchased order #502").

### Real-world Analogy
Imagine running a delivery shipping truck company. Monitoring is the dashboard instruments displaying speed, engine temperature, and gas tank levels (metrics). Logging is the driver's notebook logging: "Left warehouse at 9 AM; dropped package at house 5 at 10 AM; stopped for lunch at 12 PM" (events).

### Code Example
```javascript
// Metrics (Monitoring data sent to Prometheus/Datadog)
metrics.increment("http.requests.total", 1, { path: "/login" });

// Log Event (Qualitative text details)
logger.info("User login processed successfully", { userId: "101", ip: "192.0.2.1" });
```

### Common Interview Questions
- Why do you need both quantitative monitoring metrics and qualitative log files in production?
- Explain the concept of the Four Golden Signals in system monitoring.
- What are the operational differences between time-series metrics databases and log indices?

### Reference Links
- [Google SRE Book: Monitoring Distributed Systems](https://sre.google/sre-book/monitoring-distributed-systems/)
- [Grafana: Metrics, logs, traces](https://grafana.com/blog/2018/10/31/the-three-pillars-of-observability/)

## Application Performance Monitoring (APM)

### Definition
APM tools (Datadog, New Relic, OpenTelemetry) trace requests across distributed systems, mapping transaction traces, database query bottlenecks, and network latency across microservice boundaries.

### Real-world Analogy
Imagine tracking a customer package through a global post network. Instead of only knowing "The package left Seattle" and "The package arrived in Paris", APM is a GPS tracker that logs exactly how many minutes the package sat at the sorting desk, how long it spent inside the truck engine bay, and which plane it boarded.

### Code Example
```javascript
// OpenTelemetry manual span tracing inside a Node.js controller
const api = require("@opentelemetry/api");
const tracer = api.trace.getTracer("user-service");

async function handleRequest() {
  const span = tracer.startSpan("process-login");
  try {
    await db.validateUser();
  } finally {
    span.end(); // Record span duration
  }
}
```

### Common Interview Questions
- What is a transaction trace (distributed tracing) and how does it propagate span IDs across HTTP headers?
- How do APMs help locate database query bottlenecks?
- Compare OpenTelemetry and proprietary APM SDKs (Datadog/New Relic).

### Reference Links
- [OpenTelemetry Documentation](https://opentelemetry.io/docs/)
- [Datadog APM Guide](https://docs.datadoghq.com/tracing/)

## Structured Logging

### Definition
Structured logging formats log files as parseable JSON objects instead of loose text strings. This allows log aggregation engines (Elasticsearch, Loki) to parse, filter, and index log metadata parameters.

### Real-world Analogy
Text logging is writing diary entries: "At 2 PM Bob logged in from Berlin". Structured logging is typing the entries inside a spreadsheet template containing separate columns: Timestamp, Username, Action, Location. The spreadsheet makes it simple to filter all login actions from Berlin in seconds.

### Code Example
```javascript
const winston = require("winston");

const logger = winston.createLogger({
  level: "info",
  format: winston.format.json(), // Structured JSON logs
  transports: [
    new winston.transports.Console()
  ]
});

// Outputs: {"level":"info","message":"Order created","orderId":502,"timestamp":"..."}
logger.info("Order created", { orderId: 502 });
```

### Common Interview Questions
- Why are JSON structured logs preferred over plain text strings in production?
- What are Winston log levels and how do they map to RFC 5424 severity rules?
- How do you redact sensitive data (like passwords or credit card numbers) inside structured log formatters?

### Reference Links
- [Winston GitHub Documentation](https://github.com/winstonjs/winston)
- [Loggly: Structured Logging Benefits](https://www.solarwinds.com/resources/it-glossary/structured-logging)

## Centralized Logging

### Definition
Centralized logging aggregates log streams from multiple container nodes into a single, searchable repository using shipping agents (Fluentd, Logstash, Vector) and search index interfaces (ELK Stack, Grafana Loki, AWS CloudWatch).

### Real-world Analogy
Imagine a retail chain with 50 stores. Instead of forcing the head office manager to travel to each store and look at their local sales receipt drawers (local container logs), every store sends duplicate receipts to the head office mailroom (log shipper), which files them in a central searchable computer database.

### Code Example
```yaml
# Simple Promtail configuration snippet (ships logs to Grafana Loki)
clients:
  - url: http://loki:3100/loki/api/v1/push

scrape_configs:
  - job_name: system
    static_configs:
      - targets: [localhost]
        labels:
          job: varlogs
          __path__: /var/log/*.log
```

### Common Interview Questions
- Contrast the ELK (Elasticsearch, Logstash, Kibana) stack and the PLG (Promtail, Loki, Grafana) stack.
- What is a log shipper (e.g. Fluentd) and how does it prevent disk storage exhaust on container hosts?
- How do you search and query log indexes using PromQL/LogQL?

### Reference Links
- [Elasticsearch: ELK Stack](https://www.elastic.co/what-is/elk-stack)
- [Grafana Loki Documentation](https://grafana.com/docs/loki/latest/)

## Alerting Strategies

### Definition
Alerting strategies define conditional rules that trigger system notifications (Slack, PagerDuty) when metrics exceed thresholds, while configuring filters to prevent alert fatigue.

### Real-world Analogy
Imagine a house fire alarm. You configure a smoke detector to ring the fire station only if the room temperature exceeds 65 degrees (threshold alert). You don't set the alarm to ring the station if someone burns a piece of toast (low severity warning).

### Code Example
```yaml
# Prometheus alert rule example template
groups:
  - name: APIAlerts
    rules:
      - alert: HighErrorRate
        expr: rate(http_requests_total{status=~"5.."}[5m]) > 0.05
        for: 2m
        labels:
          severity: critical
        annotations:
          summary: "API Error rate exceeds 5% on instance {{ $labels.instance }}"
```

### Common Interview Questions
- What is Alert Fatigue and how do you structure alert severities to prevent it?
- How do you define Alert Thresholds using sliding metrics windows (e.g. for: 5m)?
- What is the difference between paging alerts (requiring wake-ups) and informational logs?

### Reference Links
- [Prometheus Alerting Rules](https://prometheus.io/docs/prometheus/latest/configuration/alerting_rules/)
- [PagerDuty: Incident Response Guides](https://response.pagerduty.com/)

## Error Tracking (Sentry)

### Definition
Error tracking platforms (Sentry) capture runtime exceptions, parsing stack traces, binding release source maps to locate error lines, recording breadcrumbs (user actions before crash), and grouping identical errors.

### Real-world Analogy
Imagine a car engine diagnostics computer. If a spark plug fails, the computer doesn't just display: "Engine failed". It captures the exact cylinder number, the gasoline flow rate before failure (breadcrumbs), and the speed the car was traveling, sending a detailed report to the mechanic.

### Code Example
```javascript
const Sentry = require("@sentry/node");

Sentry.init({
  dsn: "https://examplePublicKey@o0.ingest.sentry.io/0",
  tracesSampleRate: 1.0,
});

app.use(Sentry.Handlers.requestHandler());

app.get("/bug", (req, res) => {
  throw new Error("Critical payment bug"); // Captured by Sentry handler
});

app.use(Sentry.Handlers.errorHandler());
```

### Common Interview Questions
- What are Sentry Breadcrumbs and how do they help reproduce errors?
- Why do you need to upload JS Source Maps to Sentry during build pipelines?
- How does Sentry group individual runtime exceptions into unique issues?

### Reference Links
- [Sentry Documentation](https://docs.sentry.io/)

## Scenario-based Interview Questions for Observability

### Scenario 1
A memory leak is suspected in your Express API server. In production, container memory utilization grows incrementally over 24 hours until the container runtime kills the process (Out Of Memory - OOM crash). How do you investigate this?

*Expected Approach:*
1. Configure memory tracking metrics in your monitoring tool (e.g. track process.memoryUsage().heapUsed in Prometheus) to verify memory grows linearly without plateaus.
2. Run a heap profiler (like Clinic.js or Chrome DevTools heap snapshot comparison) on a local environment or load-testing cluster.
3. Compare heap snapshots before and after sending mock requests to identify which objects (e.g. unclosed connections or global array caches) are not cleared by Garbage Collection.

### Scenario 2
Your team receives 50 alert emails every day listing warning messages (e.g. "User login failed due to bad password" or "S3 bucket checked successfully"). Developers ignore these emails. When a critical database crash happens, no one notices for 3 hours. How do you reform this?

*Expected Approach:*
1. Address the alert fatigue issue: audit all active alarm rules.
2. Demote non-actionable warning logs (like bad passwords or success checks) from alerting channels; keep them inside structured log files for manual search queries.
3. Route critical, actionable alerts (like database offline or API error rate > 5%) to pager platforms (PagerDuty) and secure Slack channels.
4. Establish clear SRE playbooks outlining specific steps to resolve each active alert condition.
