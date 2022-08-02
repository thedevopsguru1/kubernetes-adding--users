# kubernetes-adding--users
We will create both ClusterRole and Role in this demo for our users bijou and Eric.

Bijou is our Senior DevOps Engineer and is working on mulitple projects. Hence, will create ClusterRole for her which will not restricting his permission at namespace level but at resource level.

Eric is a Junior DevOps Engineer whi has recently joined the team and is working on Brown Fox project hence, will create a Role for him to restrict his permission both at namespace and resource level
1- Let create ClusterRole and Role for bijou and eric respectively
- ClusterRole-bijou.yaml
```
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: bijou
rules:
  - apiGroups: [""]
    resources: ["pods", "services", "namespaces", "nodes"]
    verbs: ["create", "get", "update", "list", "watch", "patch"]
  - apiGroups: ["apps"]
    resources: ["deployment"]
    verbs: ["create", "get", "update", "list", "delete", "watch", "patch"]
  - apiGroups: ["rbac.authorization.k8s.io"]
    resources: ["clusterroles", "clusterrolebindings"]
    verbs: ["create", "get", "list", "watch"]
  ```


- role-eric.yaml
```
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: eric
  namespace: brown-fox
rules:
  - apiGroups: [""]
    resources: ["pods", "services", "namespaces", "nodes"]
    verbs: ["create", "get", "update", "list", "watch", "patch"]
  - apiGroups: ["apps"]
    resources: ["deployment"]
    verbs: ["create", "get", "update", "list", "delete", "watch", "patch"]
  - apiGroups: ["rbac.authorization.k8s.io"]
    resources: ["clusterroles", "clusterrolebindings"]
    verbs: ["create", "get", "list", "watch"]
  ```
  Let us now bind ClusterRole to a Adam using ClusterRoleBinding API resource
  ```
  apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: bijou
subjects:
  - kind: ServiceAccount
    name: bijou-sa
    namespace: kube-system
roleRef:
  kind: ClusterRole
  name: bijou
  apiGroup: rbac.authorization.k8s.io
 ```
 For Bill, we will be using RoleBinding API resource.
 ```
 apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: bill
  namespace: brown-fox
subjects:
  - kind: ServiceAccount
    name: bill-sa
    namespace: brown-fox
roleRef:
  kind: Role
  name: bill
  apiGroup: rbac.authorization.k8s.io
 ```
 We are now required to create Service Account for both Bijou and Eric which are attached to their roles we create above
 Service Account for Bijou
 ```
 apiVersion: v1
kind: ServiceAccount
metadata:
  name: bijou-sa
  namespace: kube-system
secrets:
  - name: bijou-secret
 ```
 Service Account for Eric
 ```
 apiVersion: v1
kind: ServiceAccount
metadata:
  name: eric-sa
  namespace: brown-fox
secrets:
  - name: eric-secret
 ```
