# -*- mode: ruby -*-
# vi: set ft=ruby :

$node_count = 3
$node_memory = 512
$ip_prefix = "192.168.99"

host_vars = {}
etcd_nodes = []

Vagrant.configure("2") do |config|
  config.vm.box = "jumperfly/centos-7"
  config.vm.box_version = "7.6.1"
  config.vm.box_check_update = false
  config.ssh.insert_key = false

  (1..$node_count).each do |i|
    config.vm.define vm_name = "etcd#{i}" do |config|
      config.vm.provision "shell", run: "always", inline: <<-SHELL
        systemctl stop firewalld
        systemctl disable firewalld
      SHELL
      etcd_nodes << vm_name
      config.vm.provider "virtualbox" do |v|
        v.memory = $node_memory
        v.cpus = 1
      end
      config.vm.hostname = vm_name
      ip = "#{$ip_prefix}.10#{i}"
      config.vm.network "private_network", ip: ip
      host_vars[vm_name] = {
        "ansible_ssh_common_args": "-o StrictHostKeyChecking=no",
        "ansible_host": ip,
        "etcd_iface": "eth1",
        "etcd_install_mode": "package"
      }
      if i == $node_count
        config.vm.box = "jumperfly/ansible-2.8"
        config.vm.box_version = "0.3"
        config.vm.provision "shell", privileged: false, inline: "cp /vagrant/vagrant_insecure_key /home/vagrant/.ssh/id_rsa && chmod 600 /home/vagrant/.ssh/id_rsa"
        config.vm.provision "shell", inline: <<-SHELL
          mkdir -p /etc/ansible/roles
          ln -snf /vagrant/ /etc/ansible/roles/etcd_node
          ansible-galaxy install --ignore-errors -r /vagrant/tests/requirements.yml -p /etc/ansible/roles
        SHELL
        config.vm.provision "ansible_local" do |ansible|
          ansible.compatibility_mode = "2.0"
          ansible.limit = "all"
          ansible.playbook = "tests/test.yml"
          ansible.host_vars = host_vars
          ansible.groups = {
            "etcd_nodes": etcd_nodes,
            "ca_nodes": [ "etcd1" ]
          }
        end
      end
    end
  end

end
