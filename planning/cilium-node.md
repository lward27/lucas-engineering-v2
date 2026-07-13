# Cilium Worker Node

## Decisions

- Cilium uses the control-plane API address `192.168.20.242:6443` rather than `127.0.0.1:6443`. Loopback only works on the control-plane node and prevents Cilium agents on workers from initializing.
- Retain the existing Cilium version, VXLAN tunnel mode, pod CIDR, Gateway API setting, and kube-proxy replacement. The worker rollout failure is solely an incorrect Kubernetes API endpoint.

## Backlog

- Replace the fixed control-plane IP with a stable internal DNS name or virtual IP before introducing additional control-plane nodes.
- Add a worker-node admission smoke test that verifies Cilium initializes and the node reaches `Ready` after every node join.
