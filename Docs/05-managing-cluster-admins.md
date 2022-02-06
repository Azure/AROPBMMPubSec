# Managing Cluster Admins

Azure Red Hat OpenShift (ARO) allows for full **cluster-admin** access on the platform.  Although ARO is a managed service, a user with **cluster-admin** level access *can irreparably damage the cluster*. As such, this level of access should be limited to a small number of people who require it.  An ARO cluster should be treated the same as a "self-managed OpenShift" cluster in this regard.

When you first log into your cluster, only the default `kubeadmin` user will have cluster-admin level access (unless you are using Red Hat Advanced Cluster Management for Kuberenetes, in which case you can fully configure your cluster before a user has logged in).

Although it is possible to add individual users as *cluster-admins* with the command:

```
oc adm policy add-cluster-role-to-user cluster-admin <username>
```

It is better idea to create a specific group that is assigned the *cluster-admin* role, then add users to the group.  This will make keeping track of (and removing) cluster-admin users easier.

The name of the group that you create is up to you.  For this example, I will use the name `department-cluster-admins`.  The *yaml* file that defines this group can ve found under `manifests/cluster-admins`.  It looks like this:

```
apiVersion: user.openshift.io/v1
kind: Group
metadata:
  name: department-cluster-admins
users:
  - bob
  - susan
  - filipe
```

And the following `ClusterRoleBinding` to bind the *cluster-admins* cluster role to the `department-cluster-admins` group:

```
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: department-cluster-admins-clusterrolebinding
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: cluster-admin
subjects:
- apiGroup: rbac.authorization.k8s.io
  kind: Group
  name: department-cluster-admins
```

As the `kubeadmin` user, apply this group to your cluster.

```
oc apply -f manifests/cluster-admins/cluster-admins-group.yaml
oc apply -f manifests/cluster-admins/cluster-admins-clusterrolebinding.yaml
```

Ideally, you would keep these manifests in a git repository where commit access is restricted to people with cluster admin responsibility.  If you want to add/remove people from the list of cluster admins for a particular cluster, you can simply add/remove them from the `cluster-admins-group.yaml` file and re-apply it.

The lifecycle of this group can be handled by a GitOps tool such as [OpenShift GitOps](https://docs.openshift.com/container-platform/4.9/cicd/gitops/understanding-openshift-gitops.html) (Argo CD) or Red Hat Advanced Cluster Management for Kuberenetes.  However, simply using the command line or another tool (Tekton, Flux, Jenkins, etc... or OpenShift UI) is fine as well.

You can read more about [user and group management in the OpenShift documentation](https://docs.openshift.com/container-platform/3.11/admin_guide/manage_users.html#managing-users-overview).

[NEXT: Platform and Application Logging](06-platform-and-app-logging.md)
---
[Back to the Table of Contents](README.md)