# LogSentinel 🛡️

**LogSentinel** is a lightweight, containerized Security Information and Event Management (SIEM) lab designed for security monitoring, log parsing, and real-time authentication attack detection.

---

## 🏗️ Architecture Overview

```
                      +-----------------------------+
                      |   attacks/                  |
                      |   attack_simulator.py       |
                      +--------------+--------------+
                                     |
                         HTTP POST   | (Port 5044)
                                     v
                      +-----------------------------+
                      |   Logstash Pipeline         |
                      |   (Grok Parsing & Filtering)|
                      +--------------+--------------+
                                     |
                                     v
                      +-----------------------------+
                      |   OpenSearch Cluster        |
                      |   (Index: security-logs-*)  |
                      |   (Port 9200)               |
                      +--------------+--------------+
                                     |
                                     v
                      +-----------------------------+
                      |   OpenSearch Dashboards     |
                      |   (SIEM Visualization UI)   |
                      |   (Port 5601)               |
                      +-----------------------------+
```

1. **Attack Simulator (`attacks/attack_simulator.py`)**: Generates real-time synthetic authentication telemetry, simulating both normal internal logins and brute-force attacks from malicious IPs.
2. **Logstash (`logstash/logstash.conf`)**: Ingests raw HTTP log events on port `5044`, parses them using Grok expressions, extracts structured fields (`timestamp`, `log_level`, `src_ip`, `action`, `user`, `failed_attempts`), and streams them to OpenSearch.
3. **OpenSearch & Dashboards (`docker-compose.yml`)**:
   - `opensearch-siem`: Fast, scalable search and analytics engine indexing security logs into `security-logs-%{+YYYY.MM.dd}`.
   - `siem-dashboards`: Web UI for querying, visualizing security telemetry, and building threat monitoring dashboards.

---

## 📁 Repository Structure

```
LogSentinel/
├── attacks/
│   └── attack_simulator.py     # Real-time brute force & normal login traffic generator
├── logstash/
│   └── logstash.conf           # HTTP input, Grok parsing filter, and OpenSearch output
├── docker-compose.yml          # Container configuration for OpenSearch and Dashboards
├── .gitignore                  # Git ignore rules for Python, logs, and environments
└── README.md                   # Project documentation
```

---

## 🚀 Getting Started

### Prerequisites

- [Docker](https://www.docker.com/) and [Docker Compose](https://docs.docker.com/compose/)
- [Logstash](https://www.elastic.co/logstash) (or run Logstash in Docker)
- [Python 3.8+](https://www.python.org/)
- `requests` library: `pip install requests`

---

### Step 1: Start OpenSearch and OpenSearch Dashboards

Launch the OpenSearch SIEM cluster:

```bash
docker-compose up -d
```

Verify that the containers are healthy:

```bash
docker ps
```

- **OpenSearch API**: [http://localhost:9200](http://localhost:9200)
- **OpenSearch Dashboards**: [http://localhost:5601](http://localhost:5601)

---

### Step 2: Start Logstash

Run Logstash using the provided configuration:

```bash
logstash -f logstash/logstash.conf
```

Or using Docker:

```bash
docker run --rm -it \
  --network host \
  -v "$(pwd)/logstash/logstash.conf:/usr/share/logstash/pipeline/logstash.conf" \
  docker.elastic.co/logstash/logstash:8.11.0
```

Logstash will listen for incoming HTTP POST events on port `5044`.

---

### Step 3: Run the Attack Simulator

Install the Python dependency:

```bash
pip install requests
```

Execute the simulator:

```bash
python attacks/attack_simulator.py
```

The script will begin streaming live events:
- **Normal Telemetry (85%)**: `INFO` logins by user `alice` from internal subnet IPs (`192.168.1.10`, `192.168.1.15`, `10.0.0.50`).
- **Anomalous Telemetry (15%)**: `WARN` brute-force authentication failures on `admin` from external malicious IP `45.33.32.156` (45–200 attempts).

---

### Step 4: Explore and Analyze in OpenSearch Dashboards

1. Open your browser and navigate to **[http://localhost:5601](http://localhost:5601)**.
2. Go to **Stack Management** > **Index Patterns** (or **Index Management**).
3. Create an index pattern: `security-logs-*` with timestamp field `timestamp`.
4. Head to **Discover** to monitor incoming telemetry in real-time.
5. Create visualizations and alerts to detect brute-force spikes (`failed_attempts > 40` or traffic from `45.33.32.156`).

---

## 🔒 Security Note

For laboratory/demonstration simplicity, demo security configuration plugins are disabled in `docker-compose.yml` (`DISABLE_SECURITY_PLUGIN=true`). For production deployments, enable SSL/TLS certificates and role-based access control (RBAC).

---

## 📄 License

This project is licensed under the MIT License - see the LICENSE file for details.
