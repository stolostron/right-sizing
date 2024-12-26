This Document described how you can interact with grafana dashboards. 

1. You can go to Grafana on Hub Cluster and search for the `ACM Right Sizing Virtualization` dashboard. Click on it, and you are ready to use ACM Right Sizing for Virtualization solution.

2. Select Specific filter criteria based on need

    ![Filters](../../data-assets/rs-virtualization/images/vm-filter.png)

   * Cluster: Dropdown contains all the managed cluster names.
   * Profile: For time being only `Max Overall` profile will be available, in future we will incorporate other mechanism to do Right Sizing along with existing `max Overall`
   * Aggregation: It contains number of days we want to get data from. Ex: for `Max Overall` if you select `30d` that means we are looking for 30 days peak/max value for the CPU/Memory and looking for recommendation based on `30d` data points. 

3. Left 4 Panels show *cluster level* maximum values over selected last days of aggregation for CPU/Mem Recommendation, CPU/Mem Utilization, CPU/Mem Request and CPU/Mem Utilization%.

    ![Left Panels](../../data-assets/rs-virtualization/images/vm-left-panel.png)

4. Graph shows CPU/Memory Utilization values for each Namespace with Pod name in parenthesis.

    ![Graph](../../data-assets/rs-virtualization/images/vm-graph.png)

5. CPU/Mem Quota table shows max values over last selected aggregation days of namespaces and/or pods available for clusters for CPU/Mem Utilization Percentage, CPU/Mem Utilization, CPU/Mem Request, CPU/Mem Recommendation.

    ![Table](../../data-assets/rs-virtualization/images/vm-table.png)

6. Click on the Column Header to sort values of particular columns.

7. Click on the Namespace Filter Icon in the Column Header to filter values based on user need.

    ![Namespace Filter](../../data-assets/rs-virtualization/images/vm-namespace-filter.png)

8. Click on the VM Name Filter Icon in the Column Header to filter the values further.

    ![VM Filter](../../data-assets/rs-virtualization/images/vm-pod-filter.png)
