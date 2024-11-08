# -*- mode: ruby -*-
# vi: set ft=ruby :

# All Vagrant configuration is done below. The "2" in Vagrant.configure
# configures the configuration version (we support older styles for
# backwards compatibility). Please don't change it unless you know what
# you're doing.
Vagrant.configure("2") do |config|
  config.vm.provision "shell", inline: <<-SHELL
     apt-get update
     apt-get install -y git
     apt-get install -y nginx
     apt-get install -y vsftpd
  SHELL

  config.vm.define "lucas" do |lucas|
    lucas.vm.box = "debian/bookworm64"
    lucas.vm.network "private_network", ip: "192.168.57.102"
    config.ssh.insert_key = false

    lucas.vm.provision "shell", inline: <<-SHELL

    mkdir -p /var/www/lucas/html
    git clone https://github.com/cloudacademy/static-website-example /var/www/lucas/html
    chown -R www-data:www-data /var/www/lucas/html
    chmod -R 755 /var/www/lucas

    cp -v /vagrant/lucas /etc/nginx/sites-available/lucas
    ln -s /etc/nginx/sites-available/lucas /etc/nginx/sites-enabled/
    cp -v /vagrant/hosts /etc/hosts

    mkdir /home/vagrant/ftp
    openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout /etc/ssl/private/vsftpd.key -out /etc/ssl/certs/vsftpd.crt -subj "/"
    cp -v /vagrant/vsftpd.conf /etc/vsftpd.conf

    systemctl restart vsftpd
    systemctl restart nginx
    systemctl status nginx
    SHELL
  end # lucas
end
