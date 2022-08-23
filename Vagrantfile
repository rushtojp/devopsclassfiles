# -*- mode: ruby -*-
# vi: set ft=ruby :

ENV['VAGRANT_DEFAULT_PROVIDER'] = 'virtualbox'

Vagrant.configure("2") do |config|
  ##### DEFINE VM #####
  config.vm.define "dockervm-sog-ubuntu-aug23" do |config|
  config.vm.hostname = "dockervm-sog-ubuntu-aug23"
  config.vm.box = "bento/ubuntu-18.04"
  config.vm.box_check_update = false
  config.vm.network "private_network", ip: "192.168.56.59"
  
  end
end
