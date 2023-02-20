# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.configure("2") do |config|
  config.vm.box = "generic/ubuntu2010"

  # Create the first VM
  config.vm.define "vm1" do |vm1|
    vm1.vm.hostname = "vm1"
    vm1.vm.network "private_network", ip: "192.168.33.10"
  end

  # Create the second VM
  config.vm.define "vm2" do |vm2|
    vm2.vm.hostname = "vm2"
    vm2.vm.network "private_network", ip: "192.168.33.11"
  end

  # Create the third VM
  config.vm.define "vm3" do |vm3|
    vm3.vm.hostname = "vm3"
    vm3.vm.network "private_network", ip: "192.168.33.12"
  end
end
