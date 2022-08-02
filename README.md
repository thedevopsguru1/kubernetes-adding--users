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
    
