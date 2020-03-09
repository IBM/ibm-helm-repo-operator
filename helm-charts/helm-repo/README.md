# Helm Repo
* Installs components for IBM helm repo service

## Introduction
Helm Repo is a helm chart repository for storing and supplying IBM and local charts.

## Chart Details
This chart deploys a single instance of a helm repo pod on the master node of a Kubernetes environment. 

## Prerequisites
* Kubernetes 1.11.0 or later, with beta APIs enabled
* IBM core services including auth-idp service, MongoDB, and management-ingress
* Cluster Admin role for installation

### PodSecurityPolicy Requirements

This chart requires a PodSecurityPolicy to be bound to the target namespace prior to installation. Choose either a predefined [`ibm-anyuid-psp`](https://ibm.biz/cpkspec-psp) PodSecurityPolicy or have your cluster administrator create a custom PodSecurityPolicy for you:
* Custom PodSecurityPolicy definition:

```
apiVersion: extensions/v1beta1
kind: PodSecurityPolicy
metadata:
  name: icp-helm-repo-psp
spec:
  allowPrivilegeEscalation: true
  fsGroup:
    rule: RunAsAny
  requiredDropCapabilities:
  - MKNOD
  allowedCapabilities:
  - SETPCAP
  - AUDIT_WRITE
  - CHOWN
  - NET_RAW
  - DAC_OVERRIDE
  - FOWNER
  - FSETID
  - KILL
  - SETUID
  - SETGID
  - NET_BIND_SERVICE
  - SYS_CHROOT
  - SETFCAP
  runAsUser:
    rule: RunAsAny
  seLinux:
    rule: RunAsAny
  supplementalGroups:
    rule: RunAsAny
  volumes:
  - configMap
  - emptyDir
  - projected
  - secret
  - persistentVolumeClaim
  forbiddenSysctls:
  - '*'
```

### Red Hat OpenShift SecurityContextConstraints Requirements

This chart requires a SecurityContextConstraints to be bound to the target namespace prior to installation. To meet this requirement there may be cluster-scoped, as well as namespace-scoped, pre- and post-actions that need to occur.

The predefined SecurityContextConstraints [`ibm-anyuid-scc`](https://ibm.biz/cpkspec-scc) has been verified for this chart. If your target namespace is not bound to this SecurityContextConstraints resource you can bind it with the following command:

`oc adm policy add-scc-to-group ibm-anyuid-scc system:serviceaccounts:<namespace>` For example, for release into the `default` namespace:
``` 
oc adm policy add-scc-to-group ibm-anyuid-scc system:serviceaccounts:default
```

* Custom SecurityContextConstraints definition:

```
apiVersion: security.openshift.io/v1
kind: SecurityContextConstraints
metadata:
  name: icp-helm-repo-scc
readOnlyRootFilesystem: false
allowedCapabilities: []
allowHostPorts: true
seLinuxContext:
  type: RunAsAny
supplementalGroups:
  type: MustRunAs
  ranges:
  - max: 65535
    min: 1
runAsUser:
  type: MustRunAsNonRoot
fsGroup:
  type: MustRunAs
  ranges:
  - max: 65535
    min: 1
volumes:
- configMap
- downwardAPI
- emptyDir
- persistentVolumeClaim
- projected
- secret
```

## Resources Required
* At least 640Mb of available memory
* At least 150m of available CPU
* At least a few GB of free storage

## Installing the Chart
_NOTE: Helm repo is intended to be installed by the Installer for internal chart management, not by a user_

Only one instance of helm repo should be installed on a cluster. 
To install the chart with the release name `helm-repo`:

```bash
$ helm install {chartname.tgz} -f {my-values.yaml} --namespace kube-system --name helm-repo --tls
```
- Replace `{chartname.tgz}` with the packaged helm-repo chart
- Replace `{my-values.yaml}` with the path to a YAML file that specifies the values that are to be used with the install command

The command deploys Helm repo on the Kubernetes cluster in the default configuration. The [configuration](#configuration) section lists the parameters that can be configured during installation.

### Verifying the Chart
Call the `/helm-repo/charts/index.yaml` endpoint to verify that helm-repo is functioning properly.

### Uninstalling the Chart

To uninstall/delete the deployment:

```console
$ helm delete helm-repo --purge --tls
```

The command removes all the Kubernetes components associated with the chart and deletes the release.

## Configuration

The following tables lists the configurable parameters of the helm repo chart and their default values.

Parameter                                       | Description                              | Default
----------------------------------------------- | ---------------------------------------- | -------
`replicaCount`|number of pod replications|1
`arch` |Node architecture to deploy to|amd64
`helmrepo.image.name`|helm repo container image name|helm-repo
`helmrepo.image.repository`|helm repo image path|quay.io/opencloudio/icp-helm-repo
`helmrepo.image.tag`|helm repo image tag|latest
`helmrepo.image.pullPolicy`|helm repo image pull policy|IfNotPresent
`helmrepo.env.CLUSTER_PORT`|helm repo cluster port|8443
`helmrepo.env.CLUSTER_CA_DOMAIN`|helm repo cluster domain|mycluster
`helmrepo.env.PROXY_ROUTE`|helm repo proxy route|helm-repo
`helmrepo.resources.limits.cpu`|helm repo cpu limits|100m
`helmrepo.resources.limits.memory`|helm repo memory limits|256Mi
`mongo.host`|mongoDB host name|mongodb
`mongo.port`|mongoDB exposed port|27017
`mongo.username.secret`|mongoDB username secret|icp-mongodb-admin
`mongo.username.key`|mongoDB username key|user
`mongo.password.secret`|mongoDB password secret|icp-mongodb-admin
`mongo.password.key`|mongoDB password key|password
`mongo.clustercertssecret`|mongoDB cluster cert secret|cluster-ca-cert
`mongo.clientcertssecret`|mongoDB client cert secret|cluster-ca-cert
`auditService.name`|audit service container name|icp-audit-service
`auditService.image.repository`|audit service image path|quay.io/opencloudio/icp-audit-service
`auditService.image.tag`|audit service image tag|latest
`auditService.image.pullPolicy`|audit service image pull policy|IfNotPresent
`auditService.resources.limits.cpu`|audit service cpu limits|100m
`auditService.resources.limits.memory`|audit service memory limits|128Mi
`auditService.config.enabled`|audit service container enabled|true
`auditService.config.audit_enabled`|audit log generating enabled|false
`auditService.config.logrotate`|audit service logrotate settings|'/var/log/audit/*.log {\n su root root\n copytruncate \n rotate 24\n hourly\n missingok\n notifempty}'
`auditService.config.logrotate_conf`|global logrotate settings|include /etc/logrotate.d

## Storage
Chart packages and chart information is stored on volume mounts within the pod as well as with MongoDB.

## Limitations
Only one instance of Helm API should run at a single time, and should be restricted to running only on the master node.

## Documentation
Managing helm repositories:
https://www.ibm.com/support/knowledgecenter/SSBS6K_3.2.0/app_center/manage_helm_repo.html
