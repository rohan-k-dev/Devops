# Exercise 6: Observing Delivery Metrics with Prometheus and Grafana

## Objective
I created a small operations-monitoring setup for a quick-commerce delivery service. Python produces the metrics, Prometheus collects them, Grafana displays them, and Jenkins coordinates the workflow.

## Files in the Monitoring Setup
- `delivery_metrics.py`: A Python script that uses the `prometheus_client` library to simulate metrics (`total_deliveries`, `pending_deliveries`, `on_the_way_deliveries`, `average_delivery_time`) and expose them on an HTTP endpoint (`/metrics`) running on port 8000. 
  *(Note: Edited to simulate `pending_deliveries` between 50-100 to trigger the HighPendingDeliveries alert threshold of 10).*
- `prometheus.yml`: Configuration file for Prometheus, defining scrape targets (`localhost:9090` and `172.17.0.1:8000` which points to the host's Docker bridge IP).
- `alert_rules.yml`: Contains Prometheus alert rules for `HighPendingDeliveries` (warning) and `HighAverageDeliveryTime` (critical).
- `Jenkinsfile`: Defines the CI/CD pipeline to build the Docker image, run the Python application, and spin up Prometheus and Grafana containers.

## Setup and Expected Results

### 1. Start the Delivery Metrics Service
Run the Python app:
```
python3 delivery_metrics.py
```
**Output:**
```
[INFO] Starting the HTTP server on port 8000...
[INFO] HTTP server started. Simulating deliveries...
[DEBUG] Total deliveries: 135
[DEBUG] Pending deliveries: 85
[DEBUG] On-the-way deliveries: 12
[DEBUG] Average delivery time: 27.50 seconds
...
```

Access metrics at `http://localhost:8000/metrics`:
```
curl http://localhost:8000/metrics
# HELP pending_deliveries Number of pending deliveries
# TYPE pending_deliveries gauge
pending_deliveries 85.0
...
```

### 2. Configure Prometheus
Run Prometheus container using the config files:
```
docker run -d --name prometheus -p 9090:9090 --network=host \
  -v ./prometheus.yml:/etc/prometheus/prometheus.yml \
  -v ./alert_rules.yml:/etc/prometheus/alert_rules.yml \
  prom/prometheus
```
Access Prometheus at `http://localhost:9090`.
- The Targets page will show `delivery_service` as UP.
- The Alerts page will show the `HighPendingDeliveries` alert in a `FIRING` state since the simulated pending deliveries (50-100) consistently exceed the threshold of 10.

### 3. Open Grafana
Run Grafana container:
```
docker run -d --name grafana -p 3000:3000 grafana/grafana
```
Access Grafana at `http://localhost:3000` (admin/admin).
- A Prometheus data source is added pointing to `http://172.17.0.1:9090`.
- A dashboard is created to visualize:
  - Total Deliveries
  - Pending Deliveries
  - On-the-Way Deliveries
  - Average Delivery Time

### 4. Automate the Stack with Jenkins
When run, the `Jenkinsfile` automates the entire process: checking Docker installation, building the application image, and deploying the Python app, Prometheus, and Grafana containers.
