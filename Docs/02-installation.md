# Installation

## Considerations Before Installation

### Networking

The Generic Subscription archetype has a VNet created already, which will be used for ARO. (basic reference here: [Concepts - Networking diagram for Azure Red Hat on OpenShift 4 | Microsoft Docs](https://docs.microsoft.com/en-us/azure/openshift/concepts-networking#whats-new-in-openshift-45))

Suggested changes for this network could be:

- renaming subnets for clarity (Pod, Service)
- right size for CIDRs?

### Firewall

Eventually will adopt this guidance: [Control egress traffic for ARO](https://docs.microsoft.com/en-us/azure/openshift/howto-restrict-egress)

For now: allowing all outbound traffic to Internet for ARO nodes, and allowing bidirectional TCP 6443 between ARO clusters for management.


## Install

After deploying the Azure Landing Zone for PubSec, and creating a generic landing zone subscription, the following steps were done to deploy ARO.

1. Register the Azure ARO resource provider:

    `az provider register -n Microsoft.RedHatOpenShift --wait`

1. Policy Assignment Changes
    - Created exclusion for "Custom - Required tags on resource group" policy on the landing zone subscription. This allows ARO to create a managed resource group.
    - Remove region requirement so that Azure Arc instance can be deployed (req until Arc is available in Canada)

1. Landing zone network changes
    - The default landing zone network must not have NSGs attached to the subnets used by ARO, remove them.
    - The route tables also must either be removed, or the firewall rules must be adjusted (final rules not yet tested for this).
    - The Oz and Rz subnets were used for master and worker, respectively.

1. ARO install was done via Azure CLI for ease of testing, future efforts will include ARM or Bicep resources.

    ```
    az aro create \
         --resource-group $RESOURCEGROUP \
         --name $CLUSTER \
         --vnet <aro-vnet> \
         --vnet-resource-group <vnet rg> \
         --master-subnet <master-subnet> \
         --worker-subnet <worker-subnet> \
         --domain <base cluster domain> \
         --pull-secret @pullsecret.txt
    ```

[NEXT: Custom Certificates](03-custom-certificates.md)
---
[Back to the Table of Contents](README.md)