# Prerequisites

- Openshift Cluster (tested with 4.15.13+ version)
- The Advanced Cluster Management operator should be installed on the hub cluster to facilitate cluster administration and data aggregation (tested with 2.10.4+ version).
- [MCO](https://github.com/stolostron/multicluster-observability-operator/) should be installed into the hub cluster with latest version. 
- `oc` CLI should be installed and configured to interact with Openshift cluster

# Steps to Enable Right Sizing Virtualization 

For enabling Right Sizing Virtualization, we need to add Prometheus recording rule as well as [custom allowlist](https://docs.redhat.com/en/documentation/red_hat_advanced_cluster_management_for_kubernetes/2.10/html-single/observability/index#creating-custom-rules) to each managed clusters.     

## Step 1: Login into Openshift Cluster
From the terminal, login into ACM hub cluster environment using login command. You can easily get it form the Hub cluster Console UI.  
```
oc login --token=*** --server=***
```

## Step 2: Setup ManagedClusterSetBinding
We will be using the `global` **ManagedClusterSet** and setting **ManagedClusterSetBinding** with the `policies` namespace using the command below.   
```
oc apply -f data-assets/rs-virtualization/deploy/rs-vm-managedclustersetbinding.yaml
```

## Step 3: Deploy Policy 
We will utilize [Policy](https://access.redhat.com/documentation/en-us/red_hat_advanced_cluster_management_for_kubernetes/2.10/html/governance/governance#policy-overview) to deploy the **recording rules** as well as **custom allowlist** to each managed clusters. 

We have created sample policy for the Prometheus Recording Rule for Virtualization, you can find it [here](../../data-assets/rs-virtualization/deploy/rs-vm-rules-policy.yaml). Using below command you can deploy policy to Hub cluster.  
```
oc apply -f data-assets/rs-virtualization/deploy/rs-vm-rules-policy.yaml
```

You can customize the cluster/namespace filter criteria based on need. Below are sample criteria used in the file where we exclude namespaces that start with `openshift` or `xyz`. Also including namespace where label is `empty` or contains `prod` or `dev` label.

- `namespace!~'openshift.*|xyz.*'`
- `(kube_namespace_labels{label_env=~"prod|dev"} or kube_namespace_labels{label_env=''})`

### Similar way you can deploy custom allowlist policy
### If you are using Right Sizing at Virtualization VM Level only:
you can find sample [here](../../data-assets/rs-virtualization/deploy/rs-vm-allowlist-policy.yaml)
```
oc apply -f data-assets/rs-virtualization/deploy/rs-vm-allowlist-policy.yaml
```

### If you are using Right Sizing at Namespace Level as well as Virtualization VM Level.
you can find sample [here](../../data-assets/rs-allowlist-policy.yaml)
```
oc apply -f data-assets/rs-allowlist-policy.yaml
```

## Step 4: Adding Policies to PolicySet
### If you are using Right Sizing at Virtualization VM Level only:
Apply the [Policy Configurations](../../data-assets/rs-virtualization/deploy/rs-vm-policyset.yaml) using the command below to enable these policies across the fleet. This will include the PolicySet, PlacementBinding, and Placement.
```
oc apply -f data-assets/rs-virtualization/deploy/rs-vm-policyset.yaml
```

### If you are using Right Sizing at Namespace Level as well as Virtualization VM Level.
Apply the [Policy Configurations](../../data-assets/rs-policyset.yaml) using the command below to enable these policies across the fleet. This will include the PolicySet, PlacementBinding, and Placement.
```
oc apply -f data-assets/rs-policyset.yaml
```

Currently, it applies to all managed clusters. You can customize it based on your needs.

**Notes**:
* There are different ways to filter specific clusters that are part of the created ManagedClusterSet. Use the sample configurations below along with the `clusterSets` configuration in the `Placement` to achieve the same. 
  * We can use **predicates** in the `Placement` to filter only a few clusters from the ManagedClusterSet based on labels: 
    ```
      predicates:
      - requiredClusterSelector:
          labelSelector:
            matchExpressions:
              - key: "vm-right-sizing"
                operator: In
                values:
                  - "true"
    ```
    In the example above, we are selecting only the clusters labeled with `vm-right-sizing=true`

    You can use below script to assign specific label to clusters. Update list of comma separated clusters in above command `local-cluster,cluster2,cluster3`
    ```
    for cluster in $(echo "local-cluster,cluster2,cluster3" | tr ',' ' '); do oc label managedcluster "$cluster" vm-right-sizing=true --overwrite; done
    ```
    In this example, we define that only clusters with the label `hive.openshift.op/managed=true` will have the policy applied.

    You can also use **clusterConditions** to filter only few managed clusters based on their status:
    ```
    clusterConditions:
    - status: "True"
      type: ManagedClusterConditionAvailable
    ```

## Step 5: Deploy Grafana Dashboard
[Here](../../data-assets/rs-virtualization/deploy/vm_right_sizing_grafana_main_dashboard.yaml) is the yaml files that contains configuration of ACM Right-Sizing grafana dashboard. Use the following command to add this dashboard on Grafana. 

```
oc create -f data-assets/rs-virtualization/deploy/vm_right_sizing_grafana_main_dashboard.yaml
```

## Step 6: Access Grafana Dashboard
You can then go to Grafana and search for the `ACM Right Sizing Openshift Virtualization Dashboard` dashboard. Click on it to view the main dashboard. Copy the URL of the Dashboard.

## Step 7: Set-Up VM Level Detailed View Dashboards
Update [Overutilized](../../data-assets/rs-virtualization/deploy/vm_right_sizing_grafana_overestimation_dashboard.yaml) and [Underutilized](../../data-assets/rs-virtualization/deploy/vm_right_sizing_grafana_underestimation_dashboard.yaml) Grafana Dashboard config file with the Main Dashboard URL.

```
"links": [
        {
          "asDropdown": false,
          "icon": "dashboard",
          "includeVars": false,
          "keepTime": false,
          "tags": [],
          "targetBlank": false,
          "title": "Back to Main Dashboard",
          "tooltip": "",
          "type": "link",
          "url": "<Main Dashboard URL>"
        }
      ],
```
## Step 8: Deploy VM level detailed View Grafana Dashboards
[There](../../data-assets/rs-virtualization/deploy/) are yaml files that contains configuration of ACM Right-Sizing Overestimation and Underestimation grafana dashboard. Use the following commands to add this dashboard on Grafana. 

```
oc create -f data-assets/rs-virtualization/deploy/vm_right_sizing_grafana_overestimation_dashboard.yaml

oc create -f data-assets/rs-virtualization/deploy/vm_right_sizing_grafana_underestimation_dashboard.yaml
```

## Step 9: All set to access Grafana Dashboard
Wait for some time for the Prometheus Recording Rules for Virtualization to get triggered, and we will have some data points. You can then go to Grafana and search for the `ACM Right Sizing Openshift Virtualization Dashboard` dashboard.
