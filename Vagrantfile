# -*- mode: ruby -*-
# vi: set ft=ruby :

unless Vagrant.has_plugin?("vagrant-hostmanager")
  raise 'vagrant-hostmanager plugin is not installed!'
end

$node_count = 3
$node_memory = 256
$ip_prefix = "192.168.99"

host_vars = {}

Vagrant.configure("2") do |config|
  config.vm.box = "centos/7"
  config.ssh.insert_key = false
  config.hostmanager.enabled = true

  config.vm.define "ca" do |config|
    config.vm.provider "virtualbox" do |v|
      v.memory = $node_memory
      v.cpus = 1
    end
    config.vm.network "private_network", ip: "#{$ip_prefix}.100"
  end

  (1..$node_count).each do |i|
    config.vm.define vm_name = "etcd#{i}" do |config|
      config.vm.provider "virtualbox" do |v|
        v.memory = $node_memory
        v.cpus = 1
      end
      config.vm.hostname = vm_name
      ip = "#{$ip_prefix}.10#{i}"
      config.vm.network "private_network", ip: ip
      host_vars[vm_name] = {
        "ip": ip
      }
      if i == $node_count
        config.vm.provision "shell", privileged: false, inline: "cp /vagrant/vagrant_insecure_key /home/vagrant/.ssh/id_rsa && chmod 600 /home/vagrant/.ssh/id_rsa"
        config.vm.provision "ansible_local" do |ansible|
          ansible.compatibility_mode = "2.0"
          ansible.playbook = "tests/vagrant-deps.yml"
        end
        config.vm.provision "ansible_local" do |ansible|
          ansible.compatibility_mode = "2.0"
          ansible.limit = "all"
          ansible.playbook = "tests/test.yml"
          ansible.host_vars = host_vars
        end
      end
    end
  end

end
