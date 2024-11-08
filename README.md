# Practica-nginx
## Modificación del documento Vagrantfile
Primero, he configurado el archivo Vagrantfile, creando una máquina virtual llamada lucas_nginx, con la cual se puede acceder a través de la dirección IP 192.168.57.103.

Luego, he modificado el archivo Vagrantfile para que copie y pegue todos los archivos que he modificado a lo largo de la práctica.

´´´
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
    cp -vr /vagrant/lucas_nginx /var/www/lucas_nginx
    chown -R www-data:www-data /var/www/lucas_nginx/html
    chmod -R 755 /var/www/lucas_nginx


    cp -v /vagrant/sites-available-lucas_nginx /etc/nginx/sites-available/lucas_nginx
    ln -s /etc/nginx/sites-available/lucas_nginx /etc/nginx/sites-enabled/
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
También, he copiado este mismo archivo y lo he añadido al directorio sites-enabled

## Modificación del archivo hosts
El archivo hosts contiene las direcciones IP de los servidores que se conectan a la red local, por lo que para que no se presente un error de conexión, he modificado el archivo hosts para que la dirección IP 192.168.57.103 apunte a la máquina virtual lucas_nginx.

´´´
127.0.0.1	localhost
127.0.0.2	bookworm
::1		localhost ip6-localhost ip6-loopback
ff02::1		ip6-allnodes
ff02::2		ip6-allrouters
192.168.57.103 lucas_nginx
´´´

## Configuración de servidor ftps en Debian
Primero, he creado una carpeta ftp en la ruta /home/vagrant/ftp. 

Para crear los certificados de seguridad necesarios, he introducido esta línea:
´´´
sudo openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout /etc/ssl/private/vsftpd.key -out /etc/ssl/certs/vsftpd.crt -subj "/"
´´´

Una vez he realizado estos pasos, he modificado el archivo vsftpd.conf para que se use el certificado que he creado y que el usuario ftp se conecte a la máquina virtual lucas_nginx. He eliminado tres líneas que habían y las he sustituido por las siguientes:
´´´
rsa_cert_file=/etc/ssl/certs/vsftpd.crt
rsa_private_key_file=/etc/ssl/private/vsftpd.key
ssl_enable=YES
allow_anon_ssl=NO
force_local_data_ssl=YES
force_local_logins_ssl=YES
ssl_tlsv1=YES
ssl_sslv2=NO
ssl_sslv3=NO
require_ssl_reuse=NO
ssl_ciphers=HIGH
local_root=/home/vagrant/ftp
´´´

## Pruebas de funcionamiento ftp
### Mediante una conexión FTP sin cifrar en el puerto 21.
Para realizar la prueba, me he instalado el paquete ftp y con este comando he comprobado su correcto funcionamiento:
```
vagrant@bookworm:/etc/nginx/sites-enabled$ ftp 192.168.57.103
Connected to 192.168.57.103.
220 (vsFTPd 3.0.3)
Name (192.168.57.103:vagrant):
```




# Cuestiones finales
## ¿Qué pasa si no hago el link simbólico entre sites-available y sites-enabled de mi sitio web?
Si no haces el link simbólico, Nginx no cargará la configuración de tu sitio y no servirá el sitio web.

## ¿Qué pasa si no le doy los permisos adecuados a /var/www/nombre_web?
Si no asignas permisos adecuados, Nginx no podrá acceder a los archivos, lo que resultará en errores o un sitio no disponible.
