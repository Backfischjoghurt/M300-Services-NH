# M300-Services-NH
==========================

 Vagrant.configure("2") do |config|

# Konfiguration VM SRVWEB
  config.vm.define "web" do |web| 
    web.vm.box = "ubuntu/xenial64"
    web.vm.hostname = "SRVWEB"
    web.vom.network "private_network", ip: "192.168.1.5"
    web.vm.network "forwarded_port", guest: 80, host: 8080
    web.vm.provider "virtualbox" do |vb|
    vb.memory = "1024"
    vb.name = "SRVWEB"
    end

    ### Installation und Konfiguration für web
    web.vm.provision "shell", inline: <<-SHELL
    
    ### Update durchführen  
    sudo apt-get update
    
    ### PHP installation
    sudo apt-get install -y php7.0-gd php7.0-json php7.0-mysql php7.0-curl \
    php7.0-intl php7.0-mcrypt php-imagick \
    php7.0-zip php7.0-xml php7.0-mbstring
    
    ### Apache2 installation
    sudo apt-get -y install apache2
    
    ### Module aktivieren
    a2enmod rewrite
    a2enmod headers
    a2enmod env
    a2enmod dir
    a2enmod mime
    
    ### Webuser Berechigungen
    sudo chown -R www-data:www-data /var/www/html/
    
    ### Apache2 Service neustarten
    service apache2 restart
    
    ### Firewall konfiguration
    ufw --force enable
    sudo ufw allow 80/tcp
    sudo ufw allow ssh

    SHELL
    end

# Konfiguration VM SRVDB
  config.vm.define "db" do |db| 
    db.vm.box = "ubuntu/xenial64"
    db.vm.hostname = "SRVDB"
    db.vm.network "private_network", ip: "192.168.1.10"
    db.vm.provider "virtualbox" do |vb|
    vb.memory = "1024"
    vb.name = "SRVDB"
    end

    ### Installation und Konfiguration für db
    db.vm.provision "shell", inline: <<-SHELL
    
    ### Update durchführen
    sudo apt-get update
    
    ### MySQL Root Benutzer Kennwort setzten (root)
    debconf-set-selections <<< 'mysql-server mysql-server/root_password password root'
    debconf-set-selections <<< 'mysql-server mysql-server/root_password_again password root'
    
    ### Installation MySQL
    sudo apt-get install -y php libapache2-mod-php php-curl php-cli php-mysql php-gd mysql-client mysql-server
    
    ### Zugriffsberechtigungen Root
    mysql -u root -proot -e "GRANT ALL PRIVILEGES ON *.* TO 'root'@'%' IDENTIFIED BY 'root'; FLUSH privileges;"
    sudo service mysql restart
    
    ### Datenbank anlegen
    mysql -u root -p "CREATE DATABASE TEST;"
    
    ### Firewall konfiguration
    ufw --force enable
    sudo ufw allow 3306/tcp
    sudo ufw allow ssh

    SHELL

  end
