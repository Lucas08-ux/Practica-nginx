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

    # Usar expect para saltarse las preguntas del comando openssl
    expect -c "
    spawn openssl req -new -x509 -days 365 -nodes -out /etc/ssl/certs/vsftpd.crt -keyout /etc/ssl/private/vsftpd.key
      expect {
        \"Country Name (2 letter code)\" {send \"US\r\"; exp_continue}
        \"State or Province Name (full name)\" {send \"California\r\"; exp_continue}
        \"Locality Name (eg, city)\" {send \"Los Angeles\r\"; exp_continue}
        \"Organization Name (eg, company)\" {send \"My Company\r\"; exp_continue}
        \"Organizational Unit Name (eg, section)\" {send \"IT\r\"; exp_continue}
        \"Common Name (e.g. server FQDN or YOUR name)\" {send \"localhost\r\"; exp_continue}
        \"Email Address\" {send \"admin@example.com\r\"; exp_continue}
      }
    "
    cp -v /vagrant/vsftpd.conf /etc/vsftpd.conf

    systemctl restart vsftpd
    systemctl restart nginx
    systemctl status nginx
    SHELL
  end # lucas
end
