# -*- mode: ruby -*-
# vim: set ft=ruby :

servers={
  :elk=>{
    :hostname => "elk",
    :box_name => "centos/7",
    :box_version => "2004.1",
    :memory => "4096",
    :cpu => "2",
    :ip => "192.168.56.7"
  },
  :backup=>{
    :hostname => "backup",
    :box_name => "centos/7",
    :box_version => "2004.1",
    :memory => "1024",
    :cpu => "1",
    :ip => "192.168.56.6"
  },
  :prometheus=>{
    :hostname => "prometheus",
    :box_name => "centos/7",
    :box_version => "2004.1",
    :memory => "1024",
    :cpu => "1",
    :ip => "192.168.56.5"
  },
  :apache_mysql_slave=>{
    :hostname => "apache-mysql-slave",
    :box_name => "centos/7",
    :box_version => "2004.1",
    :memory => "2048",
    :cpu => "1",
    :ip => "192.168.56.4"
  },
  :nginx_apache_mysql=>{
    :hostname => "nginx-apache-mysql",
    :box_name => "centos/7",
    :box_version => "2004.1",
    :memory => "2048",
    :cpu => "1",
    :ip => "192.168.56.3"
  },
  :ansible=>{
    :hostname => "ansible",
    :box_name => "centos/7",
    :box_version => "2004.1",
    :memory => "1024",
    :cpu => "1",
    :ip => "192.168.56.2"
  }
}

Vagrant.configure("2") do |config|
  servers.each_with_index do |(machinename,machineconfig),index|
      config.vm.define machinename do |node|
          node.vm.box_check_update = false
          node.vm.box = machineconfig[:box_name]
          node.vm.box_version = machineconfig[:box_version]
          node.vm.host_name = machineconfig[:hostname]
          node.vm.network "private_network", ip: machineconfig[:ip]
          node.vm.provider "virtualbox" do |vb|
              vb.customize ["modifyvm", :id, "--memory", machineconfig[:memory]]
              vb.customize ["modifyvm", :id, "--cpus", machineconfig[:cpu]]
          end
          node.vm.provision "ssh_keys", type: "shell", run: "always" do |s|
            ssh_pub_key = File.readlines("#{Dir.home}/.ssh/id_rsa.pub").first.strip
            s.inline = <<-SHELL
              sudo mkdir -p ~root/.ssh
              echo #{ssh_pub_key} >> /home/vagrant/.ssh/authorized_keys
              echo #{ssh_pub_key} >> /root/.ssh/authorized_keys
            SHELL
          end
          node.vm.provision "copy_private", after: "ssh_keys", type: "file", run: "always", source: "/root/.ssh/id_rsa", destination: "/tmp/.ssh/id_rsa"
          node.vm.provision "move_private", after: "copy_private",  type: "shell", inline: <<-'SHELL'
            cp /tmp/.ssh/id_rsa /root/.ssh/
            mv /tmp/.ssh/id_rsa /home/vagrant/.ssh/
            chown root /root/.ssh/id_rsa
          SHELL
      if index == servers.size - 1
        node.vm.provision "ansible_roles", type: "shell", inline: <<-'SHELL'
          sudo mkdir /etc/ansible/roles -p
          sudo chmod o+w /etc/ansible/roles
        SHELL
        node.vm.provision "run_playbook", run: "always", type: "ansible_local" do |ansible|
          ansible.galaxy_role_file = "requirements.yml"
          ansible.galaxy_roles_path = "/etc/ansible/roles"
          ansible.galaxy_command = "ansible-galaxy collection install -r %{role_file}"
          ansible.inventory_path = "inventory.yml"
          ansible.playbook = "playbook.yml"
          ansible.limit = "all"
          ansible.verbose = "vvv"
        end
      end
    end
  end
end