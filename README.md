# Minikube Vagrant

The goal of this project is to provide an easy and clean way to install minikube inside a vagrant guest VM using virtualbox. Without having to install any binaries on the host.

Kubernetes cluster will be exposed to the host using a private network.

```
+------------------------------------------------------+
| Host (private_network_ip)                            |
|     +-----------------------------------+            |
|     | Vagrant box                       |            |
|     |    +------------------------------+            |
|     |    | Minikube                     |            |
|     |    |   +--------------------------+            |
|     |    |   | Pods/Services/foo...     |            |
|     |    |   |                          |            |
|     |    |   |                          | <--+curl   |
|     |    |   |                          |     browser|
|     |    |   |                          |            |
|     |    |   |                          |            |
|     |    |   |                          |            |
|     |    |   |                          |            |
+-----+----+---+--------------------------+------------+
```


## Pre-requisites

Vagrant (https://www.vagrantup.com/docs/installation/)



## Options

Edit constant on top of the Vagrantfile

| Option              | Description   | Default                 |
| -------------       | ------------- | -------------           |
| KUBERNETES_VERSION  | Kubernetes version to install           | 1.26
| CONTAINER_RUNTIME   | Runtime to use (containerd, docker)     | containerd
| CLUSTER_NODES       | Number of nodes to setup                | 1
| PRIVATE_NETWORK_IP  | Prefered private network ip (from host) | 172.20.128.2

# Usage

### Install

```
vagrant up
```

### Suspend / resume

```
vagrant suspend
vagrant resume
```

### Manage kubernetes cluster

```
vagrant ssh
kubectl get pods -A
```

### Access to exposed services from host
```
vagrant ssh -c 'bash -ci tunnel'

curl PRIVATE_NETWORK_IP:80/exposed-svc
```

### Access to kubernetes dashboard from host
```
vagrant ssh -c 'bash -ci dashboard'

http://PRIVATE_NETWORK_IP:9999/api/v1/namespaces/kubernetes-dashboard/services/http:kubernetes-dashboard:/proxy/#/workloads?namespace=default
```

# Example

This example will create an ingress (foo-ingress) listening on port 80 and routing to the corresponding service (foo-service) targeting pod (foo-app) on port 8080.

```
kubectl apply -f https://raw.githubusercontent.com/flysteur-dev/minikube-vagrant/master/examples/ingress.yaml

vagrant ssh -c 'bash -ci tunnel'
```

From the host

```
curl http://172.20.128.2/foo

============================================
Request served by foo-app

HTTP/1.1 GET /foo

Host: 172.20.128.2
Accept: */*
User-Agent: curl/7.58.0
X-Forwarded-For: 10.244.0.1
X-Forwarded-Host: 172.20.128.2
X-Forwarded-Port: 80
```