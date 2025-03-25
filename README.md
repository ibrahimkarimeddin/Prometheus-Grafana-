# Monitoring Stack Installation Guide

### Prerequisites
- A Linux system (these instructions are for Linux, specifically Ubuntu/Debian)
- `wget` installed
- `sudo` access


  
##Prometheus Installation Guide

## Step 1: Create Prometheus User
Create a system user for Prometheus with no login shell:

```bash
sudo useradd --no-create-home --shell /usr/sbin/nologin prometheus
```

## Step 2: Download Prometheus
Download the latest Prometheus release (in this case, v2.41.0):

```bash
wget https://github.com/prometheus/prometheus/releases/download/v2.41.0/prometheus-2.41.0.linux-amd64.tar.gz
```

## Step 3: Extract and Prepare Prometheus
Extract the downloaded archive and navigate to the directory:

```bash
tar xvf prometheus-2.41.0.linux-amd64.tar.gz
cd prometheus-2.41.0.linux-amd64
```

## Step 4: Install Prometheus Binaries
Copy the Prometheus binaries to system directories:

```bash
sudo cp prometheus /usr/local/bin/
sudo cp promtool /usr/local/bin/
```

## Step 5: Set Permissions
Set the correct ownership for the Prometheus binaries:

```bash
sudo chown prometheus:prometheus /usr/local/bin/prometheus
sudo chown prometheus:prometheus /usr/local/bin/promtool
```

## Step 6: Create Prometheus Directories
Create necessary directories for configuration and data:

```bash
sudo mkdir /etc/prometheus
sudo mkdir /var/lib/prometheus
sudo chown prometheus:prometheus /etc/prometheus
sudo chown prometheus:prometheus /var/lib/prometheus
```

## Step 7: Configure Prometheus
Create and edit the Prometheus configuration file:

```bash
sudo nano /etc/prometheus/prometheus.yml
```

Add the following configuration:

```yaml
global:
  scrape_interval: 15s
  evaluation_interval: 15s
scrape_configs:
  # Monitor Prometheus itself
  - job_name: 'prometheus'
    static_configs:
      - targets: ['localhost:9090']
```

## Step 8: Create Systemd Service
Create a systemd service file for Prometheus:

```bash
sudo nano /etc/systemd/system/prometheus.service
```

Add the following content:

```ini
[Unit]
Description=Prometheus
After=network.target

[Service]
User=prometheus
Group=prometheus
Type=simple
ExecStart=/usr/local/bin/prometheus \
    --config.file=/etc/prometheus/prometheus.yml \
    --storage.tsdb.path=/var/lib/prometheus/ \
    --web.console.templates=/etc/prometheus/consoles \
    --web.listen-address=0.0.0.0:9090  \
    --web.console.libraries=/etc/prometheus/console_libraries \
    --log.level=info
Restart=always

[Install]
WantedBy=multi-user.target
```

## Step 9: Start and Enable Prometheus
Enable and start the Prometheus service:

```bash
sudo systemctl enable prometheus
sudo systemctl start prometheus
```

## Verification
Prometheus should now be running and accessible at `http://localhost:9090`

## Additional Notes
- The default Prometheus port is 9090
- Ensure your firewall allows traffic on port 9090 if accessing remotely
- This configuration monitors Prometheus itself
- For monitoring other services, you'll need to modify the `prometheus.yml` configuration

## Troubleshooting
- Check service status: `sudo systemctl status prometheus`
- View logs: `sudo journalctl -u prometheus`




## Grafana Installation Guide

### Step 1: Prepare System for Grafana Installation
Update system packages and install required dependencies:

```bash
sudo apt-get update
sudo apt-get install -y apt-transport-https software-properties-common wget
```

### Step 2: Add Grafana GPG Key
Import the Grafana GPG key for package verification:

```bash
wget -q -O - https://packages.grafana.com/gpg.key | sudo apt-key add -
```

### Step 3: Add Grafana APT Repository
Configure the official Grafana repository for the latest stable version:

```bash
echo "deb https://packages.grafana.com/oss/deb stable main" | sudo tee /etc/apt/sources.list.d/grafana.list
```

### Step 4: Install Grafana
Update package lists and install Grafana:

```bash
sudo apt-get update
sudo apt-get install grafana
```

### Step 5: Start and Enable Grafana Service
Enable Grafana to start on boot and start the service:

```bash
sudo systemctl enable grafana-server
sudo systemctl start grafana-server
```

## Accessing Grafana

### Default Credentials
- **Username**: `admin`
- **Password**: `admin`
- **Port**: `3000`

**Note**: You will be prompted to change the default password on first login.

### Access Methods
- Local access: `http://localhost:3000`
- Remote access: `http://your_server_ip:3000`

## Troubleshooting
- Check Grafana service status: 
  ```bash
  sudo systemctl status grafana-server
  ```
- View Grafana logs: 
  ```bash
  sudo journalctl -u grafana-server
  ```





## Node Exporter Installation Guide

### Step 1: Download Node Exporter
Download the latest Node Exporter release (v1.5.0):

```bash
wget https://github.com/prometheus/node_exporter/releases/download/v1.5.0/node_exporter-1.5.0.linux-amd64.tar.gz
```

### Step 2: Extract Node Exporter
Extract the downloaded archive:

```bash
tar xvf node_exporter-1.5.0.linux-amd64.tar.gz
```

### Step 3: Install Node Exporter Binary
Copy the Node Exporter binary to the system bin directory:

```bash
sudo cp node_exporter-1.5.0.linux-amd64/node_exporter /usr/local/bin/
```

### Step 4: Create System User
Create a system user for Node Exporter:

```bash
sudo useradd -rs /bin/false node_exporter
```

### Step 5: Create Systemd Service
Create a systemd service file for Node Exporter:

```bash
sudo nano /etc/systemd/system/node_exporter.service
```

Add the following content:

```ini
[Unit]
Description=Node Exporter
After=network.target

[Service]
User=node_exporter
Group=node_exporter
Type=simple
ExecStart=/usr/local/bin/node_exporter

[Install]
WantedBy=multi-user.target
```

### Step 6: Start and Enable Node Exporter
Enable and start the Node Exporter service:

```bash
sudo systemctl enable node_exporter
sudo systemctl start node_exporter
```

## Configuring Prometheus to Scrape Node Exporter

### Step 7: Update Prometheus Configuration
Edit the Prometheus configuration file:

```bash
sudo nano /etc/prometheus/prometheus.yml
```

Update the configuration to include Node Exporter:

```yaml
global:
  scrape_interval: 15s
  evaluation_interval: 15s

scrape_configs:
  # Monitor Prometheus itself
  - job_name: 'prometheus'
    static_configs:
      - targets: ['localhost:9090']
  
  # Monitor Node Exporter
  - job_name: 'node_exporter'
    static_configs:
      - targets: ['localhost:9100']
```

### Multiple Target Configuration
You can add multiple targets by expanding the `static_configs` section:

```yaml
- job_name: 'node_exporter'
  static_configs:
    - targets: ['localhost:9100', '192.168.1.100:9100', '192.168.1.101:9100']
```

### Step 8: Validate Prometheus Configuration
Check the configuration file for syntax errors:

```bash
promtool check config /etc/prometheus/prometheus.yml
```

### Step 9: Restart Prometheus
Apply the new configuration:

```bash
sudo systemctl restart prometheus
```

## Node Exporter Details

### Port Information
- Node Exporter runs on port **9100** by default

### Accessing Node Exporter Metrics
- Local metrics endpoint: `http://localhost:9100/metrics`
- Remote metrics endpoint: `http://server_ip:9100/metrics`

## Troubleshooting
- Check Node Exporter service status:
  ```bash
  sudo systemctl status node_exporter
  ```
- View Node Exporter logs:
  ```bash
  sudo journalctl -u node_exporter
  ```

## Monitoring Stack Integration Guide

### Integrating Grafana with Prometheus and Node Exporter

### Step 1: Access Grafana Web Interface
1. Open your web browser
2. Navigate to `http://localhost:3000`
3. Log in with your admin credentials

### Step 2: Add Prometheus as a Data Source

#### Navigate to Data Sources
1. Click on the gear icon (⚙️) in the left sidebar
2. Select "Connections"
3. Click on "Data Sources"
4. Click "Add new data source"

#### Configure Prometheus Data Source
1. Select "Prometheus" from the list of available data sources
2. In the HTTP section, set the URL to: `http://localhost:9090`
   - This is the default Prometheus server URL
3. Click "Save & Test"
   - You should see a green "Successfully queried the Prometheus API" message

### Step 3: Import Pre-built Node Exporter Dashboard

#### Find and Import Dashboard
1. Go to the "Dashboards" section in the left sidebar
2. Click "New" and then "Import"

#### Popular Node Exporter Dashboards
Here are some recommended pre-built dashboards:

1. **Node Exporter Full** (Dashboard ID: 1860)
   - Comprehensive system metrics dashboard
   - Detailed view of CPU, memory, disk, and network usage

2. **Node Exporter for Prometheus Dashboard** (Dashboard ID: 11074)
   - Clean, informative system monitoring dashboard

#### Import Process
1. Enter the Dashboard ID (e.g., 1860)
2. Click "Load"
3. Select Prometheus as the data source
4. Click "Import"

