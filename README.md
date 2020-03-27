# IBM Helm Repo Operator

Operator used to manager IBM Helm Repo service. Helm Repo is a helm chart repository for storing and supplying IBM and local charts.

## Supported platforms

### Platforms

- Red Hat OpenShift Container Plataform 4.x

### Operating Systems

- Red Hat Enterprise Linux CoreOS

## Operator versions

- 3.5.0

## Prerequisites

1. Red Hat OpenShift Container Plataform 4.x installed
1. Cluster Admin role for installation
1. [IBM Cert Manager Operator](https://github.com/IBM/ibm-cert-manager-operator)
1. [IBM MongoDB Operator](https://github.com/IBM/ibm-mongodb-operator)
1. [IBM IAM Operator](https://github.com/IBM/ibm-iam-operator)
1. [IBM Management Ingress Operator](https://github.com/IBM/ibm-management-ingress-operator)
1. [IBM Platform API Operator](https://github.com/IBM/ibm-platform-api-operator)

## Documentation

For installation and configuration, see [IBM Knowledge Center link].

### Developer guide

Information about building and testing the operator.

#### Cloning the operator repository
```
# git clone git@github.com:IBM/ibm-helm-repo-operator.git
# cd ibm-helm-repo-operator
```

#### Building the operator image
```
# make build
```

#### Installing the operator 
```
# make install
```

#### Uninstalling the operator
```
# make uninstall
```

#### Debugging the operator

Check the Cluster Service Version (CSV) installation status
```
# oc get csv
# oc describe csv ibm-helm-repo-operator.v3.5.0
```

Check the custom resource status
```
# oc describe helmrepos helm-repo
# oc get helmrepos helm-repo -o yaml
```

Check the operator status and log
```
# oc describe po -l name=ibm-helm-repo-operator
# oc logs -f $(oc get po -l name=ibm-helm-repo-operator -o name)
```

#### End-to-End testing using Operand Deployment Lifecycle Manager

See [ODLM guide](https://github.com/IBM/operand-deployment-lifecycle-manager/blob/master/docs/install/common-service-integration.md#end-to-end-test)