---
title: Testing approach for Azure landing zones
description: Testing approach for Azure landing zones
author: jtracey93
ms.author: jatracey
ms.date: 03/31/2025
ms.topic: conceptual
ms.custom: think-tank
---

# Testing approach for Azure landing zones

> [!NOTE]
 > This article only applies to Microsoft Azure and not to any other Microsoft Cloud offerings such as Microsoft 365 or Microsoft Dynamics 365.

Some organizations might want to test their Azure landing zones platform deployment's Azure Policy definitions and assignments, role-based access control (RBAC) custom roles and assignments, and so on. The tests can be completed via automation by using Azure Resource Manager templates (ARM templates), [Terraform](https://aka.ms/alz/tf), [Bicep](https://aka.ms/alz/bicep), or manually via the Azure portal. This guidance provides an approach that can be used to test changes and their impact on an Azure landing zones platform deployment.

This article can also be used with the [Platform automation and DevOps critical design area](../landing-zone/design-area/platform-automation-devops.md) guidance as it relates to the PlatformOps and Central functions teams and tasks. Furthermore, it can be combined with the guidance in [Adopt policy-driven guardrails](/azure/cloud-adoption-framework/ready/enterprise-scale/dine-guidance) for techniques on rolling out policy changes to an Azure landing zones deployment.

This guidance is most suited to organizations with robust change management processes governing changes to the production management group hierarchy. The *canary* management group hierarchy can be independently used to author and test deployments before you deploy them into the production environment.

> [!NOTE]
 > The term *canary* is used to avoid confusion with development environments or test environments. This name is used for illustration purposes only. You might define any name you deem as appropriate for your canary Azure landing zones environment.
 >
 > Similarly, the term *production environment/hierarchy* is used throughout this guidance to refer to the Azure landing zones management group hierarchy your organization might have in place that contains the Azure subscriptions and resources for your workloads.

## Platform definition

> [!IMPORTANT]
 > This guidance isn't for development environments or test environments that will be used by application or service owners known as landing zones, workloads, applications, or services. These are placed and handled within the production environment Azure landing zones management group hierarchy and associated governance (RBAC and Azure Policy). 
 >
 > For guidance on development, testing, user acceptance testing (UAT), and production environments for application workloads and teams see [Manage application development environments in Azure landing zones](/azure/cloud-adoption-framework/ready/landing-zone/design-area/management-application-environments).
 >
 > This guidance is only for platform level testing and changes in the context of Azure landing zones.

Azure landing zones help you design and deploy the required Azure platform components to enable you to construct and operationalize landing zones at scale.

The platform resources in scope for this article and this testing approach are:

| Product or service | Resource provider and type |
| :--------------- | :------------------------ |
| Management groups | `Microsoft.Management/managementGroups` |
| Management groups subscription association | `Microsoft.Management/managementGroups/subscriptions` |
| Policy definitions | `Microsoft.Authorization/policyDefinitions` |
| Policy initiative definitions or policy set definitions | `Microsoft.Authorization/policySetDefinitions` |
| Policy assignments | `Microsoft.Authorization/policyAssignments` |
| RBAC role definitions | `Microsoft.Authorization/roleDefinitions` |
| RBAC role assignments | `Microsoft.Authorization/roleAssignments` |
| Subscriptions | `Microsoft.Subscription/aliases` |

## Example scenarios and outcomes

An example of this scenario is an organization that wants to test the impact and result of a new Azure Policy to govern resources and settings in all landing zones, as per the [Policy-driven governance design principle](../landing-zone/design-principles.md#policy-driven-governance). They don't want to make this change directly to the production environment as they're concerned about the impact it might have.

Using the canary environment to test this platform change will allow the organization to implement and review the impact and result of the Azure Policy change. This process will ensure it satisfies the organization's requirements before they implement the Azure Policy to their production environment.

A similar scenario might be a change to the Azure RBAC role assignments and Microsoft Entra group memberships. It might require a form of testing before the changes are made in the production environment.

> [!IMPORTANT]
 > This isn't a common deployment approach or pattern for most customers. It isn't mandatory for Azure landing zones deployments.

[![Diagram of the management group hierarchy with the canary environment testing approach.](./media/canary-mgmt-groups.png)](./media/canary-mgmt-groups.png#lightbox)

*Figure 1: Canary management group hierarchy.*

As the diagram shows, the entire Azure landing zones production management group hierarchy is duplicated under the `Tenant Root Group`. The *canary* name is appended to the management group display names and IDs. The IDs must be unique within a single Microsoft Entra tenant.

> [!NOTE]
 > The canary management group display names can be the same as the production management group display names. This might cause confusion for users. Because of this, we recommend appending the name "canary" to the display names and to their IDs.

The canary management group hierarchy is then used to simplify testing of the following resource types:

- Management groups
  - Subscription placement
- RBAC
  - Roles (built-in and custom)
  - Assignments
- Azure Policy
  - Definitions (built-in and custom)
  - Initiatives, also known as set definitions
  - Assignments

## What if you don't want to deploy the canary management group hierarchy?

If you don't want to deploy the canary management group hierarchy, you can test platform resources within the production environment hierarchy by using [sandbox subscriptions](../considerations/sandbox-environments.md) as shown in the diagram.

[![Diagram of the testing approach that uses sandboxes.](./media/canary-sandboxes.png)](./media/canary-sandboxes.png#lightbox)

*Figure 2: Azure landing zones management group hierarchy highlighting sandboxes.*

To test Azure Policy and RBAC in this scenario, you need a single Azure subscription with the Owner RBAC role assigned to the identity you wish to complete the testing as, for example, User Account, Service Principal, or Managed Service Identity. This configuration allows you to author, assign, and remediate Azure Policy definitions and assignments within the scope of the sandbox subscription only.

This sandbox approach can also be used for RBAC testing within the subscription, for example, if you're developing a new custom RBAC role to grant permissions for a particular use case. This testing can all be done in the sandbox subscription and tested before you create and assign roles higher up in the hierarchy.

A benefit of this approach is that the sandbox subscriptions can be used for the time that they're required, and then deleted from the environment.

However, this approach doesn't allow you to test with the inheritance of RBAC and Azure policies from the management group hierarchy.

## Implementation guidance

Below is guidance on how to implement and use the canary management group hierarchy for Azure landing zones alongside a production environment Azure landing zones management group hierarchy.

>[!WARNING]
 > If you're using the portal to deploy and manage your Azure landing zones environment today, it might be difficult to adopt and use the canary approach efficiently due to a high risk of both the production and canary environments getting out-of-sync often and therefore not providing a replica-like hierarchy and environment of production.
 >
 > Consider moving to an Infrastructure as Code (IaC) deployment approach for Azure landing zones, as listed above, if you're in this scenario. Or be aware of the potential risks of configuration drift between canary and production and proceed with care. For more information, see [Use IaC to update Azure landing zones](/azure/cloud-adoption-framework/ready/considerations/infrastructure-as-code-updates).

1. Use separate Microsoft Entra service principals (SPNs) or Managed Identities (MIs) that are granted permissions over the relevant production environment or canary environment Azure landing zones management group hierarchy.
   - This guidance follows the principle of least privilege (PoLP)
2. Use separate folders within a git repository, branches, or repositories to hold the IaC for the production environment and canary environment Azure landing zones deployments.
   - Using the relevant Microsoft Entra service principals (SPNs) or Managed Identities (MIs) as part of the CI/CD pipelines depending on which hierarchy is being deployed.
3. Implement git branch policies or security for the canary environment as you have in place for the production environment.
   - Consider reducing the number of approvers and checks for the canary environment to fail-fast.
4. Use the same Azure Pipelines or GitHub actions that use environment variables to change which hierarchy is being deployed. Another option is to clone the pipelines and amend the hard-coded settings to define which hierarchy is being deployed.
   - Using [Azure Pipelines DevOps templates](/azure/devops/pipelines/process/templates) or [GitHub Actions Workflow Templates](https://docs.github.com/en/actions/learn-github-actions/sharing-workflows-with-your-organization) will help you adhere to the *Don't Repeat Yourself (DRY)* principle.
5. Have a set of canary subscriptions under a separate billing account, for example an Enterprise Agreement (EA) Account or Microsoft Customer Agreement (MCA) invoice section, that can be moved around the canary management group hierarchy as needed.
   - It might be beneficial to have a set of resources always deployed into the canary environment subscriptions to speed up testing and validation of changes to the canary environment.
6. Have a set of example application workload architectures that you can deploy into the canary subscriptions in the canary environment to test the Azure Policy and RBAC changes against. This helps you to validate the changes before you deploy and promote the changes into production.
   - These example workloads can be deployed using the same IaC templates that are used to deploy the production application workloads. This will help you to ensure that the canary environment is in sync with the production environment and that the changes you're testing are valid and applicable to the production environment.
   - You should continuously review and update the example workloads to ensure that they're relevant and up to date with the latest architecture and design patterns in your organization.
   - If you provide reference architectures to your application teams, consider deploying these into the canary environment as well. This helps you to validate the changes against the reference architectures and ensure that they're applicable to the production environment.
7. Send all Azure activity logs for all Azure subscriptions, including any canary environment subscriptions, to the production environment Azure Log Analytics workspace as per the [Azure landing zones design recommendations](../landing-zone/design-area/management.md).
   - This helps your security and operations teams to monitor the canary environment for any changes or issues that might arise from the testing of the Azure Policy and RBAC changes to the production environment.

>[!TIP]
 > If you already have Azure landing zones deployed in production and are now looking to add a canary environment. Consider cloning your current deployment of the production environment hierarchy and amend the names of resources to prefix them with your canary naming scheme.
 >
 > This is to ensure what you're deploying to enable the canary environment is in sync with production from the start. This is easily achieved when using an IaC tool alongside a git repository.

<a name='using-a-single-azure-ad-tenant'></a>

### Using a single Microsoft Entra tenant

Considerations to take into account when you use a single Microsoft Entra tenant are:

- Using a single tenant follows [Azure landing zones design recommendations](../landing-zone/design-area/azure-billing-microsoft-entra-tenant.md) for Microsoft Entra tenants.
  - In a single Microsoft Entra tenant, you can use the different Microsoft Entra groups for both production and canary Azure landing zones environments with the same users assigned to their relevant management group hierarchy.
- Increased or duplicated Microsoft Entra ID licensing costs because of multiple identities across different Microsoft Entra tenants. This is particularly relevant to customers who use Microsoft Entra ID P1 or P2 features.
- RBAC changes are more complex in both canary and production environments, as it's likely that the users and groups aren't identical across both Microsoft Entra tenants.
  - Consider that the users and groups IDs won't be the same across Microsoft Entra tenants since they're globally unique.
- Reduces complexity and management overhead caused by managing multiple Microsoft Entra tenants.
  - Privileged users that must maintain access and sign in to separate tenants to perform testing might make accidental changes to the production environment instead of the canary environment, as an example.
- Reduces the likelihood of configuration drift and deployment failures.
- Doesn't require extra security and break-glass or emergency access processes to be created.
- Reduces friction and the time required to implement changes to the Azure landing zones deployment.

## Next steps
Once you have a canary environment in place, you can start to test the Azure Policy and RBAC changes before you deploy them into your production Azure landing zones management group hierarchy.

When you have tested changes to Azure Policies in the canary environment, you can then promote them to the production environment following the same approach as documented in the [Adopt policy-driven guardrails](/azure/cloud-adoption-framework/ready/enterprise-scale/dine-guidance) guidance. This is done by using the [enforcement mode feature of policy assignments](/azure/governance/policy/concepts/assignment-structure#enforcement-mode). Using the approach documented in this guidance allows you to proceed through an extra testing phase before you enforce the Azure Policy in the production environment in its desired effect which will help you build confidence in the Azure Policy changes you're making.

You can also review sandbox environments for your application teams to use for development and testing of their workloads. This is a separate concept to the canary environment and is used to provide a safe environment for application teams to develop and test their workloads before they're deployed into production.

> [!div class="nextstepaction"]
> [Implement landing zone sandbox environments](../considerations/sandbox-environments.md)
