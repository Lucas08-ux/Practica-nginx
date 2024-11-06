# Practica-nginx
## Modificación del documento Vagrantfile
Primero, he configurado el archivo Vagrantfile, creando una máquina virtual llamada lucas_nginx, con la cual se puede acceder a través de la dirección IP 192.168.57.103.

Luego, he modificado el archivo Vagrantfile para que copie y pegue todos los archivos que he modificado a lo largo de la práctica.

´´´
Vagrant.configure("2") do |config|
  config.vm.provision "shell", inline: <<-SHELL
     apt-get update
     apt-get install -y nginx
     apt-get install -y vsftpd
  SHELL

  config.vm.define "lucas_nginx" do |lucas_nginx|
    lucas_nginx.vm.box = "debian/bookworm64"
    lucas_nginx.vm.network "private_network", ip: "192.168.57.103"

    lucas_nginx.vm.provision "shell", inline: <<-SHELL

    cp -vr /vagrant/lucas_nginx /var/www/lucas_nginx
    chown -R www-data:www-data /var/www/lucas_nginx/html
    chmod -R 755 /var/www/lucas_nginx

    cp -v /vagrant/sites-available-lucas_nginx /etc/nginx/sites-available/lucas_nginx
    cp -v /vagrant/sites-available-lucas_nginx /etc/nginx/sites-enabled/lucas_nginx
    cp -v /vagrant/hosts /etc/hosts

    mkdir /home/vagrant/ftp
    openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout /etc/ssl/private/vsftpd.key -out /etc/ssl/certs/vsftpd.crt -subj "/"
    cp -v /vagrant/vsftpd.conf /etc/vsftpd.conf

    systemctl restart nginx
    systemctl restart vsftpd
    SHELL
  end # lucas_nginx
end
´´´
## Creación de las carpetas del sitio web
He creado una carpeta llamada lucas_nginx para la web y a su vez, dentro de ella se encuentra la carpeta html. Dentro de esta última, se encuentra una carpeta llamada static-website-example, que tiene el contenido del repositiorio "https://github.com/cloudacademy/static-website-example". 

También le he dado estos dos permisos para que no haya problemas de permisos a la hora de acceder a la web:
´´´
chown -R www-data:www-data /var/www/lucas_nginx/html
    chmod -R 755 /var/www/lucas_nginx
´´´
He buscado en el navegador http://192.168.57.103/ y he podido acceder a la web.

## Configuración del servidor web nginx
He creado un archivo llamado lucas_nginx dentro de /etc/nginx/sites-available, y dentro de este archivo escrito lo siguiente:
´´´
server {
	listen 80;
	listen [::]:80;
	root /var/www/lucas_nginx/html/static-website-example;
	index index.html index.htm index.nginx-debian.html;
	server_name lucas_nginx;
	location / {
		try_files $uri $uri/ =404;
	}
}
´´´