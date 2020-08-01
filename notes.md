With both `cluster1` and `cluster2` deployed via the steps given in `reproduction-steps.md`, run the following commands:
`kubectl exec -it cluster1-pxc-0 env | grep MONITOR`
`kubectl exec -it cluster2-pxc-0 env | grep MONITOR`

You should observe that `cluster1` reports the env vars `MONITOR_HOST=%` and `MONITOR_PASSWORD=monitor`, while `cluster2` only reports `MONITOR_PASSWORD=monitor`.

Now observe this: https://github.com/percona/percona-xtradb-cluster-operator/blob/v1.5.0/build/pxc-entrypoint.sh#L296

Without `MONITOR_HOST` being set, `localhost` is used as the default in the pxc startup script. This results in the `monitor` user being created with host `localhost`, which prevents remote clients such as HAProxy from connecting to it.

To understand why `MONITOR_HOST` is not being set to `%` on the pxc submitted directly to the API, observe this block: https://github.com/percona/percona-xtradb-cluster-operator/blob/v1.5.0/pkg/pxc/app/statefulset/node.go#L150

In the above code, `MONITOR_HOST` is set to `%` if the pxc's API version is >= v1.1.0. Clearly both of our pxc's API versions are v1.5.0, so we must go still deeper.

The PXC's version is set here: https://github.com/percona/percona-xtradb-cluster-operator/blob/v1.5.0/pkg/apis/pxc/v1/pxc_types.go#L523

Of note is the fact that this function checks the annotation `kubectl.kubernetes.io/last-applied-configuration` for the API version. If you check the pxc object from cluster1 w/ `kubectl get pxc cluster1 -oyaml`, you will see that this annotation is set. This behavior is noted in the kubctl docs: https://kubernetes.io/docs/tasks/manage-kubernetes-objects/declarative-config/

Furthermore, both `cluster1` and `cluster2`'s pxc objects report `apiVersion: pxc.percona.com/v1`, when this function is looking for something like `apiVersion: pxc.percona.com/v1-5-0`. I do not know why the minor and patch version numbers are being stripped.

To sumamrize, the `monitor` user's host is being set incorrectly because of the lack of the env var `MONITOR_HOST`, which is used by the pxc-entrypoint script to create the user. This seems to be caused by incorrect parsing of the pxc object's api version by the operator code.
