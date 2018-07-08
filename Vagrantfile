# -*- mode: ruby -*-
# vi: set ft=ruby :

########## SETTINGS AND REQUIREMENTS #########
unless Vagrant.has_plugin? 'vagrant-hostmanager'
  raise 'Required plugin vagrant-hostmanager not found. Run `vagrant plugin install vagrant-hostmanager` to install.'
end

require 'yaml'

Dir.chdir(File.dirname __FILE__)

settings = YAML.load_file 'vagrant.yml' if File.exists? 'vagrant.yml'

# All Vagrant configuration is done below. The "2" in Vagrant.configure
# configures the configuration version (we support older styles for
# backwards compatibility). Please don't change it unless you know what
# you're doing.
Vagrant.configure(2) do |config|
  # The most common configuration options are documented and commented below.
  # For a complete reference, please see the online documentation at
  # https://docs.vagrantup.com.

  # Every Vagrant development environment requires a box. You can search for
  # boxes at https://atlas.hashicorp.com/search.


  ####### GENERAL SETTINGS / HOSTNAME #######
  config.hostmanager.enabled = true
  config.hostmanager.manage_host = true
  config.vm.hostname = 'drupal-dev.example.com'
  config.hostmanager.aliases = %w(drupal-dev.example.fr drupal-dev.example.co.uk drupal-dev-au.example.com drupal-dev-nl.example.com drupal-dev.example.de drupal-dev.example.es drupal-dev-br-example.com drupal-dev-it.example.com drupal-dev-es-la.example.com drupal-dev.example.ca drupal-dev-fr.example.ca drupal-dev-la.example.com drupal-dev-assets0.example.com drupal-dev-assets.example.com drupal-dev-assets1.example.com drupal-dev-assets2.example.com drupal-dev-assets3.example.com)

  # We need this so vagrant can ssh into itself when running Capistrano
  config.ssh.forward_agent = true

  ####### DEFAULT BOX (VIRTUALBOX) #########
  config.vm.box = "geerlingguy/centos7" # virtualbox, vmware_desktop

  # Disable automatic box update checking. If you disable this, then
  # boxes will only be checked for updates when the user runs
  # `vagrant box outdated`. This is not recommended.
  # config.vm.box_check_update = false

  # Create a forwarded port mapping which allows access to a specific port
  # within the machine from a port on the host machine. In the example below,
  # accessing "localhost:8080" will access port 80 on the guest machine.
   config.vm.network "forwarded_port", guest: 80, host: 8080

  # Create a private network, which allows host-only access to the machine
  # using a specific IP.
  # config.vm.network "private_network", ip: "192.168.33.10"


  ######### NETWORKING (cuz of vagrant-hostmanager) ###########
  # config.vm.network "private_network", ip: '10.212.55.50'
  # MOVED to provider-specific config.

  # Create a public network, which generally matched to bridged network.
  # Bridged networks make the machine appear as another physical device on
  # your network.
  # config.vm.network "public_network"

  # Share an additional folder to the guest VM. The first argument is
  # the path on the host to the actual folder. The second argument is
  # the path on the guest to mount the folder. And the optional third
  # argument is a set of non-required options.
  # config.vm.synced_folder "../data", "/vagrant_data"

  ######## SHARED FOLDERS ##########
  config.vm.synced_folder ".", "/data/apps/drupal/current"
  config.vm.synced_folder "/home/adri93/Escritorio/Code/TFM/drupal-vagrant/datos/", "/data/apps/drupal/shared/files"

  # Provider-specific configuration so you can fine-tune various
  # backing providers for Vagrant. These expose provider-specific options.
  # Example for VirtualBox:
  #
  # config.vm.provider "virtualbox" do |vb|
  #   # Display the VirtualBox GUI when booting the machine
  #   vb.gui = true
  #
  #   # Customize the amount of memory on the VM:
  #   vb.memory = "1024"
  # end
  #
  # View the documentation for the provider you are using for more
  # information on available options.


  ####### PROVIDER CONFIGURATION ########
  configured_cpus = (settings && (settings.has_key? 'cpus') and (settings['cpus'])) ? settings['cpus'] : 4
  configured_mem = (settings && (settings.has_key? 'memory') and (settings['memory'])) ? settings['memory'] : 2048

  config.vm.provider "virtualbox" do |vb, override|
    # Customize the amount of memory on the VM:
    vb.cpus = configured_cpus
    vb.memory = configured_mem

    machine_ip = (settings && settings.has_key?('parallels_ip') and settings['virtualbox_ip']) ? settings['virtualbox_ip'] : '10.213.55.50'
    override.vm.network "private_network", ip: machine_ip

    # NFS doesn't work for VPN users. Let them configure something else.
    # Set nfs: false in vagrant.yml to use regular VBox Shared Folders.
    if settings and settings.has_key? 'nfs' and settings['nfs'] == 'false'
      override.vm.synced_folder ".", "/data/apps/drupal/current", type: nil
      override.vm.synced_folder "data/files", "/data/apps/drupal/shared/files", :type => nil
    end
  end

  config.vm.provider "parallels" do |p, override|
    override.vm.box = 'parallels/centos-6.6'
    p.update_guest_tools = true

    p.cpus = configured_cpus
    p.memory = configured_mem

    machine_ip = (settings && settings.has_key?('parallels_ip') and settings['parallels_ip']) ? settings['parallels_ip'] : '10.212.55.50'
    override.vm.network "private_network", ip: machine_ip

    if settings and settings.has_key? 'nfs' and settings['nfs'] == 0
      override.vm.synced_folder ".", "/data/apps/drupal/current", type: nil
      override.vm.synced_folder "data/files", "/data/apps/drupal/shared/files", :type => nil
    end
  end

  config.vm.provider "lxc" do |l, override|
    # I think LXC just shares host resources by default.

    override.vm.box = 'fgrehm/centos-6-64-lxc'
    override.vm.synced_folder ".", "/data/apps/drupal/current"
    override.vm.synced_folder "data/files", "/data/apps/drupal/shared/files"

    machine_ip = (settings && settings.has_key?('lxc_ip') and settings['lxc_ip']) ? settings['lxc_ip'] : '10.214.55.50'
    override.vm.network "private_network", ip: machine_ip
  end

  # Define a Vagrant Push strategy for pushing to Atlas. Other push strategies
  # such as FTP and Heroku are also available. See the documentation at
  # https://docs.vagrantup.com/v2/push/atlas.html for more information.
  # config.push.define "atlas" do |push|
  #   push.app = "YOUR_ATLAS_USERNAME/YOUR_APPLICATION_NAME"
  # end

  # Enable provisioning with a shell script. Additional provisioners such as
  # Puppet, Chef, Ansible, Salt, and Docker are also available. Please see the
  # documentation for more information about their specific syntax and use.
  # config.vm.provision "shell", inline <<-SHELL
  #   sudo apt-get install apache2
  # SHELL

  ###### ANSIBLE SETUP ########
  config.vm.provision "ansible" do |ansible|
    ansible.playbook = "lib/ansible/test.yml"
    ansible.extra_vars = {
        'vagrant_hostmanager_aliases' => (config.hostmanager.aliases.join ' '),
    }
    ansible.verbose = 'v'

    if ENV.has_key?'ANSIBLE_START_AT_TASK'
      ansible.start_at_task = ENV['ANSIBLE_START_AT_TASK']
    end
  end

  $script = <<SCRIPT
/bin/bash
echo 'Provisioning complete! Run `vagrant provision` to re-run this.'
SCRIPT
  config.vm.provision "shell", inline: $script

  ####### CHANGES POST-SETUP #######

  # @todo: Use exampleapp user to log in by default instead of vagrant. Make Ansible output a file into /vagrant or something as a flag. See if I can somehow remove it in a vagrant destroy hook. Might need a plugin...
end
