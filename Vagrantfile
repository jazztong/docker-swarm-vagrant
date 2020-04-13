# -*- mode: ruby -*-
# vi: set ft=ruby :

$install_docker_script = <<SCRIPT
echo "Installing dependencies ..."
sudo apt-get update

echo Installing Docker...
curl -sSL https://get.docker.com/ | sh
sudo usermod -aG docker vagrant
SCRIPT

$manager_script = <<SCRIPT
echo Swarm Init...
sudo docker swarm init --listen-addr 172.20.20.11:2377 --advertise-addr 172.20.20.11:2377
sudo docker swarm join-token --quiet worker > /vagrant/worker_token
sudo docker swarm join-token --quiet manager > /vagrant/manager_token
SCRIPT

$manager_join_script = <<SCRIPT
echo Swarm Join...
sudo docker swarm join --token $(cat /vagrant/manager_token) 172.20.20.11:2377
SCRIPT

$worker_script = <<SCRIPT
echo Swarm Join...
sudo docker swarm join --token $(cat /vagrant/worker_token) 172.20.20.11:2377
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
    config.vm.synced_folder "/var/tmp/vagrant/apt-archives/", "/var/cache/apt/archives/", create: true
    config.vm.synced_folder "/var/tmp/vagrant/apt-lists/", "/var/lib/apt/lists", create: true
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
            manager.vm.network :forwarded_port, guest: 8000, host: 8000
            manager.vm.provision "shell",inline: $manager_script, privileged: true
          else
            manager.vm.provision "shell",inline: $manager_join_script, privileged: true
          end
        end
    end
    #Setup Woker Nodes
    (1..WORKERS).each do |i|
        config.vm.define "worker0#{i}" do |worker|
          worker.vm.provision "shell",inline: $worker_script, privileged: true
          worker.vm.network :private_network, ip: "#{WORKER_IP}#{i}"
          worker.vm.hostname = "worker0#{i}"
        end
    end
end