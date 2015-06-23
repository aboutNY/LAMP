# -*- mode: ruby -*-
# vi: set ft=ruby :

$version = "65"

Vagrant.configure(2) do |config|
  config.vm.box = "CentOS%s" % $version
  config.vm.box_url = "https://github.com/2creatives/vagrant-centos/releases/download/v6.5.3/centos65-x86_64-20140116.box"

  #config.vm.synced_folder = 'C:/hoge/data/source/web', '/var/www/html/', type: "nfs"
  #config.vm.synced_folder "../../Dropbox/Backup/sandbox", "/vagrant"

  config.vm.network "private_network", ip: "192.168.33.30"

  config.vm.provider "virtualbox" do |vb|
     vb.memory = 1024;
     vb.cpus = 2;
     vb.gui = false;
   end

   config.vm.provision "ansible" do |ansible|
     ansible.playbook = "playbook.yml"
     ansible.limit = "all"
   end
end
