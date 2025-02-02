# ACM Right Sizing at Namespace level

Below is the screenshot of ACM Right Sizing grafana dashboard at Namespace level. 
![ACM Right Sizing Grafana dashboard](../../data-assets/rs-namespace/images/grafana-overview.png)

You can select the cluster you are interested in exploring and view the CPU/memory recommendations, requests, and utilization for different aggregation periods.

# Metrics Used
* CPU Usage: `node_namespace_pod_container:container_cpu_usage_seconds_total:sum`
* Memory Usage: `container_memory_working_set_bytes`
* CPU Request: `kube_pod_container_resource_requests:sum{resource="cpu"}`
* Memory Request: `kube_pod_container_resource_requests:sum{resource="memory"}`

# Installation Steps 

You can checkout [this](installation-steps.md) page for detailed steps on installing ACM Right Sizing components.

# How to Use Grafana?

We have given basic instruction on how you can use grafana [here](how-to-use-grafana.md), you can check out for more details.   
