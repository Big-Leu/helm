# FastAPI Helm Chart with Observability Stack

This Helm chart deploys a FastAPI application with a comprehensive observability stack including OpenTelemetry Collector, Prometheus, Loki, and Grafana.

## Architecture

The deployment consists of the following components:

- FastAPI Application: The main application instrumented with OpenTelemetry
- OpenTelemetry Collector: Deployed as a DaemonSet to collect traces and logs
- Loki: Log aggregation system running in single-binary mode
- Prometheus: Metrics collection and storage
- Grafana: Visualization platform for logs, metrics, and traces

## Prerequisites

- Kubernetes cluster (1.19+)
- Helm 3.0+
- kubectl configured to communicate with your cluster
- Sufficient cluster resources (minimum recommendations):
  - 2 CPU cores available
  - 4GB RAM available
  - 10GB storage space

## Installation

1. Clone the repository:
   ```bash
   git clone <repository-url>
   cd fastapi-project
   ```

2. Update dependencies:
   ```bash
   helm dependency update
   ```

3. Configure values:
   Review and modify `values.yaml` for your environment:
   - Resource limits and requests
   - Service configurations
   - Persistence settings

4. Install the chart:
   ```bash
   # For first installation
   helm install my-release .

   # For upgrading existing installation
   helm upgrade --install my-release .
   ```

5. Verify the installation:
   ```bash
   # Check pod status
   kubectl get pods

   # Ensure all pods are running
   kubectl get pods -w
   ```

## Configuration

### FastAPI Application

The FastAPI application is configured to send traces and logs to the OpenTelemetry Collector. Key configurations in `values.yaml`:

```yaml
env:
  - name: BACKEND_OPENTELEMETRY_ENDPOINT
    value: "opentelemetry-collector:4317"
```

### Resource Configuration

Default resource limits and requests are set conservatively for resource-constrained environments:

```yaml
resources:
  limits:
    cpu: 200m
    memory: 256Mi
  requests:
    cpu: 100m
    memory: 128Mi
```

Adjust these values based on your environment and requirements.

### OpenTelemetry Collector

Deployed as a DaemonSet, the collector receives traces and logs from the FastAPI application and forwards them to appropriate backends. Configuration includes:

- OTLP gRPC receiver on port 4317
- Exporters for Prometheus, Loki, and traces
- Log transformation processor for enhanced log handling

### Loki

Deployed in single-binary mode with:
- No persistence (using emptyDir)
- Multi-tenancy disabled
- Direct ingestion of logs from the OpenTelemetry Collector

### Grafana

Pre-configured with data sources for:
- Loki (logs)
- Prometheus (metrics)
- Tempo/Jaeger (traces)

## Accessing the Services

1. Port-forward Grafana:
   ```bash
   kubectl port-forward svc/my-release-grafana 3000:80
   ```

2. Access Grafana:
   - URL: http://localhost:3000
   - Default credentials:
     - Username: admin
     - Password: Get using:
       ```bash
       kubectl get secret my-release-grafana -o jsonpath="{.data.admin-password}" | base64 --decode
       ```

## Monitoring

### Logs
View logs in Grafana by:
1. Navigate to Explore
2. Select Loki data source
3. Query using LogQL, e.g.:
   ```
   {app="fastapi"}
   ```

### Metrics
View metrics by:
1. Navigate to Explore
2. Select Prometheus data source
3. Query using PromQL, e.g.:
   ```
   http_requests_total{service="fastapi"}
   ```

### Traces
View traces by:
1. Navigate to Explore
2. Select the tracing data source
3. Search for traces by service name "fastapi"

## Troubleshooting

### Common Issues

1. Loki Pod Fails to Start
   - Ensure emptyDir is properly configured
   - Check Loki is in single-binary mode
   - Verify no PVC requirements
   - If pod is stuck in Terminating state:
     ```bash
     kubectl delete pod <loki-pod> --force --grace-period=0
     ```

2. OpenTelemetry Collector Issues
   - Verify collector endpoints are accessible
   - Check collector configuration in values.yaml
   - Examine collector logs:
     ```bash
     kubectl logs -l app=opentelemetry-collector
     ```
   - Ensure exporter configurations are valid (no unsupported fields)

3. FastAPI Application Not Sending Traces
   - Verify BACKEND_OPENTELEMETRY_ENDPOINT is correct
   - Check FastAPI pod logs
   - Ensure collector service is reachable
   - Verify OpenTelemetry SDK configuration

4. .gitignore Not Working
   If tracked files are not being ignored:
   ```bash
   # Remove tracked files from git's index
   git rm -r --cached .
   
   # Add all files back respecting .gitignore
   git add .
   
   # Commit the changes
   git commit -m "Reset git cache to respect .gitignore"
   ```

5. Resource Constraints
   If pods are in Pending state due to insufficient resources:
   - Check node resources: `kubectl describe node`
   - Adjust resource requests in values.yaml
   - Consider cleaning up unused resources:
     ```bash
     # Delete failed pods
     kubectl delete pods --field-selector status.phase=Failed
     
     # Clean up orphaned PVCs
     kubectl delete pvc --field-selector status.phase=Released
     ```

### Useful Commands

1. Check pod status:
   ```bash
   kubectl get pods
   ```

2. View specific pod logs:
   ```bash
   kubectl logs <pod-name>
   ```

3. Describe resources:
   ```bash
   kubectl describe pod <pod-name>
   ```

4. Port forwarding for debugging:
   ```bash
   # Grafana
   kubectl port-forward svc/my-release-grafana 3000:80

   # Prometheus
   kubectl port-forward svc/my-release-prometheus-server 9090:80

   # Loki
   kubectl port-forward svc/my-release-loki 3100:3100
   ```

## Reset Installation

If you need to start fresh:

1. Uninstall the release:
   ```bash
   helm uninstall my-release
   ```

2. Clean up persistent volumes:
   ```bash
   kubectl delete pvc --all
   ```

3. Verify all resources are removed:
   ```bash
   kubectl get all,pvc,configmap,secret -l release=my-release
   ```

## Contributing

1. Fork the repository
2. Create your feature branch
3. Commit your changes
4. Push to the branch
5. Create a new Pull Request

## License

This project is licensed under the MIT License - see the LICENSE file for details.