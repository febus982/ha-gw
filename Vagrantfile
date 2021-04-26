# -*- mode: ruby -*-
# vi: set ft=ruby :

# unless Vagrant.has_plugin?("vagrant-vbguest")
#   puts 'Installing vagrant-vbguest Plugin...'
#   system('vagrant plugin install vagrant-vbguest')
# end
#
# unless Vagrant.has_plugin?("vagrant-reload")
#   puts 'Installing vagrant-reload Plugin...'
#   system('vagrant plugin install vagrant-reload')
# end

# Vagrantfile API/syntax version.
# NB: Don't touch unless you know what you're doing!
VAGRANTFILE_API_VERSION = "2"

LBS = {
  "lb1": {
    "external_cidr": "192.168.50.151/24",
    "internal_cidr": "192.168.100.2/24",
  },
  "lb2": {
    "external_cidr": "192.168.50.152/24",
    "internal_cidr": "192.168.100.3/24",
  },
}

MASTERS = {
  "master1": {
    "internal_cidr": "192.168.100.11/24",
  },
  "master2": {
    "internal_cidr": "192.168.100.12/24",
  },
}

NODES = {
  "node1": {
    "internal_cidr": "192.168.100.21/24",
  },
}

# Do local: ansible-galaxy install -r ansible/requirements.yml

Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|
  config.vm.box = "debian/buster64"
  config.vbguest.auto_update = false

  LBS.map do |name, cidrs|
    config.vm.define name do |m|
      m.vm.hostname = name
      m.vm.network "public_network",
       bridge: "en0: Wi-Fi (Wireless)",
       auto_config: false
#        use_dhcp_assigned_default_route: true
#        ip: cidrs[:external]
      m.vm.network "private_network",
       auto_config: false,
#        ip: cidrs[:internal],
       virtualbox__intnet: true

      # delete default gw
      m.vm.provision "shell",
        run: "always",
        inline: "ip route del default dev eth0 || true"

      # default router
#       m.vm.provision "shell",
#         run: "always",
#         inline: "ip route add default via 192.168.50.1"

      m.vm.provision "ansible" do |ansible|
        ansible.playbook = "ansible/gateway.yml"
        ansible.extra_vars = {
          external_cidr: cidrs[:external_cidr],
          internal_cidr: cidrs[:internal_cidr],
          consul_join_ips: LBS.map{|k,v| "#{v[:internal_cidr][0..(v[:internal_cidr].index("/")-1)]}"},
          ansible_python_interpreter:"/usr/bin/python3",
        }
      end
    end
  end

  MASTERS.map do |name, cidrs|
    config.vm.define name do |m|
      m.vm.hostname = name
      m.vm.network "private_network", ip: cidrs[:internal_cidr],
        virtualbox__intnet: true

      # delete default gw on eth0
      m.vm.provision "shell",
        run: "always",
        inline: "ip route del default dev eth0 || true"

      # default router
      m.vm.provision "shell",
        run: "always",
        inline: "ip route add default via 192.168.100.1 || true"

      m.vm.provision "ansible" do |ansible|
        ansible.playbook = "ansible/master.yml"
        ansible.extra_vars = {
#           external_cidr: cidrs[:internal_cidr],
          internal_cidr: cidrs[:internal_cidr],
          consul_join_ips: LBS.map{|k,v| "#{v[:internal_cidr][0..(v[:internal_cidr].index("/")-1)]}"},
          ansible_python_interpreter:"/usr/bin/python3",
        }
      end
    end
  end


end
