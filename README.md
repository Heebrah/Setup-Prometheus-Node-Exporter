## **Prometheus – Monitoring & Alerting System**

**Overview**
Prometheus is an open-source **monitoring and alerting toolkit** designed for reliability and scalability, especially in **cloud-native** and **containerized environments** like Kubernetes. It collects time-series metrics, stores them efficiently, and allows users to query and visualize them.

---

### **Key Features**

1. **Time-Series Data Storage**

   * Stores metrics as **time-stamped** data points.
   * Organizes data using labels for easy filtering and grouping.

2. **Pull-Based Model**

   * Prometheus **scrapes** (pulls) metrics from targets at defined intervals using HTTP endpoints (`/metrics`).

3. **Multi-Dimensional Data Model**

   * Metrics are identified by a name and a set of **key-value pairs** (labels).

4. **Powerful Query Language (PromQL)**

   * Enables complex queries, aggregations, and filtering for real-time analysis.

5. **Service Discovery**

   * Automatically detects targets via integrations with Kubernetes, Consul, EC2, etc.

6. **Built-in Alerting**

   * Integrates with **Alertmanager** to send notifications to email, Slack, PagerDuty, etc.

7. **Visualization**

   * Works with tools like **Grafana** for dashboards.

---

### **Prometheus Architecture**

* **Prometheus Server** – Core component that scrapes metrics and stores them.
* **Exporters** – Tools that expose metrics (e.g., Node Exporter for system metrics, MySQL Exporter for DB).
* **Pushgateway** – Accepts pushed metrics from short-lived jobs.
* **Alertmanager** – Handles alert delivery and deduplication.
* **Client Libraries** – Used to instrument applications directly.

---

### **Advantages**

* Easy to set up and run standalone.
* No external dependencies for core functionality.
* Rich ecosystem of exporters.
* Highly efficient in handling millions of time series.

### **Limitations**

* Not designed for long-term storage without external systems.
* No built-in user authentication/authorization.
* Primarily pull-based (push support is limited to Pushgateway).

---

✅ **In short:** Prometheus is the backbone of modern **observability**, helping teams detect performance issues, track system health, and automate alerts in real time.

---
Here’s a **step-by-step guide** to set up **Prometheus with Node Exporter** so you can monitor server metrics like CPU, RAM, and disk usage.

---

## **1. Install Prometheus**

**On Linux (Ubuntu/Debian):**

```bash
# Create a Prometheus user
sudo useradd --no-create-home --shell /bin/false prometheus

# Download Prometheus
wget https://github.com/prometheus/prometheus/releases/latest/download/prometheus-*.linux-amd64.tar.gz

# Extract and move files
tar xvf prometheus-*.linux-amd64.tar.gz
sudo mv prometheus-*/prometheus /usr/local/bin/
sudo mv prometheus-*/promtool /usr/local/bin/

# Move config files
sudo mkdir /etc/prometheus
sudo mv prometheus-*/prometheus.yml /etc/prometheus/

# Create data directory
sudo mkdir /var/lib/prometheus

# Set ownership
sudo chown prometheus:prometheus /usr/local/bin/prometheus /usr/local/bin/promtool
sudo chown -R prometheus:prometheus /etc/prometheus /var/lib/prometheus
```

---

## **2. Install Node Exporter**

```bash
# Create user
sudo useradd --no-create-home --shell /bin/false node_exporter

# Download Node Exporter
wget https://github.com/prometheus/node_exporter/releases/latest/download/node_exporter-*.linux-amd64.tar.gz

# Extract and move binary
tar xvf node_exporter-*.linux-amd64.tar.gz
sudo mv node_exporter-*/node_exporter /usr/local/bin/

# Set ownership
sudo chown node_exporter:node_exporter /usr/local/bin/node_exporter
```

---

## **3. Create Systemd Services**

**Prometheus Service**

```bash
sudo nano /etc/systemd/system/prometheus.service
```

Paste:

```ini
[Unit]
Description=Prometheus Monitoring
Wants=network-online.target
After=network-online.target

[Service]
User=prometheus
ExecStart=/usr/local/bin/prometheus \
  --config.file=/etc/prometheus/prometheus.yml \
  --storage.tsdb.path=/var/lib/prometheus/

[Install]
WantedBy=default.target
```

**Node Exporter Service**

```bash
sudo nano /etc/systemd/system/node_exporter.service
```

Paste:

```ini
[Unit]
Description=Node Exporter
After=network.target

[Service]
User=node_exporter
ExecStart=/usr/local/bin/node_exporter

[Install]
WantedBy=default.target
```

---

## **4. Configure Prometheus to Scrape Node Exporter**

Edit:

```bash
sudo nano /etc/prometheus/prometheus.yml
```

Add under `scrape_configs`:

```yaml
scrape_configs:
  - job_name: 'node_exporter'
    static_configs:
      - targets: ['localhost:9100']
```

---

## **5. Start and Enable Services**

```bash
sudo systemctl daemon-reload
sudo systemctl enable prometheus
sudo systemctl start prometheus

sudo systemctl enable node_exporter
sudo systemctl start node_exporter
```

---

## **6. Access Prometheus & Node Exporter**

* **Prometheus UI:** `http://<server-ip>:9090`
* **Node Exporter Metrics:** `http://<server-ip>:9100/metrics`

---

## **7. (Optional) Add Grafana for Visualization**

```bash
# Install Grafana
sudo apt-get install -y apt-transport-https software-properties-common
sudo apt-get install grafana -y
sudo systemctl enable --now grafana-server
```

Then connect Grafana to Prometheus (`http://<server-ip>:9090`) and import **Node Exporter dashboard**.


## Project monitoring linux server using prometheus  node exporter
1. ### Install Prometheus
i) update packages 
```
sudo apt update && upgrade
```
![caption](/img/1.update-upgrade.jpg)

ii) create system user
```
sudo groupadd --system prometheus
sudo useradd -s /sbin/nologin --system -g prometheus prometheus
```
![caption](/img/2.user-group.jpg)

iii) create directory where prometheus will be stored
```
sudo mkdir /etc/prometheus
sudo mkdir /var/lib/prometheus
```
![caption](/img/3.mkdir-prometues.jpg)

iv) download prometheus
```
wget https://github.com/prometheus/prometheus/releases/download/v2.43.0/prometheus-2.43.0.linux-amd64.tar.gz
```
![caption](/img/4.install-prometheus.jpg)

v) extract prometheus
```
tar vxf prometheus*.tar.gz
```
![caption](/img/5.extract-prometheus.jpg)

vi)  navigate to the newly extracted Prometheus directory
```
cd ~/prometheus*/
```
vii) move the configuration files and set their ownership so that Prometheus can access them. To do this, run the following commands
```
sudo mv consoles /etc/prometheus
sudo mv console_libraries /etc/prometheus
sudo mv prometheus.yml /etc/prometheus
```
![caption](/img/9.copy.jpg)

viii) change ownership
```
sudo chown prometheus:prometheus /etc/prometheus
sudo chown -R prometheus:prometheus /etc/prometheus/consoles
sudo chown -R prometheus:prometheus /etc/prometheus/console_libraries
sudo chown -R prometheus:prometheus /var/lib/prometheus
```

ix) move prometheus to /usr/local/bin
```
sudo mv prometheus /usr/local/bin
```
![caption](/img/17move-prome.jpg)
x) checking the prometheus default files
```
sudo nano /etc/prometheus/prometheus.yml
```
xi) Create Prometheus Systemd Service

```
sudo nano /etc/systemd/system/prometheus.service
```
![caption](/img/12.create-prometheus-service.jpg)

xii) content to add to the prometheus.service

```
[Unit]
Description=Prometheus
Wants=network-online.target
After=network-online.target

[Service]
User=prometheus
Group=prometheus
Type=simple
ExecStart=/usr/local/bin/prometheus \
    --config.file /etc/prometheus/prometheus.yml \
    --storage.tsdb.path /var/lib/prometheus/ \
    --web.console.templates=/etc/prometheus/consoles \
    --web.console.libraries=/etc/prometheus/console_libraries

[Install]
WantedBy=multi-user.target
```
![caption](/img/19.cat-prom-serv.jpg)

xiv) Reloading the daemon
```
sudo systemctl daemon-reload
```
![caption](/img/14.reload-daemon.jpg)
xv) Start the prometheus service
```
sudo systemctl enable prometheus
sudo systemctl start prometheus
```
![caption](/img/15.enable-prometheus.jpg)

xvi) check the status
```
sudo systemctl status prometheus
```
![caption](/img/21.check-status.jpg)

xvii) allowing access on firewall
```
sudo ufw allow 9090/tcp
```
![caption](/img/22.allowing-port.jpg)

xviii) Can now open the prometheus on webpage with
```
http://localhost:9090
```
but if it's hosted on a public server like Ec2 instance this can be open using
public ip with port 9090

![caption](/img/24.memory-check.jpg)

2. ### Install prometheus node export
i) install the node export with this command 
```
wget https://github.com/prometheus/node_exporter/releases/download/v1.3.1/node_exporter-1.3.1.linux-amd64.tar.gz

```
ii) extract the file
```
node_exporter-1.3.1.linux-amd64.tar.gz
```
![caption](/img/30.install-prometheus-node-module.jpg)
iii) go into the node exporter
```
cd node_exporter-1.3.1.linux-amd64
```
iv) copy node_exporter into bin
```
sudo cp node_exporter /usr/local/bin
```

v) cd out of the folder

```
cd ..
```
![caption](/img/34.mv-from-bin.jpg)
vi) 

```
rm -rf ./node_exporter-1.3.1.linux-amd64
```
![caption](/img/35.rf-command.jpg)

vii) add user to the prometheus node exporter
```
sudo useradd --no-create-home --shell /bin/false node_exporter

sudo chown node_exporter:node_exporter /usr/local/bin/node_exporter

```
![caption](/img/36.useradd-chown.jpg)
viii) Go to exporter file
```
sudo nano /etc/systemd/system/node_exporter
```
![caption](/img/37.create-file.jpg)

ix) Reloading the systemctl and also starting the node_exporter
```
sudo systemctl daemon-reload
sudo systemctl enable node_exporter
sudo systemctl start node_exporter
```
![caption](/img/38.%20reload-enable-start.jpg)


x) node_export default run on port 9100 so we can give access to allow the firewall unrestriction
![caption](/img/39.allow-9100.jpg)

xi) Now going back to prometheus file to allow prometheus access the node_exporter
```
sudo nano /etc/prometheus/prometheus.yml
```
![caption](/img/40.sudo-prometheus-file.jpg)

xii) Then add the config of node_exporter into the prometheus.yml file
```
- job name: "node_exporter"
  static_configs:
  - targets: ["localhost:9100"]
```
![caption](/img/41.add-node-module.jpg)

3. ### Testing And Validation
- using the querry below
```
node_memory_MemAvailability_bytes
```

* **What it is:**
  An estimate of memory available for starting new applications without swapping.
* **Type:** Gauge.
* **Use case:**
  Detect low memory conditions before swap usage spikes.
* **Example alert rule:**

  ```promql
  node_memory_MemAvailable_bytes < 500 * 1024 * 1024
  ```

  *(alerts if available memory < 500 MB)*


![caption](/img/42.available-memory.jpg)

- Using the querry below
```
node_filesystem_avail_bytes
```

* **What it is:**
  Number of **bytes available** to non-root users on a filesystem.
* **Type:** Gauge (can go up or down).
* **Use case:**
  Check disk free space, alert if nearing low thresholds.
* **Example alert rule:**

  ```promql
  node_filesystem_avail_bytes{mountpoint="/"} < 5 * 1024 * 1024 * 1024
  ```

  *(alerts if less than 5 GB free on `/`)*

![caption](/img/43.node-file-system.jpg)

- Using the querry below

```
node_network_receive_byte_total
```
![caption](/img/44.node-network.jpg)
* **What it is:**
  Cumulative count of **bytes** received over all network interfaces since the system started.
* **Type:** Counter (keeps increasing until reboot or interface reset).
* **Use case:**
  Monitor incoming network traffic, detect spikes, or track total bandwidth usage.
* **Example query (rate over 5 minutes):**

  ```promql
  rate(node_network_receive_bytes_total[5m])
  ```



- Create a basic time-series graphs using Prometheus expressions



To create basic time-series graphs using Prometheus expressions (PromQL) as shown in your image, follow these steps in Prometheus’s built-in web UI:

---

### 🔧 **Step-by-Step Instructions**

#### **1. Open Prometheus Web UI**

* Go to your Prometheus server in a browser:

  ```
  http://<your-prometheus-host>:9090
  ```

  (Usually it's `http://localhost:9090` if you're running it locally.)

---

#### **2. Enter the PromQL Query**

* In the "Expression" input box at the top, paste:

  ```
  rate(node_cpu_seconds_total[5m])
  ```

  This expression calculates the per-second average CPU usage rate over the last 5 minutes.

---

#### **3. Click "Execute"**

* Click the **"Execute"** button.
* You'll see the raw time series data as a table.

---

#### **4. Switch to "Graph" View**

* Click the **"Graph"** tab right above the result panel.
* This displays the data as a time-series graph.

---

#### **5. (Optional) Adjust the Time Range**

* In the top-right, adjust the time range (e.g., 1h, 6h) and refresh rate as needed.

---
```
rate(node_cpu_seconds_total[5m])
```
![caption](/img/45.cpu-time-alert.jpg)

-  Set up alert rules for critical metrics like high CPU usage or low disk space

### ✅ Sample Prometheus Alert Rules
Create a file to define the rule
```
sudo nano /etc/prometheus/alert.rules.yml
```
`alert.rules.yml`:

![caption](/img/46.create-rule-file.jpg)

Put the following into a file `alert.rule.yml file`
```yaml
groups:
  - name: critical-alerts
    rules:

      # High CPU Usage Alert
      - alert: HighCPUUsage
        expr: 100 - (avg by (instance) (rate(node_cpu_seconds_total{mode="idle"}[5m])) * 100) > 80
        for: 2m
        labels:
          severity: critical
        annotations:
          summary: "High CPU usage on instance {{ $labels.instance }}"
          description: "CPU usage is above 80% for more than 2 minutes."

      # Low Disk Space Alert
      - alert: LowDiskSpace
        expr: (node_filesystem_avail_bytes{fstype!~"tmpfs|aufs|overlay"} / node_filesystem_size_bytes{fstype!~"tmpfs|aufs|overlay"}) * 100 < 15
        for: 2m
        labels:
          severity: warning
        annotations:
          summary: "Low disk space on {{ $labels.instance }} ({{ $labels.mountpoint }})"
          description: "Disk space is below 15% available on {{ $labels.mountpoint }}."
```
![caption](/img/47.alert-rules-disk-high-cpu.jpg)

### 🔧 Then Add to Your `prometheus.yml`
```
sudo nano /etc/prometheus/prometheus.yml
```
![caption](/img/40.sudo-prometheus-file.jpg)


```yaml
rule_files:
  - "alert.rules.yml"
```
![caption](/img/48.adding-alert-rule-inside-prometheus-file.jpg)
---
---

### 🔄 Reload Prometheus

* If you’re using the web UI:
  POST to:

  ```
  http://localhost:9090/-/reload
  ```

* Or simply restart the Prometheus server.

---

### 🔍 View Alerts

Go to:

```
http://localhost:9090/alerts
```

You'll see **pending**, **firing**, or **inactive** alerts based on the current metrics.
![caption](/img/49.verifying-alert.jpg)

