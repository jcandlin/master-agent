# -*- mode: ruby -*-
# vi: set ft=ruby :
 
# This configuration requires Vagrant 1.5 or newer and two plugins:
#
#   vagrant plugin install vagrant-hosts        ~> 2.1.4
#   vagrant plugin install vagrant-auto_network ~> 1.0.0
#
# After installation, the following steps will spin up a master and agent that
# can communicate with each other:
#
#     vagrant up
#     vagrant ssh puppetagent
#     sudo -i
#     puppet agent -t
Vagrant.require_version ">= 1.5.0"
require 'vagrant-hosts'
require 'vagrant-auto_network'
 
Vagrant.configure('2') do |config|
 
  config.vm.define :puppetmaster do |node|
    # An index of pre-built boxes can be found at:
    #
    #   https://vagrantcloud.com/puppetlabs
    node.vm.box = 'precise64'
 
    node.vm.hostname = 'puppetmaster.puppetdebug.vlan'
 
    # Use vagrant-auto_network to assign an IP address.
    node.vm.network :private_network, :auto_network => true
 
    # Use vagrant-hosts to add entries to /etc/hosts for each virtual machine
    # in this file.
    node.vm.provision :hosts
 
    # Bootstrap the latest version of Puppet on CentOS 6 and Set up a Puppet Master
    # that automatically signs certs from agents.
    #
    # For operating systems other than CentOS 6, a collection of bootstrap
    # scripts can be found here:
    #
    #   https://github.com/hashicorp/puppet-bootstrap
    #
    # The Puppet Labs installation docs also have some useful pointers:
    #
    #   http://docs.puppetlabs.com/guides/installation.html#installing-puppet-1
    bootstrap_script = <<-EOF
    if which puppet > /dev/null 2>&1; then
      echo 'Puppet Installed.'
    else
      echo 'Installing Puppet Master.'
      rpm -ivh http://yum.puppetlabs.com/el/6/products/x86_64/puppetlabs-release-6-10.noarch.rpm
      yum --nogpgcheck -y install puppet-server
      echo '*.puppetdebug.vlan' > /etc/puppet/autosign.conf
      /usr/bin/puppet resource service iptables ensure=stopped enable=false
      /usr/bin/puppet resource service puppetmaster ensure=running enable=true
    fi
    EOF
    node.vm.provision :shell, :inline => bootstrap_script
  end
 
  config.vm.define :puppetagent do |node|
    node.vm.box = 'precise64'
 
    node.vm.hostname = 'puppetagent.puppetdebug.vlan'
 
    node.vm.network :private_network, :auto_network => true
    node.vm.provision :hosts
 
    # Set up Puppet Agent to automatically connect with Puppet master
    bootstrap_script = <<-EOF
    if which puppet > /dev/null 2>&1; then
      echo 'Puppet Installed.'
    else
      echo 'Installing Puppet Agent.'
      rpm -ivh http://yum.puppetlabs.com/el/6/products/x86_64/puppetlabs-release-6-10.noarch.rpm
      yum --nogpgcheck -y install puppet
      puppet config set --section main server puppetmaster.puppetdebug.vlan
    fi
    EOF
    node.vm.provision :shell, :inline => bootstrap_script
  end
 
end