# -*- mode: ruby -*-
# vi: set ft=ruby :

# Порядок установки:
# 1. Скачиваем и устанавливаем virtualbox: https://www.virtualbox.org/wiki/Downloads
# 2. Можно установить Oracle VM VirtualBox Extension Pack - плагин для virtualbox
# 2.1. Скачиваем: https://download.virtualbox.org/virtualbox/6.0.2/Oracle_VM_VirtualBox_Extension_Pack-6.0.2.vbox-extpack
# 2.2. Открываем virtualbox, нажимаем "Настройки", затем "Плагины", а там небольшую кнопку "+"
# 2.3. Выбираем только что скачанный файл Oracle_VM_VirtualBox_Extension_Pack-6.0.2.vbox-extpack и подтверждаем установку плагина
# 3. Скачиваем и устанавливаем vagrant: https://www.vagrantup.com/downloads.html
# 4. Создаем папку для нашего проекта. Например "netology_db"
# 5. Копируем в нее этот файл "Vagrantfile"
# 6. В консоли, переходим в эту папку (все команды vagrant запускаем в этой папке)
# 6.1. Вводим команду "vagrant up" и ждем завершения загрузки и установки.
# 6.2. Вводим команду "vagrant ssh" и оказываемся в консоли linux
# 7. команды подключения к базам данных:
# 7.1. psql - стандартная консоль PostgreSQL (уже создана БД и суперпользователь vagrant - никаких параметров вводить не нужно)
# 7.2. pgcli - еще одна консоль для PostgreSQL (красивая и удобная - с автодополнением)
# 7.3. mongo - консоль mongodb
# 8. После окончания работы, выполните команду "vagrant halt" для остановки виртуальной машины
# 8.1. Чтобы после этого снова продолжить работу, нужно будет выполнить команды "vagrant up" (запуск vm) и "vagrant ssh" (подключение)
# 8.2. Если хотите полностью переустановить VM (с чистой БД), выполните команду "vagrant destroy" и перейдите к пункту 6.1.

Vagrant.configure("2") do |config|
  # The most common configuration options are documented and commented below.
  # For a complete reference, please see the online documentation at
  # https://docs.vagrantup.com.

  # Every Vagrant development environment requires a box. You can search for
  # boxes at https://vagrantcloud.com/search.
  config.vm.box = "ubuntu/bionic64"
  # config.vm.box_version = "20190119.0.0"
  # config.vm.box_url = "https://app.vagrantup.com/ubuntu/boxes/bionic64/versions/20190119.0.0/providers/virtualbox.box"
  # config.vm.box_url = "file://bionic-server-cloudimg-amd64-vagrant.box"

  # Disable automatic box update checking. If you disable this, then
  # boxes will only be checked for updates when the user runs
  # `vagrant box outdated`. This is not recommended.
  # config.vm.box_check_update = false

  # Create a forwarded port mapping which allows access to a specific port
  # within the machine from a port on the host machine. In the example below,
  # accessing "localhost:8080" will access port 80 on the guest machine.
  # NOTE: This will enable public access to the opened port
  # config.vm.network "forwarded_port", guest: 80, host: 8080

  # Create a forwarded port mapping which allows access to a specific port
  # within the machine from a port on the host machine and only allow access
  # via 127.0.0.1 to disable public access
  config.vm.network "forwarded_port", guest: 5432, host: 5432, host_ip: "127.0.0.1"
  # config.vm.network "forwarded_port", guest: 27017, host: 27017, host_ip: "127.0.0.1"

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
  config.vm.synced_folder ".", "/vagrant", :mount_options => ["dmode=777","fmode=666"]

  # Provider-specific configuration so you can fine-tune various
  # backing providers for Vagrant. These expose provider-specific options.
  # Example for VirtualBox:
  #
  config.vm.provider "virtualbox" do |vb|
    # Display the VirtualBox GUI when booting the machine
    # vb.gui = true
  
    # Customize the amount of memory on the VM:
    vb.memory = "2048"
  end
  #
  # View the documentation for the provider you are using for more
  # information on available options.

  # Enable provisioning with a shell script. Additional provisioners such as
  # Puppet, Chef, Ansible, Salt, and Docker are also available. Please see the
  # documentation for more information about their specific syntax and use.
  config.vm.provision "shell", inline: <<-SHELL
    LANGUAGE=en_US
    ENCODING=UTF-8
    PROJECT=vagrant
    NETOLOGY_DATA="/usr/local/share/netology"
    apt-get -y update
    apt-get -y upgrade
    apt-get install -y locales
    localedef -i ${LANGUAGE} -c -f ${ENCODING} -A /usr/share/locale/locale.alias ${LANGUAGE}.${ENCODING}
    apt-get -y install postgresql mongodb mc
    pg_dropcluster --stop 10 main
    pg_createcluster --locale ${LANGUAGE}.${ENCODING} --start 10 main
    apt-get -y install postgresql-client libpq-dev python3.6 python3-pip unzip git virtualenv gcc python3.6-dev uuid-dev libcap-dev libpcre3-dev pgcli
    pip3 install requests tqdm

    tee -a /home/vagrant/.bashrc <<EOF
 
export APP_MONGO_HOST=localhost
export APP_MONGO_PORT=27017
export APP_POSTGRES_HOST=localhost
export APP_POSTGRES_PORT=5432
export APP_REDIS_HOST=localhost
export APP_REDIS_PORT=6379
export NETOLOGY_DATA="${NETOLOGY_DATA}"
EOF

    chown vagrant:vagrant /home/vagrant/.bashrc

    mkdir ${NETOLOGY_DATA}
    mkdir ${NETOLOGY_DATA}/raw_data
    mkdir ${NETOLOGY_DATA}/pg_data
    mkdir ${NETOLOGY_DATA}/data

    chmod -R 777 ${NETOLOGY_DATA}

    mkdir -p /tmp/deploy
    chmod 777 /tmp/deploy
    tee /tmp/deploy/database.sql << EOF
CREATE DATABASE ${PROJECT} ENCODING '${ENCODING}' TEMPLATE template0;
CREATE ROLE ${PROJECT} WITH LOGIN PASSWORD '${PROJECT}';
GRANT ALL PRIVILEGES ON DATABASE ${PROJECT} TO ${PROJECT};
ALTER USER ${PROJECT} WITH SUPERUSER;
ALTER USER ${PROJECT} WITH CREATEROLE;
ALTER USER ${PROJECT} WITH CREATEDB;
ALTER USER ${PROJECT} WITH REPLICATION;
EOF
    chmod 755 /tmp/deploy/database.sql
    su -c "psql -f /tmp/deploy/database.sql" postgres
  SHELL
end

