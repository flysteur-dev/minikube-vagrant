# -*- mode: ruby -*-
# vi: set ft=ruby :

KUBERNETES_VERSION = "1.26.0"
CONTAINER_RUNTIME  = "containerd"
CLUSTER_NODES      = "1"
PRIVATE_NETWORK_IP = "172.20.128.2"

$install_docker = <<SCRIPT
#!/bin/bash

if ! [ -x "$(command -v docker)" ]; then
  curl -fsSL https://get.docker.com -o get-docker.sh
  sudo sh ./get-docker.sh
  sudo usermod -aG docker vagrant && newgrp docker
fi
SCRIPT

$install_minikube = <<SCRIPT
#!/bin/bash

if ! [ -x "$(command -v minikube)" ]; then
  curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64
  sudo install minikube-linux-amd64 /usr/local/bin/minikube

  # Usefull alias to access kubernetes services inside guest vm
  echo 'alias kubectl="minikube kubectl --"' >> ~/.bashrc
  echo 'alias tunnel="minikube tunnel --bind-address '${PRIVATE_NETWORK_IP}'"' >> ~/.bashrc
  echo 'alias dashboard="(trap 'kill 0' SIGINT; minikube dashboard --url & kubectl proxy --port 9999 --address='0.0.0.0' --disable-filter=true)"' >> ~/.bashrc
fi
SCRIPT

$starting_minikube = <<SCRIPT
#!/bin/bash

minikube start \
  --memory max \
  --driver docker \
  --kubernetes-version v${KUBERNETES_VERSION} \
  --container-runtime=${CONTAINER_RUNTIME} \
  --nodes ${CLUSTER_NODES}

# Enable addons
minikube addons enable ingress
minikube addons enable metrics-server
SCRIPT



Vagrant.configure("2") do |config|
  config.vm.box = "hashicorp/bionic64"

  config.vm.provider "virtualbox" do |vb|
    vb.name = "minikube"
    vb.cpus = 2
    vb.memory = "2048"
  end

  config.vm.network :private_network, ip: PRIVATE_NETWORK_IP

  config.vm.provision "shell", inline: $install_docker, reset: true, privileged: false
  config.vm.provision "shell", inline: $install_minikube, privileged: false, env: {
    "PRIVATE_NETWORK_IP" => PRIVATE_NETWORK_IP
  }
  config.vm.provision "shell", inline: $starting_minikube, privileged: false, env: {
    "KUBERNETES_VERSION" => KUBERNETES_VERSION,
    "CONTAINER_RUNTIME"  => CONTAINER_RUNTIME,
    "CLUSTER_NODES"      => CLUSTER_NODES
  }
end
