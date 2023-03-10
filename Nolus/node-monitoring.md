# Installation
## Port Configuration
Here is my Node Port
```
RPC = 55657
gRPC = 55090
```
## Installing Required Dependencies
```
sudo apt-get update
sudo apt-get upgrade -y
sudo apt install jq -y
sudo apt install python3-pip -y
sudo pip install yq
```

## Install cosmos-exporter
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
## Install Node Exporter
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
