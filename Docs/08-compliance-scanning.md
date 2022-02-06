# Compliance Scanning and Reporting

It is recommended to install the official [Compliance Operator](https://docs.openshift.com/container-platform/4.9/security/compliance_operator/compliance-operator-understanding.html) and configure the appropriate scan profiles in order to produce compliance reports for your OpenShift clusters.

The compliance operator includes a number of different profiles, but the one that is most relevant to Canadian organizations is the `moderate` profile, which includs **NIST-800-53** controls, which closely map to **ITSG-33**.

Installing the Compliance Operator is a *cluster admin* task that can be accomplished in a number of ways:
1. You can install the operator using the OpenShift Administration console.
2. You can use either `oc` or `kubectl` to *apply* the manifests to deploy the operator.
3. You can use a GitOps tool such as Argo CD (OpenShift GitOps)
4. Or you can automatically deploy the Compliance Operator and scanning profiles to all of your clusters with Advanced Cluster Management for Kubernets.

For simplicity sake, the below example will show you how to deploy the operator and configure scans using the `oc` command line interface.  This will create two different scans - for the "platform" (OpenShift/Kubernetes), and one for the underlying operating system (Red Hat Enterprise Linux CoreOS).

To install the *Compliance Operator* in the `openshift-compliance` namespace (the namespace will also be created):

```
oc apply -k manifests/compliance/operator
```

Once the operator has finished installing, you can create the scans:

```
oc apply -k manifests/compliance/scan
```

After a few minutes, the initial scan will be complete.  There are a number of different ways to view the results and apply remediations.  Please see the [Compliance Operator](https://docs.openshift.com/container-platform/4.9/security/compliance_operator/compliance-operator-understanding.html) docs for more information on this topic.  Some general guidance and (unofficial) tooling to view scans an be found [in this repository](https://github.com/pittar/ocp4-compliance-pbmm#viewing-scan-results).

