# 👁️ Enterprise Observability: ELK Stack

![Elasticsearch](https://img.shields.io/badge/Elasticsearch-005571?style=for-the-badge&logo=elasticsearch&logoColor=white)
![Logstash](https://img.shields.io/badge/Logstash-005571?style=for-the-badge&logo=logstash&logoColor=white)
![Kibana](https://img.shields.io/badge/Kibana-005571?style=for-the-badge&logo=kibana&logoColor=white)
![Docker](https://img.shields.io/badge/Docker-Ready-blue?style=for-the-badge&logo=docker)

## 📌 Project Overview
A production-grade Centralized Logging and Observability platform utilizing the **ELK Stack** (Elasticsearch, Logstash, Kibana) deployed via Docker Compose. This architecture is designed to ingest, parse, and visualize logs from microservices and distributed systems in real-time.

## 📂 Repository Structure
```text
.
├── logstash/
│   └── logstash.conf      # TCP input and JSON parsing configurations
├── assets/
│   ├── discover.png
│   ├── docker-ps.png
│   ├── docker-stats.png
│   ├── gelf-integration.png
│   ├── kibana-index.png
│   └── project-diagram.png
├── docker-compose.yml     # Orchestrates the ELK stack & virtual networks
└── README.md              # Project documentation


```

## 🏗️ System Architecture

The logging pipeline is engineered to decouple log ingestion from storage. Logstash acts as the primary buffer and parser, transforming raw JSON logs before persisting them into Elasticsearch for rapid querying via Kibana.
<p align="center">
  <img src="./assets/project-diagram.png" width="100%">
  <br>
  <em><b>Figure 1:</b> System Architecture Diagram </em>
</p>

## 💡 Key Features & Technical Decisions

- **Kernel Optimization:** Requires host machine `vm.max_map_count` modification to prevent Elasticsearch out-of-memory crashes, ensuring enterprise-level stability and performance.
- **Resource Constraints:** Elasticsearch JVM options (`ES_JAVA_OPTS`) are strictly configured to a 512MB limit (`-Xms512m -Xmx512m`) to prevent container resource exhaustion and maintain host efficiency.
- **Strict Network Isolation:** The entire observability stack communicates over a custom internal Docker bridge network (`observability_net`), securing database traffic and preventing unauthorized external access.
- **Decoupled Pipeline Architecture:** Utilizing Logstash as an intermediary buffer enables advanced log filtering, JSON parsing, and data mutation before the payload reaches the Elasticsearch storage tier.
- **Real-Time Data Indexing:** Dynamically generates daily index patterns (`enterprise-logs-%{+YYYY.MM.DD}`) for efficient log rotation and lifecycle management.

## 📸 Project Showcase (Proof of Concept)

### 1. Real-Time Log Ingestion & JSON Parsing
The core of the observability platform. Logstash successfully parses incoming JSON payloads, breaking them down into searchable fields (`app`, `engineer`, `level`), which are instantly indexed and visualized in Kibana.

<p align="center">
  <img src="assets/discover.png" width="100%" style="border-radius: 8px; border: 1px solid #ddd; box-shadow: 0 4px 8px rgba(0,0,0,0.1);">
</p>

### 2. Automated Index Discovery
Proof of seamless integration between the storage tier (Elasticsearch) and the visualization tier (Kibana), successfully matching dynamically generated daily indices.

<p align="center">
  <img src="assets/kibana-index.png" width="100%" style="border-radius: 8px; border: 1px solid #ddd; box-shadow: 0 4px 8px rgba(0,0,0,0.1);">
</p>

### 3. Infrastructure Health & Resource Constraints
Executing `docker compose ps` and `docker stats` demonstrates the stack running reliably. Notice that the Elasticsearch container adheres strictly to the defined JVM heap size constraints (`-Xms512m -Xmx512m`), preventing host machine resource exhaustion.

<p align="center">
  <img src="assets/docker-ps.png" width="48%" style="border-radius: 8px; border: 1px solid #ddd;">
  <img src="assets/docker-stats.png" width="48%" style="border-radius: 8px; border: 1px solid #ddd;">
</p>

🚀 Getting Started
1. Host Machine Preparation (Crucial)

Increase the virtual memory mapping limits on your Linux host:
sudo sysctl -w vm.max_map_count=262144

2. Build and Deploy

Launch the observability stack:
docker compose up -d

3. Verify Ingestion via Netcat

Simulate a microservice sending a JSON log to the pipeline:
echo '{"app": "Test-Service", "message": "The Enterprise Observability Stack is fully operational!", "level": "INFO"}' | nc localhost 5000

4. Access the Dashboard

Navigate to http://localhost:5601, create a Data View for enterprise-logs-*, and start exploring your data in the Discover tab.


## 🔗 Advanced Integration: Docker GELF Logging

This section demonstrates how to seamlessly route logs from an external Multi-Tier architecture (Nginx & Node.js) directly into the ELK stack without modifying any application source code. This is achieved using Docker's native `gelf` logging driver.

### Step 1: Prepare Logstash for GELF
Configure Logstash to listen for incoming Docker logs via UDP. Update `logstash.conf` to include the `gelf` input plugin:

```text
input {
  gelf {
    port => 12201
  }
}

```
Ensure port 12201:12201/udp is exposed in the ELK docker-compose.yml.

### Step 2: Configure the External Application
In the `docker-compose.yml` of your target application (e.g., the Nginx Load Balancer), override the default json-file logger by adding the `gelf` logging driver:

```yaml
  nginx:
    image: nginx:latest
    # ... other configurations ...
    logging:
      driver: gelf
      options:
        gelf-address: "udp://127.0.0.1:12201"
        tag: "nginx-loadbalancer"
```

### Step 3: Traffic Simulation & Stress Testing
To verify the pipeline, a stress test was executed against the load balancer using a simple Bash loop to generate 100 concurrent requests:

```bash
for i in {1..100}; do curl -s http://localhost > /dev/null; echo "Request $i sent"; done

```

### Step 4: Centralized Visualization
The architecture successfully streamed, parsed, and visualized the traffic in real-time within Kibana. The dashboard below illustrates the traffic spike and the successfully parsed GELF payload.

<p align="center">
  <img src="assets/gelf-integration.png" width="100%" style="border-radius: 8px; border: 1px solid #ddd; box-shadow: 0 4px 8px rgba(0,0,0,0.1);">
</p>


**Architected by:** Ahmed Mohamed Abbas Bahij

[![](https://img.shields.io/badge/LinkedIn-Connect-blue?style=for-the-badge&logo=linkedin)](https://www.linkedin.com/in/ahmedabbas99)

Cloud Infrastructure & DevOps Engineer 
