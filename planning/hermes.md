# Hermes Deployment

## Decisions

- Hermes runs as a StatefulSet in `lucas-engineering-v2`; the desktop endpoint and external `EndpointSlice` mode are not used.
- Hermes uses Fireworks' OpenAI-compatible endpoint with `accounts/fireworks/models/glm-5p2` as its sole configured provider and default model.
- The v2 `hermes-api-keys` secret contains only `FIREWORKS_API_KEY` and `TELEGRAM_BOT_TOKEN`. SSH, kubeconfig, and GitHub credentials are intentionally not copied.
- The chart's kubectl and GitHub CLI tools are disabled to keep the in-cluster agent capability set narrow by default.
- Hermes is deployed through its own Flux Kustomization, independent of the suspended agent/product phase, so it cannot trigger unrelated releases.
- The dashboard remains a ClusterIP service. Cloudflare Tunnel must route to the dashboard service, and Cloudflare Access must protect the published hostname.
- The custom network policy accepts dashboard traffic only from the `cloudflared` namespace and permits DNS plus outbound HTTPS for Fireworks and Telegram.
- The single v2 node has only 100m CPU request headroom, so Hermes requests 80m CPU and 224Mi memory across its three containers while retaining the chart's burst limits.

## Backlog

- Decide which Hermes plugins, if any, should be explicitly installed and enabled after the baseline Telegram workflow is proven.
- Add a machine-readable smoke test for Telegram polling and a Fireworks inference request that does not expose credentials.
- Add a Cloudflare Access service-token strategy before exposing any non-browser Hermes API endpoint.
- Revisit existing platform workload requests or add node capacity before assigning Hermes sustained CPU-intensive work.
