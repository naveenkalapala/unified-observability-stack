# 📊 Observability: Metrics, Logs & Traces (MELT)

Modern observability is built on three core data signals — **Metrics**, **Logs**, and **Traces**. These work together to provide deep visibility into the health, behavior, and performance of software systems, especially in microservice and cloud-native environments.

---

## 📈 1. Metrics

### ✅ What are Metrics?

Metrics are **numerical values** that represent measurements over time. They are **aggregated, structured, and queryable**, making them efficient for real-time monitoring.

### 📌 Examples:
- CPU usage: `70%`
- Requests per second: `1500 RPS`
- Error rate: `1.2%`
- Memory usage: `3.5 GB`

### 🧰 Use Cases:
- Alerting on thresholds (e.g., high latency)
- Monitoring trends (e.g., traffic spikes)
- Performance dashboards
- Scaling decisions (e.g., auto-scaling pods)

### 🔧 Tools:
- Prometheus
- CloudWatch
- Datadog Metrics
- OpenTelemetry SDK

---

## 📝 2. Logs

### ✅ What are Logs?

Logs are **timestamped text records** that capture discrete events. They provide **context-rich details** about what happened inside a system.

### 📌 Examples:
- `"GET /api/products - 500 Internal Server Error"`
- `"User login failed for user123"`
- `"Kafka consumer disconnected"`

### 🧰 Use Cases:
- Debugging application failures
- Forensic analysis and audits
- Filtering and searching events
- Correlating with metrics/traces for RCA

### 🔧 Tools:
- ELK Stack (Elasticsearch, Logstash, Kibana)
- Loki
- Fluentd / Fluent Bit
- Cloud-native logs (e.g., GCP Logging, AWS CloudWatch Logs)

---

## 🔍 3. Traces

### ✅ What are Traces?

Traces track the **end-to-end flow of a request** across services. They consist of **spans**, which represent individual operations within a trace.

### 📌 Examples:
- HTTP request trace from frontend → API → DB
- A POST request taking 300ms due to DB latency
- Tracing a Kafka message from producer to consumer

### 🧰 Use Cases:
- Pinpointing performance bottlenecks
- Visualizing request paths
- Understanding service-to-service dependencies
- Root cause analysis for slow transactions

### 🔧 Tools:
- OpenTelemetry
- Jaeger / Zipkin / Tempo
- AWS X-Ray
- Datadog APM

---

## 🧠 Summary Table

| Aspect   | Metrics                     | Logs                               | Traces                            |
|----------|-----------------------------|-------------------------------------|-----------------------------------|
| Type     | Numeric measurements        | Textual event records               | Distributed request flows         |
| Format   | Structured key/value        | Unstructured/structured text        | Spans and traces (structured)     |
| Volume   | Low (aggregated)            | Medium to High                      | High (but sampled)                |
| Retention| Long                        | Medium                              | Short                             |
| Best For | Alerting, Dashboards        | Debugging, Error Investigation      | Performance, Dependency Mapping   |

---

## 🔗 Combined Use

In a real incident:
- **Metrics** alert you that latency is high.
- **Logs** show an exception in the payment service.
- **Traces** help pinpoint that the DB query inside the payment service is taking 900ms.

This is why all **three pillars are essential** for modern observability.

---

## 📚 References

- [OpenTelemetry Docs](https://opentelemetry.io/docs/)
- [Prometheus](https://prometheus.io/)
- [Jaeger Tracing](https://www.jaegertracing.io/)
- [Grafana Loki](https://grafana.com/oss/loki/)
