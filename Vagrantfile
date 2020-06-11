IMAGE_NAME = "centos/7"
MASTER_COUNT = 2
WORKER_COUNT = 2

Vagrant.configure("2") do |config|
    config.ssh.insert_key = false

    config.vm.provider "virtualbox" do |v|
        v.memory = 1024
        v.cpus = 2
    end

    (1..MASTER_COUNT).each do |i|
        config.vm.define "master-#{i}" do |master|
            master.vm.box = IMAGE_NAME
            master.vm.network "private_network", ip: "192.168.50.#{i + 10}"
            master.vm.hostname = "master-#{i}"
        end
    end


    (1..WORKER_COUNT).each do |i|
        config.vm.define "node-#{i}" do |node|
            node.vm.box = IMAGE_NAME
            node.vm.network "private_network", ip: "192.168.50.#{i + 20}"
            node.vm.hostname = "node-#{i}"
        end
    end

    config.vm.define "k8s-haproxy" do |lb|
        lb.vm.box = IMAGE_NAME
        lb.vm.network "private_network", ip: "192.168.50.50"
        lb.vm.hostname = "k8s-lb"
    end

    config.vm.define "home" do |home|
        home.vm.box = IMAGE_NAME
        home.vm.network "private_network", ip: "192.168.50.100"
        home.vm.hostname = "home"
    end

end