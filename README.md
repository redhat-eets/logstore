# logstore

There is an issue in kubelet that results in log lines from before a log rotation being unavailable via the API (e.g. `kubectl logs ...`). Log following (`kubectl logs --follow ...`) does not avoid issues with log rotation; it silently stops receiving new lines when a log rotation occurs. Those issues mean that there is no reliable way to collect unbroken logs from a cluster through the k8s API.

This tool works around that issue by creating a debug pod on the node, downloading all old logs for the pod including ones that have been rotated and archived, then merging them together and discarding any older than the requested duration.

It has only been tested on a Single-Node Openshift cluster.

It requires the `oc` tool available locally and a sufficiently privileged cluster account to run a debug pod on the node.

## Usage

By default this tool retrieves logs for the `linuxptp-daemon-container` from the openshift-ptp operator:

```shell
go run main.go --kubeconfig ~/kubeconfig-puffin  --duration 100s --output ./logs.log
```

To get logs for a different container you must specify the namespace, pod, and container. The namespace and pod name can be file globs to handle auto-generated names:
```shell
go run main.go --kubeconfig ~/kubeconfig-puffin --duration 100s --output ./logs.log --namespace 'openshift-network-operator' --pod 'network-operator-*' --container network-operator
```

## Limitations

* Not timezone-aware for calculating the duration of logs.
* Will not retrieve log lines that happen after downloading has started.
* Does not support streaming logs - if you need this behaviour look at the vse-sync-collection-tools.
