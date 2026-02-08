# Hydra Infrastructure

This repository contains ArgoCD Applications for the core infrastructure of Hydra.
New Applications added to `applications/` will be deployed automatically to the cluster under the `infrastructure` Project.
See [Jack-Gledhill/hydra-bootstrap](https://github.com/Jack-Gledhill/hydra-bootstrap) for the ArgoCD manifests that bootstrap this repository onto a Kubernetes cluster.

## Sync Waves

In short, Sync Waves control the order in which manifests are applied to the cluster - with lower numbered waves being applied before higher numbered waves.
See [the ArgoCD documentation](https://argo-cd.readthedocs.io/en/stable/user-guide/sync-waves/) for more information on Sync Waves.

Wave 0:

- Kube State Metrics
- MetalLB
- NFS CSI
- Node Exporter
- Local Path Provisioner
- Reloader
- Sealed Secrets

Wave 1:

- Cert Manager
- CloudNative Postgres
- Prometheus Operator
- Traefik