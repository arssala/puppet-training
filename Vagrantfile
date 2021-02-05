# -*- mode: ruby -*-
# vi: set ft=ruby :


Vagrant.configure("2") do |config|
  config.vm.define "puppet_master" do |subconfig|
    subconfig.vm.box = "centos/7"
    subconfig.vm.hostname = "puppet"
    subconfig.vm.network :private_network, ip: "192.168.199.10"
#    subconfig.vm.network "forwarded_port", guest: 80, host: 8080
    subconfig.vm.synced_folder ".", "/vagrant", type: "rsync", rsync__exclude: ".git/"
    subconfig.vm.provider "virtualbox" do |vb|
			vb.memory = "8000"
		end
  end

  config.vm.define "puppet_agent" do |subconfig|
    subconfig.vm.box = "centos/7"
    subconfig.vm.hostname = "agent"
    subconfig.vm.network :private_network, ip: "192.168.199.11"
    subconfig.vm.synced_folder ".", "/vagrant", type: "rsync", rsync__exclude: ".git/"
  end

  config.vm.define "puppet_gitlab" do |subconfig|
    subconfig.vm.box = "centos/7"
    subconfig.vm.hostname = "gitlab"
    subconfig.vm.network :private_network, ip: "192.168.199.11"
    subconfig.vm.synced_folder ".", "/vagrant", type: "rsync", rsync__exclude: ".git/"
  end
end
