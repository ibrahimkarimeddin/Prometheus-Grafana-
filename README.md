# Prometheus Installation Guide

## Prerequisites
- A Linux system (these instructions are for Linux, specifically Ubuntu/Debian)
- `wget` installed
- `sudo` access

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




# Grafana Installation Guide

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

## Additional Configuration
- Configure data sources
- Create dashboards
- Set up alerts

## Security Recommendations
- Change default password immediately
- Configure authentication methods
- Use HTTPS for remote access
- Set up firewall rules to restrict access to port 3000

## Common Next Steps
1. Connect Grafana to Prometheus as a data source
2. Import or create monitoring dashboards
3. Configure additional data sources as needed
