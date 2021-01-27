# -*- mode: ruby -*-
# vi: set ft=ruby :


master_ram=1536
node_ram=1536
nfs_ram=512

# node count 1..10
node_count=1

Vagrant.configure("2") do |config|
  config.vm.box = "ubuntu/bionic64"
  config.vm.synced_folder './tmp', '/conf.tmp'
  config.vm.synced_folder './tests', '/tests'
  config.vm.synced_folder './k8s_manifests', '/deploy'
  config.vm.provision "file", source: "hosts", destination: "/tmp/hosts"
  config.vm.provision "file", source: "~/.ssh/id_rsa.pub", destination: "~/.ssh/my.pub"
  config.vm.provision "shell", inline: <<-SHELL
    cat /tmp/hosts >> /etc/hosts
    cat /home/vagrant/.ssh/my.pub >> /home/vagrant/.ssh/authorized_keys
    apt update
    apt install tmux
  SHELL
  # create k8s master node
  config.vm.define "master" do |master|
      master.vm.provider "virtualbox" do |vb|
        vb.gui = false
        vb.memory = "#{master_ram}"
        vb.linked_clone = true
      end
      master.vm.hostname = "master"
      master.vm.network :"private_network", ip: "10.0.0.10", :name => 'vboxnet0', :adapter => 2
      master.vm.provision "ansible" do |ansible|
          ansible.verbose = "v"
          ansible.playbook = "playbooks/k8s_master.yml"
          ansible.raw_arguments = ["--diff"]
      end
  end
  # create k8s nodes
  (100..100+node_count-1).each do |i|
      config.vm.define "node#{i}" do |node00|
          node00.vm.provider "virtualbox" do |vb|
            vb.gui = false
            vb.memory = "#{master_ram}"
            vb.linked_clone = true
          end
          node00.vm.network :"private_network", ip: "10.0.0.#{i}", :name => 'vboxnet0', :adapter => 2
          node00.vm.hostname = "node#{i}"
          node00.vm.provision "ansible" do |ansible|
              ansible.verbose = "v"
              ansible.playbook = "playbooks/k8s_node.yml"
              ansible.raw_arguments = ["--diff"]
          end
      end
  end
  # create NFS server for persistent volumes
  config.vm.define "nfs" do |nfs|
      nfs.vm.provider "virtualbox" do |vb|
        vb.gui = false
        vb.memory = "#{master_ram}"
        vb.linked_clone = true
      end
      nfs.vm.hostname = "nfs"
      nfs.vm.network :"private_network", ip: "10.0.0.5", :name => 'vboxnet0', :adapter => 2
      nfs.vm.provision "ansible" do |ansible|
          ansible.verbose = "v"
          ansible.playbook = "playbooks/nfs.yml"
          ansible.raw_arguments = ["--diff"]
      end
  end
  config.trigger.before :up do |trigger|
    trigger.info = "Create tmp/ dir"
    trigger.run = {inline: "mkdir -p tmp/"}
  end
  config.trigger.before :destroy do |trigger|
    trigger.info = "Clean up tmp/*"
    trigger.run = {inline: "sh -c 'rm -rf tmp/*'"}
  end
end
