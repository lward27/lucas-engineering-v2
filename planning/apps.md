# Apps Deployment

## Decisions

- v2 uses Flux Kustomizations split by phase: sources, foundations, observability, CI/registry, core apps, then agent/product apps.
- The core and agent/product phases are present but suspended because they depend on private images, app credentials, or remote Cloudflare routes that are not safe to invent in Git.
- Public exposure stays out of Cilium Gateway and ingress-nginx. Cloudflare Tunnel routes must point directly at service DNS names after the target service is Ready.
- The registry, Tekton, and observability ingress templates are disabled in values because v2 exposure is Cloudflare-managed.
- RabbitMQ is represented locally instead of by copying the old raw manifest because the old manifest committed static credential values.

## Backlog

- Add Terraform or Cloudflare API automation for tunnel public hostnames if this repo should own remote Cloudflare route state.
- Vendor or convert raw old manifest directories into real Helm charts or Kustomizations before enabling homepage, Directus, Ghost, finance services, scraper manager, yfinance wrapper, and RAG service.
- Seed `registry-v2.lucas.engineering` before switching private-image workloads away from `registry.lucas.engineering`.
- Add a post-registry activation pass that unsuspends `apps-core`, then `apps-agent-products`, after the required secrets exist and image pulls are verified.
- Replace old Argo/cert-manager/ingress assumptions inside upstream charts so values can disable every Ingress cleanly.
