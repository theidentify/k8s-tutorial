# What is namespace?
- Organise resources in namespaces
- Virtual cluster inside a cluster

# 4 Namespaces per Default
```console
$ kubectl get namespace
NAME              STATUS   AGE
default           Active   7d
kube-node-lease   Active   7d
kube-public       Active   7d
kube-system       Active   7ds
```

kube-system
- Do not create or modify in kube-system
- System processes
- Master and Kubectl process

kube-public
- publicly accessible data
- A configmap, which contains cluster information

kube-node-lease
- heartbeats of nodes
- each node has associated lease object in namespace
- determines the availability of a node

default
- resources you create are located here

# Create your own
Solution 1. kubectl create namespace [my-namespace]
Solution 2. Create name space with a configuration file

# Why to use namespace
1. If Everything in one Namespace, make no "Overview"; So Resources grouped in Namespaces such as
   1. Database
   2. Monitoring
   3. Elastic Stack
   4. Nginx-Ingress
2. Conflicts: Many teams, same application.
  - Scenario: only default namespace. each team deploy same name of deployment. Team A apply my-app deployment same as Team B, which conflict in name but different configuration.
  - Result: They override the first teams deployment!
  - Solution: split namespace to ProjectA namespace and ProjectB namespace
3. Resource Sharing: Staging and Development / Blue-Green Deployment
4. Access and Resource Limits on Namespaces. Each team has own, isolated environment.
   - Limit: CPU, RAM, Storage per NS

# Use Cases when to use Namespaces
1. Structure your components
2. Avoid conflicts between teams
3. Share services between different environments
4. Access and Resource Limits on Namespaces Level

# Characteristic of Namespaces
- You can't access most resources from another Namespace. Each NS must define own ConfigMap
- You can access Service in another Namespace
- Components, which can't be created within a Namespace such as vol and node
  - live globally in a cluster
  - you can't isolate them
  - ```console $ kubectl api-resources --namespaced=false```

# Create component in a Namespace
```console
// Solution1
$ kubectl apply -f mysql-configmap.yaml --namespace=my-namespace

// Solution2
apiVersion: v1
kind: ConfigMap
metadata:
  name: mysql-configmap
  namespace: my-namespace
data:
  db_url: mysql-service.database
```
# Change active namespace
- Change the active namespace with kubens!

```console
$ brew install kubectx

$ kubens                        
<default>
kube-node-lease
kube-public
kube-system

$ kubens my-namespace                  
Context "minikube" modified.
Active namespace is "my-namespace".

$ kubens             
default
kube-node-lease
kube-public
kube-system
<my-namespace>
```