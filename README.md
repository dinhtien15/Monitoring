# Monitoring
### Install Prometheus

Download and Install Prometheus
```
wget https://github.com/prometheus/prometheus/releases/download/v2.37.6/prometheus-2.37.6.linux-amd64.tar.gz
tar xvfz prometheus-*.tar.gz
rm prometheus-*.tar.gz
sudo mkdir /etc/prometheus /var/lib/prometheus
cd prometheus-2.37.6.linux-amd64
sudo mv prometheus promtool /usr/local/bin/
sudo mv prometheus.yml /etc/prometheus/prometheus.yml
sudo mv consoles/ console_libraries/ /etc/prometheus/
prometheus --version
```

Configure Prometheus as a Service
```
sudo useradd -rs /bin/false prometheus
sudo chown -R prometheus: /etc/prometheus /var/lib/prometheus
```
Create a prometheus.service 
```
sudo vi /etc/systemd/system/prometheus.service

[Unit]
Description=Prometheus
Wants=network-online.target
After=network-online.target
[Service]
User=prometheus
Group=prometheus
Type=simple
Restart=on-failure
RestartSec=5s
ExecStart=/usr/local/bin/prometheus \
--config.file /etc/prometheus/prometheus.yml \
--storage.tsdb.path /var/lib/prometheus/ \
--web.console.templates=/etc/prometheus/consoles \
--web.console.libraries=/etc/prometheus/console_libraries \
--web.listen-address=0.0.0.0:9090 \
--web.enable-lifecycle \
--log.level=info
[Install]
WantedBy=multi-user.target
```
Reload the systemctl daemon
```
sudo systemctl daemon-reload
sudo systemctl enable prometheus
sudo systemctl start prometheus
sudo systemctl status prometheus
```

Access the Prometheus web interface and dashboard at http://local_ip_addr:9090


### Install NodeExporter
```
wget https://github.com/prometheus/node_exporter/releases/download/v1.5.0/node_exporter-1.5.0.linux-amd64.tar.gz
tar xvfz node_exporter-*.tar.gz
sudo mv node_exporter-1.5.0.linux-amd64/node_exporter /usr/local/bin
rm -r node_exporter-1.5.0.linux-amd64*
node_exporter
sudo useradd -rs /bin/false node_exporter
```

sudo vi /etc/systemd/system/node_exporter.service
```
[Unit]
Description=Node Exporter
Wants=network-online.target
After=network-online.target
[Service]
User=node_exporter
Group=node_exporter
Type=simple
Restart=on-failure
RestartSec=5s
ExecStart=/usr/local/bin/node_exporter
[Install]
WantedBy=multi-user.target
```

Reload
```
sudo systemctl enable node_exporter
sudo systemctl daemon-reload
sudo systemctl start node_exporter
sudo systemctl status node_exporter
```

### Configure Prometheus to Monitor Client Nodes

sudo vi /etc/prometheus/prometheus.yml
```
- job_name: "remote_collector"
scrape_interval: 10s
static_configs:
- targets: ["remote_addr:9100"]
```

Restart prometheus service
```
sudo systemctl restart prometheus
```


### Install and Deploy the Grafana Server

Install some required utilities using apt
```
sudo apt-get install -y apt-transport-https software-properties-common
```
Import the Grafana GPG key
```
sudo wget -q -O /usr/share/keyrings/grafana.key https://apt.grafana.com/gpg.key
```
Add the Grafana “stable releases” repository
```
echo "deb [signed-by=/usr/share/keyrings/grafana.key] https://apt.grafana.com stable main" | sudo tee -a /etc/apt/sources.list.d/grafana.list
```
Update the packages in the repository, including the new Grafana package
```
sudo apt-get update
```
Install the open-source version of Grafana
```
sudo apt-get install grafana
```
Reload the systemctl daemon
```
sudo systemctl daemon-reload
sudo systemctl enable grafana-server.service
sudo systemctl start grafana-server
sudo systemctl status grafana-server
```

### Integrate Grafana and Prometheus
```
- Using a web browser, visit port 3000 of the monitoring server. For example, enter http://local_ip_addr:3000, replacing local_ip_addr with the actual IP address. Grafana displays the login page. Use the user name admin and the default password password. Change the password to a more secure value when prompted to do so.
- After a successful password change, Grafana displays the Grafana Dashboard.
- Go to Connection → Datasources
- At the next display, click the Add data source button.
- Choose Prometheus as the data source.
- For a local Prometheus source, as described in this guide, set the URL to http://localhost:9090. Most of the other settings can remain at the default values. However, a non-default Timeout value can be added here.
- When satisfied with the settings, select the Save & test button at the bottom of the screen.
- If all settings are correct, Grafana confirms the Data source is working
```


### Install alert manager
Create a user and group for Alertmanager
```
sudo useradd -M -r -s /bin/false alertmanager
```

Download and install the Alertmanager binaries
```
wget https://github.com/prometheus/alertmanager/releases/download/v0.20.0/alertmanager-0.20.0.linux-amd64.tar.gz
tar xvfz alertmanager-0.20.0.linux-amd64.tar.gz
sudo cp alertmanager-0.20.0.linux-amd64/alertmanager /usr/local/bin/
sudo chown alertmanager:alertmanager /usr/local/bin/alertmanager
sudo mkdir -p /etc/alertmanager
sudo cp alertmanager-0.20.0.linux-amd64/alertmanager.yml /etc/alertmanager
sudo chown -R alertmanager:alertmanager /etc/alertmanager
sudo mkdir -p /var/lib/alertmanager
sudo chown alertmanager:alertmanager /var/lib/alertmanager
```

Create a systemd unit for Alertmanager
```
sudo vi /etc/systemd/system/alertmanager.service
[Unit]
Description=Prometheus Alertmanager
Wants=network-online.target
After=network-online.target

[Service]
User=alertmanager
Group=alertmanager
Type=simple
ExecStart=/usr/local/bin/alertmanager
--config.file /etc/alertmanager/alertmanager.yml
--storage.path /var/lib/alertmanager/

[Install]
WantedBy=multi-user.target
```

Start and enable the alertmanager service
```
sudo systemctl enable alertmanager
sudo systemctl start alertmanager
```

Verify the service is running and you can reach it
```
sudo systemctl status alertmanager
curl localhost:9093
```
