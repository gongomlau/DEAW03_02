# Utilizamos una imagen oficial de Ubuntu
FROM ubuntu:22.04

# Damos información sobre la imagen que estamos creando
LABEL \
    version="2.0" \
    description="Ubuntu + Apache2 + virtual host + FTP" \
    maintainer="lagonzalez <lagonzalez@birt.eus>"

# Evitar prompts interactivos
ENV DEBIAN_FRONTEND=noninteractive

# Actualizamos la lista de paquetes e instalamos servicios necesarios
RUN apt-get update && \
    apt-get install -y nano apache2 proftpd proftpd-mod-crypto git openssh-client openssh-server&& \
    rm -rf /var/lib/apt/lists/*

# Creamos directorios para los sitios web y configuraciones
RUN mkdir -p /var/www/html/sitioprimero /var/www/html/sitiosegundo
RUN mkdir /var/run/sshd
RUN mkdir -p /var/ftp/pub

# Configuramos Apache para evitar el error de ServerName
RUN echo "ServerName localhost" >> /etc/apache2/apache2.conf

# Copiamos archivos al contenedor
COPY index1.html /var/www/html/sitioprimero/index.html
COPY index2.html /var/www/html/sitiosegundo/index.html
COPY sitio1.conf /etc/apache2/sites-available/
COPY sitio2.conf /etc/apache2/sites-available/
COPY sitio1.key /etc/ssl/private/
COPY sitio1.cer /etc/ssl/certs/
COPY proftpd.conf /etc/proftpd/proftpd.conf

# Habilitamos los sitios y el módulo SSL
RUN a2ensite sitio1.conf && \
    a2ensite sitio2.conf && \
    a2enmod ssl

# Configuramos FTP
RUN sed -i 's/#LoadModule mod_tls.c/LoadModule mod_tls.c/' /etc/proftpd/modules.conf

# Crear usuario FTP
RUN useradd -d /var/www/html/sitioprimero -s /usr/sbin/nologin laura1 && \
    echo "laura1:1234" | chpasswd && \
    chown -R laura1:laura1 /var/www/html/sitioprimero

# Permisos usuario FTP para el directorio público
RUN chown -R ftp:nogroup /var/ftp/pub

# Generar certificados TLS para ProFTPD
RUN openssl req -x509 -nodes -days 365 \
-newkey rsa:2048 \
-keyout /etc/ssl/private/proftpd.key \
-out /etc/ssl/certs/proftpd.crt \
-subj "/C=ES/ST=Bizkaia/L=Bilbao/O=Laura Gonzalez/CN=FTP"

# Crear segundo usuario FTP
RUN useradd -m -d /var/www/html/sitiosegundo -s /bin/bash laura2 && \
    echo "laura2:1234" | chpasswd && \
    chown -R laura2:laura2 /var/www/html/sitiosegundo

# Crear carpeta .ssh
RUN mkdir -p /root/.ssh

# Copiar clave SSH
COPY id_rsa /root/.ssh/id_rsa

# Permisos correctos
RUN chmod 600 /root/.ssh/id_rsa

# Añadir GitHub a known_hosts
RUN ssh-keyscan github.com >> /root/.ssh/known_hosts

# Clonar el repositorio privado
RUN git clone git@github.com:deaw-birt/UD3-ftp_anonimo.git /var/ftp/pub || mkdir /repo_anonimo

# Ajustar permisos para FTP anónimo
RUN chown -R ftp:nogroup /var/ftp/pub

# Borrar SOLO la clave privada (opcional)
RUN rm /root/.ssh/id_rsa

# Exponemos los puertos
EXPOSE 80
EXPOSE 443
EXPOSE 21
EXPOSE 30000-30010

# Comando por defecto al iniciar el contenedor
CMD ["sh", "-c", "service proftpd start && service ssh start &&apachectl -D FOREGROUND"]