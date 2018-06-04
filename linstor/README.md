# kubernetes linstor-provisioner

[![Docker Repository on Quay](https://quay.io/repository/external_storage/linstor-provisioner/status "Docker Repository on Quay")](https://quay.io/repository/external_storage/linstor-provisioner)

- pv provisioned as ${namespace}-${pvcName}-${pvName}
- pv recycled as archived-${namespace}-${pvcName}-${pvName}

# deploy
- modify and deploy `deploy/deployment.yaml`
- modify and deploy `deploy/class.yaml`

## ARM based
To deploy on ARM based (Raspberry PI) use `deploy/deployment-arm.yaml` instead of `deploy/deployment.yaml`

# authorization

If your cluster has RBAC enabled or you are running OpenShift you must
authorize the provisioner. If you are in a namespace/project other than
"default" either edit `deploy/auth/clusterrolebinding.yaml` or edit the `oadm
policy` command accordingly.

## RBAC
```console
$ kubectl create -f deploy/auth/serviceaccount.yaml
serviceaccount "linstor-provisioner" created
$ kubectl create -f deploy/auth/clusterrole.yaml
clusterrole "linstor-provisioner-runner" created
$ kubectl create -f deploy/auth/clusterrolebinding.yaml
clusterrolebinding "run-linstor-provisioner" created
$ kubectl patch deployment linstor-provisioner -p '{"spec":{"template":{"spec":{"serviceAccount":"linstor-provisioner"}}}}'
```

## OpenShift
```console
$ oc create -f deploy/auth/serviceaccount.yaml
serviceaccount "linstor-provisioner" created
$ oc create -f deploy/auth/openshift-clusterrole.yaml
clusterrole "linstor-provisioner-runner" created
$ oadm policy add-scc-to-user hostmount-anyuid system:serviceaccount:default:linstor-provisioner
$ oadm policy add-cluster-role-to-user linstor-provisioner-runner system:serviceaccount:default:linstor-provisioner
$ oc patch deployment linstor-provisioner -p '{"spec":{"template":{"spec":{"serviceAccount":"linstor-provisioner"}}}}'
```

# test
- `kubectl create -f deploy/test-claim.yaml`
- `kubectl create -f deploy/test-pod.yaml`
- check the folder and file "SUCCESS" created
- `kubectl delete -f deploy/test-pod.yaml`
- `kubectl delete -f deploy/test-claim.yaml`
- check the folder renamed to `archived-???`
