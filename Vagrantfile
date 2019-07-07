# -*- mode: ruby -*-
# vi: set ft=ruby :

image = "sbeliakou/centos"

Vagrant.configure("2") do |config|
    [
        { :hostname => 'master', :ip => '192.168.56.224', :ram => 1536 },
        { :hostname => 'worker', :ip => '192.168.56.225', :ram => 1024 }
    ].each do |node|
        config.vm.define node[:hostname] do |nodeconfig|
            nodeconfig.vm.box = image
            nodeconfig.vm.hostname = node[:hostname]
            nodeconfig.vm.network :private_network, ip: node[:ip]
            nodeconfig.ssh.insert_key = false
            
            nodeconfig.vm.provider :virtualbox do |vb|
                vb.name = node[:hostname]
                vb.memory = node[:ram] ? node[:ram] : "1536"
                vb.customize ["modifyvm", :id, "--natdnshostresolver1", "on"]
                vb.customize ["modifyvm", :id, "--cpuexecutioncap", "30"]
            end
            
            nodeconfig.vm.provision 'shell' do |shell|
                shell.inline = 'echo provisioning'
            end
        end
    end
end
