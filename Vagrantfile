#test committ
Vagrant.require_version "= 2.3.0"
Vagrant.configure("2") do |config|
    config.vm.box = "generic/ubuntu1804"
    config.vm.box_version = "4.1.12"
    config.vm.hostname = "drone-host"
    config.vm.network "private_network", ip: "192.168.50.4"
    config.vm.provider "virtualbox" do |v|
        v.name = "drone-vm"
        v.customize ["modifyvm", :id, "--cpuexecutioncap", "50"]
        v.memory = 512
        v.cpus = 1
    end
    config.vm.synced_folder ".", "/vagrant", type: "rsync", rsync__exclude: [".git/", ".gitignore"]
end