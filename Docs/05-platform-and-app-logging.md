# Platform and Application Logging

Several options exist for platform and application log aggregation in an OpenShift cluster.  These options apply to any cluster, including Azure Red Hat OpenShift.  The most common options include:

* OpenShift Cluster Logging: A *fuly supported* EFK (Elasticsearch, FluentD, Kibana) stack that is installed and managed by the OpenShift Logging operator.  OpenShift Cluster logging is a good solution when the requirement is to centralize logging within individual clusters.  This logging stack is fully integrated with the ubiquitous OpenShift role-based access control - users can only view and search logs for projects/namespaces that they have acccess to.

* OpenShift Log Forwarding: This also uses the fully supported *OpenShift Cluster Logging* stack, but instead of deploying Elasticsearch, the logging stack is configured to forward logs to an external log aggregation sync, such as Elasticsearch, Azure Log Analytics, Splunk, etc...

* Azure Arc can be used to collect logs and capture them in the central Log Analytics workspace.  Arc is an Azure service that will incur additional charges based on cluster size.

## OpenShift Cluster Logging - Full EFK Stack

OpenShift Cluster Logging is installed as a "Day 2" task using the [OpenShift Logging Operator](https://docs.openshift.com/container-platform/4.9/logging/cluster-logging-deploying.html).  For high availability logging, this will deploy Elasticsearch across three nodes, as well as FluentD collectors on each node, and Kibana (fully integrated with OpenShift OAuth) for log search.

Ensure you have capacity to deploy OpenShift Logging in your cluster, as Elasticsearch requires a minimum of 16GB memory per instance.

## OpenShift Cluster Logging - Log Forwarding

OpenShift Cluster Logging can be configured to forward logs to external 3rd party logging systems.  The benefit to this approach is the platform continues to gather logs from the platform as well as applications running on it, but will forward to the logs to a central system where they can be correlated with other logs for systems that are not running on OpenShift.

This also means you do not have to deploy Elasticsearch into your cluster, which can save a significant amount of cluster resources.

For more information on log forwarding, please see the [official OpenShift Logging documentation](https://docs.openshift.com/container-platform/4.9/logging/cluster-logging-external.html).

## Azure Arc for ARO

Connect Azure Red Hat OpenShift to the main Log Analytics workspace using Azure Arc. From the management VM, run the following commands. Note that the AZ cli commands here are formatted for Linux, so if the VM is Windows run from a WSL install for ease of use.  Azure Arc is a service that comes at an additional cost and is not currently available in Canadian regions (as of Jan 2022). Azure Arc persists a small amount of metadata about resources in the region it is provisioned in, but no customer data. See [here](https://docs.microsoft.com/en-us/azure/azure-arc/servers/agent-overview#instance-metadata) for more info.

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

    

1. Finally, enable log collection to Log Analytics workspace already created in the core.  

    Locate or derive the resource id for the workspace.
    `LAWORKSPACEID="<workspace resource id>"`

    `az k8s-extension create -n azuremonitor-containers -c $aroCluster -g $aroRG -t connectedClusters --extension-type Microsoft.AzureMonitor.Containers --configuration-settings logAnalyticsWorkspaceResourceID=$LAWORKSPACEID --only-show-errors -o jsonc`

Logs will start to flow to the workspace in a short while.