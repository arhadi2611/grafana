# GKE Autopilot Monitoring with Grafana Cloud

This repository contains the configuration and deployment files to monitor Google Kubernetes Engine (GKE) Autopilot clusters using Grafana Cloud.

## 🎯 Architecture Overview

```
GKE Autopilot Cluster
    ├── Grafana Agent (DaemonSet)
    │   ├── Collects metrics from Kubernetes API
    │   ├── Scrapes application metrics
    │   └── Forwards to Grafana Cloud
    └── Applications with /metrics endpoints
            ↓
    Grafana Cloud (Remote)
    ├── Prometheus (Metrics)
    ├── Loki (Logs)
    └── Tempo (Traces)
```

## ✅ Prerequisites

- GKE Autopilot cluster running
- Grafana Cloud account ([Sign up here](https://grafana.com/auth/sign-up/create-user))
- `kubectl` configured to access your cluster
- `gcloud` CLI installed and authenticated
- `helm` 3.x installed

## 🚀 Quick Start

### 1. Clone this repository

```bash
git clone https://github.com/your-org/gke-autopilot-grafana-monitoring.git
cd gke-autopilot-grafana-monitoring
```

### 2. Configure Grafana Cloud credentials

Get your credentials from Grafana Cloud:
- Navigate to your Grafana Cloud stack
- Go to "Connections" → "Add new connection" → "Hosted Prometheus metrics"
- Copy your endpoint URL, username, and API key

Create a `.env` file:

```bash
cp .env.example .env
# Edit .env with your credentials
```

### 3. Deploy monitoring stack

```bash
# Create namespace
kubectl create namespace monitoring

# Create secrets
./scripts/create-secrets.sh

# Deploy Grafana Agent
helm repo add grafana https://grafana.github.io/helm-charts
helm repo update

helm install grafana-agent grafana/grafana-agent \
  -f helm/grafana-agent-values.yaml \
  --namespace monitoring
```

### 4. Import Grafana dashboards

```bash
# Dashboards are in the dashboards/ directory
# Import them via Grafana Cloud UI or use the API script
./scripts/import-dashboards.sh
```

## 📁 Repository Structure

```
.
├── README.md
├── .env.example                      # Environment variables template
├── helm/
│   └── grafana-agent-values.yaml    # Helm values for Grafana Agent
├── manifests/
│   ├── namespace.yaml               # Monitoring namespace
│   ├── configmap.yaml               # Grafana Agent configuration
│   ├── secret.yaml                  # Secrets template
│   └── servicemonitor.yaml          # Service monitor examples
├── dashboards/
│   ├── gke-autopilot-overview.json  # Main cluster dashboard
│   ├── gke-workload-metrics.json    # Workload-specific metrics
│   ├── gke-cost-optimization.json   # Cost and resource optimization
│   └── gke-pod-resources.json       # Pod-level metrics
├── alerts/
│   ├── cluster-alerts.yaml          # Cluster-level alerts
│   └── workload-alerts.yaml         # Application alerts
├── scripts/
│   ├── create-secrets.sh            # Create Kubernetes secrets
│   ├── import-dashboards.sh         # Import dashboards to Grafana Cloud
│   └── validate-config.sh           # Validate configuration
└── docs/
    ├── SETUP.md                     # Detailed setup guide
    ├── TROUBLESHOOTING.md           # Common issues and solutions
    └── DASHBOARDS.md                # Dashboard documentation
```

## ⚙️ Key Metrics Monitored

### Cluster Metrics
- Node resource utilization (CPU, Memory, Disk)
- Pod count and status
- Cluster autoscaling events
- Network traffic and bandwidth

### Workload Metrics
- Deployment status and health
- Pod restart counts
- Resource requests vs actual usage
- Application-specific metrics

### Cost Optimization Metrics
- Over-provisioned resources
- Resource efficiency ratios
- Bin packing efficiency
- Spot/Preemptible node usage

## Grafana Agent Configuration

The Grafana Agent is deployed as a DaemonSet to collect metrics from:

1. **Kubernetes API** - Cluster and node metrics
2. **cAdvisor** - Container metrics
3. **Kubelet** - Pod and container stats
4. **Application Pods** - Custom metrics via ServiceMonitors

##  🎯 Supported Integrations

- ✅ Prometheus metrics
- ✅ Kubernetes logs via Loki
- ✅ Distributed tracing via Tempo
- ✅ Custom application metrics
- ✅ Alert rules and notifications

## ⚠️ Security Considerations

- Secrets stored in Kubernetes Secrets (base64 encoded)
- RBAC roles with minimum required permissions
- Network policies for agent communication
- TLS encryption for remote write endpoints

## Customization

### Adding Custom Metrics

1. Create a ServiceMonitor in `manifests/servicemonitor.yaml`
2. Add scrape config to `helm/grafana-agent-values.yaml`
3. Apply changes: `kubectl apply -f manifests/servicemonitor.yaml`

### Adding Alerts

1. Add alert rules to `alerts/` directory
2. Apply via Grafana Cloud UI or Terraform
3. Configure notification channels

## Cost Considerations

### Grafana Cloud Pricing
- Free tier: 10K series, 50GB logs, 50GB traces
- Pro tier: Pay as you grow

### Optimization Tips
- Use metric relabeling to reduce cardinality
- Set appropriate scrape intervals (30s-60s)
- Filter unnecessary labels
- Use recording rules for complex queries

## Monitoring Best Practices

1. **Start with defaults** - Use provided dashboards and customize
2. **Set meaningful alerts** - Focus on actionable alerts
3. **Monitor the monitor** - Track Grafana Agent health
4. **Regular reviews** - Review and optimize metric collection
5. **Document changes** - Keep README and docs updated

## Troubleshooting

### Grafana Agent not sending metrics

```bash
# Check agent logs
kubectl logs -n monitoring -l app.kubernetes.io/name=grafana-agent

# Verify configuration
kubectl get configmap -n monitoring grafana-agent -o yaml

# Test connectivity
kubectl run -it --rm debug --image=curlimages/curl --restart=Never -- \
  curl -v https://prometheus-prod-01-eu-west-0.grafana.net/api/prom/push
```

### Missing metrics in Grafana Cloud

1. Verify remote_write endpoint in agent config
2. Check authentication credentials
3. Review metric relabeling rules
4. Check Grafana Cloud usage limits

See [TROUBLESHOOTING.md](docs/TROUBLESHOOTING.md) for more details.

## Contributing

1. Fork the repository
2. Create a feature branch
3. Make your changes
4. Test in a development cluster
5. Submit a pull request

## 📚 Resources

- [GKE Autopilot Documentation](https://cloud.google.com/kubernetes-engine/docs/concepts/autopilot-overview)
- [Grafana Cloud Documentation](https://grafana.com/docs/grafana-cloud/)
- [Grafana Agent Documentation](https://grafana.com/docs/agent/latest/)
- [Kubernetes Monitoring Guide](https://kubernetes.io/docs/tasks/debug/debug-cluster/resource-metrics-pipeline/)

## License

MIT License - see [LICENSE](LICENSE) file for details

## 🙋 Support

- Issues: [GitHub Issues](https://github.com/your-org/gke-autopilot-grafana-monitoring/issues)
- Discussions: [GitHub Discussions](https://github.com/your-org/gke-autopilot-grafana-monitoring/discussions)
- Email: devops@yourcompany.com

## Changelog

See [CHANGELOG.md](CHANGELOG.md) for version history and updates.
