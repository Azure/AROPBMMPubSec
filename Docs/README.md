# Documentation of the Solution

The information here does not replace official Microsoft or Red Hat documentation, but is intended to offer guidance for specific areas of configuration or controls.

## General Guidelines

As a rule, settings or configurations will be provided to accelerate ATO (Authority to Operate) approvals for Canadian Government PBMM, and so certain configurations will not be emphasized, such as abundant public IP interfaces, Azure resources not available in Canada, etc.
In general, ARO deployment goals include:

- ARO private cluster config (avoid public endpoints and IPs as much as possible)
- Accelerate ATO for PBMM with technical controls

## Before You Begin

In order to have access to the catalog of Red Hat container images and OperatorHub, you should be sure to include the "pull secret" associated with your organizational Red Hat account.  You can download your "pull secret" from the [Red Hat cloud console](https://cloud.redhat.com/openshift/install/pull-secret).  Once you have this file, keep it handy for when you run the `az` command to deploy your clusters.

### Custom Domain

The ability to assign a custom domain to the ARO installation is key for most customers, but the installation can proceed without it.

Validation of the custom domain requires DNS records to be created for API, etc.

### Red Hat Pull Secret

Automation of this will not be possible, but it is a pre-req.  Of particular note if the install is using a Windows machine, ensure that the pull secret file is using "LF" format and not "CRLF" as is common for Windows.

### Management Options

The ARO cluster is created as a private cluster, meaning the admin and management interfaces are only available from the VNet in the subscription.  Create a VM (Linux or Windows) in the landing zone to complete the configuration of the cluster.  Install the OC command line tools.

## Installation

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

## Azure AD Integration

To allow admins and developers to access the cluster using Azure AD accounts, follow these instructions.

1. Configure the cluster to use Azure AD for authentication, as per [Configure Azure AD auth for Openshift 4](https://docs.microsoft.com/en-us/azure/openshift/configure-azure-ad-ui)

## Platform Logging

Several options exist for ARO logging. Azure Arc can be used to collect logs and capture them in the central Log Analytics workspace.

### Azure Arc for ARO

Connect ARO to the main Log Analytics workspace using Azure Arc. From the management VM, run the following commands. Note that the AZ cli commands here are formatted for Linux, so if the VM is Windows run from a WSL install for ease of use.

1. Add AZ CLI extensions.

    `az extension add -n log-analytics -o table`

    `az extension add -n sentinel -o table`

    `az extension add -n connectedk8s -o table`

    `az extension add -n k8s-extension -o table`

    `az provider register --namespace Microsoft.Kubernetes -o table --wait`

    `az provider register --namespace Microsoft.KubernetesConfiguration -o table --wait`

    `az provider register --namespace Microsoft.ExtendedLocation -o table --wait`

1. Obtain kubeadmin credentials for command line (if not already obtained).  

    `az aro list-credentials --name <clustername> --resource-group <rgname>`

    `oc login <apiserverurl> -u kubeadmin -p <kubeadmin password>`

1. Add a user policy for Azure Arc.

    `oc adm policy add-scc-to-user privileged system:serviceaccount:azure-arc:azure-arc-kube-aad-proxy-sa`

1. Connect the cluster to Azure Arc.

    `aroCluster=<ARO cluster name>`
    `aroRG=<ARO rg name>`
    `az connectedk8s connect -n $aroCluster -g $aroRG --only-show-errors -o jsonc --location eastus`

    Note that Azure Arc is not currently available in Canadian regions (as of Jan 2022). Azure Arc persists a small amount of metadata about resources in the region it is provisioned in, but no customer data. See [here](https://docs.microsoft.com/en-us/azure/azure-arc/servers/agent-overview#instance-metadata) for more info.

1. Finally, enable log collection to Log Analytics workspace already created in the core.  

    Locate or derive the resource id for the workspace.
    `LAWORKSPACEID="<workspace resource id>"`

    `az k8s-extension create -n azuremonitor-containers -c $aroCluster -g $aroRG -t connectedClusters --extension-type Microsoft.AzureMonitor.Containers --configuration-settings logAnalyticsWorkspaceResourceID=$LAWORKSPACEID --only-show-errors -o jsonc`

Logs will start to flow to the workspace in a short while.


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
ref: Replacing the default ingress certificate - Configuring certificates | Security and compliance | OpenShift Container Platform 4.6
Adding API server certificates - Configuring certificates | Security and compliance | OpenShift Container Platform 4.6

## Management and Operations

## Logging strategies

see Set up Azure Arc for App Service, Functions, and Logic Apps - Azure App Service | Microsoft Docs
Azure AD integration for cluster management plane
