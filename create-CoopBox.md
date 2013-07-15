---
layout: page
title: Créer son image CoopBox 
header: Créer votre propre image VirtualBox contenant des outils coopératifs
group: navigation-main
---
{% include JB/setup %}

Inspiré fortement de <http://www.howtoforge.com/perfect-server-debian-wheezy-nginx-bind-dovecot-ispconfig-3>

### Pré-requis:
 - Installer VirtualBox
 - Télécharger l'image iso de Debian 7.1 pour processeurs i386 <http://cdimage.debian.org/debian-cd/7.1.0/i386/iso-cd/debian-7.1.0-i386-netinst.iso>

### Installer Debian dans VirtualBox
 - Lancer VirtualBox
 - Créer une nouvelle machine virtuelle (icone soleil Bleu "Nouvelle") et indiquer nom : CoopBox, système linux debian, 384mo de RAM, disque dynamique, maximum 8 go, 1 processeur
 - Editer les préférences de la machine virtuelle CoopBox (icone engrenage jaune "Configuration") :
   - sous menu "Système", onglet "Carte mère" : désactiver la disquette
   - sous menu "Stockage", Controleur : IDE, pour le lecteur CD cliquer sur l'icone CD puis "Choisissez un fichier de CD virtuel", parcourir votre machine pour faire le lien avec votre image ISO de Debian précedement téléchargée. 
   - sous menu "Son" : décocher "Activer le son"
   - sous menu "Réseau" : Accès par pont (puis choisir l'interface réseau par ou internet passe Ethernet ou Wifi (Wlan))
 - Fermer les préferences
 - Démarrer la machine CoopBox (icone fleche verte "Afficher") (cocher ne plus affiche le message, s'il vous parle d'un affichage 32bit, OSEF)
 - Choisir votre langue, French - Francais par exemple, idem pour Pays, disposition du clavier
 - Nom de machine : "coopbox" ce nom sera visible sur le réseau local, adapter le pour vos besoins, (juste pas le droit aux caractères spéciaux et espace) a vous de voir si vous préférez un autre nom)
 - mot de passe superutilisateur : j'ai mis "coopboxroot"  
 - Nom complet du nouvel utilisateur : "CoopBox" (à adapter)
 - Identifiant pour le compte utilisateur : "coopbox"
 - Mot de passe : "coopbox"
 - Partition : assisté - utiliser un disque entier
 - Partition : tout dans une seule partition
 - Partition : valider et appliquer les changements
 - Miroirs : France debian.proxad.net
 - Mandataire : laisser vide
 - Etude statistique pour les paquets : non
 - Programmes : tout déchocher sauf serveur ssh et utilitaires usuels 
 - Installer Grub : oui

Voila! C'est installé, reste plus qu'a enlever le cdrom virtuel et redémarrer la machine virtuelle
 
### Configurer le serveur web

J'ai choisi d'installer les versions dans les dépots pour les outils serveurs (sauf si pas assez récents pour faire tourner les applis, dans ce cas on compile pour nodejs)

### Première mise à jour et ajouts de petits outils utiles
> Il faut s'identifier en root (mot de passe: coopboxroot) pour installer les paquets 
    apt-get update && apt-get upgrade
    apt-get install ssh openssh-server openssl sudo vim-nox mc ntp ntpdate git-core curl build-essential libssl-dev pkg-config avahi-daemon

### Partage de fichiers avec Windows (Samba)
    apt-get install samba smbclient
    vi /etc/samba/smb.conf
changer: wins support = yes
    /etc/init.d/samba restart
### NGINX, PHP, MySQL, PhpMyAdmin
    apt-get install mysql-client mysql-server nginx php5-fpm php5-mysql php5-curl php5-gd php5-intl php5-imagick php5-imap php5-mcrypt php5-memcache php5-memcached php5-ming php5-ps php5-pspell php5-recode php5-snmp php5-sqlite php5-tidy php5-xmlrpc php5-xsl php-pear php-xml-parser php5-cli memcached php-apc fcgiwrap phpmyadmin 
#### Mysql et phpmyadmin
Quand l'installateur de Mysql demande quel serveur web est utilisé, ne rien cocher, pas de serveur web a reconfigurer automatiquement.
Pour phpmyadmin : pas de db-commons

    vi /etc/nginx/sites-enabled/default
changer localhost en coopbox.local
activer php, ajouter les index.php et ajouter les directives pour phpmyadmin:

    location /phpmyadmin {
           root /usr/share/;
           index index.php index.html index.htm;
           location ~ ^/phpmyadmin/(.+\.php)$ {
                   try_files $uri =404;
                   root /usr/share/;
                   fastcgi_pass unix:/var/run/php5-fpm.sock;
                   fastcgi_param HTTPS on; # <-- add this line
                   fastcgi_index index.php;
                   fastcgi_param SCRIPT_FILENAME $request_filename;
                   include /etc/nginx/fastcgi_params;
                   fastcgi_param PATH_INFO $fastcgi_script_name;
                   fastcgi_buffer_size 128k;
                   fastcgi_buffers 256 4k;
                   fastcgi_busy_buffers_size 256k;
                   fastcgi_temp_file_write_size 256k;
                   fastcgi_intercept_errors on;
           }
           location ~* ^/phpmyadmin/(.+\.(jpg|jpeg|gif|css|png|js|ico|html|xml|txt))$ {
                   root /usr/share/;
           }
    }
    location /phpMyAdmin {
           rewrite ^/* /phpmyadmin last;
    }

    /etc/init.d/nginx start


#### Php
Editer le fichier de configuration de php
    vi /etc/php5/fpm/php.ini

Et changer les valeurs suivantes
    cgi.fix_pathinfo=0
    [...]
    date.timezone="Europe/Paris"
    [...]
    post_max_size = 64M
    [...]
    upload_max_filesize = 1000M

    vi /etc/php5/fpm/pool.d/www.conf
Changer user et group : mettre coopbox

#### Création des répertoires par défaut de la CoopBox
    mkdir /home/coopbox/src
    mkdir /home/coopbox/www
    mkdir /home/coopbox/import
    mkdir /home/coopbox/export
    mkdir /home/coopbox/public
    chown coopbox:coopbox /home/coopbox -R
	

### NodeJS et NPM
    cd /home/coopbox/src
    git clone https://github.com/joyent/node.git
    cd node
    git checkout v0.10
    	./configure --openssl-libpath=/usr/lib/ssl
    make
    make install


### Etherpad

    cd /home/coopbox/src
    git clone git://github.com/ether/etherpad-lite.git

> copier settings.json, mettre une base mysql

### ScrumbLR
    cd /home/coopbox/src
    apt-get install redis-server
    git clone https://github.com/aliasaria/scrumblr.git
    cd scrumblr
    npm install
    node server.js 9002

### Ethercalc
    npm i -g ethercalc
    ethercalc

### Etherdraw (marche pas)
    cd /home/coopbox/src
    apt-get install libcairo2-dev
    git clone git://github.com/JohnMcLear/draw.git
    cd draw
    bin/run.sh

### Forever (pour garder les applis nodejs toujours ouvertes, relancees automatiquement)
    npm install forever -g

    vi /etc/init.d/nodejs-forever
> copier le code suivant

    #!/bin/sh

    ### BEGIN INIT INFO
    # Provides:          nodejs-forever
    # Required-Start:    $local_fs $remote_fs $network $syslog
    # Required-Stop:     $local_fs $remote_fs $network $syslog
    # Default-Start:     2 3 4 5
    # Default-Stop:      0 1 6
    # Short-Description: starts all coopbox nodejs apps
    # Description:       starts all coopbox nodejs apps using start-stop-daemon
    ### END INIT INFO

    PATH="/usr/local/sbin:/usr/local/bin:/sbin:/bin:/usr/sbin:/usr/bin:/opt/node/bin"
    LOGFILE="/var/log/etherpad-lite/etherpad-lite.log"
    EPLITE_DIR="/usr/share/etherpad-lite"
    EPLITE_BIN="bin/safeRun.sh"
    USER="coopbox"
    GROUP="coopbox"
    DESC="Etherpad Lite"
    NAME="etherpad-lite"

    set -e

    . /lib/lsb/init-functions

    start() {
      echo "Starting $DESC... "
      cd /home/coopbox/src/scrumblr/
      forever start server.js 9002
      forever start /usr/local/bin/ethercalc
      cd /home/coopbox/src/etherpad
      forever start bin/run.sh
      echo "done"
    }

    stop() {
      echo "Stopping $DESC... "
      forever stopall
      echo "done"
    }

    status() {
      #status_of_proc -p /var/run/$NAME.pid "" "etherpad-lite" && exit 0 || exit $?
      forever list
    }

    case "$1" in
      start)
        start
        ;;
      stop)
        stop
        ;;
      restart)
        stop
        start
        ;;
      status)
        status
        ;;
      *)
        echo "Usage: $NAME {start|stop|restart|status}" >&2
        exit 1
        ;;
    esac

    exit 0


    chmod +x /etc/init.d/nodejs-forever
    update-rc.d nodejs-forever defaults
    service nodejs-forever start

### YesWiki
    cd /home/coopbox/src
    git clone https://github.com/mrflos/yeswiki.git
    cd yeswiki
    git checkout cerco_php54
Faire les liens symboliques vers /home/coopbox/www (sauf files, cache et themes)
Puis installer le wiki à partir de l'adresse web <http://coopbox.local/>

### Owncloud
cf. <http://www.crucial.com.au/blog/2013/04/11/setting-up-the-new-owncloud-5-0-with-nginx-and-mysql/>
cf. <http://doc.owncloud.org/server/5.0/admin_manual/installation.html>
    cd /home/coopbox/www
    wget http://download.owncloud.org/community/owncloud-5.0.7.tar.bz2
    tar -xjf owncloud-5.0.7.tar.bz2
    rm owncloud-5.0.7.tar.bz2
    mkdir owncloud/data
    chown -R coopbox:coopbox owncloud
    chmod 777 owncloud/data



