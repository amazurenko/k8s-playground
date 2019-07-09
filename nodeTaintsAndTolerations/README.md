# Taints ant Tolerations

source: https://kubernetes.io/docs/concepts/configuration/taint-and-toleration/

UseCases:
- Dedicated Nodes
- Nodes with Special Hardware
- Taint based Evictions (A per-pod-configurable eviction behavior when there are node problems, which is described in the next section)


Useful commands:

`kubectl taint nodes node1 key=value:NoSchedule`
places a taint on node node1. The taint has key key, value `value`, and taint effect `NoSchedule`. This means that no pod will be able to schedule onto node1 unless it has a matching toleration.

To remove the taint added by the command above, you can run:

`kubectl taint nodes node1 key:NoSchedule-`

There're the following actions:
- NoSchedule
- NoExecute
- PreferNoSchedule


You can put multiple taints on the same node and multiple tolerations on the same pod. The way Kubernetes processes multiple taints and tolerations is like a filter: start with all of a node’s taints, then ignore the ones for which the pod has a matching toleration; the remaining un-ignored taints have the indicated effects on the pod. In particular,

- if there is at least one un-ignored taint with effect NoSchedule then Kubernetes will not schedule the pod onto that node
- if there is no un-ignored taint with effect NoSchedule but there is at least one un-ignored taint with effect PreferNoSchedule then Kubernetes will try to not schedule the pod onto the node
- if there is at least one un-ignored taint with effect NoExecute then the pod will be evicted from the node (if it is already running on the node), and will not be scheduled onto the node (if it is not yet running on the node).

For example, imagine you taint a node like this:
```
kubectl taint nodes node1 key1=value1:NoSchedule
kubectl taint nodes node1 key1=value1:NoExecute
kubectl taint nodes node1 key2=value2:NoSchedule
```

And a Pod has two tolerations:
```
tolerations:
- key: "key1"
  operator: "Equal"
  value: "value1"
  effect: "NoSchedule"
- key: "key1"
  operator: "Equal"
  value: "value1"
  effect: "NoExecute"
```

!!! In this case, the pod will not be able to schedule onto the node, because there is no toleration matching the third taint. 
But it will be able to continue running if it is already running on the node when the taint is added, because the third taint is the only one of the three that is not tolerated by the pod.

Normally, if a taint with effect `NoExecute` is added to a node, then any pods that do not tolerate the taint will be evicted immediately, and any pods that do tolerate the taint will never be evicted. However, a toleration with `NoExecute` effect can specify an optional `tolerationSeconds` field that dictates how long the pod will stay bound to the node after the taint is added. For example,

```
tolerations:
- key: "key1"
  operator: "Equal"
  value: "value1"
  effect: "NoExecute"
  tolerationSeconds: 3600
```
