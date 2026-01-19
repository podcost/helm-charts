# PodCost Helm Charts

Helm charts for deploying the PodCost Kubernetes metrics collection agent.

## Overview

PodCost is a Kubernetes cost monitoring solution that helps you track and optimize your cluster costs. This Helm chart deploys a lightweight agent that collects metrics from your cluster and sends them to the PodCost dashboard.

## Prerequisites

- Kubernetes 1.19+
- Helm 3.0+
- Metrics Server installed in your cluster
- API key from [PodCost Dashboard](https://podcost.io/add-cluster)

## Usage

### Add the Helm Repository

```bash
helm repo add podcost https://helm.podcost.io
helm repo update
```

### Install PodCost Agent

```bash
helm install podcost-agent podcost/pod-cost \
  --set config.apiKey="YOUR_API_KEY" \
  --set config.clusterName="my-cluster" \
  --set config.cloudProvider="aws-eks" \
  --set config.region="us-east-1" \
  --namespace podcost \
  --create-namespace
```

Get your API key from the [PodCost Dashboard](https://podcost.io/add-cluster).

### Install with GPU Metrics

If you have GPU nodes and want to collect GPU metrics:

```bash
helm install podcost-agent podcost/pod-cost \
  --set config.apiKey="YOUR_API_KEY" \
  --set config.clusterName="my-cluster" \
  --set config.cloudProvider="aws-eks" \
  --set config.region="us-east-1" \
  --set config.enableGpuMetrics=true \
  --set config.dcgmEndpoint="http://dcgm-exporter.monitoring:9400/metrics" \
  --namespace podcost \
  --create-namespace
```

## Configuration

| Parameter | Description | Default |
|-----------|-------------|---------|
| `config.apiKey` | API key from podcost.io dashboard (required) | `""` |
| `config.clusterName` | Name to identify your cluster | `""` |
| `config.apiEndpoint` | PodCost API endpoint | `"https://ingest.podcost.io/v1"` |
| `config.collectionInterval` | Metrics collection interval in seconds | `60` |
| `config.cloudProvider` | Cloud provider: `aws-eks`, `gcp-gke`, `azure-aks` | `"aws-eks"` |
| `config.region` | Cloud region for pricing (e.g., `us-east-1`, `eu-west-1`) | `"us-east-1"` |
| `config.enableGpuMetrics` | Enable GPU metrics collection | `true` |
| `config.dcgmEndpoint` | DCGM Exporter endpoint for detailed GPU metrics | `""` |
| `image.repository` | Agent Docker image repository | `mgabribrahim/podcost-agent` |
| `image.tag` | Agent Docker image tag | `"latest"` |
| `image.pullPolicy` | Image pull policy | `Always` |
| `replicaCount` | Number of agent replicas | `1` |
| `resources.requests.memory` | Memory request | `"128Mi"` |
| `resources.requests.cpu` | CPU request | `"100m"` |
| `resources.limits.memory` | Memory limit | `"256Mi"` |
| `resources.limits.cpu` | CPU limit | `"200m"` |
| `serviceAccount.create` | Create service account | `true` |
| `serviceAccount.name` | Service account name | `"pod-cost"` |

## Cloud Provider Values

| Provider | Value |
|----------|-------|
| Amazon EKS | `aws-eks` |
| Google GKE | `gcp-gke` |
| Azure AKS | `azure-aks` |

## What Data is Collected

The agent collects the following metrics:

- **Node metrics**: CPU/memory usage and capacity, instance types, GPU count
- **Pod metrics**: CPU/memory usage per container
- **Workload metadata**: Deployments, StatefulSets, DaemonSets
- **GPU metrics**: Utilization, memory usage, temperature (if enabled)
- **Kubernetes events**: Scaling events, pod lifecycle events (for root cause analysis)

All data is sent securely to the PodCost API using your API key.

## RBAC Permissions

The agent requires the following Kubernetes permissions:

- `get`, `list`, `watch` on `nodes`, `pods`, `namespaces`, `events`
- `get`, `list` on `metrics.k8s.io` resources
- `get`, `list` on `deployments`, `statefulsets`, `daemonsets`

## Upgrading

```bash
helm repo update
helm upgrade podcost-agent podcost/pod-cost --namespace podcost
```

## Uninstall

```bash
helm uninstall podcost-agent --namespace podcost
kubectl delete namespace podcost
```

## Troubleshooting

### Check agent logs

```bash
kubectl logs -l app.kubernetes.io/name=pod-cost -n podcost
```

### Verify metrics server is running

```bash
kubectl get pods -n kube-system | grep metrics-server
```

### Check RBAC permissions

```bash
kubectl auth can-i list pods --as=system:serviceaccount:podcost:pod-cost
```

## Support

- Documentation: [https://docs.podcost.io](https://docs.podcost.io)
- Issues: [https://github.com/podcost/helm-charts/issues](https://github.com/podcost/helm-charts/issues)

## License

MIT License - see [LICENSE](LICENSE) for details.
