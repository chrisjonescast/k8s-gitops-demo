# k8s-gitops-demo

A deliberately tiny GitOps playground for `eks-11081-cj`.

**Argo CD watches this repo.** It does not wait for a push notification. It polls,
diffs this folder against the live cluster, and corrects any drift it finds.

```
apps/demo/          <- what Argo syncs
  namespace.yaml
  deployment.yaml
  service.yaml
```

## The experiment that makes GitOps click

1. Change `replicas: 2` to `replicas: 4` in `apps/demo/deployment.yaml`
2. Commit and push to `main`
3. Watch Argo CD sync it, or wait for the poll (default 3 minutes)

Nobody ran `kubectl apply`. The cluster converged because the desired state in git
changed.

## The half people forget

Delete a pod by hand:

```bash
kubectl -n demo delete pod -l app=demo-web
```

Argo recreates it within the self-heal window, because the live state drifted from
git. That is the difference between this and a CI pipeline that only runs `kubectl
apply` on push: a CI pipeline does not fix drift, Argo CD does.

## Tear down

```bash
kubectl -n argocd delete application demo
```

With `prune: true` set, deleting the Application does not delete the workloads.
Delete the namespace instead:

```bash
kubectl delete namespace demo
```
