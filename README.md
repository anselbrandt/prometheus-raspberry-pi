# prometheus-raspberry-pi

### Install Prometheus

```
sudo useradd \
    --system \
    --no-create-home \
    --shell /bin/false prometheus
```

```
wget https://github.com/prometheus/prometheus/releases/download/v2.53.1/prometheus-2.53.1.linux-arm64.tar.gz

tar -xvf prometheus-2.53.1.linux-arm64.tar.gz

sudo mkdir -p /data /etc/prometheus

cd prometheus-2.53.1.linux-arm64

sudo mv prometheus promtool /usr/local/bin/

sudo mv consoles/ console_libraries/ /etc/prometheus/

sudo mv prometheus.yml /etc/prometheus/prometheus.yml

sudo chown -R prometheus:prometheus /etc/prometheus/ /data/

sudo nano /etc/systemd/system/prometheus.service
```

prometheus.service

```
[Unit]
Description=Prometheus
Wants=network-online.target
After=network-online.target

StartLimitIntervalSec=500
StartLimitBurst=5

[Service]
User=prometheus
Group=prometheus
Type=simple
Restart=on-failure
RestartSec=5s
ExecStart=/usr/local/bin/prometheus \
  --config.file=/etc/prometheus/prometheus.yml \
  --storage.tsdb.path=/data \
  --web.console.templates=/etc/prometheus/consoles \
  --web.console.libraries=/etc/prometheus/console_libraries \
  --web.listen-address=0.0.0.0:9090 \
  --web.enable-lifecycle

[Install]
WantedBy=multi-user.target
```

```
sudo systemctl enable prometheus

sudo systemctl start prometheus

sudo systemctl status prometheus
```

### Prometheus Node Exporter

```
sudo useradd \
    --system \
    --no-create-home \
    --shell /bin/false node_exporter
```

```
wget https://github.com/prometheus/node_exporter/releases/download/v1.8.2/node_exporter-1.8.2.linux-arm64.tar.gz

tar -xvf node_exporter-1.8.2.linux-arm64.tar.gz

sudo mv \
  node_exporter-1.8.2.linux-arm64/node_exporter \
  /usr/local/bin/

node_exporter --version
```

```
sudo nano /etc/systemd/system/node_exporter.service
```

```
[Unit]
Description=Node Exporter
Wants=network-online.target
After=network-online.target

StartLimitIntervalSec=500
StartLimitBurst=5

[Service]
User=node_exporter
Group=node_exporter
Type=simple
Restart=on-failure
RestartSec=5s
ExecStart=/usr/local/bin/node_exporter \
    --collector.logind

[Install]
WantedBy=multi-user.target
```

```
sudo systemctl enable node_exporter

sudo systemctl start node_exporter

sudo systemctl status node_exporter

curl http://localhost:9100/metrics

curl -s localhost:9100/metrics | grep cpu
```

# Grafana

`sudo apt-get install -y apt-transport-https software-properties-common`

`wget -q -O - https://packages.grafana.com/gpg.key | sudo apt-key add -`

```
echo "deb https://packages.grafana.com/oss/deb stable main" | sudo tee -a /etc/apt/sources.list.d/grafana.list
```

```
sudo apt-get update
sudo apt-get -y install grafana
```

`sudo systemctl enable grafana-server`

`sudo systemctl start grafana-server`

`sudo systemctl status grafana-server`

# Reload Prometheus Config

`curl -X POST -u admin:admin http://localhost:9090/-/reload`

# Optional - Add all data sources

`sudo nano /etc/grafana/provisioning/datasources/datasources.yaml`

```
apiVersion: 1

datasources:
  - name: Prometheus
    type: prometheus
    url: http://localhost:9090
    isDefault: true
```

`sudo systemctl restart grafana-server`

# Update Prometheus Targets

`sudo nano /etc/prometheus/prometheus.yml`

```
  - job_name: "prometheus"
    static_configs:
      - targets: ["localhost:9090"]
  - job_name: "node"
    static_configs:
      - targets: ["localhost:9100"]

```

Check config is valid:

`promtool check config /etc/prometheus/prometheus.yml`

Use POST request to update config:

`curl -X POST http://localhost:9090/-/reload`
