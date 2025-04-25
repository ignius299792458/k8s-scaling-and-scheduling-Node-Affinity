# Kubernetes Node Affinity

Node Affinity is a feature in Kubernetes that gives you fine-grained control over which nodes your pods can be scheduled on. It's more expressive and flexible than using node selectors and provides a way to constrain which nodes your pod can be scheduled on based on node labels.

## How Node Affinity Works

Node Affinity uses label selectors with additional operators and features to enable more complex constraints and preferences for pod placement. There are two types of node affinity:

1. **Required Node Affinity (`requiredDuringSchedulingIgnoredDuringExecution`)**: 
   - Pods will only be scheduled on nodes that satisfy the affinity expressions
   - If no nodes match, pods will remain in pending state
   - "Required" means the pod must be scheduled on a matching node
   - "IgnoredDuringExecution" means if node labels change after pod is scheduled, the pod won't be evicted

2. **Preferred Node Affinity (`preferredDuringSchedulingIgnoredDuringExecution`)**: 
   - The scheduler will try to find nodes that match the affinity expressions
   - If no nodes match, the pod will still be scheduled on any available node
   - You can set a weight (1-100) to prioritize different preferences

## Basic Example

Here's a simple example of node affinity:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: with-node-affinity
spec:
  affinity:
    nodeAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        nodeSelectorTerms:
        - matchExpressions:
          - key: kubernetes.io/e2e-az-name
            operator: In
            values:
            - us-east-1a
            - us-east-1b
      preferredDuringSchedulingIgnoredDuringExecution:
      - weight: 1
        preference:
          matchExpressions:
          - key: disk-type
            operator: In
            values:
            - ssd
  containers:
  - name: nginx
    image: nginx
```

In this example:
- The pod MUST be scheduled on a node with the label `kubernetes.io/e2e-az-name` set to either `us-east-1a` or `us-east-1b`
- The scheduler will PREFER nodes with the label `disk-type` set to `ssd`, but will schedule on other nodes if none are available

## Operators

Node Affinity supports several operators:

- `In`: The label value must match one of the specified values
- `NotIn`: The label value must not match any of the specified values
- `Exists`: The node must have a label with the specified key (value not checked)
- `DoesNotExist`: The node must not have a label with the specified key
- `Gt`: The label value must be greater than the specified value (numeric)
- `Lt`: The label value must be less than the specified value (numeric)

## Practical Use Cases

1. **Hardware constraints**: Schedule GPU workloads only on nodes with GPUs

```yaml
nodeAffinity:
  requiredDuringSchedulingIgnoredDuringExecution:
    nodeSelectorTerms:
    - matchExpressions:
      - key: accelerator
        operator: In
        values:
        - gpu-a100
```

2. **Topology spreading**: Distribute pods across different availability zones

```yaml
nodeAffinity:
  preferredDuringSchedulingIgnoredDuringExecution:
  - weight: 100
    preference:
      matchExpressions:
      - key: topology.kubernetes.io/zone
        operator: In
        values:
        - us-east-1a
  - weight: 50
    preference:
      matchExpressions:
      - key: topology.kubernetes.io/zone
        operator: In
        values:
        - us-east-1b
```

3. **Cost optimization**: Prefer standard nodes but allow premium nodes when needed

```yaml
nodeAffinity:
  preferredDuringSchedulingIgnoredDuringExecution:
  - weight: 100
    preference:
      matchExpressions:
      - key: node-type
        operator: In
        values:
        - standard
```

## Best Practices

1. **Use labels consistently**: Develop a consistent labeling strategy for your nodes
2. **Combine with other scheduling features**: Use with pod affinity, anti-affinity, taints, and tolerations for complex scheduling requirements
3. **Don't over-constrain**: Very specific affinity rules might prevent pods from scheduling
4. **Test scheduling behavior**: Verify that your affinity rules work as expected
5. **Document your affinity rules**: Make sure team members understand the placement constraints

Node Affinity is a powerful tool for ensuring your workloads run on appropriate nodes while allowing flexibility in your Kubernetes cluster management.
