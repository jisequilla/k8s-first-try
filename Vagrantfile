# -*- mode: ruby -*-
# vi: set ft=ruby :

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
  # 	config.vm.box = "base"

  # Disable automatic box update checking. If you disable this, then
  # boxes will only be checked for updates when the user runs
  # `vagrant box outdated`. This is not recommended.
  # config.vm.box_check_update = false

  # Create a forwarded port mapping which allows access to a specific port
  # within the machine from a port on the host machine. In the example below,
  # accessing "localhost:8080" will access port 80 on the guest machine.
  # config.vm.network "forwarded_port", guest: 80, host: 8080

  # Create a private network, which allows host-only access to the machine
  # using a specific IP.
  # config.vm.network "private_network", ip: "192.168.33.10"

  # Create a public network, which generally matched to bridged network.
  # Bridged networks make the machine appear as another physical device on
  # your network.
  # config.vm.network "public_network"

  # Share an additional folder to the guest VM. The first argument is
  # the path on the host to the actual folder. The second argument is
  # the path on the guest to mount the folder. And the optional third
  # argument is a set of non-required options.
  # config.vm.synced_folder "../data", "/vagrant_data"

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

  # Define a Vagrant Push strategy for pushing to Atlas. Other push strategies
  # such as FTP and Heroku are also available. See the documentation at
  # https://docs.vagrantup.com/v2/push/atlas.html for more information.
  # config.push.define "atlas" do |push|
  #   push.app = "YOUR_ATLAS_USERNAME/YOUR_APPLICATION_NAME"
  # end

  # Enable provisioning with a shell script. Additional provisioners such as
  # Puppet, Chef, Ansible, Salt, and Docker are also available. Please see the
  # documentation for more information about their specific syntax and use.
  # config.vm.provision "shell", inline: <<-SHELL
  #   sudo apt-get update
  #   sudo apt-get install -y apache2
  # SHELL

  config.hostmanager.enabled = true

  config.vm.synced_folder ".", "/vagrant", type: "virtualbox", disabled: false,
  rsync__exclude: ".git/"

  $centos_box = "centos/7"
  $fedora_box ="fedora/27-cloud-base"

  $script_host_file = <<SCRIPT
	echo I provisioning host file...
  sudo cp /vagrant/config/hosts /etc/hosts
SCRIPT

# Configure new ssh key for server and creates a copy for the rest of the servers.
$script_ssh_credentials_creation = <<SCRIPT
        echo "Creating and updateing credentials for direct ssh"
        if [ ! -f /home/vagrant/.ssh/id_rsa.pub ]; then
          ssh-keygen -t rsa -N "" -f /home/vagrant/.ssh/id_rsa
        fi

        cp /home/vagrant/.ssh/id_rsa.pub /vagrant/control.pub

        cat << 'SSHEOF' > /home/vagrant/.ssh/config
        Host *
        StrictHostKeyChecking no
        UserKnownHostsFile=/dev/null
SSHEOF
        chown -R vagrant:vagrant /home/vagrant/.ssh/
SCRIPT


    $script_copy_key = 'cat /vagrant/control.pub >> /home/vagrant/.ssh/authorized_keys'

    $script_install_ansible = <<SCRIPT
    	echo Installin ansible.
        
	#
	# Update and install basic linux programs for development
	#
	sudo yum update -y
	sudo yum install -y wget
	sudo yum install -y curl
	sudo yum install -y vim
	sudo yum install -y git
	sudo yum install -y build-essential
	sudo yum install -y unzip
	#
	# Install Ansible
	# 
	cd /usr/local/src
	sudo yum install -y git python-jinja2 python-paramiko PyYAML make MySQL-python
  	sudo yum upgrade -y python-setuptools
	sudo yum install -y python-pip python-wheel
	sudo yum install -y python-setuptools
  	sudo yum install -y epel-release
  	sudo yum install -y ansible

	
SCRIPT

  #config.vm.box = $current_box 

  #config.vm.synced_folder "./vagrant_data/", "/home/vagrant/sync", type: "virtualbox"

  config.vm.define "bastion", primary: true do |controlserver|
    
    controlserver.vm.box = $centos_box
    controlserver.vm.hostname = "bastion"
    controlserver.vm.network "private_network", ip: "192.168.135.10"

    config.vm.provider :virtualbox do |vb|
      vb.customize ["modifyvm", :id, "--memory", "512"]
      vb.customize ["modifyvm", :id, "--cpus", "1"]
    end
   
    # Configures new ssh key for server and creates a copy for the rest of the servers.
    controlserver.vm.provision "shell", inline: $script_ssh_credentials_creation 

    # Install Ansible in control server  
    #controlserver.vm.provision "shell", inline: $script_install_ansible

    # Provision hosts file
    controlserver.vm.provision "shell", inline: $script_host_file
    
    # Copy ansible playbook and roles. 
    #controlserver.vm.provision "file", source: "./ansible/" , destination: "$HOME/ansible/"
  end

  config.vm.define "master-service-1" do |masterserver1|
   masterserver1.vm.box = $centos_box
   masterserver1.vm.network "private_network", ip: "192.168.135.111"
   masterserver1.vm.hostname = "k8s-master"
   config.vm.provider :virtualbox do |vb|
      vb.customize ["modifyvm", :id, "--memory", "2048"]
      vb.customize ["modifyvm", :id, "--cpus", "2"]
    end
   masterserver1.vm.provision 'shell', inline: $script_copy_key
   # Provision hosts file
   masterserver1.vm.provision "shell", inline: $script_host_file
  end		      

  config.vm.define "node-service-1" do |nodeseserver1|
    nodeseserver1.vm.box = $centos_box
    nodeseserver1.vm.network "private_network", ip: "192.168.135.121"
    nodeseserver1.vm.hostname = "worker-node-1"
    config.vm.provider :virtualbox do |vb|
      vb.customize ["modifyvm", :id, "--memory", "2048"]
      vb.customize ["modifyvm", :id, "--cpus", "1"]
    end
    nodeseserver1.vm.provision 'shell', inline: $script_copy_key
    # Provision hosts file
    nodeseserver1.vm.provision "shell", inline: $script_host_file
  end

  config.vm.define "node-service--2" do |nodeserver2|
    nodeserver2.vm.box = $centos_box
    nodeserver2.vm.network "private_network", ip: "192.168.135.122"
    nodeserver2.vm.hostname = "worker-node-2"
    config.vm.provider :virtualbox do |vb|
      vb.customize ["modifyvm", :id, "--memory", "2048"]
      vb.customize ["modifyvm", :id, "--cpus", "1"]
    end
    nodeserver2.vm.provision 'shell', inline: $script_copy_key
    # Provision hosts file
    nodeserver2.vm.provision "shell", inline: $script_host_file
  end

end
