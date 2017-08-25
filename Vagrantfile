# -*- mode: ruby -*-
# vi: set ft=ruby :

require 'yaml'

DEFAULT_BASE_BOX = "centos-7.3-base"

environment = ENV['VAGRANT_ENV'] || 'localenv'

hosts = YAML.load_file("vagrant_envs/#{environment}.yml")

def insert_ansible_key(config, osx = false)
  dest_folder = osx ? "/Users/vagrant" : "/home/vagrant"
  config.vm.provision "shell", 
       inline: "cat /vagrant/ssh_keys/provisioning.pub >> #{dest_folder}/.ssh/authorized_keys"
end

def modify_box(virtualbox_config, host_config)
    virtualbox_config.memory = host_config['memory'] if host_config['memory']
    virtualbox_config.cpus = host_config['cpu'] if host_config['cpu']
end

Vagrant.configure("2") do |config|

  hosts.each do |host|
    config.vm.define host['name'] do |node|
      node.vm.box = host['box'] ||= DEFAULT_BASE_BOX
      
      node.vm.network :private_network, ip: host["ip"]
      if host["osx"]
        node.vm.synced_folder ".", "/vagrant", type: "rsync" 
      end 
      insert_ansible_key(node, host["osx"])
      node.vm.provider :virtualbox do | vb |
        vb.customize ["modifyvm", :id, "--natdnshostresolver1", "on"]
        vb.customize ["modifyvm", :id, "--natdnsproxy1", "on"]
        modify_box(vb, host)
      end
    end
  end

end
