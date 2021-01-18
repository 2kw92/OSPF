# -*- mode: ruby -*-
# vim: set ft=ruby :

MACHINES = {
:r1 => {
        :box_name => "centos/7",
        :net => [
                   {ip: '172.16.1.1', adapter: 2, netmask: "255.255.255.248", virtualbox__intnet: "r1"},
                   {ip: '192.168.0.1', adapter: 3, netmask: "255.255.255.248", virtualbox__intnet: "r1r2"},
                   {ip: '192.168.2.1', adapter: 4, netmask: "255.255.255.248", virtualbox__intnet: "r1r3"},
                ]
  },

:r2 => {
        :box_name => "centos/7",
        :net => [
                   {ip: '172.16.2.1', adapter: 2, netmask: "255.255.255.248", virtualbox__intnet: "r2"},
                   {ip: '192.168.0.2', adapter: 3, netmask: "255.255.255.248", virtualbox__intnet: "r1r2"},
                   {ip: '192.168.1.2', adapter: 4, netmask: "255.255.255.248", virtualbox__intnet: "r2r3"},
                ]
  },

:r3 => {
        :box_name => "centos/7",
        :net => [
                   {ip: '172.16.3.1', adapter: 2, netmask: "255.255.255.248", virtualbox__intnet: "r3"},
                   {ip: '192.168.2.2', adapter: 3, netmask: "255.255.255.248", virtualbox__intnet: "r1r3"},
                   {ip: '192.168.1.1', adapter: 4, netmask: "255.255.255.248", virtualbox__intnet: "r2r3"},
                ]
  }
  
}

Vagrant.configure("2") do |config|

  MACHINES.each do |boxname, boxconfig|

    config.vm.define boxname do |box|

        box.vm.box = boxconfig[:box_name]
        box.vm.host_name = boxname.to_s

        boxconfig[:net].each do |ipconf|
          box.vm.network "private_network", ipconf
        end
        
        if boxconfig.key?(:public)
          box.vm.network "public_network", boxconfig[:public]
        end

        box.vm.provision "shell", inline: <<-SHELL
          mkdir -p ~root/.ssh
                cp ~vagrant/.ssh/auth* ~root/.ssh
        SHELL
        
        case boxname.to_s
        when "r1"
          box.vm.provision "shell", path: "r1.sh"
        when "r2"
          box.vm.provision "shell", path: "r2.sh"
        when "r3"
          box.vm.provision "shell", path: "r3.sh"
        end

      end

  end
  
  
end