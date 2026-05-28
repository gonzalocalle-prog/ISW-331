# Week 16 - Monitoring and Logging

> **Block 5: Cloud and Production**
> This week focuses on a practical reality of software in production: if you cannot see what the system is doing, you cannot operate it well. We should leave with a clear mental model of logs, metrics, traces, dashboards, alerts, and the tool choices available when managed cloud services are not an option.

---

## Part 1 - Why Monitoring and Logging Matter

Most projects break in predictable ways once they leave local development:

- A route starts failing only in production
- A deployment succeeds, but the app is slow
- A background task crashes silently
- A third-party API times out
- A database query becomes much slower with real data

Without observability, debugging becomes guesswork.

> **Key idea:** Monitoring and logging are not "nice to have" production extras. They are part of how professional teams reduce downtime, understand failures, and improve reliability.

This week should help answer three questions:

1. What is happening?
2. Why is it happening?
3. How quickly can we detect and respond?

---

## Part 2 - Monitoring, Logging, and Observability

These words are related, but they are not identical.

| Term | What it means | Example |
|------|----------------|---------|
| **Logging** | Recording discrete events that happened in the system | `User 42 submitted order 991` |
| **Monitoring** | Watching signals over time to detect health or performance changes | Error rate increased from 1% to 8% |
| **Observability** | Having enough system signals to understand internal behavior from external outputs | Tracing a slow request across frontend, API, and database |
| **Alerting** | Notifying the team when a defined threshold or condition is met | Trigger an alert when p95 latency is above 1.5s |
| **Error tracking** | Capturing exceptions with context such as stack traces and request metadata | Sentry issue grouped by stack trace |

### The three main telemetry signals

Modern observability usually revolves around three core signals:

| Signal | Best for | Example question |
|--------|----------|------------------|
| **Logs** | Detailed event history and debugging context | What exactly happened before the crash? |
| **Metrics** | Trends, dashboards, and alert thresholds | Is the error rate rising? |
| **Traces** | Following one request across multiple steps or services | Where is this request spending time? |

Some teams also collect profiles, session replay, or uptime checks, but logs, metrics, and traces are the core foundation.

---

## Part 3 - What Good Logs Look Like

Beginner teams often log too little or log the wrong thing.

### Weak logging

```text
Something failed
```

This is almost useless because it has no context.

### Better logging

```json
{
  "level": "error",
  "message": "Payment provider request failed",
  "requestId": "9d2d1d44",
  "route": "/api/orders",
  "userId": 42,
  "statusCode": 502,
  "provider": "Stripe",
  "timestamp": "2026-05-28T14:35:00Z"
}
```

### Principles for useful application logs

- Prefer structured logs over random free-text logs
- Include identifiers such as `requestId`, route, user id when appropriate, and environment
- Log meaningful state changes, errors, retries, and external dependency failures
- Avoid logging secrets, tokens, passwords, or private personal data
- Use log levels intentionally: `debug`, `info`, `warn`, `error`

### Common log events worth capturing

- Application startup and shutdown
- Request received and request completed
- Authentication failures and authorization denials
- Unhandled exceptions
- Third-party API failures or timeouts
- Database connection or query failures
- Background job start, retry, and failure

---

## Part 4 - Metrics That Matter

Metrics should help notice behavior changes early.

### Good starter metrics

- Request count
- Error rate
- Average or p95 response time
- CPU or memory usage if available
- Number of failed background jobs
- Database query duration for key operations

### A simple practical baseline

For most projects in this course, a dashboard with these three metrics is already valuable:

1. Request volume over time
2. Error rate over time
3. Response latency over time

If the project has authentication or payments domain metrics are essential too:

- Login failures
- Orders created
- Checkout success rate
- File upload failures

### Metrics to teach with care

We do not need a complex SRE program this week. Keep the focus on metrics that are actionable.

For example:

- "CPU is 53%" is interesting, but not always useful by itself
- "Checkout error rate doubled after deploy" is immediately actionable

---

## Part 5 - Traces and Why They Matter

Tracing becomes important when one user request passes through multiple steps.

```text
Browser action
   -> frontend request
   -> API route
   -> service layer
   -> database query
   -> external provider
   -> response
```

If the request is slow, logs alone may not explain where time was spent. A trace can show which span took too long.

### When tracing helps most

- A page is slow, but only for some users
- One endpoint makes multiple downstream calls
- The system uses queues, workers, or separate services
- The database is not always the slow part

For a simple monolith, tracing is still useful as a concept even if implementation remains basic.

---

## Part 6 - Dashboards, Alerts, and Operational Thinking

Collecting telemetry is not enough. Teams need a way to look at it and react.

### Dashboards

Dashboards answer the question:

> **What is the current state of the system?**

A good beginner dashboard might contain:

- Requests per minute
- Error rate
- p95 latency
- Top exception types
- Recent deployment marker if available

### Alerts

Alerts answer a different question:

> **When should someone care right now?**

Example beginner alerts:

- Error rate above 5% for 5 minutes
- p95 latency above 1500 ms for 10 minutes
- Service unavailable or health endpoint failing
- More than 5 unhandled exceptions in 10 minutes

### Common alerting mistakes

- Alerting on everything and creating noise
- Alerting on raw infrastructure values with no business context
- Defining alerts but never testing them
- Not documenting who should check the alert and what to do next

---

## Part 7 - Managed Cloud Solutions

Managed observability platforms are usually the fastest way to get value in production.

| Managed option | Strengths | Good fit |
|----------------|-----------|----------|
| **Azure Application Insights / Azure Monitor** | Easy integration, metrics, logs, traces, dashboards, alerts | Projects already deployed on Azure |
| **AWS CloudWatch + X-Ray** | Strong AWS integration for logs, metrics, tracing, alarms | Projects deployed on AWS |
| **Google Cloud Logging + Monitoring + Trace** | Unified visibility inside GCP | Projects deployed on GCP |
| **Datadog** | Fast setup, rich dashboards, broad integrations | Teams willing to use a commercial SaaS platform |
| **New Relic** | Good all-in-one experience for APM and monitoring | Teams wanting a hosted observability suite |

### Advantages of managed tools

- Faster setup
- Less operational overhead
- Built-in alerts and dashboards
- Easier authentication and permissions inside the same cloud

### Trade-offs

- Cost can grow with usage
- Vendor-specific workflows can limit portability
- Some teams cannot use them because of budget, access, or platform constraints

---

## Part 8 - What If You Do Not Have Application Insights or X-Ray?

This is an important real-world question.

If your app is not on Azure or AWS, or you do not want to depend on a managed service, you still have solid options.

> **Important principle:** Observability is a capability, not a vendor product.

You can build that capability with open-source tools and open standards.

### Open standards first: OpenTelemetry

**OpenTelemetry** is one of the most important pieces of modern observability.

It is not a dashboard by itself. It is a vendor-neutral standard and ecosystem for collecting telemetry.

Use it to:

- Instrument applications for logs, metrics, and traces
- Export data to managed tools or open-source backends
- Avoid hard-coding your app to one provider too early

OpenTelemetry is a strong candidate.

---

## Part 9 - Open-Source Alternatives

Here are practical open-source options to use.

| Need | Open-source option | What it does |
|------|--------------------|--------------|
| **Metrics collection** | **Prometheus** | Scrapes and stores time-series metrics |
| **Dashboards and visualization** | **Grafana** | Builds dashboards and visualizes many data sources |
| **Log aggregation** | **Loki** | Stores and queries logs with Grafana integration |
| **Distributed tracing** | **Jaeger** or **Tempo** | Stores and explores traces |
| **All-in-one observability** | **SigNoz** | Combines metrics, traces, logs, and dashboards around OpenTelemetry |
| **Error tracking** | **Sentry self-hosted** | Tracks exceptions and application errors |
| **Search-heavy log analytics** | **OpenSearch** or **ELK/Elastic stack** | Indexes and searches logs and events |
| **Log management UI** | **Graylog** | Log collection, search, and dashboards |

### Common open-source stacks

#### Stack A - Lightweight metrics and dashboards

- Prometheus
- Grafana

Good when the goal is to track service health, request counts, and latency.

#### Stack B - Logs plus dashboards

- Loki
- Grafana
- Promtail or another log shipper

Good when log search with a smaller operational footprint than a large search cluster.

#### Stack C - Traces plus metrics plus logs

- OpenTelemetry instrumentation
- Grafana
- Prometheus
- Loki
- Tempo

Good when the course wants a modern, vendor-neutral observability story.

#### Stack D - One platform approach

- SigNoz

Good when one open-source tool with a more integrated experience.

### How to choose

| Situation | Good default |
|----------|---------------|
| You want the simplest cloud-native option | Application Insights, CloudWatch, or another managed platform |
| You want portability across providers | OpenTelemetry + a backend of your choice |
| You want dashboards and metrics first | Prometheus + Grafana |
| You mostly need logs and debugging | Loki + Grafana or Graylog |
| You want an integrated open-source platform | SigNoz |
| You mainly need exception grouping and stack traces | Sentry |

---

## Part 10 - A Practical Mental Model for the Course

We do not need to master every observability product. We do need to understand how the pieces fit together.

```text
Application code
   -> instrumentation
   -> telemetry collected
   -> storage/backend
   -> dashboards and alerts
   -> human response
```

### Recommended teaching baseline

For this course, a strong baseline is:

1. Structured logs in the application
2. At least three useful metrics
3. Error tracking for exceptions
4. One dashboard that combines health signals
5. One or two tested alerts

Tracing can be introduced as an extension when the project complexity justifies it.

---

## Part 11 - Suggested Exercise and Deliverable

### Suggested exercise

Instrument a small application with:

- Structured request logs
- Error tracking
- Three basic metrics
- One dashboard
- One alert rule

Choose one of these paths:

- **Managed path:** Application Insights, CloudWatch, or another hosted observability service
- **Open-source path:** OpenTelemetry with Prometheus, Grafana, Loki, Tempo, Jaeger, SigNoz, or Sentry self-hosted

### Deliverable expectations

A strong Week 16 deliverable should include:

- A dashboard with at least three meaningful metrics
- Evidence of structured logging or error tracking
- At least one alert defined and explained
- A short rationale for the chosen stack
- A note describing what open-source or managed alternative could replace the chosen tools

### Reflection questions

1. Which signal was most useful for understanding failures in your project: logs, metrics, or traces?
2. What would you miss if you only had logs and no metrics?
3. If you had to move away from your current cloud provider tomorrow, which parts of your observability setup would still transfer?
4. Which alerts are actually actionable for your project, and which would only create noise?

---

## Recommended Reading and Exploration

- [OpenTelemetry Docs](https://opentelemetry.io/docs/) - The main vendor-neutral observability standard
- [Prometheus Overview](https://prometheus.io/docs/introduction/overview/) - A strong starting point for metrics
- [Grafana Docs](https://grafana.com/docs/) - Dashboards, visualization, and observability tooling
- [Loki Overview](https://grafana.com/oss/loki/) - Log aggregation designed to work well with Grafana
- [Jaeger Docs](https://www.jaegertracing.io/docs/) - Distributed tracing concepts and tooling
- [SigNoz Docs](https://signoz.io/docs/) - An open-source observability platform built around OpenTelemetry
- [Sentry Docs](https://docs.sentry.io/) - Error tracking and performance monitoring