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
    # He tenido que poner esto, puesto que no me dejaba de ninguna manera acceder mediante ssh a la máquina virtual, comente si es necesario esta línea
    config.ssh.insert_key = false

    lucas.vm.provision "shell", inline: <<-SHELL

    mkdir -p /var/www/lucas/html
    git clone https://github.com/cloudacademy/static-website-example /var/www/lucas/html
    chown -R www-data:www-data /var/www/lucas/html
    chmod -R 755 /var/www/lucas

    cp -v /vagrant/lucas /etc/nginx/sites-available/lucas
    ln -s /etc/nginx/sites-available/lucas /etc/nginx/sites-enabled/
    cp -v /vagrant/hosts /etc/hosts

    # Crear usuario FTP
    useradd -m usuarioftp
    echo "usuarioftp:usuarioftp" | sudo chpasswd
    mkdir /home/usuarioftp/ftp
    chown usuarioftp:usuarioftp /home/usuarioftp/ftp
    chmod 755 /home/usuarioftp/ftp

    # Me salto las preguntas del comando openssl
    openssl req -x509 -nodes -days 365 -newkey rsa:2048 \
      -keyout /etc/ssl/private/vsftpd.key -out /etc/ssl/certs/vsftpd.crt \
      -subj "/C=US/ST=State/L=City/O=Organization/OU=OrgUnit/CN=localhost"

    cp -v /vagrant/vsftpd.conf /etc/vsftpd.conf

    # Segunda web
    mkdir -p /var/www/web2/html
    git clone https://github.com/Lucas08-ux/web2.git /var/www/web2/html
    chown -R www-data:www-data /var/www/web2/html
    chmod -R 755 /var/www/web2
    cp -v /vagrant/web2 /etc/nginx/sites-available/web2
    ln -s /etc/nginx/sites-available/web2 /etc/nginx/sites-enabled/

    systemctl restart vsftpd
    systemctl restart nginx
    systemctl status nginx
    SHELL
  end # lucas
end
