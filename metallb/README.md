# MetalLB

[MetalLB](https://metallb.io) is a bare-metal load balancer that exposes services via ARP or BGP.
In this cluster, we're only using ARP mode, because it's simpler to set up and works well for my use-case.

## Why is this needed?

First off, it's worth mentioning that this cluster aims to be Highly Available.
That means that it can recover from node failures without any downtime, including for Pod storage and networking too.

Without a service load balancer, core services like Traefik wouldn't be accessible from outside the cluster without binding it to a specific node.
This is because incoming traffic is passed to the cluster via a static IP address configured in my router's port forwarding rules.
Hence, we need a way to expose services on a static IP address, but also allow the node to change at any time. This is what MetalLB does.