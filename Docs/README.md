# Documentation of the Solution

The information here does not replace official Microsoft or [Red Hat documentation](https://docs.openshift.com/aro/4/welcome/index.html), but is intended to offer guidance for specific areas of configuration or controls.

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
5. [Managing Cluster Admins](05-managing-cluster-admins.md)
6. [Platform and Application Logging](06-platform-and-app-logging.md)
7. [Application Monitoring](07-app-monitoring.md)
8. [Compliance Scanning and Reporting](08-compliance-scanning.md)




