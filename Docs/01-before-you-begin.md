# Before You Begin

## Custom Domain

The ability to assign a custom domain to the Azure Red Hat OpenShift installation is key for most customers.  You do not require a custom domain to deploy a cluster, but it is an **install-time decision**.  If you deploy a cluster without a custom domain, you can't switch to using a custom domain after the fact.

As an example, if you would like to use the domain `aro-dev.department.ca`, you will need to add two **A** records with a low TTL (time to live), for example 300 seconds.  One for `api` and one for `*.apps`.  These **A** records will point to the `api` load balancer and the `*.apps` ingress load balancer respectively.

Once your ARO cluster has finished deploying, you can find the IP address of these load balancers by running the following command:

```
az aro show \
  -n $CUSTER \
  -g $RESOURCEGROUP \
  --query '{api:apiserverProfile.ip, ingress:ingressProfiles[0].ip}
```

Assingn the api server IP to the `api` **A** record, and the ingress IP to the `*.apps` **A** record.

You should now be able to access your cluster at: `https://console-openshift-console.apps.aro-dev.department.ca`

## Red Hat Pull Secret

In order to have access to the catalog of Red Hat container images and OperatorHub, you should be sure to include the "pull secret" associated with your organizational Red Hat account.  You can download your "pull secret" from the [Red Hat cloud console](https://cloud.redhat.com/openshift/install/pull-secret).  Once you have this file, keep it handy for when you run the `az` command to deploy your clusters.

If the install is run from a Windows machine, ensure that the pull secret file is using "LF" format and not "CRLF" as is common for Windows.

## Management Options

The ARO cluster is created as a private cluster, meaning the admin and management interfaces are only available from the VNet in the subscription.  Create a VM (Linux or Windows) in the landing zone to complete the configuration of the cluster.  Install the OC command line tools.