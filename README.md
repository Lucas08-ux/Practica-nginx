# Practica-nginx
## Modificación del documento Vagrantfile
Primero, he configurado el archivo Vagrantfile, creando una máquina virtual llamada lucas_nginx, con la cual se puede acceder a través de la dirección IP 192.168.57.103.

Luego, he modificado el archivo Vagrantfile para que copie y pegue todos los archivos que he modificado a lo largo de la práctica y haga todos los cambios necesarios para que el sitio web funcione correctamente.

```
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
```
## Creación de las carpetas del sitio web
He creado una carpeta llamada lucas_nginx para la web y a su vez, dentro de ella se encuentra la carpeta html. Dentro de esta última, se encuentra una carpeta llamada static-website-example, que tiene el contenido del repositiorio "https://github.com/cloudacademy/static-website-example". 

También le he dado estos dos permisos para que no haya problemas de permisos a la hora de acceder a la web:
```
  chown -R www-data:www-data /var/www/lucas/html
  chmod -R 755 /var/www/lucas
```

## Configuración del servidor web nginx
He creado un archivo llamado lucas_nginx dentro de /etc/nginx/sites-available, y dentro de este archivo escrito lo siguiente:
```
server {
	listen 80;
	listen [::]:80;
	root /var/www/lucas/html;
	index index.html index.htm index.nginx-debian.html;
	server_name lucas;
	location / {
		try_files $uri $uri/ =404;
	}
}
```
También, he copiado este mismo archivo y lo he añadido al directorio sites-enabled

## Modificación del archivo hosts
El archivo hosts contiene las direcciones IP de los servidores que se conectan a la red local, por lo que para que no se presente un error de conexión, he modificado el archivo hosts para que la dirección IP 192.168.57.102 apunte a la máquina virtual lucas_nginx en el archivo, específicamente en la we lucas. He hecho esto tanto en windows 11, que es el sistema operativo que usamos en la máquina virtual, como en debian, donde he hecho esto en el archivo hosts:

```
127.0.0.1	localhost
127.0.0.2	bookworm
::1		localhost ip6-localhost ip6-loopback
ff02::1		ip6-allnodes
ff02::2		ip6-allrouters
192.168.57.102 lucas
182.168.57.102 web2
```
He buscado en el navegador http://lucas y he podido acceder a la web.

## Configuración de servidor ftps en Debian
Primero, he creado un usuario:
```
  useradd -m usuarioftp
  echo "usuarioftp:usuarioftp" | sudo chpasswd
```
Para crear los certificados de seguridad necesarios, he introducido estas líneas en el vagrantfile para que se salte las preguntas del comando openssl:
```
openssl req -x509 -nodes -days 365 -newkey rsa:2048 \
      -keyout /etc/ssl/private/vsftpd.key -out /etc/ssl/certs/vsftpd.crt \
      -subj "/C=US/ST=State/L=City/O=Organization/OU=OrgUnit/CN=localhost"
```

Una vez he realizado estos pasos, he modificado el archivo vsftpd.conf para que se use el certificado que he creado y que el usuario ftp se conecte a la máquina virtual lucas_nginx. He eliminado tres líneas que habían y las he sustituido por las siguientes:
```
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
local_root=/home/usuarioftp/ftp
```
He tenido que modificar en este mismo archivo y añadir estas tres líneas para que me permitiera transferir archivos desde el servidor ftp:

```
write_enable=YES       # Permite subir archivos
local_enable=YES       # Permite a los usuarios locales iniciar sesión
chroot_local_user=NO   # Restringe a los usuarios a su directorio home
```

## Segunda web
Mediante una conexión FTP en Filezilla en el puerto 21, he accedido a la carpeta ftp y he subido una pagina web mía.

Una vez subida la pagina web a /home/usuarioftp/ftp/web2, he transferido la carpeta web2 a la carpeta /var/www/web2.

```
mv /home/usuarioftp/ftp/web2 /var/www/web2
```

He añadido esta línea al archivo hosts tanto de windows 11 como de debian:
```
192.168.57.102  web2
```
He creado un archivo web2 dentro de /etc/nginx/sites-available, y dentro de este archivo he escrito lo siguiente:
```
server {
	listen 80;
	listen [::]:80;
	root /var/www/web2;
	index index.html index.htm index.nginx-debian.html;
	server_name web2;
	location / {
		try_files $uri $uri/ =404;
	}
}
```
He copiado este mismo archivo y lo he añadido al directorio sites-enabled. De esta manera, ahora tenemos dos sitios web en nuestra máquina virtual lucas_nginx. He recreado esta situación en el vagrantfile, pero como no se puede hacer ftp, he copiado mi web de un repositorio que he creado en github.

# Cuestiones finales
## ¿Qué pasa si no hago el link simbólico entre sites-available y sites-enabled de mi sitio web?
Si no haces el link simbólico, Nginx no cargará la configuración de tu sitio y no servirá el sitio web.

## ¿Qué pasa si no le doy los permisos adecuados a /var/www/nombre_web?
Si no asignas permisos adecuados, Nginx no podrá acceder a los archivos, lo que resultará en errores o un sitio no disponible.
