# YAML Configuration File in Kubernetes

# Overview:
- The 3 parts of configuration file
- Connecting Deployments to Service to Pods
- Demo

# Each configuration file has 3 parts
1) metadata
2) specification
3) status - Automatically generated and added by Kubernetes!: to compare between Desired? <-> Actual? is matched for self healing (Question.1)

[Note] Attributes of "spec" are specific to kind!

# Template in Deployment
- has it's own "metadata"
- applies to Pod
- blueprint for a Pod

```console
// Example
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  labels:
    app: nginx
spec:
  replicas: 2
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
        - name: nginx
          image: nginx:1.16
          ports:
            - containerPort: 80
```

# Connecting Deployment to Pods
- Pods get the label through the template blueprint
- This label is matched by the selector
  `selector:
    matchLabels:
      app: nginx`

# Connecting Services to Deployments
- The label in Deployment is matched by the service configuration 
```console
---- Deployment ----
  metadata:
  name: nginx-deployment
  labels:
    app: nginx
  ...

---- Service ----
  apiVersion: v1
  kind: Service
  metadata:
    name: nginx-servicce
  spec:
    selector:
      app: nginx
      ports: ...
    
```

# Ports in Service in Pod
- containerPort in Pod = 8080 is match to service targetPort = 8080

DB Service - port:80 -> nginx Service - targetPort: 8080 -> Pod

# Demo
```console
$ kubectl apply -f nginx-deployment.yaml
deployment.apps/nginx-deployment created

$ kubectl apply -f nginx-service.yaml
service/nginx-service created

$ kubectl get pod 
NAME                                READY   STATUS    RESTARTS   AGE
nginx-deployment-7956bd8bb9-6smvb   1/1     Running   0          107s
nginx-deployment-7956bd8bb9-92sxw   1/1     Running   0          107s

$ kubectl get services
NAME            TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)   AGE
kubernetes      ClusterIP   10.96.0.1      <none>        443/TCP   2d2h
nginx-service   ClusterIP   10.107.70.39   <none>        80/TCP    116s

$ kubectl describe service nginx-service
Name:              nginx-service
Namespace:         default
Labels:            <none>
Annotations:       <none>
Selector:          app=nginx
Type:              ClusterIP
IP Family Policy:  SingleStack
IP Families:       IPv4
IP:                10.107.70.39
IPs:               10.107.70.39
Port:              <unset>  80/TCP
TargetPort:        8080/TCP
Endpoints:         172.17.0.3:8080,172.17.0.4:8080
Session Affinity:  None
Events:            <none>

$ kubectl get pod -o wide
NAME                                READY   STATUS    RESTARTS   AGE     IP           NODE       NOMINATED NODE   READINESS GATES
nginx-deployment-7956bd8bb9-6smvb   1/1     Running   0          3m55s   172.17.0.3   minikube   <none>           <none>
nginx-deployment-7956bd8bb9-92sxw   1/1     Running   0          3m55s   172.17.0.4   minikube   <none>           <none>

---- Get result the k8s status
$ kubectl get deployment nginx-deployment -o yaml > nginx-deployment-result.yaml

$ kubectl delete -f nginx-deployment.yaml
deployment.apps "nginx-deployment" deleted

$ kubectl delete -f nginx-service.yaml
service "nginx-service" deleted
```

Question
1) Where does K8s get this status data?
[Answer]: etcd is the cluster brain, Cluster changes get stored in the key value store, So Etcd holds the current status of any K8s component!
   