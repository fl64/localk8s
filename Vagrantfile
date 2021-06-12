# -*- mode: ruby -*-
# vi: set ft=ruby :

_k8s_master_node_ip="192.168.111.10"
_k8s_worker_node_net="192.168.111"
_k8s_bootstrap_token="2i9vqf.h8blqvijpi41qum6"
_k8s_version="1.19.0"

default_node_count=1
master_ram=1536
node_ram=3072
nfs_ram=512

# node count 1..10
node_count = (ENV['K8S_NODE_COUNT'] || default_node_count).to_i
tags = (ENV['K8S_TAGS'] || "all")

Vagrant.configure("2") do |config|
  config.vm.box = "ubuntu/focal64"
  config.vm.synced_folder './tests', '/tests'
  config.vm.provision "file", source: "~/.ssh/id_rsa.pub", destination: "~/.ssh/my.pub"
  config.vm.provision "shell", inline: <<-SHELL
    cat /home/vagrant/.ssh/my.pub >> /home/vagrant/.ssh/authorized_keys
  SHELL
  # create k8s master node
  config.vm.define "master" do |master|
      master.vm.provider "virtualbox" do |vb|
        vb.gui = false
        vb.memory = "#{master_ram}"
        vb.linked_clone = true
      end
      master.vm.hostname = "master"
      master.vm.network :"private_network", ip: "#{_k8s_master_node_ip}", :name => 'vboxnet0', :adapter => 2
      master.vm.provision "ansible" do |ansible|
          ansible.galaxy_roles_path = "roles/"
          ansible.verbose = "vv"
          ansible.playbook = "playbooks/k8s_master.yml"
          ansible.raw_arguments = ["--diff"]
          ansible.tags = "#{tags}"
          ansible.extra_vars = {
            _k8s_version: "#{_k8s_version}",
            _k8s_bootstrap_token: "#{_k8s_bootstrap_token}",
            _k8s_master_node_ip: "#{_k8s_master_node_ip}"
          }
      end
  end

  # create k8s nodes
  (100..100+node_count-1).each do |i|
      config.vm.define "node#{i}" do |node00|
          node00.vm.provider "virtualbox" do |vb|
            vb.gui = false
            vb.memory = "#{node_ram}"
            vb.linked_clone = true
          end
          node00.vm.network :"private_network", ip: "#{_k8s_worker_node_net}.#{i}", :name => 'vboxnet0', :adapter => 2
          node00.vm.hostname = "node#{i}"
          node00.vm.provision "ansible" do |ansible|
              ansible.galaxy_roles_path = "roles/"
              ansible.verbose = "v"
              ansible.playbook = "playbooks/k8s_node.yml"
              ansible.raw_arguments = ["--diff"]
              ansible.extra_vars = {
                _k8s_version: "#{_k8s_version}",
                _k8s_bootstrap_token: "#{_k8s_bootstrap_token}",
                _k8s_master_node_ip: "#{_k8s_master_node_ip}"
              }
              ansible.tags = "#{tags}"
          end
      end
  end
  # # create NFS server for persistent volumes
  # config.vm.define "nfs" do |nfs|
  #     nfs.vm.provider "virtualbox" do |vb|
  #       vb.gui = false
  #       vb.memory = "#{master_ram}"
  #       vb.linked_clone = true
  #     end
  #     nfs.vm.hostname = "nfs"
  #     nfs.vm.network :"private_network", ip: "10.0.0.5", :name => 'vboxnet0', :adapter => 2
  #     nfs.vm.provision "ansible" do |ansible|
  #         ansible.verbose = "v"
  #         ansible.playbook = "playbooks/nfs.yml"
  #         ansible.raw_arguments = ["--diff"]
  #     end
  # end
  # config.trigger.before :up do |trigger|
  #   trigger.info = "Create tmp/ dir"
  #   trigger.run = {inline: "mkdir -p tmp/"}
  # end
  # config.trigger.before :destroy do |trigger|
  #   trigger.info = "Clean up tmp/*"
  #   trigger.run = {inline: "sh -c 'rm -rf tmp/*'"}
  # end
end
