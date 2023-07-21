Занятие 15 - Автоматизация администрирования. Ansible-1
Цели занятия -
автоматизировать рутинные задачи администрирования;
изучить ansible - инвентори, модули, плейбуки, роли, переменные; объяснить разницу с другими инструментами - chef/puppet/salt.
Домашнее задание - 
Подготовить стенд на Vagrant как минимум с одним сервером. На этом сервере используя Ansible необходимо развернуть nginx со следующими условиями:
необходимо использовать модуль yum/apt;
конфигурационные файлы должны быть взяты из шаблона jinja2 с перемененными;
после установки nginx должен быть в режиме enabled в systemd;
должен быть использован notify для старта nginx после установки;
сайт должен слушать на нестандартном порту - 8080, для этого использовать переменные в Ansible.
Решение -
1. Получаем файл Vagrantfile
2. Создаем файл nginx.conf.j2:
worker_processes 1;

events { worker_connections 1024; }

http {
    server {
        listen {{ nginx_port }};

        location / {
            root /usr/share/nginx/html;
            index index.html;
        }

        error_page 500 502 503 504 /50x.html;
        location = /50x.html {
            root /usr/share/nginx/html;
        }
    }
}
3. Создаем файл playbook.yml:
---
- hosts: all
  become: true

  tasks:
  - name: Install nginx
    apt:
      name: nginx
      state: latest

  - name: Copy nginx config
    template:
      src: nginx.conf.j2
      dest: /etc/nginx/nginx.conf
    notify:
      - Start nginx

  handlers:
  - name: Start nginx
    systemd:
      name: nginx
      state: started
      enabled: true
4. Исправляем файл  Vagrantfile:
# -*- mode: ruby -*-
# vi: set ft=ruby :

# All Vagrant configuration is done below. The "2" in Vagrant.configure
# configures the configuration version (we support older styles for
# backwards compatibility). Please don't change it unless you know what
# you're doing.
Vagrant.configure("2") do |config|
  # The most common configuration options are documented and commented below.
  # For a complete reference, please see the online documentation at
  # https://docs.vagrantup.com.

  # Every Vagrant development environment requires a box. You can search for
  # boxes at https://vagrantcloud.com/search.
  config.vm.box = "ubuntu/focal64"

  # Disable automatic box update checking. If you disable this, then
  # boxes will only be checked for updates when the user runs
  # `vagrant box outdated`. This is not recommended.
  config.vm.box_check_update = false

  # Create a forwarded port mapping which allows access to a specific port
  # within the machine from a port on the host machine. In the example below,
  # accessing "localhost:8080" will access port 80 on the guest machine.
  # NOTE: This will enable public access to the opened port
  config.vm.network "forwarded_port", guest: 80, host: 8080

  # Create a forwarded port mapping which allows access to a specific port
  # within the machine from a port on the host machine and only allow access
  # via 127.0.0.1 to disable public access
  # config.vm.network "forwarded_port", guest: 80, host: 8080, host_ip: "127.0.0.1"

  # Create a private network, which allows host-only access to the machine
  # using a specific IP.
  # config.vm.network "private_network", ip: "192.168.33.10"

  # Create a public network, which generally matched to bridged network.
  # Bridged networks make the machine appear as another physical device on
  # your network.
  # config.vm.network "public_network"

  # Share an additional folder to the guest VM. The first argument is
  # the path on the host to the actual folder. The second argument is
  # the path on the guest to mount the folder. And the optional third
  # argument is a set of non-required options.
  # config.vm.synced_folder "../data", "/vagrant_data"

  # Provider-specific configuration so you can fine-tune various
  # backing providers for Vagrant. These expose provider-specific options.
  # Example for VirtualBox:
  #
  # config.vm.provider "virtualbox" do |vb|
  #   # Display the VirtualBox GUI when booting the machine
  #   vb.gui = true
  #
  #   # Customize the amount of memory on the VM:
  #   vb.memory = "1024"
  # end
  #
  # View the documentation for the provider you are using for more
  # information on available options.

  # Enable provisioning with a shell script. Additional provisioners such as
  # Ansible, Chef, Docker, Puppet and Salt are also available. Please see the
  # documentation for more information about their specific syntax and use.
  
  config.vm.provision "ansible" do |ansible|
    ansible.playbook = "playbook.yml"
    ansible.extra_vars = { nginx_port: 8080 }
  end

   config.vm.provision "shell", inline: <<-SHELL
#     sudo apt install epel-release -y
#     sudo apt install ansible -y
#     sudo apt install nginx -y
#     sudo systemctl enable nginx
#     sudo systemctl start nginx
#     sudo nginx -t
     sudo systemctl status nginx
   SHELL
end
5. Запускаем - script --timing=time_homework_log homework.log
6. Получаем результат в homework.log
7. Все! Задание понравилось, пришлось перерыть кучу Интернета и посмотреть еще раз занятие.
8. Опишу как я до этого докатился -
Задание начал делать еще в июне. Помучавшись с неделю, бросил его и начал делать занятие по докеру.
И тут у меня возникает такая-же проблема - отказ доступа по порту 8080. Я понял, что если не вернусь
к выполнению занятия 15 не сделаю и это. Ошибка оказалась в отсутствии проброса порта в Vagrantfile.
Как только это сделал все прошло на Ура! Спасибо!
