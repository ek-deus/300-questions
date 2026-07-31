Installing node_exporter
Steps to install node_exporter

Add user for node_exporter

sudo useradd --no-create-home --shell /bin/false node_exporter
Download node_exporter

cd
wget https://github.com/prometheus/node_exporter/releases/download/v1.12.1/node_exporter-1.12.1.darwin-amd64.tar.gz
Extract node_exporter

tar xvf node_exporter-0.18.1.linux-amd64.tar.gz
Copy node_exporter to /opt

sudo mv node_exporter-0.18.1.linux-amd64 /opt/node_exporter
sudo chown -R node_exporter:node_exporter /opt/node_exporter
Create service file for systemd

sudo nano /etc/systemd/system/node_exporter.service
Fillin as follows:

[Unit]
Description=Node Exporter
Wants=network-online.target
After=network-online.target

[Service]
User=node_exporter
Group=node_exporter
Type=simple
ExecStart=/opt/node_exporter/node_exporter --collector.systemd

[Install]
WantedBy=multi-user.target
Start the service with systemd and verify it runs

sudo systemctl daemon-reload
sudo systemctl start node_exporter && sudo journalctl -f --unit node_exporter
On the prometheus server, dont' forget to add the static config for the collection of data!
