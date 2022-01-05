# -*- mode: ruby -*-
# vim: set ft=ruby :
home = ENV['HOME']
ENV["LC_ALL"] = "en_US.UTF-8"


Vagrant.configure(2) do |config|
  # ПЕРВАЯ ВИРТУАЛЬНАЯ МАШИНА
    config.vm.define "server-selinux" do |server|
    # имя виртуальной машины
    server.vm.box = 'vedoff/centos-7-5' 
    server.vm.provider "virtualbox" do |vb|
      vb.name = "server-selinux"
    end
    # hostname виртуальной машины
    server.vm.hostname = "server-selinux"
    # настройки сети
    server.vm.network "private_network", ip: "192.168.56.216"
    server.vm.synced_folder ".", "/vagrant",  
          type: "rsync",
          rsync_auto: "true",
          rsync_exclude: [".git/",".vagrant/",".gitignore","Vagrantfile"]
          #server.vm.provision "shell", path: "provision/prepare-host.sh"
  # ssh-pub add in server
  config.vm.provision "shell", inline: <<-SHELL
  cat /vagrant/ssh/vagrant-key.pub >> /home/vagrant/.ssh/authorized_keys
  SHELL
    end

  # ВТОРАЯ ВИРТУАЛЬНАЯ МАШИНА
#    config.vm.define "server-02-debian" do |client|
    # имя виртуальной машины
#    client.vm.box = 'debian/buster64' 
#    client.vm.provider "virtualbox" do |vbc|
#      vbc.name = "server-02-debian"
#    end
    # hostname виртуальной машины
#    client.vm.hostname = "server-02-debian"
    # настройки сети
#    client.vm.network "private_network", ip: "192.168.56.214"
#    client.vm.synced_folder ".", "/vagrant",  
#          type: "rsync",
#          rsync_auto: "true",
#          rsync_exclude: [".git/",".vagrant/",".gitignore","Vagrantfile"]
          #client.vm.provision "shell", path: "provision/prepare-host.sh"
# ssh-pub add in server
#  config.vm.provision "shell", inline: <<-SHELL
#  cat /vagrant/provision/vagrant-key.pub >> /home/vagrant/.ssh/authorized_keys
#  SHELL

#    end
end
