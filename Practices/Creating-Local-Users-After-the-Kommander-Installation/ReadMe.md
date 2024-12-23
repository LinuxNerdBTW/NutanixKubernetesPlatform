Creating Local User and Assign cluster-admin role 
---

1. Create a configMap resource with the credentials of the new local user

Create password hash for admin user
```
[root@bastion ]# htpasswd -bnBC 10 "" P@ssw0rd@123 | tr -d ':\n' && echo
$2y$10$L.7RxrXu5mDsB5FzX7wZo.6gwZ4j8RRFTOOib1U4S0pakGcGTtPzm
[root@bastion ]# 

[root@bastion ]# cat <<EOF | kubectl apply -f -
apiVersion: v1
kind: ConfigMap
metadata:
  name: dex-overrides
  namespace: kommander
data:
  values.yaml: |
    config:
      staticPasswords:
      - email: admin # can be email or username
        hash: $2y$10$L.7RxrXu5mDsB5FzX7wZo.6gwZ4j8RRFTOOib1U4S0pakGcGTtPzm
EOF
```
2. Open the Dex AppDeployment to edit it.
```
[root@bastion ]# kubectl edit -n kommander appdeployment dex

apiVersion: apps.kommander.d2iq.io/v1alpha3
kind: AppDeployment
metadata:
...
spec:
  appRef:
    kind: ClusterApp
    name: dex-2.11.1
  clusterConfigOverrides:
  - clusterSelector:
      matchExpressions:
      - key: kommander.d2iq.io/cluster-name
        operator: In
        values:
        - host-cluster
    configMapName: dex-kommander-overrides
  configOverrides: # Copy and paste this section.
    name: dex-overrides
status:
...

```

Adding RBAC Roles to Local Users

```
cat <<EOF | kubectl apply -f -
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: cluster-admin
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: cluster-admin
subjects:
- apiGroup: rbac.authorization.k8s.io
  kind: User
  name: <example_email>
EOF

```

Now you should be abled to login with admin user.....

