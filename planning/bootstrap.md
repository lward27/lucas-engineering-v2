# Bootstrap

## Decisions

- Use the `lucas-engineering-v2` kube context only. The older `lucas-engineering` cluster is out of scope.
- Bootstrap Flux from committed manifests instead of `flux bootstrap github` because no GitHub token is available in the shell. The repository is public, so the cluster can reconcile it over HTTPS without a deploy key.
- Keep the Flux root at `clusters/lucas-engineering-v2` and split infrastructure into Flux-managed child Kustomizations. Gateway API CRDs reconcile before Cilium so the service mesh path has its API surface before the Cilium Helm upgrade.
- Preserve the existing Cilium Helm chart version and k3s-specific values, then add `gatewayAPI.enabled=true`. Upgrading Cilium is a separate change, not part of bootstrapping.
- Restart Cilium agent, Envoy, and operator after the Helm upgrade. The chart updated the ConfigMap and release state, but the already-running pods did not pick up the Gateway API controller settings until restarted.

## Backlog

- Decide the external Gateway exposure model. The cluster currently has Cilium LB IPAM enabled but no committed LoadBalancer IP pool; Gateway listeners will need a pool or host-network mode before they are useful outside the cluster.
- Add a smoke-test workload and HTTPRoute once an application namespace exists. That should verify Gateway API routing through Cilium rather than just controller readiness.
- Decide whether the Cilium cluster name should move from `default` to `lucas-engineering-v2`. It is cosmetic for this single cluster today, but should be intentional before clustermesh or shared identity assumptions are introduced.
- Add a managed rollout trigger for future Cilium config changes if this repo starts changing Helm values often.
