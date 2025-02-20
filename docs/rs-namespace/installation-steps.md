# Prerequisites

- Openshift Cluster (tested with 4.15.13+ version)
- The Advanced Cluster Management operator should be installed on the hub cluster to facilitate cluster administration and data aggregation (tested with 2.10.4+ version).
- [MCO](https://github.com/stolostron/multicluster-observability-operator/) should be installed into the hub cluster with latest version. 
- `oc` CLI should be installed and configured to interact with Openshift cluster

# Steps to Enable ACM Right Sizing Dev Preview 

For enabling ACM Right Sizing Dev Preview, we need to add Prometheus recording rule as well as [custom allowlist](https://docs.redhat.com/en/documentation/red_hat_advanced_cluster_management_for_kubernetes/2.10/html-single/observability/index#creating-custom-rules) to each managed clusters.     

## Step 1: Login into Openshift Cluster
From the terminal, login into ACM hub cluster environment using login command. You can easily get it form the Hub cluster Console UI.  
```
oc login --token=*** --server=***
```

## Step 2: Setup ManagedClusterSetBinding
We will be using the  `global` **ManagedClusterSet** and setting **ManagedClusterSetBinding** with the `policies` namespace using the command below.    
```
oc apply -f data-assets/rs-namespace/deploy/rs-managedclustersetbinding.yaml
```

## Step 3: Deploy Policy 
We will utilize [Policy](https://access.redhat.com/documentation/en-us/red_hat_advanced_cluster_management_for_kubernetes/2.10/html/governance/governance#policy-overview) to deploy **Recording rule** as well as **custom allowlist** to each managed clusters. 

We have created sample policy for the Prometheus Recording Rule, you can find it [here](../../data-assets/rs-namespace/deploy/rs-rules-policy.yaml). Using below command you can deploy policy to Hub cluster.  
```
oc apply -f data-assets/rs-namespace/deploy/rs-rules-policy.yaml
```

You can customize the cluster/namespace filter criteria based on need. Below are sample criteria used in the file where we exclude namespaces that start with `openshift` or `xyz`. Also including namespace where label is `empty` or contains `prod` or `dev` label.

- `namespace!~'openshift.*|xyz.*'`
- `(kube_namespace_labels{label_env=~"prod|dev"} or kube_namespace_labels{label_env=''})`

### Similar way you can deploy custom allowlist policy.
### If you are using Right Sizing at Namespace Level only:
you can find sample [here](../../data-assets/rs-namespace/deploy/rs-allowlist-policy.yaml)
```
oc apply -f data-assets/rs-namespace/deploy/rs-allowlist-policy.yaml
```

### If you are using Right Sizing at Namespace Level as well as Virtualization VM Level.
you can find sample [here](../../data-assets/rs-allowlist-policy.yaml)
```
oc apply -f data-assets/rs-allowlist-policy.yaml
```

## Step 4: Adding Policies to PolicySet
### If you are using Right Sizing at Namespace Level only:
Apply the [Policy Configurations](../../data-assets/rs-namespace/deploy/rs-policyset.yaml) using the command below to enable these policies across the fleet. This will include the PolicySet, PlacementBinding, and Placement.
```
oc apply -f data-assets/rs-namespace/deploy/rs-policyset.yaml
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
              - key: "right-sizing"
                operator: In
                values:
                  - "true"
    ```
    In the example above, we are selecting only the clusters labeled with `right-sizing=true`

    You can use below script to assign specific label to clusters. Update list of comma separated clusters in above command `local-cluster,cluster2,cluster3`   
    ```
    for cluster in $(echo "local-cluster,cluster2,cluster3" | tr ',' ' '); do oc label managedcluster "$cluster" right-sizing=true --overwrite; done
    ```

  * You can also use **clusterConditions** to filter only few managed clusters based on their status:
    ```
    clusterConditions:
    - status: "True"
      type: ManagedClusterConditionAvailable
    ```
    In this example, we are selecting all the managed clusters where the `ManagedClusterConditionAvailable` status is `True`.

## Step 5: Deploy Grafana Dashboard
[Here](../../data-assets/rs-namespace/deploy/acm_right_sizing_grafana_dashboard.yaml) is the yaml file that contains configuration of ACM Right-Sizing grafana dashboard. Use the following command to add this dashboard on Grafana. 

```
oc create -f data-assets/rs-namespace/deploy/acm_right_sizing_grafana_dashboard.yaml
```

## Step 6: Access Grafana Dashboard
Wait for some time for the Prometheus Recording Rules to get triggered, and we will have some data points. You can then go to Grafana and search for the `ACM Right-Sizing Namespace` dashboard. Click on it, and you are ready to use ACM Right Sizing solution.
