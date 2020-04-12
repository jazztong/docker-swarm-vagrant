# -*- mode: ruby -*-
# vi: set ft=ruby :

$install_docker_script = <<SCRIPT
echo "Installing dependencies ..."
sudo apt-get update
echo Installing Docker...
curl -sSL https://get.docker.com/ | sh
sudo usermod -aG docker vagrant
SCRIPT

BOX_NAME = "ubuntu/xenial64"
MEMORY = "512"
MANAGERS = 2
MANAGER_IP = "172.20.20.1"
WORKERS = 2
WORKER_IP = "172.20.20.10"
CPUS = 2
VAGRANTFILE_API_VERSION = "2"

Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|
    #Common setup
    config.vm.box = BOX_NAME
    config.vm.synced_folder ".", "/vagrant"
    config.vm.provision "shell",inline: $install_docker_script, privileged: true
    config.vm.provider "virtualbox" do |vb|
      vb.memory = MEMORY
      vb.cpus = CPUS
    end
    #Setup Manager Nodes
    (1..MANAGERS).each do |i|
        config.vm.define "manager0#{i}" do |manager|
          manager.vm.network :private_network, ip: "#{MANAGER_IP}#{i}"
          manager.vm.hostname = "manager0#{i}"
          if i == 1
            #Only configure port to host for Manager01
            manager.vm.network :forwarded_port, guest: 8080, host: 8080
            manager.vm.network :forwarded_port, guest: 5000, host: 5000
            manager.vm.network :forwarded_port, guest: 9000, host: 9000
          end
        end
    end
    #Setup Woker Nodes
    (1..WORKERS).each do |i|
        config.vm.define "worker0#{i}" do |worker|
            worker.vm.network :private_network, ip: "#{WORKER_IP}#{i}"
            worker.vm.hostname = "worker0#{i}"
        end
    end
end