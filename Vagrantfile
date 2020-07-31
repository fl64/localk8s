# -*- mode: ruby -*-
# vi: set ft=ruby :
goss_version="v0.3.9"


vms = {
  "nfs"    =>  { :ip => "10.0.0.5",  :ram => 512,  :playbook => "playbooks/nfs.yml" },
  "master" =>  { :ip => "10.0.0.10", :ram => 1536, :playbook => "playbooks/k8s.yml" },
  "node01" =>  { :ip => "10.0.0.20", :ram => 1536, :playbook => "playbooks/k8s.yml" },
  "node02" =>  { :ip => "10.0.0.21", :ram => 1536, :playbook => "playbooks/k8s.yml" },
  "node03" =>  { :ip => "10.0.0.22", :ram => 1536, :playbook => "playbooks/k8s.yml" }

}

Vagrant.configure("2") do |config|
  config.vm.box = "ubuntu/bionic64"
  vms.each_with_index do |(name, conf), index|
    config.vm.define name do |machine|

      machine.vm.provider "virtualbox" do |vb|
        vb.gui = false
        vb.memory = "#{conf[:ram]}"
        vb.linked_clone = true
      end
      machine.vm.synced_folder './tests', '/tests'
      machine.vm.synced_folder './k8s_manifests', '/deploy'
      machine.vm.network :"private_network", ip: "#{conf[:ip]}"
      machine.vm.hostname = "#{name}"
      machine.vm.provision "file", source: "hosts", destination: "/tmp/hosts"
      machine.vm.provision "file", source: "~/.ssh/id_rsa.pub", destination: "~/.ssh/my.pub"
      machine.vm.provision "shell", inline: <<-SHELL
        cat /tmp/hosts >> /etc/hosts
        cat /home/vagrant/.ssh/my.pub >> /home/vagrant/.ssh/authorized_keys
      SHELL
      machine.vm.provision "ansible" do |ansible|
        ansible.verbose = "v"
        #ansible.galaxy_role_file = "requirements.yml"
        #ansible.galaxy_roles_path = "./roles/"
        ansible.playbook = "#{conf[:playbook]}"
        ansible.raw_arguments = ["--diff"]
        ansible.limit = "all"
        ansible.groups = {
          "masters" => ["master"],
          "nodes" =>   ["node01","node02","node03"],
          "nfs" => ["nfs"]
        }
      end
      #machine.vm.provision "shell", inline: "goss -g /tmp/goss.yml validate"
    end
  end
end
