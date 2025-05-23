---
title: Logging and reporting decision guide
description: Develop a core logging, reporting, and monitoring strategy to ensure your organization meets uptime, security, and policy compliance goals.
author: Zimmergren
ms.author: tozimmergren
ms.date: 10/12/2022
ms.topic: conceptual
ms.subservice: caf-general
ms.custom: internal
---

# Logging and reporting decision guide

All organizations need mechanisms for notifying IT teams of performance, uptime, and security issues before they become serious problems. A successful monitoring strategy shows you how the individual components that make up your workloads and networking infrastructure are performing. Within the context of a public cloud migration, integrating logging and reporting with any of your existing monitoring systems, is critical to success. Also, surfacing important events and metrics to the appropriate IT staff is vital in ensuring your organization meets its uptime, security, and policy compliance goals.

:::image type="content" source="../../_images/decision-guides/decision-guide-logging-and-reporting.png" alt-text="Diagram that shows the plotting logging, reporting, and monitoring options from least to most complex, aligned with jump links below." lightbox="../../_images/decision-guides/decision-guide-logging-and-reporting.png":::

Jump to: [Planning your monitoring infrastructure](#plan-your-monitoring-infrastructure) | [Cloud-native](#cloud-native) | [On-premises extension](#on-premises-extension) | [Gateway aggregation](#gateway-aggregation) | [Hybrid monitoring (on-premises)](#hybrid-monitoring-on-premises) | [Hybrid monitoring (cloud-based)](#hybrid-monitoring-cloud-based) | [Multicloud](#multicloud) | [Learn more](#learn-more)

The inflection point when determining a cloud logging and reporting strategy is based primarily on:

- Existing investments your organization has made in operational processes.
- Any requirements you have to support a multicloud strategy.

Activities in the cloud are logged and reported in multiple ways. Cloud-native and centralized logging are two common managed service options that are driven by the subscription design and the number of subscriptions.

## Plan your monitoring infrastructure

When planning your deployment, consider where your logging data stores and how you'll integrate cloud-based reporting and monitoring services with your existing processes and tools.

| Question | Cloud-native | On-premises extension | Hybrid monitoring | Gateway aggregation |
|-----|-----|-----|-----|-----|
| Do you have an existing on-premises monitoring infrastructure? | No | Yes | Yes |  No |
| Do you have requirements preventing storage of log data on external storage locations? | No | Yes | No | No |
| Do you need to integrate cloud monitoring with on-premises systems? | No | No | Yes | No |
| Do you need to process or filter telemetry data before submitting it to your monitoring systems? | No | No | No | Yes |

### Cloud-native

A cloud-native SaaS solution such as [Azure Monitor](/azure/azure-monitor/overview), is the easier choice if:

- Your organization currently lacks established logging and reporting systems.
- Your planned deployment doesn't need to be integrated with existing on-premises or other external monitoring systems.

In this scenario, all your log data records and stores in the cloud. Azure platform and Azure Monitor provide the logging and reporting tools that process and surface information to your IT staff.

As needed, implement custom logging solutions based on Azure Monitor for each subscription or workload in smaller or experimental deployments. These solutions are organized centrally to monitor log data across your entire cloud estate.

**Cloud-native assumptions:** Using a cloud-native logging and reporting system assumes the following circumstances:

- You don't need to integrate the log data from your cloud workloads into existing on-premises systems.
- You won't be using your cloud-based reporting systems to monitor on-premises systems.

### On-premises extension

It might require substantial redevelopment effort for applications and services migrating to the cloud to use cloud-based logging and reporting solutions such as Azure Monitor. In this case, consider allowing the workloads to continue sending telemetry data to existing on-premises systems.

To support this approach, your cloud resources must communicate directly with your on-premises systems through a combination of [hybrid networking](../../ready/azure-best-practices/define-an-azure-network-topology.md) and [cloud-hosted domain services](../identity/index.md#cloud-hosted-domain-services). With this communication in place, the cloud virtual network functions as a network extension of the on-premises environment. So, cloud-hosted workloads can communicate directly with your on-premises logging and reporting system.

This approach capitalizes on your existing investment in monitoring tooling with limited modification to any cloud-deployed applications or services. Also, this approach is often the fastest way to support monitoring during a lift and shift migration. But it won't capture log data produced by cloud-based PaaS and SaaS resources. It also omits any VM-related logs generated by the cloud platform itself, such as VM status. As a result, this pattern should be a temporary solution until you implement a more comprehensive hybrid monitoring solution.

**On-premises-only assumptions:** Using an on-premises logging and reporting system assumes the following circumstances:

- You maintain log data only in your on-premises environment. Maintain your log data in this way either in support of technical requirements or due to regulatory or policy requirements.
- Your on-premises systems don't support hybrid logging and reporting or gateway aggregation solutions.
- Your cloud-based applications can submit telemetry directly to your on-premises logging systems. Or your monitoring agents that submit to on-premises can be deployed to workload VMs.
- Your workloads don't depend on PaaS or SaaS services that require cloud-based logging and reporting.

### Gateway aggregation

A log data [gateway aggregation](/azure/architecture/patterns/gateway-aggregation) service might be required for scenarios where:

- The amount of cloud-based telemetry data is large.
- Existing on-premises monitoring systems need log data modified before processing.

A gateway service deploys to your cloud provider. Then, you configure relevant applications and services to submit telemetry data to the gateway instead of a default logging system. The gateway then processes the data: aggregating, combining, or otherwise formatting it before then submitting the data to your monitoring service for ingestion and analysis.

Also, you can use a gateway to aggregate and pre-process telemetry data bound for cloud-native or hybrid systems.

Gateway aggregation assumptions:

- You expect large volumes of telemetry data from your cloud-based applications or services.
- You need to format or otherwise optimize telemetry data before submitting it to your monitoring systems.
- Your monitoring systems have APIs or other mechanisms available to ingest log data after processing by the gateway.

### Hybrid monitoring (on-premises)

A hybrid monitoring solution combines log data from both your on-premises and cloud resources to provide an integrated view into your IT estate's operational status.

If you have an existing investment in on-premises monitoring systems that would be difficult or costly to replace, you might need to integrate the telemetry from your cloud workloads into preexisting on-premises monitoring solutions. In a hybrid on-premises monitoring system, on-premises telemetry data continues to use the existing on-premises monitoring system. Cloud-based telemetry data is either sent to the on-premises monitoring system directly, or the data is sent to Azure Monitor then compiled and ingested into the on-premises system at regular intervals.

**On-premises hybrid monitoring assumptions:** Using an on-premises logging and reporting system for hybrid monitoring assumes the following conditions:

- You must use existing on-premises reporting systems to monitor cloud workloads.
- You maintain ownership of log data on-premises.
- Your on-premises management systems have APIs or other mechanisms available to ingest log data from cloud-based systems.

> [!TIP]
> As part of the iterative nature of cloud migration, transitioning from distinct cloud-native and on-premises monitoring to a partial hybrid approach is likely as the integration of cloud-based resources and services into your overall IT estate matures.

### Hybrid monitoring (cloud-based)

If you don't have a compelling need to maintain an on-premises monitoring system, or you want to replace on-premises monitoring systems with a centralized cloud-based solution, you can also integrate on-premises log data with Azure Monitor to provide a centralized cloud-based monitoring system.

Mirroring the on-premises centered approach, in this scenario, cloud-based workloads would submit telemetry direct to Azure Monitor. And on-premises applications and services would either submit telemetry directly to Azure Monitor, or aggregate that data on-premises for ingestion into Azure Monitor at regular intervals. Azure Monitor would then serve as your primary monitoring and reporting system for your entire IT estate.

**Cloud-based hybrid monitoring assumptions:** Using cloud-based logging and reporting systems for hybrid monitoring assumes the following conditions:

- You don't depend on existing on-premises monitoring systems.
- Your workloads don't have regulatory or policy requirements to store log data on-premises.
- Your cloud-based monitoring systems have APIs or other mechanisms available to ingest log data from on-premises applications and services.

### Multicloud

Integrating logging and reporting capabilities across a multicloud platform can be complicated. Services offered between platforms are often not directly comparable, and logging and telemetry capabilities provided by these services differ as well.

Multicloud logging support often requires the use of gateway services to process log data into a common format before submitting data to a hybrid logging solution.

## Learn more

[Azure Monitor](/azure/azure-monitor/overview) is the default reporting and monitoring service for Azure. It provides:

- A unified platform for collecting application telemetry, host telemetry (such as VMs), container metrics, Azure platform metrics, and event logs.
- Visualization, queries, alerts, and analytical tools. It can provide insights into virtual machines, guest operating systems, virtual networks, and workload application events.
- [REST APIs](/azure/azure-monitor/essentials/rest-api-walkthrough) for integration with external services and automating the monitoring and alerting services.
- [Integration](/azure/azure-monitor/partners) with many popular third-party vendors.

## Next steps

Logging and reporting is just one of the core infrastructure components requiring architectural decisions during a cloud adoption process. Visit the architectural decision guides overview to learn about alternative patterns or models to use when making design decisions for other types of infrastructure.

> [!div class="nextstepaction"]
> [Architectural decision guides](../index.md)
