# 🚀 Ansible Monitoring Stack

A production-ready Ansible project to deploy a complete monitoring infrastructure using Prometheus, Grafana, Alertmanager, and Node Exporter. Perfect for DevOps engineers, SREs, and monitoring setups in AWS.

## 📋 Table of Contents

- [Features](#-features)
- [Prerequisites](#-prerequisites)
- [Project Structure](#-project-structure)
- [Quick Start](#-quick-start)
- [Inventory Setup](#-inventory-setup)
- [Deployment](#-deployment)
- [Access Services](#-access-services)
- [Configuration](#-configuration)
- [Best Practices](#-best-practices-implemented)
- [Troubleshooting](#-troubleshooting)
- [Future Enhancements](#-future-enhancements)
- [Use Cases](#-use-cases)

## 📊 Features

- ✅ **Centralized Monitoring Node** - Single source of truth for metrics
- ✅ **Distributed Node Exporters** - Monitor multiple servers simultaneously
- ✅ **Alertmanager Integration** - Intelligent alert routing and grouping
- ✅ **PagerDuty Ready** - Seamless incident management integration
- ✅ **EC2 Service Discovery** - Auto-discover AWS instances
- ✅ **Idempotent Ansible Roles** - Safe to run repeatedly without breaking setup
- ✅ **Role-Based Design** - Modular and reusable components
- ✅ **Security Hardened** - Dedicated system users, no root service execution
- ✅ **Scalable Architecture** - Easy to add new monitoring nodes

## ⚙️ Prerequisites

- **Ansible** >= 2.9
- **Ubuntu/Debian** servers (tested on Ubuntu 20.04, 22.04)
- **SSH Access** with sudo privileges
- **AWS Credentials** (optional, for EC2 service discovery)
- **IAM Role** attached to Prometheus/Grafana server (for EC2 discovery)

## 📂 Project Structure

```
monitoring-stack/
├── inventory.ini                 # Inventory file with host definitions
├── playbook.yml                  # Main playbook orchestrating roles
├── README.md                     # This file
├── ansible.cfg                   # Ansible configuration
└── roles/
    ├── common/                   # Common setup (packages, users, directories)
    │   ├── tasks/
    │   │   └── main.yml
    │   └── handlers/
    │       └── main.yml
    ├── prometheus/               # Prometheus monitoring server
    │   ├── tasks/
    │   │   └── main.yml
    │   ├── templates/
    │   │   ├── prometheus.yml.j2
    │   │   └── prometheus.service.j2
    │   ├── files/
    │   └── handlers/
    │       └── main.yml
    ├── grafana/                  # Grafana visualization dashboards
    │   ├── tasks/
    │   │   └── main.yml
    │   ├── templates/
    │   │   ├── grafana.ini.j2
    │   │   └── datasources.yml.j2
    │   └── handlers/
    │       └── main.yml
    ├── alertmanager/             # Alert routing and management
    │   ├── tasks/
    │   │   └── main.yml
    │   ├── templates/
    │   │   ├── alertmanager.yml.j2
    │   │   └── alertmanager.service.j2
    │   └── handlers/
    │       └── main.yml
    └── node_exporter/            # Prometheus node exporter
        ├── tasks/
        │   └── main.yml
        ├── templates/
        │   └── node_exporter.service.j2
        └── handlers/
            └── main.yml
```

## 🚀 Quick Start

### 1. Clone or Initialize Project

```bash
mkdir -p monitoring-stack && cd monitoring-stack
# Copy the project structure and files here
```

### 2. Update Inventory

Edit `inventory.ini` with your server IPs:

```ini
[monitor]
monitor1 ansible_host=10.0.1.100

[nodes]
node1 ansible_host=10.0.2.100
node2 ansible_host=10.0.2.101
node3 ansible_host=10.0.2.102

[all:vars]
ansible_user=ubuntu
ansible_ssh_private_key_file=~/.ssh/id_rsa
ansible_python_interpreter=/usr/bin/python3
```

### 3. Run Deployment

```bash
ansible-playbook -i inventory.ini playbook.yml
```

For verbose output:

```bash
ansible-playbook -i inventory.ini playbook.yml -v
```

For specific host group:

```bash
ansible-playbook -i inventory.ini playbook.yml --limit nodes
```

## 🖥️ Inventory Setup

### Basic Setup

```ini
[monitor]
monitor1 ansible_host=YOUR_MONITOR_IP

[nodes]
node1 ansible_host=NODE_IP_1
node2 ansible_host=NODE_IP_2
```

### AWS EC2 Setup (with tags)

```ini
[all:vars]
aws_region=us-east-1
node_tag_name=monitoring:true
ansible_user=ubuntu
ansible_ssh_private_key_file=~/.ssh/your-key.pem

[monitor]
prometheus-server ansible_host=10.0.1.100

[nodes]
# Auto-discovered via EC2 tags in prometheus.yml.j2
```

### Advanced Setup with Variables

```ini
[monitor]
monitor1 ansible_host=10.0.1.100 prometheus_retention_days=30 prometheus_scrape_interval=15s

[nodes:children]
prod_nodes
staging_nodes

[prod_nodes]
prod-node1 ansible_host=10.0.2.100
prod-node2 ansible_host=10.0.2.101

[staging_nodes]
staging-node1 ansible_host=10.0.3.100

[all:vars]
ansible_user=ubuntu
ansible_python_interpreter=/usr/bin/python3
```

## 📦 Deployment

### Standard Deployment

```bash
# Deploy to all hosts
ansible-playbook -i inventory.ini playbook.yml

# Deploy with specific tags
ansible-playbook -i inventory.ini playbook.yml --tags "prometheus,grafana"

# Deploy to specific group
ansible-playbook -i inventory.ini playbook.yml --limit monitor
```

### Dry Run (Check Mode)

```bash
# Preview changes without applying
ansible-playbook -i inventory.ini playbook.yml --check
```

### Step-by-Step Deployment

```bash
# Deploy common dependencies
ansible-playbook -i inventory.ini playbook.yml --tags "common"

# Deploy node exporters
ansible-playbook -i inventory.ini playbook.yml --tags "node_exporter"

# Deploy Prometheus
ansible-playbook -i inventory.ini playbook.yml --tags "prometheus"

# Deploy Grafana
ansible-playbook -i inventory.ini playbook.yml --tags "grafana"

# Deploy Alertmanager
ansible-playbook -i inventory.ini playbook.yml --tags "alertmanager"
```

## 🌐 Access Services

Once deployment is complete, access the monitoring stack:

| Service | URL | Port | Default Credentials |
|---------|-----|------|---------------------|
| **Prometheus** | http://<monitor_ip>:9090 | 9090 | None (no auth) |
| **Grafana** | http://<monitor_ip>:3000 | 3000 | admin / admin |
| **Alertmanager** | http://<monitor_ip>:9093 | 9093 | None (no auth) |
| **Node Exporter** | http://<node_ip>:9100/metrics | 9100 | None (metrics only) |

### Example Access

```bash
# Prometheus
curl http://monitor1:9090/api/v1/targets

# Grafana
curl -u admin:admin http://monitor1:3000/api/datasources

# Node Exporter
curl http://node1:9100/metrics
```

## 🔐 Security

### Default Credentials - ⚠️ CHANGE IMMEDIATELY

**Grafana:**
- Username: `admin`
- Password: `admin`

### Change Grafana Password

```bash
# Via API
curl -X PUT \
  -H "Content-Type: application/json" \
  -d '{"oldPassword":"admin","newPassword":"YourNewPassword123!","confirmNew":"YourNewPassword123!"}' \
  http://admin:admin@monitor1:3000/api/user/password

# Or via Web UI
# 1. Login with admin:admin
# 2. Click user icon (top right) → Change Password
# 3. Enter new password and save
```

### Security Best Practices

- ✅ Change default Grafana password immediately
- ✅ Use SSH keys, not passwords for Ansible
- ✅ Run services under dedicated non-root users
- ✅ Restrict firewall rules to necessary ports
- ✅ Use HTTPS/TLS for production (see Future Enhancements)
- ✅ Implement authentication for Prometheus and Alertmanager
- ✅ Regularly update system packages

```bash
# Update all packages after deployment
sudo apt update && sudo apt upgrade -y
```

## 🔧 Configuration

### Prometheus Configuration

Edit `roles/prometheus/templates/prometheus.yml.j2`:

```yaml
global:
  scrape_interval: 15s
  scrape_timeout: 10s
  evaluation_interval: 15s

scrape_configs:
  - job_name: 'prometheus'
    static_configs:
      - targets: ['localhost:9090']
  
  - job_name: 'nodes'
    static_configs:
      - targets: 
        - '10.0.2.100:9100'
        - '10.0.2.101:9100'
```

### AWS EC2 Service Discovery

Update Prometheus to auto-discover EC2 instances:

```yaml
scrape_configs:
  - job_name: 'ec2-nodes'
    ec2_sd_configs:
      - region: us-east-1
        port: 9100
    relabel_configs:
      - source_labels: [__meta_ec2_tag_Name]
        target_label: instance
      - source_labels: [__meta_ec2_tag_monitoring]
        regex: 'true'
        action: keep
```

### Alertmanager Configuration

Edit `roles/alertmanager/templates/alertmanager.yml.j2`:

**PagerDuty Integration:**

```yaml
global:
  resolve_timeout: 5m

route:
  receiver: 'pagerduty-receiver'
  routes:
    - match:
        severity: critical
      receiver: pagerduty-receiver
      repeat_interval: 5m

receivers:
  - name: 'pagerduty-receiver'
    pagerduty_configs:
      - service_key: YOUR_PAGERDUTY_SERVICE_KEY
        description: '{{ .GroupLabels.alertname }}'
```

**Slack Integration:**

```yaml
receivers:
  - name: 'slack-receiver'
    slack_configs:
      - api_url: YOUR_SLACK_WEBHOOK_URL
        channel: '#alerts'
        title: 'Alert: {{ .GroupLabels.alertname }}'
        text: '{{ range .Alerts }}{{ .Annotations.description }}{{ end }}'
```

### Grafana Datasources

Edit `roles/grafana/templates/datasources.yml.j2`:

```yaml
apiVersion: 1

datasources:
  - name: Prometheus
    type: prometheus
    url: http://localhost:9090
    access: proxy
    isDefault: true
    editable: true
```

### Node Exporter Custom Flags

Edit `roles/node_exporter/tasks/main.yml`:

```yaml
- name: Start node_exporter with custom flags
  systemd:
    name: node_exporter
    enabled: yes
    state: started
    daemon_reload: yes
  environment:
    ARGS: "--collector.filesystem.mount-points-exclude=^/(sys|proc|dev|host|etc)($$|/)"
```

## 🧠 Best Practices Implemented

### 1. **Idempotency**
Roles ensure that repeated runs do not break the setup. All tasks are designed to be safe to execute multiple times.

```yaml
- name: Create prometheus user
  user:
    name: prometheus
    shell: /bin/false
    state: present
```

### 2. **Role-Based Design**
Each component is modular and reusable, following Ansible best practices:
- `common`: Base OS setup
- `prometheus`: Metrics collection
- `grafana`: Visualization
- `alertmanager`: Alert routing
- `node_exporter`: Host metrics

### 3. **Security**
- Dedicated system users (no root execution)
- Proper file permissions
- Service isolation

```yaml
- name: Set prometheus config permissions
  file:
    path: /etc/prometheus
    owner: prometheus
    group: prometheus
    mode: '0755'
    recurse: yes
```

### 4. **Scalability**
Add new monitoring nodes easily by updating the inventory file. No playbook changes needed.

```ini
# Just add new nodes to inventory
[nodes]
node1 ansible_host=10.0.2.100
node2 ansible_host=10.0.2.101
node999 ansible_host=10.0.2.999  # New node
```

### 5. **Separation of Concerns**
Each role handles only one responsibility, making maintenance and testing easier.

## ❗ Troubleshooting

### Check Service Status

```bash
# On monitor server
systemctl status prometheus
systemctl status grafana-server
systemctl status alertmanager

# On node servers
systemctl status node_exporter
```

### View Service Logs

```bash
# Recent logs
journalctl -u prometheus -n 50 -f

# Full logs
journalctl -u prometheus -b

# Grafana logs
journalctl -u grafana-server -f

# Alertmanager logs
journalctl -u alertmanager -f
```

### Prometheus Not Scraping Targets

```bash
# Check Prometheus targets page
curl http://localhost:9090/api/v1/targets | jq '.data.activeTargets'

# Verify node_exporter is running on target
curl http://<target_ip>:9100/metrics

# Check Prometheus configuration
curl http://localhost:9090/api/v1/query?query=up
```

### Grafana Cannot Connect to Prometheus

1. Verify Prometheus is running: `systemctl status prometheus`
2. Check Prometheus connectivity: `curl http://localhost:9090/-/healthy`
3. Verify datasource configuration in Grafana UI
4. Check Grafana logs: `journalctl -u grafana-server -f`

### Node Exporter Not Responding

```bash
# Check if service is running
systemctl status node_exporter

# Check port availability
netstat -tlnp | grep 9100

# Restart the service
sudo systemctl restart node_exporter

# Check logs
journalctl -u node_exporter -f
```

### Ansible Connection Issues

```bash
# Test SSH connectivity
ansible all -i inventory.ini -m ping

# Run with verbose output
ansible-playbook -i inventory.ini playbook.yml -vvv

# Check SSH key permissions
chmod 600 ~/.ssh/your-key.pem
```

### Port Already in Use

```bash
# Find process using port 9090 (example)
lsof -i :9090

# Kill process
kill -9 <PID>

# Or change port in configuration and redeploy
```

### Permission Denied Errors

```bash
# Ensure ansible user has sudo privileges
sudo usermod -aG sudo ubuntu

# Check sudoers file
sudo visudo

# Ensure Ansible can use sudo without password
echo "ubuntu ALL=(ALL) NOPASSWD:ALL" | sudo tee /etc/sudoers.d/ansible
```

## 📊 Monitoring Checks

### Key Metrics to Monitor

```promql
# Node availability
up{job="nodes"}

# CPU usage
rate(node_cpu_seconds_total[5m])

# Memory usage
(1 - (node_memory_MemAvailable_bytes / node_memory_MemTotal_bytes)) * 100

# Disk usage
(node_filesystem_avail_bytes / node_filesystem_size_bytes) * 100

# Network interface
rate(node_network_transmit_bytes_total[5m])
```

## 🎯 Advanced Usage

### EC2 Service Discovery with IAM Role

1. **Attach IAM Policy to EC2 Instance:**

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "ec2:DescribeInstances",
        "ec2:DescribeTags"
      ],
      "Resource": "*"
    }
  ]
}
```

2. **Update Prometheus Configuration:**

```yaml
scrape_configs:
  - job_name: 'ec2'
    ec2_sd_configs:
      - region: us-east-1
        port: 9100
    relabel_configs:
      - source_labels: [__meta_ec2_tag_monitoring]
        regex: 'true'
        action: keep
```

3. **Redeploy Prometheus:**

```bash
ansible-playbook -i inventory.ini playbook.yml --tags prometheus
```

### Custom Alert Rules

Create `roles/prometheus/files/alert_rules.yml`:

```yaml
groups:
  - name: custom_alerts
    interval: 30s
    rules:
      - alert: HighCPUUsage
        expr: (100 - (avg by (instance) (rate(node_cpu_seconds_total{mode="idle"}[5m])) * 100)) > 80
        for: 5m
        labels:
          severity: warning
        annotations:
          summary: "High CPU usage on {{ $labels.instance }}"
```

### Grafana Dashboard Import

Grafana comes with built-in dashboards. To import custom dashboards:

1. Visit http://<monitor_ip>:3000/dashboards
2. Click "New" → "Import"
3. Enter dashboard ID or paste JSON
4. Select Prometheus as datasource
5. Click "Import"

Popular Dashboard IDs:
- `3662` - Prometheus server
- `1860` - Node Exporter
- `6417` - Grafana Dashboard

## 📦 Future Enhancements

- [ ] **Grafana Dashboards** - Auto-import popular dashboards
- [ ] **Slack/Email Alerts** - Multi-channel notifications
- [ ] **TLS/HTTPS** - Secure communications
- [ ] **Kubernetes Helm** - Deploy on K8s clusters
- [ ] **Docker Compose** - Containerized alternative
- [ ] **Loki Integration** - Log aggregation
- [ ] **Jaeger Integration** - Distributed tracing
- [ ] **Automatic Backup** - Grafana backup automation
- [ ] **Prometheus HA** - High availability setup
- [ ] **VictoriaMetrics** - Alternative time-series database

## 🎯 Use Cases

This monitoring stack is perfect for:

- 🏢 **DevOps Engineers** - Infrastructure monitoring
- 👨‍💼 **SREs** - Service reliability and observability
- ☁️ **AWS Environments** - Cloud infrastructure monitoring
- 📚 **Interview Projects** - Demonstrate DevOps skills
- 🏗️ **Proof of Concepts** - Quick monitoring setup
- 🚀 **Startup Infrastructure** - Cost-effective monitoring

## 📝 Resume Points

Add this to your resume:

> "Implemented a centralized monitoring infrastructure using Prometheus, Grafana, and Alertmanager deployed via Ansible automation, with EC2 service discovery integration for automatic node detection in AWS environments."

Or:

> "Architected and deployed production-ready monitoring stack with Prometheus metrics collection, Grafana visualization dashboards, and Alertmanager for intelligent alert routing, supporting horizontal scalability across 100+ nodes."

## 📞 Support & Contributing

For issues or questions:

1. Check the **Troubleshooting** section
2. Review Ansible logs with `-v` flag
3. Verify all prerequisites are met
4. Test SSH connectivity with `ansible all -m ping`

## 📄 License

MIT License - Feel free to use and modify

## 👨‍💻 Author

DevOps Monitoring Project

---

**Last Updated:** April 2026

**Version:** 1.0.0

**Status:** Production Ready ✅
