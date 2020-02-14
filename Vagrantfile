# -*- mode: ruby -*-
# vi: set ft=ruby :

$node_memory = 512
$ip_prefix = "192.168.99"

Vagrant.configure("2") do |config|
  config.vm.box = "jumperfly/centos-7"
  config.vm.box_version = "7.7.3"
  config.vm.box_check_update = false
  config.ssh.insert_key = false
  config.vm.provider "virtualbox" do |v|
    v.memory = $node_memory
    v.cpus = 1
  end

  (1..4).each do |i|
    config.vm.define vm_name = "etcd#{i}" do |config|
      config.vm.provision "shell", inline: <<-SHELL
        systemctl stop firewalld
        systemctl disable firewalld
      SHELL
      config.vm.hostname = vm_name
      ip = "#{$ip_prefix}.10#{i}"
      config.vm.network "private_network", ip: ip
    end
  end

  config.vm.define "ansible" do |config|
    config.vm.box = "jumperfly/ansible-2.8"
    config.vm.box_version = "5.5"
    config.vm.hostname = "ansible"
    config.vm.network "private_network", ip: "#{$ip_prefix}.100"
    config.vm.provision "shell", inline: <<-SHELL
      mkdir -p /etc/ansible/roles
      ln -snf /vagrant/ /etc/ansible/roles/etcd_node
      ansible-galaxy install --ignore-errors -r /vagrant/tests/requirements.yml -p /etc/ansible/roles
      if ! rpm -q sshpass > /dev/null; then
        yum -y install sshpass
      fi
    SHELL
    config.vm.provision "ansible_local" do |ansible|
      ansible.compatibility_mode = "2.0"
      ansible.limit = "all"
      ansible.playbook = "tests/test.yml"
      ansible.inventory_path = "tests/inventory-vagrant"
    end
  end

end
