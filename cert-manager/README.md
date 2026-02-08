# Cert Manager

[Cert Manager](https://cert-manager.io) automatically provisions SSL certificates for the cluster's ingress routes.
It provides all services exposed via [Traefik](https://traefik.io) with a certificate from [Let's Encrypt](https://letsencrypt.org).

## Why not use Traefik's built-in ACME client?

As per the [docs](https://doc.traefik.io/traefik/providers/kubernetes-ingress/#letsencrypt-support-with-the-ingress-provider), Traefik's built-in ACME client doesn't support High-Availability environments.
Since Traefik handles the entire cluster's ingress, it is critical infrastructure, and so needs to be Highly Available.

Additionally, Cert Manager is a lot more feature-rich and can be used for services outside Traefik.

## Why the DNS01 challenge?

While Cert Manager has great support for both `http01` and `dns01` challenges, `dns01` is more reliable for my use-case.
This is because `dns01` only relies on having a good connection with the Cloudflare API, while `http01` requires the cluster to be accessible from the internet, where there a number of potential points of failure.
It would also require me to have an ingress controller already in place, even though the ingress controller I'm using relies on Cert Manager working, creating an unnecessary cyclic dependency.

Additionally, the `dns01` challenge allows wildcard certificates.

## Why not use a static certificate?

While I could easily generate some certificates with Cloudflare and pass them to Traefik manually, this is not as cool.
Cert Manager completely automates the process of renewing certificates, reduces the amount of stuff I need to manually add to the cluster and makes me feel really smart all in one go.

## Why use a ClusterIssuer?

Because the majority of certificate requests will be for the same domain, so there's no point in having to specify the configuration multiple times.

In the rare exceptions where other domains may be used, I will create scoped Issuers within the namespaces they're needed.

## How are certificates requested?

Certificates are requested by adding an annotation to the Ingress resource, rather than directly through Traefik.
This allows Ingress resources to use different Issuers if needed, while still using the same Traefik configuration.

## Adding new domains

Cert Manager can issue certificates for any domain that we can prove ownership of.
Hence, tenants can configure Hydra to issue certificates for their own domain simply by [adding an Issuer](https://cert-manager.io/docs/configuration/acme/).

## Development Log

### 7th July 2025

Was having issues with `ClusterIssuer` failing to apply due to the validating webhook failing.
Ostensibly, the issue was the certificate used for the webhook was not signed by a trusted authority.

After many hours of digging, I figured out that cert-manager's `cainjector` pod was meant to add this certificate to the cluster's trusted CA bundle, but it was failing to do so.
This was because the `cainjector` pod was silently failing to create lease objects (used for leader election) in the `kube-system` namespace on account of missing permissions.
It wasn't until I had taken a deep look at the RBAC manifests that I realised that permissions for lease objects was controlled by a Role that was meant to be in the `kube-system` namespace.
The namespace was being overwritten by kustomize to be in the `cert-manager` namespace, which meant `cainjector` was unable to create the objects it needed and therefore failed to add the certificate to the cluster's trusted CA bundle.

The fix was infuriatingly simple: remove the namespace field from the Kustomize object.
