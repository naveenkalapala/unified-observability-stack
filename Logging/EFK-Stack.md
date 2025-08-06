
### Logging is one of the three core pillars of observability, alongside metrics and tracing. In simple terms:

### What is Logging?
- Logging is the practice of recording discrete events and messages produced by applications and infrastructure. Logs provide detailed context and are essential for debugging and root cause analysis.

---

### Why Logging Matters in Observability

- Observability is about understanding what's going on inside your system, especially when things go wrong. Logs help answer questions like:

- What happened?

- When did it happen?

- Why did it happen?

- What was the system doing at that time?

Logs provide context that metrics and traces might miss.

---
## 1. Introduction to EFK Stack
- **Overview**: The EFK Stack is a powerful logging and observability framework that integrates Elasticsearch, Fluent Bit, and Kibana to collect, store, search, and visualize log data.
- **Significance in Observability**:
  - Centralized logging for distributed systems.
  - Scalable storage and real-time analysis of large log volumes.
  - Intuitive visualization for actionable insights.
  - Supports troubleshooting, monitoring, and compliance auditing.
- **Use Cases**: Effective for microservices, cloud-native applications, and DevOps environments requiring real-time log analysis.
---
## 2. Architecture of the EFK Stack


---
### Component Roles
- **Elasticsearch**:
  - **Functionality**: A distributed, RESTful search and analytics engine built on Apache.
  - **Purpose**:
    - Stores log data in a scalable, indexed format.
    - Enables full-text search and complex queries for rapid log retrieval.
    - Supports horizontal scaling for high availability and performance.
  - **Key Features**:
    - JSON-based document storage.
    - Near real-time search capabilities.
    - Sharding and replication for fault tolerance.
- **Fluent Bit**:
  - **Functionality**: A lightweight, high-performance log processor and forwarder.
  - **Purpose**:
    - Collects logs from various sources (e.g., applications, containers, or system logs).
    - Processes and filters logs before forwarding them to Elasticsearch.
    - Supports multiple input/output plugins for flexibility.
  - **Key Features**:
    - Low memory and CPU footprint (~450 KB).
    - Extensible plugin architecture (e.g., Kubernetes, Docker, Syslog).
    - Log parsing, filtering, and enrichment capabilities.
- **Kibana**:
  - **Functionality**: A web-based visualization and analytics platform.
  - **Purpose**:
    - Visualizes log data stored in Elasticsearch through dashboards, charts, and graphs.
    - Enables users to query logs and create real-time monitoring dashboards.
    - Facilitates log exploration and correlation for root cause analysis.
  - **Key Features**:
    - Interactive dashboards and visualizations (e.g., histograms, heatmaps).
    - Advanced search and filtering tools.
    - Integration with Elasticsearch for seamless data access.

---

### Why are we creating the SVC Account here?

- We're creating the svc account here to allow the EBS CSI driver to interact with AWS resources, specifically for managing EBS volumes in the Kubernetes cluster.

### Step-by-Step Setup of EFK Stack

### 1) Create IAM Role for Service Account
```
eksctl create iamserviceaccount \
    --name ebs-csi-controller-sa \
    --namespace kube-system \
    --cluster observability \
    --role-name AmazonEKS_EBS_CSI_DriverRole \
    --role-only \
    --attach-policy-arn arn:aws:iam::aws:policy/service-role/AmazonEBSCSIDriverPolicy \
    --approve
```
- This command creates an IAM role for the EBS CSI controller.
- IAM role allows EBS CSI controller to interact with AWS resources, specifically for managing EBS volumes in the Kubernetes cluster.
- We will attach the Role with service account

### 2) Retrieve IAM Role ARN
```
ARN=$(aws iam get-role --role-name AmazonEKS_EBS_CSI_DriverRole --query 'Role.Arn' --output text)
```
- Command retrieves the ARN of the IAM role created for the EBS CSI controller service account.

### 3) Deploy EBS CSI Driver
```
eksctl create addon --cluster observability --name aws-ebs-csi-driver --version latest \
    --service-account-role-arn $ARN --force
```
- Above command deploys the AWS EBS CSI driver as an addon to your Kubernetes cluster.
- It uses the previously created IAM service account role to allow the driver to manage EBS volumes securely.

### 4) Create Namespace for Logging
```
kubectl create namespace logging
```

### 5) Install Elasticsearch on K8s

```
helm repo add elastic https://helm.elastic.co
```
```
helm install elasticsearch \
 --set replicas=1 \
 --set volumeClaimTemplate.storageClassName=gp2 \
 --set persistence.labels.enabled=true elastic/elasticsearch -n logging
```

- Installs Elasticsearch in the `logging` namespace.
- It sets the number of replicas, specifies the storage class, and enables persistence labels to ensure
data is stored on persistent volumes.

### 6) Retrieve Elasticsearch Username & Password
- **Username**:
```
kubectl get secrets --namespace=logging elasticsearch-master-credentials -ojsonpath='{.data.username}' | base64 -d
 ```
 - **Password**:
```
kubectl get secrets --namespace=logging elasticsearch-master-credentials -ojsonpath='{.data.password}' | base64 -d
```
- Retrieves the password for the Elasticsearch cluster's master credentials from the Kubernetes secret.
- The password is base64 encoded, so it needs to be decoded before use.
- **Note**: Please write down the password for future reference
---

### 7) Install Kibana
```
helm install kibana --set service.type=LoadBalancer elastic/kibana -n logging
```
- Kibana provides a user-friendly interface for exploring and visualizing data stored in Elasticsearch.
- It is exposed as a LoadBalancer service, making it accessible from outside the cluster.
---

### 8) Install Fluentbit with Custom Values/Configurations
- **Note**: Please update the `HTTP_Passwd` field in the `fluentbit-values.yml` file with the password retrieved earlier in step 6: 
```
helm repo add fluent https://fluent.github.io/helm-charts
```
```
helm install fluent-bit fluent/fluent-bit -f fluentbit-values.yaml -n logging
```

## ✅ Conclusion
- We have successfully installed the EFK stack in our Kubernetes cluster, which includes Elasticsearch for storing logs, Fluentbit for collecting and forwarding logs, and Kibana for visualizing logs.
- To verify the setup, access the Kibana dashboard by entering the `LoadBalancer DNS name followed by :5601 in your browser.
    - `http://LOAD_BALANCER_DNS_NAME:5601`
- Use the username and password retrieved in step 6 to log in.
- Once logged in, create a new data view in Kibana and explore the logs collected from your Kubernetes cluster.

---
## 3. Logging Process with EFK

### Step-by-Step Process
1. **Log Collection (Fluent Bit)**:
   - Fluent Bit collects logs from various sources, such as application logs, system logs, or container logs (e.g., Docker, Kubernetes).
   - Uses input plugins to ingest logs (e.g., `tail` for file-based logs, `systemd` for system logs).
   - Parses and enriches logs with metadata (e.g., timestamps, Kubernetes pod names).
   - Filters irrelevant data or transforms log formats for consistency.
2. **Log Forwarding (Fluent Bit)**:
   - Fluent Bit forwards processed logs to Elasticsearch via the HTTP output plugin.
   - Buffers logs to handle network issues or Elasticsearch downtime.
   - Supports secure transmission with TLS/SSL.
3. **Log Storage and Indexing (Elasticsearch)**:
   - Elasticsearch receives logs as JSON documents and stores them in indices.
   - Indexes logs based on time or application for efficient retrieval.
   - Enables full-text search and aggregations for log analysis.
4. **Log Visualization and Analysis (Kibana)**:
   - Kibana queries Elasticsearch indices to retrieve log data.
   - Users create dashboards to visualize log patterns, errors, or performance metrics.
   - Supports real-time monitoring and alerting based on log thresholds.
---
### Key Features Enhancing Observability
- **Elasticsearch**:
  - Scalable indexing for handling petabytes of log data.
  - Advanced querying (e.g., regex, range queries) for pinpointing issues.
  - Machine learning capabilities for anomaly detection.
- **Fluent Bit**:
  - Lightweight deployment, ideal for resource-constrained environments like IoT or Kubernetes.
  - Multiline log parsing for complex log formats (e.g., stack traces).
  - Dynamic routing to multiple destinations (e.g., Elasticsearch, S3).
- **Kibana**:
  - Customizable dashboards for different stakeholders (e.g., developers, SREs).
  - Integration with alerting tools for proactive issue resolution.
  - Log correlation across distributed systems for end-to-end tracing.
---
### Example Scenarios
- **Microservices Debugging**:
  - A microservices-based e-commerce platform generates logs across multiple services. Fluent Bit collects logs from Kubernetes pods, enriches them with pod metadata, and forwards them to Elasticsearch. Developers use Kibana to trace errors across services, identifying a failing payment API.
- **Real-Time Monitoring**:
  - A cloud-native application logs performance metrics. Fluent Bit aggregates logs, Elasticsearch indexes them, and Kibana displays latency trends in real-time, enabling SREs to detect and resolve bottlenecks.
- **Compliance Auditing**:
  - A financial application uses EFK to store access logs for regulatory compliance. Kibana dashboards provide auditors with detailed reports on user activity.
---
## 5. Conclusion
- The EFK Stack is a cornerstone of modern observability, offering a scalable, flexible, and efficient solution for logging. By combining Fluent Bit’s lightweight log collection, Elasticsearch’s robust storage and search, and Kibana’s intuitive visualization, EFK enables organizations to monitor, troubleshoot, and optimize complex systems. Its ability to handle diverse log sources and provide real-time insights makes it indispensable for cloud-native, microservices, and DevOps environments.
---
