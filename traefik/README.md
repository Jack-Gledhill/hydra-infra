# Traefik

[Traefik](https://traefik.io) is the cluster's ingress controller, handling all external traffic and routing it to the appropriate services.
This integrates nicely with [MetalLB](https://metallb.io) and [Cert Manager](https://cert-manager.io), with the former providing High Availability to Traefik, and the latter ensuring routes are properly secured with valid SSL certificates.

## Why not use the Traefik CRDs?

I've used them before. They work quite well, but I find I just don't need a lot of the functionality they provide.
Instead, I prefer to use the standard Kubernetes Ingress resource.

## What about TCP/UDP routing?

While Traefik does have native support for this, I won't be using it.
Instead, services will be exposed directly via MetalLB, which reduces complexity and the dependency chain.

## What about middleware?

I've added the Middleware CRD to the cluster, so middlewares can be added to Ingress through [these annotations](https://doc.traefik.io/traefik/routing/providers/kubernetes-ingress/#on-ingress).

## Why DaemonSet instead of Deployment?

Both Deployment and DaemonSet would've done the job when it comes to High Availability.
Ultimately, DaemonSet is slightly faster to write the manifests for and makes things slightly simpler to understand.
Traefik is written in Go and is incredibly efficient, so I'm not concerned about wasted compute resources.

Were I to use a Deployment instead, I would specify a `replicas` count that is equal to quorum, and add Pod anti-affinity rules.