# Documentation of the Solution

The information here does not replace official Microsoft or Red Hat documentation, but is intended to offer guidance for specific areas of configuration or controls.

## General Guidelines

As a rule, settings or configurations will be provided to accelerate ATO (Authority to Operate) approvals for Canadian Government PBMM, and so certain configurations will not be emphasized, such as abundant public IP interfaces, Azure resources not available in Canada, etc.
In general, ARO deployment goals include:

- ARO private cluster config (avoid public endpoints and IPs as much as possible)
- Accelerate ATO for PBMM with technical controls

This guide is composed of the following sections:

1. [Before You Begin - Prerequisites](01-before-you-begin.md)
2. [Installation](02-installation.md)
3. [Installing Custom Ingress and API Certificates](03-custom-certificates.md)
4. [Configuring Azure AD for Authentication](04-azure-ad-integration.md)
5. [Platform and Application Logging](05-platform-and-app-logging.md)
6. Managing Cluster Admins
7. Cluster and Application Logging
8. [Compliance Scanning and Reporting](08-compliance-scanning.md)


### Networking

The Generic Subscription archetype has a VNet created already, which will be used for ARO. (basic reference here: [Concepts - Networking diagram for Azure Red Hat on OpenShift 4 | Microsoft Docs](https://docs.microsoft.com/en-us/azure/openshift/concepts-networking#whats-new-in-openshift-45))

Suggested changes for this network could be:

- renaming subnets for clarity (Pod, Service)
- right size for CIDRs?

### Firewall

Eventually will adopt this guidance: [Control egress traffic for ARO](https://docs.microsoft.com/en-us/azure/openshift/howto-restrict-egress)

For now: allowing all outbound traffic to Internet for ARO nodes, and allowing bidirectional TCP 6443 between ARO clusters for management.

### Application Presentation

Certificates for internal/external situations
ref: Replacing the default ingress certificate - Configuring certificates | Security and compliance | OpenShift Container Platform 4.9
Adding API server certificates - Configuring certificates | Security and compliance | OpenShift Container Platform 4.9

## Management and Operations

## Logging strategies

see Set up Azure Arc for App Service, Functions, and Logic Apps - Azure App Service | Microsoft Docs
Azure AD integration for cluster management plane
