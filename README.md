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
