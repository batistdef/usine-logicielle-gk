# -*- mode: ruby -*-
# vi: set ft=ruby :

VAGRANTFILE_API_VERSION = "2"

# Toutes les VMs sont definies dans ce fichier
hosts = YAML.load_file('hosts.yml')

Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|
  # Verification de la presence de certains plugins
  required_plugins = %w(vagrant-hostmanager vagrant-vbguest)
  plugin_installed = false
  required_plugins.each do |plugin|
    unless Vagrant.has_plugin?(plugin)
          system "vagrant plugin install #{plugin}"  
      plugin_installed = true
    end
  end

  # Restart vagrant si on a installe un plugin
  if plugin_installed === true
    exec "vagrant #{ARGV.join' '}"
  end

  # Host Manager configuration
  config.hostmanager.enabled = true
  config.hostmanager.manage_host = true
  config.hostmanager.manage_guest = true
  config.hostmanager.include_offline= true
  config.hostmanager.ignore_private_ip = false

  # No updates
  config.vm.box_check_update = false
  config.vbguest.auto_update = false

  $script_inject_pk =<<-'SCRIPT'
    cat /vagrant/id_rsa.pub >> /home/vagrant/.ssh/authorized_keys
  SCRIPT

  hosts.each do |host|
    config.vm.define host['name'] do |node|
      # - HOST SETTINGS -
      node.vm.hostname = host['name']
      node.vm.network :private_network, ip: host['ip']
      node.vm.box = host['os'] 
      node.hostmanager.aliases = host['aliases']

      node.vm.provision "shell", inline: $script_inject_pk

      # - PROVISION -
      if host['provisioner'] == 'shell'
        node.vm.provision "shell",privileged: true, path: host['playbook']
      elsif host['provisioner'] == 'ansible'
        node.vm.provision "ansible" do |ansible|
          ansible.playbook = host['playbook']
          ansible.verbose = "False"
        end
      end

      # - PROVIDER -
      node.vm.provider :virtualbox do |vb|
        vb.gui = false
        vb.name = host['name']
        vb.customize ["modifyvm", :id, "--memory", host['mem'], "--cpus", host['cpus'], "--hwvirtex", "on","--groups", "/usine-logicielle",
        "--natdnshostresolver1", "on", "--cableconnected1", "on"]
      end      
    end
  end
end
