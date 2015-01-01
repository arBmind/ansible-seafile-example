# -*- mode: ruby -*-
# vi: set ft=ruby :

provision_target = ENV['PROVISION_TARGET'] || 'vagrant' # target hosts configuration used (shell provisioning)
provision_groups = ENV['PROVISION_GROUPS'] || 'vagrant' # target group configuration used (ansible provisioning)
provision_action = ENV['PROVISION_ACTION'] || 'provision' # Ansible playbook used
vm_ansible = ENV['VM_ANSIBLE'] || false # install Ansible into the VM (required for Windows)

Vagrant.configure('2') do |config|
  config.vm.box = "hashicorp/precise64"

  config.vm.network "forwarded_port", guest: 80, host: 8081 # webserver
  config.vm.network "private_network", type: "dhcp"
  config.ssh.forward_agent = true

  config.vm.provider "virtualbox" do |v|
    v.memory = 1024
    v.cpus = 2
  end

  # config.vm.synced_folder "../ansible-seafile", "/opt/ansible/roles/dresden-weekly.seafile"

  if vm_ansible
    config.vm.provision :shell, keep_color: true, inline: <<-SHELL_END
      export REPO_USER=#{ENV['REPO_USER']}
      export REPO_PASSWORD=#{ENV['REPO_PASSWORD']}
      export SECRET_KEY_BASE=#{ENV['SECRET_KEY_BASE']}
      export PROVISION_ARGS=#{ENV['PROVISION_ARGS']}
      export ANSIBLE_HOST_KEY_CHECKING=False
      export PYTHONUNBUFFERED=1
      export ANSIBLE_FORCE_COLOR=1
      export BASE_FOLDER=/vagrant/ansible
      export ANSIBLE_DIR=/opt/ansible
      export RUN_ANSIBLE=true
      exec /vagrant/ansible/run.sh #{provision_target} #{provision_action}
    SHELL_END
  else
    base_folder = ENV['BASE_FOLDER'] || "#{Dir.pwd}/ansible"
    ansible_dir = ENV['ANSIBLE_DIR'] || "#{Dir.pwd}/.ansible"
    ansible_roles_path = ENV['ANSIBLE_ROLES_PATH'] || "#{ansible_dir}/roles"

    ansible_groups = "#{base_folder}/groups/#{provision_groups}.yml"
    ansible_playbook = "#{base_folder}/#{provision_action}.yml"
    raise "missing groups #{ansible_groups}" unless File.file?(ansible_groups)
    raise "missing playbook #{ansible_playbook}" unless File.file?(ansible_playbook)

    require 'yaml'
    ansible_groups = YAML.load(File.read(ansible_groups))

    %x[ #{base_folder}\run.sh ] # install proper Ansible
    File.write('ansible.cfg', "[defaults]\nroles_path=#{ansible_roles_path}")
    config.vm.provision "ansible" do |ansible|
      ansible.sudo = true
      ansible.playbook = ansible_playbook
      ansible.groups = ansible_groups
    end
    # no cleanup possible
  end
end
