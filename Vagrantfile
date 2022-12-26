Vagrant.require_version "= 2.3.0"
Vagrant.configure("2") do |config|
    # (Kubernetes master)
    config.vm.define "master" do |master|
        master.vm.box = "generic/ubuntu1804"
        master.vm.box_version = "4.1.12"
        master.vm.hostname = "drone-host"
        master.vm.network "private_network", ip: "192.168.50.4"
        master.vm.provider "virtualbox" do |v|
            v.name = "drone-vm"
            v.customize ["modifyvm", :id, "--cpuexecutioncap", "50"]
            v.memory = 2048
            v.cpus = 2
        end
    end

    # creating second VM (Kubernetes node)
    config.vm.define "node" do |node|
        node.vm.box = "generic/ubuntu1804"
        node.vm.box_version = "4.1.12"
        node.vm.hostname = "kubernetes-node"
        node.vm.network "private_network", ip: "192.168.50.5"
        node.vm.provider "virtualbox" do |v|
            v.name = "kub-node-vm"
            v.customize ["modifyvm", :id, "--cpuexecutioncap", "50"]
            v.memory = 2048
            v.cpus = 2
        end
    end
end