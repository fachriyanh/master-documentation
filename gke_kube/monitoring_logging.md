# Monitoring and Logging

This section describes how to set up monitoring and logging for applications deployed in GKE using Google Cloud Operations Suite, Prometheus, Grafana, and GitLab CI/CD monitoring.

---

## Integration with Google Cloud Operations Suite (formerly Stackdriver)

**Google Cloud Operations Suite** provides powerful built-in logging, monitoring, and alerting for GKE clusters.

### Key Features:
- Logs from all Kubernetes system components and containers.
- Metrics collection (CPU, memory, networking, disk).
- Alerting based on metrics thresholds.

### Steps to Enable:
1. When creating the GKE cluster, enable **Cloud Operations for GKE**.
2. Ensure that the GKE nodes have the correct IAM permissions (`roles/logging.logWriter` and `roles/monitoring.metricWriter`).
3. Use the Google Cloud Console to access logs and metrics:
   - Logs Explorer for viewing application logs.
   - Metrics Explorer for creating dashboards and setting up alerts.

**Example Command to Create a Cluster with Monitoring:**

gcloud container clusters create my-cluster
--zone us-central1-a
--enable-stackdriver-kubernetes

---

## Setup Prometheus and Grafana for Custom Metrics

**Prometheus** and **Grafana** allow detailed monitoring and visualization of application-specific metrics within Kubernetes.

### Prometheus Setup:
- Deploy Prometheus in the GKE cluster using Helm:
  
  helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
  helm install prometheus prometheus-community/kube-prometheus-stack

- Configure Prometheus to scrape metrics endpoints in your applications.

### Grafana Setup:
- Grafana is typically installed alongside Prometheus using the kube-prometheus-stack Helm chart.
- Access Grafana dashboard using a LoadBalancer or Ingress.
- Import Kubernetes or custom metric dashboards from the Grafana marketplace.

### Example Metrics to Monitor:
- Application response time
- Request rate
- Error rate
- Database query times

---

## GitLab CI/CD Pipeline Monitoring

Monitoring GitLab pipelines helps track performance, failures, and optimize build time.

### How to Set Up:
1. GitLab UI provides default pipeline duration metrics and success/failure rates.
2. Enable **Pipeline Insights** feature in GitLab for aggregated metrics.
3. For deeper monitoring:
   - Export GitLab CI metrics to Prometheus using GitLab's Prometheus integration.
   - Create dashboards in Grafana using GitLab’s pipeline metrics.
   
### Important Metrics:
- Pipeline duration over time.
- Success vs failure rates.
- Job execution time breakdown.
- Stale or canceled pipelines.

---

# Summary

- Use **Google Cloud Operations Suite** for easy and centralized logging and monitoring of GKE applications.
- Deploy **Prometheus and Grafana** to monitor custom metrics inside Kubernetes.
- Monitor **GitLab CI/CD pipelines** to optimize performance and reliability.
- Set up alerts for critical failures or abnormal performance.

---