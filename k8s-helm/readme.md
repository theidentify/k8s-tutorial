# Helm explained
- Main concepts of Helm
- Helm changes a lot between versions, so:
  - Understand the basic common principles and use cases

## Overview:
- What is Helm?
- What are Helm Charts?
- How to use them?
- When to use them?
- What is Tiller?

### What is Helm?
- Package Manager for kubernetes
- To package YAML Files and distribute them in public and private repositories

--- 

### Helm Charts?
- Bundle of YAML Files
- Create your own Helm Charts with Helm
- Push them to Helm Repository
- Download and use existing ones
- So you can reuse that configuration!

#### Sharing Helm Charts
- helm search <keyword>
- or Helm hub
  - Public Registries
  - Private Registries

#### Templating Engine
- Deployment and Service configurations almost the same!
- Most values are same
- Prictical for CI / CD
  - In your Build you can replace the values on the fly

#####  How ?
1. Define a common blueprint
2. Dynamic values are replaced by placeholders

```yaml
# Template Yaml Config
apiVersion: v1
kind: Pod
metadata:
  name: {{{ .Values.name }}}
spec:
  containers:
  - name: {{ .Values.container.name }}
    image: {{ .Values.container.image }}
    port: {{ .Values.container.port }}

-- 
# values.yaml
name: my-app
container:
  name: my-app-container
  image: my-app-image
  port: 9001

# Values defined either via yaml file or with --set flag
```

##### Another Use Case for Helm features
- Same Applications across different environments
  - Deployment
  - Staging
  - Production

--- 
### Helm Chart Structure
```bash
# Directory structure:
# .
# mychart/
#  |--Chart.yaml
#  |--values.yaml
#  |--charts/
#  |--templates/
#  ...
# Top level mychart folder [name of chart]
# Chart.yaml meta info about chart
# values.yaml values for template Files
# charts folder chart dependencies
# templates folder the actual template files

# Template files will be filled with the values from values.yaml

$ helm install <chartname>
```


### Values injection into template files
- From Default file [values.yaml]
- From Custom file to override values [my-values.yaml]

```yaml
# values.yaml
imageName: myapp
port: 8080
version: 1.0.0
--
# my-values.yaml
version: 2.0.0
```
```bash

$ helm install --values=my-values.yaml <chartname>

# OR on Command Line:
$ helm install --set version=2.0.0

```
```yaml
# result .Values object
imageName: myapp
port: 8080
version: 2.0.0
```

### Release Management
#### For version 2 comes in two parts:
1. Client (helm CLI) ```$ helm install <chartname>```
2. Sends requests to Tiller
3. Server execute the requests and create component from there yaml files inside the cluster

- Keeping trackof all chart executions:
  - Change are applied to existing deployment instead of creating a new one
  - Handling rollbacks

#### Downsides of Tiller
- Tiller has too much power inside of K8s Cluster
- Security Issue
- Solves the Security Concern 👍 In Helm 3 Tiller got removed

```bash
$ helm install <chartname>

$ helm upgrade <chartname>

$ helm rollback <chartname>
```
|Revision|Request|
|--------|-------|
|1| Installed chart |
|2| Upgraded to v1.0.0 |
|3| Rolled back to 1

