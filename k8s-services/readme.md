# K8s Services
- What is a Kubernetes Service and when we need it?
- Different Service types explained
  - ClusterIP Services
  - NodePort Services
  - Headless Services
  - LoadBalancer Services
- Differences between them and when to use which one

## What is a Kubernetes Service and when we need it?
- Each Pod has its own IP address, But Pod are ephemeral - are destroyed frequently
- Service:
  - Stable IP address to access that Pod
  - Loadbalancing
  - Loose coupling
  - within & outside cluster

## ClusterIP Services
- default type
- Only accessible within cluster
```yaml
# No type specified here:
apiVersion: v1
kind: Service
metadata:
  name: microservice-one-service
spec:
  selector:
    app: microservice-one
  ports:
    - protocol: TCP
      port: 3200
      targetPort: 3000
      ...
```

### Service Communication: selector
- Pods are identified via selectors
- key value pairs
- labels of pods
- random label names

### Service Communication: port
- Pods with multiple ports is specified with targetPort

### Service Endpoints
- K8s creates Endpoint object
- same name as Service
- keeps track of, which Pods are the member/endpoints of the Service
```bash
$ kubectl get endpoints
```

## Headless Services
- Client wants to communicate with 1 specific Pod directly
- Pods want to talk directly with specific Pod
- So, not randomly selected
- Use Case: Stateful applications, like databases Only Master is allowed to write to DB
  
### Client need to figure out Ip address of each Pod
1. Option1 - API call to k8s API Server
   - makes app too tied to K8s API
   - inefficient
2. Option2 - DNS Lookup
   - DNS Lookup for Service - returns single IP address (ClusterIP)
   - Set ClusterIP To "None" - returns Pod IP address instead

```yaml
apiVersion: v1
kind: Service
metadata:
  name: mongodb-service-headless
spec:
  clusterIP: None
  selector:
    app: mongodb
  ports:
    - protocol: TCP
      port: 27017
      targetPort: 27017
```

## 3 Service type attributes
```yaml
# ClusterIP
apiVersion: v1
kind: Service
metadata:
  name: my-service
spec:
  type: ClusterIP

# NodePort
apiVersion: v1
kind: Service
metadata:
  name: my-service
spec:
  type: NodePort

# LoadBalancer
apiVersion: v1
kind: Service
metadata:
  name: my-service
spec:
  type: LoadBalancer
```

## NodePort Services
- External traffic has access to fixed port on each Worker Node
- ClusterIP Service is automatically created
- Range: 30000 - 32767
- Not secure!

```yaml
apiVersion: v1
kind: Service
metadata:
  name: ms-service-nodeport
spec:
  type: NodePort
  selector:
    app: microservice-one
  ports:
    - protocol: TCP
      port: 3200
      targetPort: 3000
      nodePort: 30008
```

## LoadBalancer Services
- Becomes accessible externally through cloud providers LoadBalancer
- NodePort and ClusterIP Service are created automatically
- LoadBalancer Service is an extension of NodePort Service
- NodePort Service is an extension of ClusterIP Service

```yaml
apiVersion: v1
kind: Service
metadata:
  name: ms-service-loadbalancer
spec:
  type: LoadBalancer
  selector:
    app: microservice-one
  ports:
    - protocol: TCP
      port: 3200
      targetPort: 3000
      nodePort: 30010
```