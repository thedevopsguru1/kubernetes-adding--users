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
  Let us now bind ClusterRole to a Bijou using ClusterRoleBinding API resource
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
Now we need to create tokens using Secret API resource which will be used in our kubeconfig file.

Secret for Bijou
```
apiVersion: v1
kind: Secret
metadata:
  name: bijou-secret
  namespace: kube-system
  annotations:
    kubernetes.io/service-account.name: bijou-sa
type: kubernetes.io/service-account-token
```
Secret for Eric
```
apiVersion: v1
kind: Secret
metadata:
  name: bill-secret
  namespace: brown-fox
  annotations:
    kubernetes.io/service-account.name: bill-sa
type: kubernetes.io/service-account-token
```
Finally, let us generate kubeconfig file for Bijou and Eric.
Sample File:
```
apiVersion: v1
clusters:
- cluster:
    certificate-authority-data: {cluster-ca}
    server: {server-dns}
  name: {cluster-name}
contexts:
- context:
    cluster: {cluster-name}
    user: {user-name}
  name: {context-name}
current-context: {context-name}
kind: Config
users:
- name: {user-name}
  user:
    token: {secret-token}
```
We need to replace all the values within curly braces with their actual values, so let's keep going.

Type the below command to fetch Certificate and Server information:
```
kubectl config view --flatten --minify

```
Copy certificate-authority-data, server and name field from the output and replace them with their respective fields in the sample file.
![image](https://user-images.githubusercontent.com/107158398/182351373-0abf6ddd-7dd9-4c18-972b-c5a7bca8b3b0.png)

It's now time to grab the secret token. Use the following command:
```
# Fetching Bijou's secret
kubectl get secrets -A

# Copy the token name starting bijou-secret
kubectl describe secrets {token-name} -A

# Fetching Eric's secret
kubectl get secrets -A

# Copy the token name starting Eric-secret
kubectl describe secrets {token-name} -A
```
Copy the token value and paste it in sample file. The kubeconfig file is now complete and should look like this:
```
apiVersion: v1
clusters:
- cluster:
    certificate-authority-data: LS0tLS1CRUdJTiBDRVJUSUZJQ0...............Sk1TTG5Ndk1yNmYzdFNl
    server: https://01F4..............BEA3.sk1.ap-south-1.eks.amazonaws.com
  name: arn:aws:eks:ap-south-1:006378141167:cluster/brown-fox-cluster
contexts:
- context:
    cluster: arn:aws:eks:ap-south-1:006378141167:cluster/brown-fox-cluster
    user: eric
  name: arn:aws:eks:ap-south-1:006378141167:cluster/brown-fox-cluster
current-context: arn:aws:eks:ap-south-1:006378141167:cluster/brown-fox-cluster
kind: Config
users:
- name: eric
  user:
    token: eyJhbGciOiJSUzI1NiIsImtpZCI6IiJ9.eyJpc3MiOiJ...............SNFZmRdIf

```
Note: You will have to generate a separate kubeconfing file for Bijou using his secret.
Ask Bijou and Eric to create If they don't have it yet , a kubeconfig file.
```
touch ~/.kube/config
```
```
vi(m) ~/.kube/config 
```
Send them the kubeconfig so they can paste it on their one.

```
apiVersion: v1
clusters:
- cluster:
    certificate-authority-data: LS0tLS1CRUdJTiBDRVJUSUZJQ0...............Sk1TTG5Ndk1yNmYzdFNl
    server: https://01F4..............BEA3.sk1.ap-south-1.eks.amazonaws.com
  name: arn:aws:eks:ap-south-1:006378141167:cluster/brown-fox-cluster
contexts:
- context:
    cluster: arn:aws:eks:ap-south-1:006378141167:cluster/brown-fox-cluster
    user: eric
  name: arn:aws:eks:ap-south-1:006378141167:cluster/brown-fox-cluster
current-context: arn:aws:eks:ap-south-1:006378141167:cluster/brown-fox-cluster
kind: Config
users:
- name: eric
  user:
    token: eyJhbGciOiJSUzI1NiIsImtpZCI6IiJ9.eyJpc3MiOiJ...............SNFZmRdIf
```
Ask them of add the context as well to bind the cluster to the user
To test custom kubeconfig file type the following command:
```
kubectl get pods -A
```
