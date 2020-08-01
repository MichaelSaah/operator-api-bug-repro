1) Apply `bundle.yaml` to bare k8s cluster w/ `kubectl apply -f bundle.yaml`. Wait until the operator is up and ready.
2) Apply `ssl-secrets.yaml` and `secrets.yaml` similarly.
3) Apply `cr.yaml` with `kubectl apply -f cr.yaml`. This is a valid pxc cr with minor changes from the cr provided in the operator repo.
4) Observe the pxc come up normally. It can be logged into at this point to verify functionality.
5) Now submit `cr-api.json` to the operator API. This is a json verison of the manifest found in `cr-api.yaml`, which differs from `cr.yaml` only in the pxc's name. Submission can be done by running `kubectl proxy --port=8080` in one shell and `curl 127.0.0.1:8080/apis/pxc.percona.com/v1-5-0/namespaces/default/perconaxtradbclusters -X POST -d @./cr-api.json -H "Content-Type: Application/json"` in another.
6) Observe that this pxc comes up normally and reports itself as "ready," but the HAProxy log will display an error like this:
`ERROR 1045 (28000): Access denied for user 'monitor'@'10-2-11-2.cluster2-haproxy-replicas.default.svc.cluster.local' (using password: YES)`. Observe that cluster2 cannot be logged into via HAProxy.
7) At this point, the bug has been replicated. To understand the source of the bug, see `notes.md` in this directory.
