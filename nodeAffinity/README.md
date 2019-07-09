#Affinity and anti-affinity

Source: https://kubernetes.io/docs/concepts/configuration/assign-pod-node/

The key enhancements are:

- the language is more expressive (not just “AND of exact match”)
- you can indicate that the rule is “soft”/“preference” rather than a hard requirement, so if the scheduler can’t satisfy it, the pod will still be scheduled
- you can constrain against labels on other pods running on the node (or other topological domain), rather than against labels on the node itself, which allows rules about which pods can and cannot be co-located

The behavior similar to `nodeSelector`, but more flexible

##Node affinity

There are the following types of affinities:

already implemented:
- requiredDuringSchedulingIgnoredDuringExecution
- preferredDuringSchedulingIgnoredDuringExecution

planned in the nearest future:

- requiredDuringSchedulingRequiredDuringExecution
- requiredDuringSchedulingIgnoredDuringExecution

Example:

```
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
            - e2e-az1
            - e2e-az2
      preferredDuringSchedulingIgnoredDuringExecution:
      - weight: 1
        preference:
          matchExpressions:
          - key: another-node-label-key
            operator: In
            values:
            - another-node-label-value
  containers:
  - name: with-node-affinity
    image: k8s.gcr.io/pause:2.0
```

This node affinity rule says the pod can only be placed on a node with a label whose key is kubernetes.io/e2e-az-name and whose value is either e2e-az1 or e2e-az2. In addition, among nodes that meet that criteria, nodes with a label whose key is another-node-label-key and whose value is another-node-label-value should be preferred.

You can see the operator In being used in the example. The new node affinity syntax supports the following operators: In, NotIn, Exists, DoesNotExist, Gt, Lt. You can use NotIn and DoesNotExist to achieve node anti-affinity behavior, or use node taints to repel pods from specific nodes.

!!! If you specify both nodeSelector and nodeAffinity, both must be satisfied for the pod to be scheduled onto a candidate node.

## Inter-pod affinity and anti-affinity
As with node affinity, there are currently two types of pod affinity and anti-affinity, called requiredDuringSchedulingIgnoredDuringExecution and preferredDuringSchedulingIgnoredDuringExecution which denote “hard” vs. “soft” requirements. See the description in the node affinity section earlier. An example of requiredDuringSchedulingIgnoredDuringExecution affinity would be “co-locate the pods of service A and service B in the same zone, since they communicate a lot with each other” and an example preferredDuringSchedulingIgnoredDuringExecution anti-affinity would be “spread the pods from this service across zones” (a hard requirement wouldn’t make sense, since you probably have more pods than zones).

Example:

```
apiVersion: v1
kind: Pod
metadata:
  name: with-pod-affinity
spec:
  affinity:
    podAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
      - labelSelector:
          matchExpressions:
          - key: security
            operator: In
            values:
            - S1
        topologyKey: failure-domain.beta.kubernetes.io/zone
    podAntiAffinity:
      preferredDuringSchedulingIgnoredDuringExecution:
      - weight: 100
        podAffinityTerm:
          labelSelector:
            matchExpressions:
            - key: security
              operator: In
              values:
              - S2
          topologyKey: failure-domain.beta.kubernetes.io/zone
  containers:
  - name: with-pod-affinity
    image: k8s.gcr.io/pause:2.0
```
