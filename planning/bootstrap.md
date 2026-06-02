# Bootstrap

## Decisions

- Use the `lucas-engineering-v2` kube context only. The older `lucas-engineering` cluster is out of scope.
- Bootstrap Flux from committed manifests instead of `flux bootstrap github` because no GitHub token is available in the shell. The repository is public, so the cluster can reconcile it over HTTPS without a deploy key.
- Keep the Flux root at `clusters/lucas-engineering-v2` and split infrastructure into Flux-managed child Kustomizations. Gateway API CRDs reconcile before Cilium so the service mesh path has its API surface before the Cilium Helm upgrade.
- Preserve the existing Cilium Helm chart version and k3s-specific values, then add `gatewayAPI.enabled=true`. Upgrading Cilium is a separate change, not part of bootstrapping.
- Restart Cilium agent, Envoy, and operator after the Helm upgrade. The chart updated the ConfigMap and release state, but the already-running pods did not pick up the Gateway API controller settings until restarted.
- Reverted the first Cloudflare/Gateway/Flux Web UI attempt after the v2 API endpoint stopped responding over TCP during reconciliation. Keep the rollback in Git until the node is inspected from console or local access.
- Retry starts with the Cloudflare tunnel runner only, using the manually created `cloudflared/tunnel-token` secret and pinning `cloudflare/cloudflared` to `2026.5.2`.
- Reverted the later Cilium Gateway-only step after reproducing the TCP blackhole to `192.168.20.242:6443` over the VPN path. Keep v2 Cloudflare origins pointed directly at in-cluster services until the Gateway/VPN interaction is understood.
- Add a startup probe to `cloudflared` because `/ready` can fail while edge discovery and QUIC registration are still settling.
- Add Flux Web UI without Gateway API. Cloudflare should route `flux-v2.lucas.engineering` directly to the `flux-web` service in `flux-system`; use this first-level hostname instead of `flux.v2.lucas.engineering` so Cloudflare Universal SSL can cover it without deeper-subdomain certificate work.
- Expose Hubble UI through Cloudflare Tunnel at `hubble-v2.lucas.engineering`, routed directly to `hubble-ui.kube-system.svc.cluster.local:80`, and protect it with Cloudflare Access because Hubble UI has no built-in external auth boundary.

## Backlog

- Decide the external Gateway exposure model. The cluster currently has Cilium LB IPAM enabled but no committed LoadBalancer IP pool; Gateway listeners will need a pool or host-network mode before they are useful outside the cluster.
- Add a smoke-test workload and HTTPRoute once an application namespace exists. That should verify Gateway API routing through Cilium rather than just controller readiness.
- Decide whether the Cilium cluster name should move from `default` to `lucas-engineering-v2`. It is cosmetic for this single cluster today, but should be intentional before clustermesh or shared identity assumptions are introduced.
- Add a managed rollout trigger for future Cilium config changes if this repo starts changing Helm values often.
- Retry Flux Web UI without Cilium Gateway by pointing the Cloudflare tunnel public hostname directly at the in-cluster Flux Web service.
- Investigate why creating a Cilium Gateway for `*.v2.lucas.engineering` breaks TCP access to the node over `utun4` while ICMP continues. Do not reintroduce Gateway API exposure until this is explained.
