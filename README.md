 #仮想マシンにLAMP環境立ち上げ
## vagrantfile 作成

```
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
     ansible.inventory_path = "hosts"
     ansible.limit = "all"
   end
end
```



## 鍵の作成と登録

```
$ ssh-keygen -t rsa
$ cat .ssh/id_rsa.pub | ssh vagrant@192.168.59.34 "cat >> .ssh/authorized_keys"
$ echo "192.168.59.34" > hosts
$ ssh vagrant@192.168.59.34
```
## ansible テスト

```
CA0220:db 01008053$ ansible all -i hosts -m ping -u vagrant

```
```
192.168.33.30 | success >> {
    "changed": false,
    "ping": "pong"
}
```

## playbook.yml作成

以下のように書いていきます

```
- hosts: web
 user: vagrant
 sudo: yes
 vars:
       mysql_root_pw: "3bcdj2caS2"
 tasks:
   - name: Install basepackage
     yum: name={{ item }} state=present
     with_items:
       - wget
       - ntp
       - vim
   - name: SELinux Disable
     command: setenforce 0
     ignore_errors: True
   - name: Edit selinux config
     command: sed -i -e "s/^SELINUX=enforing/SELINUX=disabled/g" /etc/selinux/config
   - name: stop iptabes
     service: name=iptables state=stopped
   - name: Install apache
     yum: name=httpd state=latest
     notify:
       - restart apache
   - name: Install php
     yum: name={{ item }} state=present
     with_items:
       - php
       - php-devel
       - php-mbstring
       - php-mysql
       - php-gd
     notify:
       - restart apache
   - name: Install mysql
     yum: name=mysql-server state=installed
     notify:
       - mysql setup
       - mysql set password
 handlers:
   - name: restart apache
     service: name=httpd state=restarted enabled=yes
   - name: mysql setup
     service: name=mysqld state=started enabled=yes
   - name: mysql set password
     command: mysqladmin -u root password "{{ mysql_root_pw }}"
```


## ansible実行

```
ansible-playbook playbook.yml -i hosts
```
