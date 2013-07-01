---
layout: page
title: Créer son image CoopBox 
header: Créer votre propre image Virtual Box contenant your virtual server from scratch
group: navigation-main
---
{% include JB/setup %}

Inspiré fortement de http://www.howtoforge.com/perfect-server-debian-wheezy-nginx-bind-dovecot-ispconfig-3

## Pré-requis:
 - installer VirtualBox
 - télécharger l'image iso de Debian 7.1 pour processeurs i386 http://cdimage.debian.org/debian-cd/7.1.0/i386/iso-cd/debian-7.1.0-i386-netinst.iso

## Installer Debian dans VirtualBox
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
 - Nom de machine : "coopbox.florian" ce nom sera visible sur le réseau local, adapter le pour vos besoins, changer le prénom au minimum (le point n'est pas obligatoire, juste pas le droit aux caractères spéciaux) a vous de voir si vous préférez un autre nom)
 - mot de passe superutilisateur : j'ai mis "coopboxroot"  
 - Nom complet du nouvel utilisateur : "CoopBox de Florian" (à adapter)
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
 
## configurer le serveur web avec des versions récentes

J'ai choisi d'installer les versions dans les dépots pour les outils serveurs (sauf si pas assez récents pour faire tourner les applis, dans ce cas ppa)

### Premiere mise à jour et ajouts de petits outils utiles
Il faut s'identifier en root (mot de passe: coopboxroot) pour installer les paquets 
	apt-get update && apt-get upgrade
	apt-get install ssh openssh-server vim-nox mc ntp ntpdate samba avahi-daemon
	vi /etc/samba/smb.conf
changer: wins support = yes
	/etc/init.d/samba restart
### NGINX, PHP, MySQL, PhpMyAdmin, Composer
	apt-get install mysql-client mysql-server openssl sudo nginx php5-fpm php5-mysql php5-curl php5-gd php5-intl php5-imagick php5-imap php5-mcrypt php5-memcache php5-memcached php5-ming php5-ps php5-pspell php5-recode php5-snmp php5-sqlite php5-tidy php5-xmlrpc php5-xsl memcached php-apc fcgiwrap phpmyadmin git-core curl build-essential libssl-dev pkg-config

#### Mysql
pas de serveur web a reconfigurer automatiquement
phpmyadmin : pas de db-commons




	vi /etc/php5/fpm/php.ini
[...]
cgi.fix_pathinfo=0
[...]
date.timezone="Europe/Paris"
[...]


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

Création des répertoires par défaut de la CoopBox
	mkdir /home/coopbox/src
	mkdir /home/coopbox/www
	mkdir /home/coopbox/import
	mkdir /home/coopbox/export
	mkdir /home/coopbox/public

	

### NodeJS et NPM
	cd /home/coopbox/src
	git clone https://github.com/joyent/node.git
	cd node
	git checkout v0.10
 	./configure --openssl-libpath=/usr/lib/ssl
	make
	make install
  chown coopbox:coopbox /home/coopbox -R

### Etherpad
	cd /home/coopbox/src
	git clone git://github.com/ether/etherpad-lite.git
> copier settings.json, mettre une base mysql
  mkdir  /var/log/etherpad-lite
  chown coopbox:coopbox /var/log/etherpad-lite
  vi /etc/init.d/etherpad-lite

  #!/bin/sh

  ### BEGIN INIT INFO
  # Provides:          etherpad-lite
  # Required-Start:    $local_fs $remote_fs $network $syslog
  # Required-Stop:     $local_fs $remote_fs $network $syslog
  # Default-Start:     2 3 4 5
  # Default-Stop:      0 1 6
  # Short-Description: starts etherpad lite
  # Description:       starts etherpad lite using start-stop-daemon
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
    
    start-stop-daemon --start --chuid "$USER:$GROUP" --background --make-pidfile --pidfile /var/run/$NAME.pid --exec $EPLITE_DIR/$EPLITE_BIN -- $LOGFILE || true
    echo "done"
  }

  #We need this function to ensure the whole process tree will be killed
  killtree() {
      local _pid=$1
      local _sig=${2-TERM}
      for _child in $(ps -o pid --no-headers --ppid ${_pid}); do
          killtree ${_child} ${_sig}
      done
      kill -${_sig} ${_pid}
  }

  stop() {
    echo "Stopping $DESC... "
     while test -d /proc/$(cat /var/run/$NAME.pid); do
      killtree $(cat /var/run/$NAME.pid) 15
      sleep 0.5
    done
    rm /var/run/$NAME.pid
    echo "done"
  }

  status() {
    status_of_proc -p /var/run/$NAME.pid "" "etherpad-lite" && exit 0 || exit $?
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


  chmod +x /etc/init.d/etherpad-lite
  ln -s /home/coopbox/src/etherpad-lite /usr/share/etherpad-lite
  update-rc.d etherpad-lite defaults
  service etherpad-lite start

### GIT, btsync, outils devs (vi optimisé, yeoman)

And now some code:

    // Code is just text indented a bit
    which(is_easy) to_remember();
