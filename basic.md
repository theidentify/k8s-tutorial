## Meaning
Deployment
- is information about blueprint for creating Pods
- most basic configuration for deployment (name and image to use)
- rest defaults
  confusing: Pod is smallest unit But, you are creating Deployment instead
             because Deployment is abstraction over Pods

Replicaset
- is managing the replicas of a Pod

## Main kubectl command
Create deployment
```console
$ kubectl create deployment nginx-depl --image=nginx
deployment.apps/nginx-depl created
```

Check deployment status
```console
$ kubectl get deployment                            
NAME         READY   UP-TO-DATE   AVAILABLE   AGE
nginx-depl   0/1     1            0           15s
```

Check pod is already created
```console
$ kubectl get pod       
NAME                          READY   STATUS    RESTARTS   AGE
nginx-depl-5ddc44dd46-fftxq   1/1     Running   0          26s
``` 

Check replicas set
```console
$ kubectl get replicaset
NAME                    DESIRED   CURRENT   READY   AGE
nginx-depl-5ddc44dd46   1         1         1       80s
```

Edit deployment
```console
-- Try to edit deployment configuration
$ kubectl edit deployment nginx-depl
deployment.apps/nginx-depl edited

-- After edited pod has auto generate new one
$ kubectl get pod       
NAME                          READY   STATUS              RESTARTS   AGE
nginx-depl-5ddc44dd46-fftxq   1/1     Running             0          38m
nginx-depl-7d459cf5c8-k9gz6   0/1     ContainerCreating   0          8s

-- Terminate old pod
$ kubectl get pod
NAME                          READY   STATUS    RESTARTS   AGE
nginx-depl-7d459cf5c8-k9gz6   1/1     Running   0          19s

-- Replicas has automatically updated
$ kubectl get replicaset
NAME                    DESIRED   CURRENT   READY   AGE
nginx-depl-5ddc44dd46   0         0         0       44m
nginx-depl-7d459cf5c8   1         1         1       5m47s
```

Debugging pods - Logs to console 
```console
$ kubectl logs mongo-depl-85ddc6d66-bdcdb
{"t":{"$date":"2022-06-08T15:30:53.800+00:00"},"s":"I",  "c":"NETWORK",  "id":4915701, "ctx":"-","msg":"Initialized wire specification","attr":{"spec":{"incomingExternalClient":{"minWireVersion":0,"maxWireVersion":13},"incomingInternalClient":{"minWireVersion":0,"maxWireVersion":13},"outgoing":{"minWireVersion":0,"maxWireVersion":13},"isInternalClient":true}}}...
```

Debugging pods - Get info about pod
```console
$ kubectl describe pod mongo-depl-85ddc6d66-bdcdb
Events:
  Type    Reason     Age    From               Message
  ----    ------     ----   ----               -------
  Normal  Scheduled  3m14s  default-scheduler  Successfully assigned default/mongo-depl-85ddc6d66-bdcdb to minikube
  Normal  Pulling    3m14s  kubelet            Pulling image "mongo"
  Normal  Pulled     2m43s  kubelet            Successfully pulled image "mongo" in 30.664007014s
  Normal  Created    2m43s  kubelet            Created container mongo
  Normal  Started    2m43s  kubelet            Started container mongo
```

Debugging pods - Get interactive terminal
```console
$ kubectl exec -ti mongo-depl-85ddc6d66-bdcdb -- bin/bash
root@mongo-depl-85ddc6d66-bdcdb:/# 
```

Delete deployment
```console
$ kubectl get deployment
NAME         READY   UP-TO-DATE   AVAILABLE   AGE
mongo-depl   1/1     1            1           7m44s
nginx-depl   1/1     1            1           85m

$ kubectl delete deployment mongo-depl
deployment.apps "mongo-depl" deleted

$ kubectl get pod       
NAME                          READY   STATUS    RESTARTS   AGE
nginx-depl-7d459cf5c8-k9gz6   1/1     Running   0          48m

$ kubectl get replicaset
NAME                    DESIRED   CURRENT   READY   AGE
nginx-depl-5ddc44dd46   0         0         0       87m
nginx-depl-7d459cf5c8   1         1         1       49m

$ kubectl delete deployment nginx-depl
deployment.apps "nginx-depl" deleted

$ kubectl get replicaset
No resources found in default namespace.
```

Apply deployment configuration file
```console
$ kubectl apply -f nginx-deployment.yaml
deployment.apps/nginx-deployment created

-- After change to 2 replicas
$ kubectl apply -f nginx-deployment.yaml
deployment.apps/nginx-deployment configured

$ kubectl get pod                       
NAME                                READY   STATUS    RESTARTS   AGE
nginx-deployment-7956bd8bb9-47tgj   1/1     Running   0          11s
nginx-deployment-7956bd8bb9-f8k62   1/1     Running   0          11s
```

Delete deployment with configuration file
```console
$ kubectl delete -f nginx-deployment.yaml 
deployment.apps "nginx-deployment" deleted

$ kubectl get pod                        
No resources found in default namespace.
```

## Layer of abstraction
1. Deployment manages a ... :arrow_down: 
2. Replicaset manages a ... :arrow_down: 
3. Pod is an abstraction of a ... :arrow_down: 
4. Container

[Tips] Everything below Deployment is handled by kubernetes