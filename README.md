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


