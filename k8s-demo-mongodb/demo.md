Secret Configuration File
- kind: Secret
- metadata / name: a random name
- type: "Opaque" - default for arbitrary key-value pairs

```console
// Storing the data in a Secret component doesn't automatically make it secure.

// There are built-in mechanism (like encryption) for basic security, which are not enabled by default

// For test only
$ echo -n 'username' | base64 
dXNlcm5hbWU=

$ echo -n 'password' | base64
cGFzc3dvcmQ=

$ kubectl apply -f mongo-secret.yaml
secret/mongodb-secret created

$ kubectl get secret
NAME                  TYPE                                  DATA   AGE
default-token-wnctx   kubernetes.io/service-account-token   3      6d2h
mongodb-secret        Opaque                                2      32s
```

Secret can be referenced now in Deployment
```console
---- mongo.yaml ----
  env:
  - name: MONGO_INITDB_ROOT_USERNAME
    valueFrom:
      secretKeyRef:
        name: mongodb-secret
        key: mongo-root-username 
  - name: MONGO_INITDB_ROOT_PASSWORD
    valueFrom:
      secretKeyRef:
        name: mongodb-secret
        key: mongo-root-password 
  ...

$ kubectl apply -f mongo.yaml
deployment.apps/mongodb-deployment created

$ kubectl get all   
NAME                                      READY   STATUS    RESTARTS   AGE
pod/mongodb-deployment-7bb6c6c4c7-575zk   1/1     Running   0          23s

NAME                 TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)   AGE
service/kubernetes   ClusterIP   10.96.0.1    <none>        443/TCP   6d2h

NAME                                 READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/mongodb-deployment   1/1     1            1           23s

NAME                                            DESIRED   CURRENT   READY   AGE
replicaset.apps/mongodb-deployment-7bb6c6c4c7   1         1         1       23s

$ kubectl get pod
NAME                                  READY   STATUS    RESTARTS   AGE
mongodb-deployment-7bb6c6c4c7-575zk   1/1     Running   0          54s
```

Server Configuration File
- kind: "Service"
- metadata / name: a random name
- selector: to connect to Pod through label
- ports:
    port: Service port
    targetPort: containerPort of Deployment

```console
$ kubectl apply -f mongo.yaml
deployment.apps/mongodb-deployment unchanged
service/mongodb-service created

$ kubectl get service
NAME              TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)     AGE
kubernetes        ClusterIP   10.96.0.1      <none>        443/TCP     6d2h
mongodb-service   ClusterIP   10.96.142.39   <none>        27017/TCP   35s

$ kubectl describe service mongodb-service
Name:              mongodb-service
Namespace:         default
Labels:            <none>
Annotations:       <none>
Selector:          app=mongodb
Type:              ClusterIP
IP Family Policy:  SingleStack
IP Families:       IPv4
IP:                10.96.142.39
IPs:               10.96.142.39
Port:              <unset>  27017/TCP
TargetPort:        27017/TCP
Endpoints:         172.17.0.3:27017
Session Affinity:  None
Events:            <none>

$ kubectl get pod -o wide
NAME                                  READY   STATUS    RESTARTS   AGE     IP           NODE       NOMINATED NODE   READINESS GATES
mongodb-deployment-7bb6c6c4c7-575zk   1/1     Running   0          9m28s   172.17.0.3   minikube   <none>           <none>
```

ConfigMap
- external configuration
- centralized
- other components can use it

ConfigMap Configureation File
- kind: "ConfigMap"
- metadata / name: a random name
- data: the actual contents in key-value pairs

```console
// ConfigMap must already be in the k8s cluster, when referencing it!

$ kubectl apply -f mongo-configmap.yaml 
configmap/mongodb-configmap created

$ kubectl apply -f mongo-express.yaml  
deployment.apps/mongo-express created

$ kubectl get all | grep mongo-express
pod/mongo-express-68c4748bd6-84c7j        1/1     Running   0          36s
deployment.apps/mongo-express        1/1     1            1           36s
replicaset.apps/mongo-express-68c4748bd6        1         1         1       36s

$ kubectl logs mongo-express-68c4748bd6-84c7j                          
Welcome to mongo-express
-----------------------
```

Way to make it an External Service
- type: "LoadBalancer" 
  // assigns service an external IP address and so accepts external requests
  // But internal service also acts as a loadbalancer!
- nodePort: must be between 30000-32767
  // Port for external IP & Port you need to put into browser

```console
$ kubectl apply -f mongo-express.yaml 
deployment.apps/mongo-express unchanged
service/mongo-express-service created

$ kubectl get service
NAME                    TYPE           CLUSTER-IP       EXTERNAL-IP   PORT(S)          AGE
kubernetes              ClusterIP      10.96.0.1        <none>        443/TCP          6d2h
mongo-express-service   LoadBalancer   10.106.132.185   <pending>     8081:30000/TCP   10s
mongodb-service         ClusterIP      10.96.142.39     <none>        27017/TCP        23m

// For minikube
$ minikube service mongo-express-service
🏃  Starting tunnel for service mongo-express-service.
🎉  Opening service default/mongo-express-service in default browser...
❗  Because you are using a Docker driver on darwin, the terminal needs to be open to run it.
```

# Request Flow
Client request -> Mongo Express External Service -> Mongo Express Pod -> Mongo DB Internal Service -> Mongo DB Pod
