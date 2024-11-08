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

  config.vm.define "lucas_nginx" do |lucas_nginx|
    lucas_nginx.vm.box = "debian/bookworm64"
    lucas_nginx.vm.network "private_network", ip: "192.168.57.103"

    lucas_nginx.vm.provision "shell", inline: <<-SHELL

    mkdir -p /var/www/lucas_nginx/html
    git clone https://github.com/cloudacademy/static-website-example /var/www/lucas_nginx/html
    chown -R www-data:www-data /var/www/lucas_nginx/html
    chmod -R 755 /var/www/lucas_nginx

    cp -v /vagrant/lucas_nginx /etc/nginx/sites-available/lucas_nginx
    ln -s /etc/nginx/sites-available/lucas_nginx /etc/nginx/sites-enabled/
    cp -v /vagrant/hosts /etc/hosts

    # mkdir /home/vagrant/ftp
    # openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout /etc/ssl/private/vsftpd.key -out /etc/ssl/certs/vsftpd.crt -subj "/"
    # cp -v /vagrant/vsftpd.conf /etc/vsftpd.conf

    systemctl restart vsftpd
    systemctl restart nginx
    systemctl status nginx
    SHELL
  end # lucas_nginx
end
