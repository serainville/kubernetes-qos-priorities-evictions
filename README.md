# QoS, Priorities, evictions
The following repo was created for testing Kubernetes' eviction behaviours, and how its behavour can be manipulated through QoS classes and Priorty classes.

## Notes
### QoS
- QoS do not set precedence, but help determine in what order pods are evicted.

### Priorities and Priority Classes
- Priorities can be used to set eviction order

### Nodes
- Nodes under pressure evict pods to reclaim resources. 
- Nodes apply a taint to prevent pods from being scheduled
- Nodes pressure taints must be manually removed


### Precedence
1. QoS BestEffort (no requests/limits)
2. Priority Class
    1. QoS Burstable
    2. QoS Guaranteed

- A **BestEffort** workload will evict before any other workload
- A **BestEffort** workload in a low-priority class will be evicted before a **BestEffort** workload in a high-priority class.
- A **Guaranteed** workload in a low-priority class will be evicted before a **burstable** workloads in a high-priority class.
- **BestEffort** workloads will not be able to schedulable on nodes with pressure taints.
- **Burstable** and **Guaranteed** workloads can be scheduled on nodes with pressure taints if no longer under pressure.


## Demo
### Understanding QOS
1. Apply the different deployments found under /serverlab
2. Review pods for each deployment and what QoS class is applied

### PriorityClasses
1. Review the manifests under /priority-classes
2. The three classes defined (low, standard, and high) all have different priority values.
3. Review what the values mean and how they are used to manipulate the order of pod evictions.

### Applying PriorityClasses to Pods
1. Deploy at least one of the priorty classes found under '/priority-classes'
2. Edit one of the deployment manifests under `/serverlab`
3. Add `priorityClassName: low-priority` to the template spec of the deployment. Change the `priorityClassName` value to the name of the PriorityClass you deployed in step 1.
4. Rechedule the pod for deployment
5. Check the pod's priority value by running `kubectl describe pod -n serverlab POD_NAME`

## Attack the Cluster
Cookie_Monster is a application specifically designed to consume the available resources of a container host. To test Kubernete's behaviour with evictions and limts, for example, deploy the application to your cluster.

### Getting Started
1. Deploy Cookie Monster!
    ```shell
    kubectl deploy -f deploy-chomper.yaml
    ```
2. Monitor how many resouces Cookie Monster has consumed waiting for a cookie
    ```shell
    kubectl logs -f POD
    ```
3. Allow Cookie Monster to run until all resources have been consumed.
4. If you would like to stop Cookie Monster, give him a cookie!
    ```shell
    kubectl exec POD -- touch /cookie

### Adjust PriorityClass
Apply a PriorityClass to Cookie Monster and monitor how that impacts Kubernetes eviction order.
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: cookie-monster
  labels:
    app: cookie-monster
spec:
  replicas: 1
  selector:
    matchLabels:
      app: cookie-monster
  template:
    metadata:
      labels:
        app: cookie-monster
    spec:
      priorityClassName: high-priority
      containers:
      - name: cookie-monster
        image: serverlab/cookie-monster:0.1.0
        imagePullPolicy: Always

```
