
### Grafana and Prometheus Setup through docker

#### Step 1: Prepare the Environment

```bash
mkdir monitoring
cd monitoring
touch docker-compose.yml
```

#### Step 2: Create `.env` File for Credentials

```bash
cat > .env <<EOL
# Grafana credentials
GRAFANA_USERNAME=aeroadmin
GRAFANA_PASSWORD=X6mhD3r2a10P
EOL
```

#### Step 3: Create `docker-compose.yml` File

```yaml
services:
  prometheus:
    image: prom/prometheus:latest
    container_name: prometheus
    ports:
      - "4000:9090"
    volumes:
      - ./prometheus.yml:/etc/prometheus/prometheus.yml
    networks:
      - internal
    logging:
      driver: "json-file"
      options:
        max-size: "10m"
        max-file: "3"
  grafana:
    image: grafana/grafana:latest
    container_name: grafana
    ports:
      - "3900:3000"
    environment:
      - GF_SECURITY_ADMIN_USER=${GRAFANA_USERNAME}
      - GF_SECURITY_ADMIN_PASSWORD=${GRAFANA_PASSWORD}
      - GF_SERVER_HTTP_PORT=3000
    volumes:
      - grafana-data:/var/lib/grafana
    networks:
      - internal
    logging:
      driver: "json-file"
      options:
        max-size: "10m"
        max-file: "3"
volumes:
  grafana-data:
networks:
  internal:
    driver: bridge
```

#### Step 4: Create `prometheus.yml` File

```yaml
global:
  scrape_interval: 15s
scrape_configs:
  - job_name: "node_exporter"
    static_configs:
      - targets:
          - "<node-exporter-private-ip-1>:9100"
          - "<node-exporter-private-ip-2>:9100"
          - "<node-exporter-private-ip-3>:9100"
```

#### Step 5: Deploy Grafana and Prometheus

```bash
docker compose up -d
```

### Node Exporter on Apache NiFi Nodes

#### Step 1: Download and Install Node Exporter

```bash
wget https://github.com/prometheus/node_exporter/releases/download/v1.6.0/node_exporter-1.6.0.linux-amd64.tar.gz
tar -xvf node_exporter-1.6.0.linux-amd64.tar.gz
sudo mv node_exporter-1.6.0.linux-amd64/node_exporter /usr/local/bin/
sudo chmod +x /usr/local/bin/node_exporter
```

#### Step 2: Create Node Exporter Service

```bash
sudo vi /etc/systemd/system/node_exporter.service
```

Add the following:

```ini
[Unit]
Description=Node Exporter
After=network.target
[Service]
User=root
ExecStart=/usr/local/bin/node_exporter --web.listen-address=<private-ip>:9100
Restart=always
[Install]
WantedBy=multi-user.target
```

#### Step 3: Start the Node Exporter Service

```bash
sudo systemctl daemon-reload
sudo systemctl start node_exporter
sudo systemctl enable node_exporter
```

#### Step 4: Verify Node Exporter

Access `http://<NiFi-EC2-IP>:9100/metrics` in a browser to ensure it is running.

## Verification

### Prometheus Targets

1. Open Prometheus web interface: `http://<prometheus-server-ip>:9090`.
2. Go to **Status > Targets** and verify that all Node Exporter instances are listed under the `node_exporter` job.

### Metrics in Grafana

1. Open Grafana: `http://<monitoring-server-ip>:3000`.
2. Log in with the credentials from the `.env` file.
3. Add a new data source for Prometheus:
   - Navigate to **Configuration > Data Sources**.
   - Click **Add data source**.
   - Select **Prometheus** from the list.
   - Set the URL to `http://prometheus:9090`.
   - Click **Save & Test** to verify the connection.

### Import Node Exporter Dashboard

1. Navigate to **Dashboards > Import**.
2. Enter the dashboard ID `1860` and click **Load**.
3. Select the Prometheus data source created earlier.
4. Click **Import** to add the Node Exporter Full dashboard.
5. View real-time metrics and configure alerts as needed.

## Conclusion

By following this guide, you can set up Grafana and Prometheus for monitoring your Apache NiFi cluster. With the Node Exporter dashboard, you can visualize system metrics and set up alerts to ensure your cluster remains performant and reliable.

## Contributing

Contributions are welcome! Please feel free to submit a Pull Request.

## License

This project is licensed under the MIT License - see the LICENSE file for details.
