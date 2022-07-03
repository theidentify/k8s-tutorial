# StatefulSet for stateful applications
## stateful applications
- examples of stateful applications:
  - database
  
## stateless applcations
- don't keep record of state
- each request is completely new

## Deployment of stateful and stateless applications
- stateless applications deployed using Deployment (replicate your app)
- stateful applications deployed using StatefulSet (replicate Pods)
- Both manage Pods based on container specification
- configure storage the same way

## Pod Identity
- sticky identity for each pod ex. mysql-1, mysql-2, mysql-3
- created from same specification, but not interchangeable
- persistent identifier across any re-scheduling
- Next pod is only created, if previous is up and running
- Delete StatefulSet or scale down to 1 relica, Deletion in reverse order, starting from the last one
  
## To fixed predictable names with 2 Pod endpoints
- Own DNS Endpoint from Service

1. loadbalancer service
2. Individual service name for each pod ${pod name}.${governing service domain}

## 2 characteristics
1. predictable pod name [mysql-0]
2. fixed individual DNS name [mysql-0.svc2]
### When Pod restarts:
- Ip address changes
- name and endpoint stays same

## Replicating stateful applcations
- It's complex
- K8s helps you
- You still need to do a lot:
  - Configuring the cloning and data synchronization
  - Make remote storage available
  - Managing and backup
- "Stateful application not perfect for containerized environment"