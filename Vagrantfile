# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.configure('2') do |config|
  config.vm.box = "ubuntu/trusty64"
  config.vm.network "private_network", type: "dhcp"
  config.vm.synced_folder ".", "/vagrant", :mount_options => ["fmode=666"]
  config.ssh.insert_key = false

  # use vagrant-hostmanager plugin
  #   vagrant plugin install vagrant-hostmanager
  config.hostmanager.enabled = true
  config.hostmanager.ip_resolver = proc do |vm, resolving_vm|
    if vm.id
      `VBoxManage guestproperty get #{vm.id} "/VirtualBox/GuestInfo/Net/1/V4/IP"`.split()[1]
    end
  end

  config.vm.define "seafile-vm" do |seafile|
    seafile.vm.hostname = "seafile-vm"

    # forwarded ports
    seafile.vm.network "forwarded_port", guest: 80, host: 8080 # http
    seafile.vm.network "forwarded_port", guest: 443, host: 4443 # https
    seafile.vm.network "forwarded_port", guest: 10001, host: 10001 # Ccnet Daemon
    seafile.vm.network "forwarded_port", guest: 12001, host: 12001 # Seafile Daemon

    # disable shared folder
    seafile.vm.synced_folder ".", "/vagrant", disabled: true

    # make machine faster
    seafile.vm.provider "virtualbox" do |v|
      v.memory = 1024
      v.cpus = 2
    end
  end

  config.vm.define "ansible-vm", primary: true do |ansible|
    ansible.vm.hostname = "ansible-vm"
    ansible.ssh.forward_agent = true
  end
end
