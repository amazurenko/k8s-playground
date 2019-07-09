source: https://kubernetes.io/docs/concepts/configuration/assign-pod-node/

Useful commands:
Create node Label: `kubectl label nodes <node-name> <label-key>=<label-value>`
Delete node LabelL `kubectl label nodes <node-name> <label-key>-`
Show labels: `kubectl get nodes --show-labels`



Examples
kubectl label nodes k8s-worker-1 disktype=ssd
kubectl get nodes --show-labels
kubectl describe node k8s-worker-1


Scenario:
- Create node label for worker1: disktype=ssd
- Create node label for worker2: disktype=hdd
- Deploy pod with set nodeSelector multiple times, check behaviour
- delete label from node, which used in pod definition
- deploy pode and check behavior
- return node label back and check status of pod
