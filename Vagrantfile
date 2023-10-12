# -*- mode: ruby -*-
# vi: set ft=ruby :

# All Vagrant configuration is done below. The "2" in Vagrant.configure
# configures the configuration version (we support older styles for
# backwards compatibility). Please don't change it unless you know what
# you're doing.
#$script = <<-SCRIPT
#echo I am provisioning...

#!/usr/bin/env bash
#install package updates
#sudo apt-get update
#install required packages
#sudo apt-get install -y build-essential python3-pip apt-transport-https ca-certificates curl
#echo "***** Install UNZIP*******"
#sudo apt-get install -y unzip
#echo "***** Install AWS CLI*******"
#curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
#unzip awscliv2.zip
#sudo ./aws/install
#install kubectl
#echo "*****Install Kubectl*******"
#curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
#sudo install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl
#curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add -
#curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add -
#sudo touch /etc/apt/sources.list.d/kubernetes.list
#echo "deb http://apt.kubernetes.io/ kubernetes-xenial main" | sudo tee -a /etc/apt/sources.list.d/kubernetes.list

#echo "*****Create homebrew install script for post install*******"
#echo ' NONINTERACTIVE=1 /bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"' >installbrew.sh
#sudo chmod +x installbrew.sh
#sudo chmod 777 installbrew.sh
#eval "$(/home/linuxbrew/.linuxbrew/bin/brew shellenv)"
#brew install derailed/k9s/k9s


require "yaml"
settings = YAML.load_file "settings.yaml"

IP_SECTIONS = settings["network"]["control_ip"].match(/^([0-9.]+\.)([^.]+)$/)
# First 3 octets including the trailing dot:
IP_NW = IP_SECTIONS.captures[0]
# Last octet excluding all dots:
IP_START = Integer(IP_SECTIONS.captures[1])
NUM_WORKER_NODES = settings["nodes"]["workers"]["count"]

#SCRIPT
NUM_WORKER_NODES=1
IP_NW="10.0.0."
IP_START=10

require "yaml"
settings = YAML.load_file "settings.yaml"


Vagrant.configure("2") do |config|
  #config.vm.box = "rethinc-oss/ubuntu-2204"
  config.vm.box = "bento/ubuntu-20.04"
  #using 20.04 as there are issues with 22.04 and K8s
  #config.vm.box = "bento/ubuntu-20.04"
  config.vbguest.auto_update = false
  #config.vm.disk :disk, size: "20GB", primary: true
  config.vm.provision "shell", env: { "IP_NW" => IP_NW, "IP_START" => IP_START, "NUM_WORKER_NODES" => NUM_WORKER_NODES }, inline: <<-SHELL
      apt-get update -y
      echo "$IP_NW$((IP_START)) master-node" >> /etc/hosts
      for i in `seq 1 ${NUM_WORKER_NODES}`; do
        echo "$IP_NW$((IP_START+i)) worker-node0${i}" >> /etc/hosts
      done
  SHELL
  
  #create Control Plane vm
  config.vm.define "cp" do |cp|
     
    cp.winrm.timeout = 1800 # 30 minutes
    cp.vm.boot_timeout = 1800 # 30 minutes
    cp.vm.synced_folder "C:\\Users\\apatel", "/mnt/", type: "virtualbox"
    cp.vm.network "private_network", ip: IP_NW + "#{IP_START}"
      #virtualbox__intnet: true

    #setting memory/cpu on the vm    
    cp.vm.provider "virtualbox" do |vb|
      vb.cpus = settings["nodes"]["control"]["cpu"]
      vb.memory = settings["nodes"]["control"]["memory"]
      if settings["cluster_name"] and settings["cluster_name"] != ""
        vb.customize ["modifyvm", :id, "--groups", ("/" + settings["cluster_name"])]
      end
    end
    cp.vm.provision "shell",
      env: {
        "DNS_SERVERS" => settings["network"]["dns_servers"].join(" "),
        "ENVIRONMENT" => settings["environment"],
        "KUBERNETES_VERSION" => settings["software"]["kubernetes"],
        "OS" => settings["software"]["os"]
      },
      path: "scripts/common.sh"
  end

  #create worker node vm

  (1..NUM_WORKER_NODES).each do |i|

    config.vm.define "node0#{i}" do |node|
      node.vm.synced_folder "C:\\Users\\apatel", "/mnt/", type: "virtualbox"
      node.vm.hostname = "worker-node0#{i}"
      node.vm.network "private_network", ip: IP_NW + "#{IP_START + i}"
        #virtualbox__intnet: true
      node.vm.provider "virtualbox" do |vb|
          vb.cpus = settings["nodes"]["workers"]["cpu"]
          vb.memory = settings["nodes"]["workers"]["memory"]
          if settings["cluster_name"] and settings["cluster_name"] != ""
            vb.customize ["modifyvm", :id, "--groups", ("/" + settings["cluster_name"])]
          end
      end
      node.vm.provision "shell",
        env: {
          "DNS_SERVERS" => settings["network"]["dns_servers"].join(" "),
          "ENVIRONMENT" => settings["environment"],
          "KUBERNETES_VERSION" => settings["software"]["kubernetes"],
          "OS" => settings["software"]["os"]
        },
        path: "scripts/common.sh"
    end
  end
end

  


