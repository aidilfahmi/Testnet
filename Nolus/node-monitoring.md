# Installation NODE BINARY
## Port Configuration
Here is my Node Port
```
RPC = 55657
gRPC = 55090
Prometheus PORT from config.toml : 55660 
```
## Installing Required Dependencies
```
sudo apt-get update
sudo apt-get upgrade -y
sudo apt install docker
sudo apt install docker-compose
sudo apt install jq -y
sudo apt install python3-pip -y
sudo pip install yq
```

## COSMOS EXPORTER
### Install Binary
```
wget https://github.com/solarlabsteam/cosmos-exporter/releases/download/v0.2.2/cosmos-exporter_0.2.2_Linux_x86_64.tar.gz
tar xvfz cosmos-exporter*
sudo cp ./cosmos-exporter /usr/bin
rm cosmos-exporter* -rf
```
### Create cosmos_exporter user
```
sudo useradd -rs /bin/false cosmos_exporter
```
### Create Service
```
sudo tee <<EOF >/dev/null /etc/systemd/system/cosmos-exporter.service
[Unit]
Description=Cosmos Exporter
After=network-online.target
[Service]
User=cosmos_exporter
Group=cosmos_exporter
TimeoutStartSec=0
CPUWeight=95
IOWeight=95
ExecStart=cosmos-exporter --denom unls --denom-coefficient 1000000 --bech-prefix nolus --tendermint-rpc http://localhost:55657 --node localhost:55090
Restart=always
RestartSec=2
LimitNOFILE=800000
KillSignal=SIGTERM
[Install]
WantedBy=multi-user.target
EOF
```
## NODE EXPORTER
### Install Binary
```
wget https://github.com/prometheus/node_exporter/releases/download/v1.3.1/node_exporter-1.3.1.linux-amd64.tar.gz
tar xvfz node_exporter-*.*-amd64.tar.gz
sudo mv node_exporter-*.*-amd64/node_exporter /usr/local/bin/
rm node_exporter-* -rf
```
### Create node_exporter user
```
sudo useradd -rs /bin/false node_exporter
```
### Create Service
```
sudo tee <<EOF >/dev/null /etc/systemd/system/node_exporter.service
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
EOF
```
### Installing and Start Services
```
sudo systemctl daemon-reload
sudo systemctl enable cosmos-exporter
sudo systemctl start cosmos-exporter
sudo systemctl enable node_exporter
sudo systemctl start node_exporter
```

## PROMETHEUS
```
custom port web 7090
```
### Installing Binary
```
wget https://github.com/prometheus/prometheus/releases/download/v2.42.0/prometheus-2.42.0.linux-amd64.tar.gz
tar xvfz prometheus-*.*-amd64.tar.gz
sudo mv prometheus-*.*-amd64/prometheus /usr/local/bin/
rm prometheus-* -rf
```
### Create Prometheus.yml
```
sudo mkdir -p /etc/prometheus
```
```diff
- Edit Port, Nolus address and Validator Address with your own configurations.

sudo tee /etc/prometheus/prometheus.yml > /dev/null <<EOF
global:
  scrape_interval: 15s
  scrape_timeout: 10s
  evaluation_interval: 15s
alerting:
  alertmanagers:
    - follow_redirects: true
      scheme: http
      timeout: 10s
      api_version: v2
      static_configs:
        - targets:
            - alertmanager:9093
rule_files:
  - /etc/prometheus/alerts/alert.rules
scrape_configs:
  - job_name: prometheus
    metrics_path: /metrics
    static_configs:
      - targets:
          - localhost:7090 # Prometheus will be running
  - job_name: cosmos
    metrics_path: /metrics
    static_configs:
      - targets:
          - 127.0.0.1:55660 # Port Prometheus from Nolus
        labels: {}
  - job_name: node
    metrics_path: /metrics
    static_configs:
      - targets:
          - 127.0.0.1:9100 # Port node-exporter
        labels:
          instance: nolus-rila
  - job_name: validators
    metrics_path: /metrics/validators
    static_configs:
      - targets:
          - 127.0.0.1:9300 # Port cosmos-exporter
        labels: {}
  - job_name: validator
    metrics_path: /metrics/validator
    relabel_configs:
      - source_labels:
          - address
        target_label: __param_address
    static_configs:
      - targets:
          - 127.0.0.1:9300 # Port cosmos-exporter
        labels:
          address: # validator-address
  - job_name: wallet
    metrics_path: /metrics/wallet
    relabel_configs:
      - source_labels:
          - address
        target_label: __param_address
    static_configs:
      - targets:
          - 127.0.0.1:9300  # Port cosmos-exporter
        labels:
          address: # nolus-address
EOF
```
### Create Prometheus Service

```diff
-This configuration using custom port 7090.
-If you are using default port (9090), please delete this part: --web.listen-address="0.0.0.0:7090"
sudo tee /etc/systemd/system/prometheus.service > /dev/null <<EOF
[Unit]
Description=Prometheus
After=network.target
[Service]
User=root
Type=simple
ExecStart=/usr/local/bin/prometheus --config.file="/etc/prometheus/prometheus.yml" --web.listen-address="0.0.0.0:7090"
[Install]
WantedBy=multi-user.target
EOF
```
### Enable and start Service
```
sudo systemctl daemon-reload
sudo systemctl enable prometheus
sudo systemctl start prometheus
```
## GRAFANA
### Default Installation GRAFANA
```
sudo docker run -d --name=grafana -p 3000:3000 grafana/grafana-enterprise
```
### Installation GRAFANA with PUBLIC DASHBOARD ENABLE
```
sudo docker run -d -p 3000:3000   -e "GF_FEATURE_TOGGLES_ENABLE=publicDashboards"   grafana/grafana-enterprise
```
# SETTING GRAFANA DASHBOARD
to be continue...
