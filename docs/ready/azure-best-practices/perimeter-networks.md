---
title: Perimeter networks
description: Learn about perimeter networks, and see how to use Azure components to set up effective perimeter networks for your organization.
author: tracsman
ms.author: jonor
ms.date: 01/18/2023
ms.topic: conceptual
ms.custom: think-tank, virtual-network
---

# Perimeter networks

Perimeter networks, sometimes called demilitarized zones (DMZs), help provide secure connectivity between cloud networks, on-premises or physical datacenter networks, and the internet. 

In effective perimeter networks, incoming packets flow through security appliances that are hosted in secure subnets, before the packets can reach back-end servers. Security appliances include firewalls, network virtual appliances (NVAs), and other intrusion detection and prevention systems. Internet-bound packets from workloads must also flow through security appliances in the perimeter network before they can leave the network.

Usually, central IT teams and security teams are responsible for defining operational requirements for perimeter networks. Perimeter networks can provide policy enforcement, inspection, and auditing.

Perimeter networks can use the following Azure features and services:

- [Virtual networks][virtual-networks], [user-defined routes][user-defined-routes], and [network security groups (NSGs)][network-security-groups]
- [Azure Firewall][azure-firewall]
- [Azure Web Application Firewall][appgwwaf] on [Azure Application Gateway][appgw]
- [Azure Web Application Firewall][afdwaf] on [Azure Front Door][afd]
- [Other network virtual appliances (NVAs)][network-virtual-appliances]
- [Azure Load Balancer][alb]
- [Public IP addresses][public-ip]

For more information about perimeter networks, see [The virtual datacenter: A network perspective][perimeter-network].

For example templates you can use to implement your own perimeter networks, see the reference architecture [Implement a secure hybrid network](/azure/architecture/reference-architectures/dmz/secure-vnet-dmz).

## Perimeter network topology

The following diagram shows an example [hub and spoke network](./hub-spoke-network-topology.md) with perimeter networks that enforce access to the internet and to an on-premises network.

![Diagram that shows an example of a hub and spoke network topology with two perimeter networks.](../../_images/azure-best-practices/network-high-level-perimeter-networks.png)

The perimeter networks are connected to the DMZ hub. In the DMZ hub, the perimeter network to the internet can scale up to support many lines of business. This support uses multiple farms of web application firewalls (WAFs) and Azure Firewall instances that help protect the spoke virtual networks. The hub also allows connectivity to on-premises or partner networks via virtual private network (VPN) or Azure ExpressRoute as needed.

## Virtual networks

Perimeter networks are typically built within virtual networks. The virtual network uses multiple subnets to host the different types of services that filter and inspect traffic to or from other networks or the internet. These services include NVAs, WAFs, and Application Gateway instances.

## User-defined routes

In a hub and spoke network topology, you must guarantee that traffic generated by virtual machines (VMs) in the spokes passes through the correct virtual appliances in the hub. This traffic routing requires [user-defined routes][user-defined-routes] in the subnets of the spokes.

User-defined routes can guarantee that traffic passes through specified custom VMs, NVAs, and load balancers. The route sets the front-end IP address of the internal load balancer as the next hop. The internal load balancer distributes the internal traffic to the virtual appliances in the load balancer back-end pool.

You can use user-defined routes to direct traffic through firewalls, intrusion detection systems, and other virtual appliances. Customers can route network traffic through these security appliances for security boundary policy enforcement, auditing, and inspection. 

## Azure Firewall

[Azure Firewall][azure-firewall] is a managed cloud-based firewall service that helps protect your resources in virtual networks. Azure Firewall is a fully stateful managed firewall with built-in high availability and unrestricted cloud scalability. You can use Azure Firewall to centrally create, enforce, and log application and network connectivity policies across subscriptions and virtual networks.

Azure Firewall uses a static public IP address for virtual network resources. External firewalls can use the static public IP to identify traffic that originates from your virtual network. Azure Firewall works with Azure Monitor for logging and analytics.

## Network virtual appliances

You can manage perimeter networks with access to the internet through Azure Firewall or through a farm of firewalls or WAFs. An Azure Firewall instance and an NVA firewall can use a common administration plane with a set of security rules. These rules help protect the workloads hosted in the spokes, and control access to on-premises networks. Azure Firewall has built-in scalability, and NVA firewalls can be manually scaled behind a load balancer.

Different lines of business use many different web applications, which can suffer from various vulnerabilities and potential exploits. A WAF detects attacks against HTTP/S web applications in more depth than a generic firewall. Compared with traditional firewall technology, WAFs have a set of specific features to help protect internal web servers from threats.

A firewall farm has less specialized software than a WAF, but also has a broader application scope to filter and inspect any type of egress and ingress traffic. If you use an NVA approach, you can find and deploy software from the Azure Marketplace.

Use one set of Azure Firewall instances or NVAs for traffic that originates on the internet, and another set for traffic that originates on-premises. Using only one set of firewalls for both kinds of traffic is a security risk, because there's no security perimeter between the two sets of network traffic. Using separate firewall layers reduces the complexity of checking security rules, and clarifies which rules correspond to which incoming network requests.

## Azure Load Balancer

[Azure Load Balancer][alb] offers a high-availability Layer 4 transmission control protocol/user datagram protocol (TCP/UDP) load balancing service. This service can distribute incoming traffic among service instances defined by a load-balancing set. Load Balancer can redistribute traffic from front-end public or private IP endpoints, with or without address translation, to a pool of back-end IP addresses like NVAs or VMs.

Load Balancer can also probe the health of the server instances. When an instance fails to respond to a probe, the load balancer stops sending traffic to the unhealthy instance.

In the hub and spoke network topology example, you deploy an external load balancer to both the hub and the spokes. In the hub, the load balancer efficiently routes traffic to services in the spokes. The load balancer in the spokes manages application traffic.

## Azure Front Door

[Azure Front Door][afd] is a highly available and scalable web application acceleration platform and global HTTPS load balancer. You can use Azure Front Door to build, operate, and scale out your dynamic web application and static content. Azure Front Door runs in more than 100 locations at the edge of Microsoft's global network.

Azure Front Door provides your application with:

- Unified regional and stamp maintenance automation.
- Business continuity and disaster recovery (BCDR) automation.
- Unified client and user information.
- Caching.
- Service insights.

Azure Front Door offers performance, reliability, and service-level agreements (SLAs) for support. Azure Front Door also offers compliance certifications and auditable security practices that Azure develops, operates, and natively supports.

## Application Gateway

[Application Gateway][appgw] is a dedicated virtual appliance that gives you a managed application delivery controller. Application Gateway offers various Layer 7 load-balancing capabilities for your application.

Application Gateway helps you optimize web farm productivity by offloading CPU-intensive secure socket layer (SSL) termination. Application Gateway also provides other Layer 7 routing capabilities, such as:  

- Round-robin distribution of incoming traffic.
- Cookie-based session affinity.
- URL path-based routing.
- Hosting multiple websites behind a single application gateway.

The [Application Gateway with WAF SKU][appgwwaf] includes a WAF, and provides protection to web applications from common web vulnerabilities and exploits. You can configure Application Gateway as an internet-facing gateway, an internal-only gateway, or a combination of both.

## Public IPs

With some Azure features, you can associate service endpoints to a [public IP][public-ip] address. This option provides access to your resource from the internet. The endpoint uses network address translation (NAT) to route traffic to the internal address and port on the Azure virtual network. This path is the main way for external traffic to pass into the virtual network. You can configure public IP addresses to control what traffic passes in, and how and where it's translated into the virtual network.

## Azure DDoS Protection

[Azure DDoS Protection][ddos] provides extra mitigation capabilities to help protect your Azure resources in virtual networks from distributed denial of service (DDoS) attacks. DDoS Protection has two SKUs, DDoS IP Protection and DDoS Network Protection. For more information, see [About Azure DDoS Protection SKU Comparison](/azure/ddos-protection/ddos-protection-sku-comparison).

DDoS Protection is easy to enable and requires no application changes. You can tune protection policies through dedicated traffic monitoring and machine-learning algorithms. DDoS Protection applies protection to IPv4 Azure public IP addresses that are associated with resources deployed in virtual networks. Example resources include Load Balancer, Application Gateway, and Azure Service Fabric instances.

Real-time telemetry is available through Azure Monitor views, both during an attack and for historical purposes. You can add application-layer protection by using Web Application Firewall in Application Gateway. 

## Next steps

Learn how to efficiently manage common communication or security requirements by using the [hub and spoke network topology](hub-spoke-network-topology.md) model.

<!-- links -->

[virtual-networks]: /azure/virtual-network/virtual-networks-overview
[network-security-groups]: /azure/virtual-network/network-security-groups-overview
[user-defined-routes]: /azure/virtual-network/virtual-networks-udr-overview#user-defined
[network-virtual-appliances]: /azure/architecture/reference-architectures/dmz/nva-ha
[azure-firewall]: /azure/firewall/overview
[perimeter-network]: ../../resources/networking-vdc.md
[alb]: /azure/load-balancer/load-balancer-overview
[ddos]: /azure/ddos-protection/ddos-protection-overview
[public-ip]: /azure/virtual-network/ip-services/virtual-network-public-ip-address
[afd]: /azure/frontdoor/front-door-overview
[afdwaf]: /azure/web-application-firewall/afds/afds-overview
[appgw]: /azure/application-gateway/overview
[appgwwaf]: /azure/web-application-firewall/ag/ag-overview
