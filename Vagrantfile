# -*- mode: ruby -*-
# vi: set ft=ruby :

VAGRANTFILE_API_VERSION = "2"

application_slug = "bamboo"
application_ram  = 4192
application_cpus = 2

Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|
    config.vm.hostname = "local.#{application_slug}.net"
    config.vm.box = "bento/ubuntu-16.04"

    # Application provisioning
    ansible = lambda do |override|
      # Install githooks
      override.vm.provision :shell, inline: "cd /vagrant && test -d .git && (test -d .git/hooks || mkdir .git/hooks) && for h in $(ls tools/githooks/); do test -f .git/hooks/`basename \"$h\"` || ( cp \"$h\" .git/hooks/`basename \"$h\"` && chmod 0755 .git/hooks/`basename \"$h\"` ); done"

      # Bootstrap Ansible
      override.vm.provision :shell, inline: "test ! -x /usr/bin/apt-get || test -x /usr/bin/ansible || ( apt-get update && apt-get install -y software-properties-common && apt-add-repository ppa:ansible/ansible && apt-get update && apt-get install -y ansible )"
      override.vm.provision :shell, privileged: false, keep_color: true, inline: "PYTHONUNBUFFERED=1 ANSIBLE_FORCE_COLOR=true ansible-playbook -i /vagrant/ansible/inventories/vagrant/hosts /vagrant/ansible/site.yml"
    end

    # Parallels
    config.vm.provider :parallels do |p, override|
        p.name = application_slug
        p.memory = application_ram
        p.cpus = application_cpus
        p.update_guest_tools = true

        ansible.call override
    end

    # VirtualBox
    config.vm.provider :virtualbox do |v, override|
        v.name = application_slug
        v.memory = application_ram
        v.cpus = application_cpus

        ansible.call override
    end

    # Networking
    begin
        config.vm.network :private_network, ip: Socket::getaddrinfo(config.vm.hostname, 'http', nil, Socket::SOCK_STREAM)[0][3]
    rescue SocketError
        $stderr.puts "Could not resolve " + config.vm.hostname + " (see README.md) - one will be assigned automatically"
    end

    # File sharing
    if RUBY_PLATFORM =~ /cygwin|mswin|mingw|bccwin|wince|emx/
        config.vm.synced_folder   "./cache/apt/archives", "/var/cache/apt/archives"
    else
        config.vm.synced_folder   "./cache/apt/archives", "/var/cache/apt/archives"
        config.vm.provider :virtualbox do |v, override|
            override.vm.synced_folder   "./cache/apt/archives", "/var/cache/apt/archives", type: "nfs"
            override.vm.synced_folder   ".", "/vagrant", type: "nfs"
        end
    end

    config.vm.post_up_message = "Server '" + application_slug + "' is running - open http://#{config.vm.hostname}"
end
