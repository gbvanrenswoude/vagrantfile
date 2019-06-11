# -*- mode: ruby -*-
# vi: set ft=ruby :

# install vagrant-diskresize
# install vagrant proxyconf
# for minikube set virtualbox networking mode NAT

Vagrant.configure("2") do |config|
  config.vm.box = "ubuntu/bionic64"    # find more https://app.vagrantup.com/boxes/search
  config.disksize.size = '40GB'  # vagrant plugin install vagrant-disksize
  config.vm.box_check_update = false
  config.vm.network :forwarded_port, guest: 8080, host: 8080 # https://www.vagrantup.com/intro/getting-started/networking.html
  config.vm.network :forwarded_port, guest: 8000, host: 8000
  #  config.ssh.insert_key = false
  config.vm.provider :virtualbox do |vb|
    vb.customize ["modifyvm", :id, "--natdnshostresolver1", "on"] # off
	vb.customize ["modifyvm", :id, "--natdnsproxy1", "on"] # off
    # Enable symlinks in vagrant shared folder, https://coderwall.com/p/b5mu2w
    vb.customize ["setextradata", :id, "VBoxInternal2/SharedFoldersEnableSymlinksCreate/v-root", "1"]
    vb.customize ["setextradata", :id, "VBoxInternal2/SharedFoldersEnableSymlinksCreate/vagrant-root", "1"]
    vb.customize ["setextradata", :id, "VBoxInternal2/SharedFoldersEnableSymlinksCreate/vagrant", "1"]
    vb.memory = 2048
  end

  config.vm.provision "file", source: "~/.ssh/config", destination: "/home/vagrant/.ssh/config"
  config.vm.synced_folder ENV['HOME'] + "/.aws", "/home/vagrant/.aws" # use saml to aws sts keys conversion chrome/firefox plugin and sync key directly to box
  config.ssh.forward_agent = true

  # if Vagrant.has_plugin?("vagrant-proxyconf")
	#   config.proxy.http     = "http://someproxy:9400"
	#   config.proxy.https    = "http://someproxy:9400"
  #   config.proxy.no_proxy = "localhost,127.0.0.1,company.com"
  # end

  # if add pubkey
  if File.exists?(File.join(Dir.home, ".ssh", "id_rsa.pub"))
    ssh_key = File.read(File.join(Dir.home, ".ssh", "id_rsa.pub"))
    config.vm.provision :shell, :inline => "echo 'Copying local SSH key to VM for provisioning...' && echo '#{ssh_key}' >> /home/vagrant/.ssh/authorized_keys"
  end

  config.vm.provision "shell", inline: <<-SHELL
    chmod 600 /home/vagrant/.ssh/config
    sudo apt-get -y install apt-transport-https ca-certificates software-properties-common zsh
    sudo apt-key adv --keyserver hkp://p80.pool.sks-keyservers.net:80 --recv-keys 58118E89F3A912897C070ADBF76221572C52609D
    sudo echo "deb https://apt.dockerproject.org/repo ubuntu-trusty main" > /etc/apt/sources.list.d/docker.list
    sudo apt-add-repository ppa:ansible/ansible
    sudo apt-get -y update
    sudo apt-get -y install docker.io unzip wget jq git #ansible  its better to install ansible using pip for python dependencies
    sudo gpasswd -a vagrant docker
    sudo wget https://releases.hashicorp.com/packer/1.4.1/packer_1.4.1_linux_amd64.zip
    sudo unzip packer_1.4.1_linux_amd64.zip -d /usr/local/bin
    sudo wget https://releases.hashicorp.com/terraform/0.11.0/terraform_0.11.0_linux_amd64.zip
    sudo unzip terraform_0.11.0_linux_amd64.zip -d /usr/local/bin
    wget https://storage.googleapis.com/kubernetes-release/release/v1.12.0/bin/linux/amd64/kubectl
    chmod +x kubectl
    sudo mv kubectl /usr/local/bin/
    wget https://storage.googleapis.com/kubernetes-helm/helm-v2.12.2-linux-amd64.tar.gz
    tar -xvzf helm-v2.12.2-linux-amd64.tar.gz
    sudo mv linux-amd64/helm /usr/bin

#   pip
    apt-get -y install python-pip
    sudo pip install --upgrade virtualenv

#   ansible
    sudo pip install ansible

#   sceptre
    sudo pip install sceptre

#   AWS CLI
    sudo pip install awscli
    touch ~/.bash_profile
    export PATH=~/.local/bin:$PATH
    source ~/.bash_profile

#   Concourse (fly)
    # wget --no-check-certificate "https://concourse.corp.com/api/v1/cli?arch=amd64&platform=linux" -O fly
    # sudo install fly /usr/local/bin

#   aws iam authenticator
#   minikube

  SHELL

end
