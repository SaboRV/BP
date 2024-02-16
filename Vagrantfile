# -*- mode: ruby -*-
# vi: set ft=ruby :

VAGRANTFILE_API_VERSION = "2"
Vagrant.configure(VAGRANTFILE_API_VERSION) do |config|


 config.vm.define "bs" do |bs| 
 bs.vm.box = "gusztavvargadr/ubuntu-server-2204-lts"
 bs.vm.box_version = "2204.0.2312"
 bs.vm.network "private_network", ip: "192.168.56.160",  virtualbox__intnet: "net1" 
 bs.vm.hostname = "backup-server" 
 bs.vm.provider :virtualbox do |vb|
      vb.name = "bs"
      vb.memory = 4096
      vb.cpus = 4
    end
              bs.vm.provision "shell", inline: <<-SHELL
            mkdir -p ~root/.ssh; cp ~vagrant/.ssh/auth* ~root/.ssh
            sed -i '65s/PasswordAuthentication no/PasswordAuthentication yes/g' /etc/ssh/sshd_config
            systemctl restart sshd
          SHELL
 end 
 config.vm.define "sc" do |sc| 
 sc.vm.box = "gusztavvargadr/ubuntu-server-2204-lts"
 sc.vm.box_version = "2204.0.2312"
 sc.vm.network "private_network", ip: "192.168.56.150",  virtualbox__intnet: "net1" 
 sc.vm.hostname = "server-client"
 sc.vm.provider :virtualbox do |vb|
      vb.name = "sc"
      vb.memory = 2048 
      vb.cpus = 2 
    end
              sc.vm.provision "shell", inline: <<-SHELL
            mkdir -p ~root/.ssh; cp ~vagrant/.ssh/auth* ~root/.ssh
            sed -i '65s/PasswordAuthentication no/PasswordAuthentication yes/g' /etc/ssh/sshd_config
            systemctl restart sshd
          SHELL
 end 
end 
