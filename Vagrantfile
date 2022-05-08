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
  # config.vm.box_check_update = false

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
  
  config.vm.provision "shell", inline: <<-SHELL
    apt update
    mkdir /work
    cd /work
    wget "https://www.redmine.org/releases/redmine-4.2.5.tar.gz"
    tar xzvf redmine-4.2.5.tar.gz -C /opt
    wget "https://downloads.mysql.com/archives/get/p/23/file/mysql-5.5.62-linux-glibc2.12-x86_64.tar.gz"
  SHELL
  config.vm.provision "file", source: "./config/database.yml", destination: "/opt/redmine-4.2.5/config/database.yml"
  config.vm.provision "file", source: "./config/initializers/mysqlpls.rb", destination: "/opt/redmine-4.2.5/config/initializers/mysqlpls.rb"
  config.vm.provision "file", source: "./createdb.sql", destination: "/tmp/createdb.sql"
  config.vm.provision "file", source: "./redmine.conf", destination: "/tmp/redmine.conf"
  config.vm.provision "shell", inline: "mv /tmp/createdb.sql /work/createdb.sql"
  config.vm.provision "shell", inline: "mv /tmp/redmine.conf /work/redmine.conf"
  config.vm.provision "shell", inline: <<-SHELL
    groupadd mysql
    useradd -g mysql mysql
    cd /work
    tar xzvf mysql-5.5.62-linux-glibc2.12-x86_64.tar.gz
    mv mysql-5.5.62-linux-glibc2.12-x86_64  /usr/local/mysql
    cd /usr/local/mysql
    chown -R mysql:mysql *
    apt install -y libaio1 libncurses5
    scripts/mysql_install_db --user=mysql
    chown -R root .
    chown -R mysql data
    cp support-files/my-medium.cnf /etc/my.cnf
    bin/mysqld_safe --user=mysql & cp support-files/mysql.server /etc/init.d/mysql.server
    ln -s /usr/local/mysql/bin/mysql /usr/local/bin/mysql
    /etc/init.d/mysql.server start
    update-rc.d -f mysql.server defaults
    bin/mysqladmin -u root password redmine
    mysql -u root -predmine < /work/createdb.sql
  SHELL
  config.vm.provision "shell", inline: <<-SHELL
    apt install software-properties-common
    apt-add-repository -y ppa:rael-gc/rvm
    apt update
    apt install -y rvm libssl-dev libreadline-dev zlib1g-dev gcc g++ make apache2
    echo "source /etc/profile.d/rvm.sh" | tee -a /etc/profile
    source /etc/profile.d/rvm.sh
    rvm install 2.7.2
    rvm use 2.7.2 --default
    gem install bundler
    gem install passenger
    cd /opt/redmine-4.2.5
    bundle install --without development test
    bundle exec rake generate_secret_token
    RAILS_ENV=production bundle exec rake db:migrate
    RAILS_ENV=production REDMINE_LANG=ja bundle exec rake redmine:load_default_data
    groupadd redmine
    useradd -g redmine redmine
    mkdir -p tmp tmp/pdf public/plugin_assets tmp/cache
    chown -R redmine:redmine files log tmp public/plugin_assets
    chown redmine:redmine config/environment.rb 
    chmod -R 755 files log tmp public/plugin_assets
    chmod -R 777 tmp/cache
    ln -s /opt/redmine-4.2.5/public /var/www/html/redmine
    chown -R redmine:redmine /var/www/html/redmine
    # bundle exec rails server webrick -e production
  SHELL

  config.vm.provision "shell", inline: <<-SHELL
    apt install -y libcurl4-openssl-dev apache2-dev libapr1-dev libaprutil1-dev
    dd if=/dev/zero of=/swap bs=1M count=1024
    mkswap /swap
    swapon /swap
    passenger-install-apache2-module --auto --languages ruby
    cp /work/redmine.conf /etc/apache2/conf-available/redmine.conf
    passenger-install-apache2-module --snippet >> /etc/apache2/conf-available/redmine.conf 
    a2enconf redmine.conf
    apache2ctl restart
    systemctl reload apache2
  SHELL
end
