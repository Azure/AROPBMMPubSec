# Documentation of the Solution

The information here does not replace official Microsoft or Red Hat documentation, but is intended to offer guidance for specific areas of configuration or controls.

## General Guidelines

As a rule, settings or configurations will be provided to accelerate ATO (Authority to Operate) approvals for Canadian Government PBMM, and so certain configurations will not be emphasized, such as abundant public IP interfaces, Azure resources not available in Canada, etc.
In general, ARO deployment goals include:

- ARO private cluster config (avoid public endpoints, IPs if possible)
- Accelerate ATO for PBMM with technical controls

## Installation

After deploying the Azure Landing Zone for PubSec, and creating a generic landing zone subscription, the following steps were done to deploy ARO.

1. Register the Azure ARO resource provider:
`az provider register -n Microsoft.RedHatOpenShift --wait`
1. Policy Assignment Changes
    - Created exclusion for "Custom - Required tags on resource group" policy on the landing zone subscription. This allows ARO to create a managed resource group.

### Custom Domain

The ability to assign a custom domain to the ARO installation is key for most customers, but the installation can proceed without it. Validation of the custom domain requires DNS records to be created for API, etc.

### Red Hat Pull Secret

Automation of this will not be possible, but it is a pre-req.

### Networking

The Generic Subscription archetype has a VNet created already, which will be used for ARO. (basic reference here: [Concepts - Networking diagram for Azure Red Hat on OpenShift 4 | Microsoft Docs](https://docs.microsoft.com/en-us/azure/openshift/concepts-networking#whats-new-in-openshift-45))

Suggested changes for this network could be:

- renaming subnets for clarity (Pod, Service)
- right size for CIDRs?

### Firewall

Azure Firewall rule changes for ARO deployments:
operations VM rules for downloads, etc.

### Application Presentation

Certificates for internal/external situations
ref: Replacing the default ingress certificate - Configuring certificates | Security and compliance | OpenShift Container Platform 4.6
Adding API server certificates - Configuring certificates | Security and compliance | OpenShift Container Platform 4.6

## Management and Operations

## Logging strategies

see Set up Azure Arc for App Service, Functions, and Logic Apps - Azure App Service | Microsoft Docs
Azure AD integration for cluster management plane