# ProjetInfraB2

Notre ProjetInfra est un système de stockage accessible en ligne depuis une interface web.
Pour ce projet nous utilisons deux Rspberry pi 4 et une clé USB ou disque dure externe (DDE) pour le stockage.

## Stockage

### Samba
Pour partager les fichiers à l'autre Raspberry nous utiliserons Samba.

1. Mise en place
```
sudo apt install -y samba
sudo reboot

sudo smbpasswd -a pi
sudo usermod -a -G sambashare pi
sudo chmod 777 -R /data/usbshare/
sudo chown -R pi:sambashare /data/usbshare/
```
Création du point de montage du DDE
`sudo mkdir -p /data/usbshare`

2. Configuration du serveur

Dans `/etc/samba/smb.conf` ajouter à la fin

```
[USBShare]
   comment = Partage USB du Raspberry Pi
   path = /data/usbshare
   valid user = pi
   directory mask = 750
   create mask = 750
   browsable = yes
   writable = yes
   guest ok = no
```

Puis on redémarre la Raspberry `sudo reboot`. 

### Cryptsetup

Pour la partie Stockage nous allons formater et chiffrer notre DDE à l'aide de cryptsetup puis le monter sur le dossier créer précédemment.
```
sudo apt install -y cryptsetup
sudo cryptsetup luksFormat -c aes-xts-plain64 -s 512 -h sha512 /dev/sdaXX
sudo cryptsetup luksOpen /dev/sdaXX share
sudo mkfs.ext3 /dev/mapper/share
sudo mount -t ext3 /dev/mapper/share /data/USBShare
```

Puis on redémarre la Raspberry `sudo reboot`. 

## Web

### Mise en place de l'accès Internet :

Veillez pour mettre en place le projet à bien ouvrir un port sur l'accès internet de votre machine qui redirige vers votre machine.

Vérifier aussi qu'il n'y ai aucun autre service sur votre machine qui occupe le port 80 (utilisé par le service) Sinon, modifier le port connecté à HAProxy (dans docker-compose.yml, ligne 27 -> `ports: - 80:80`, il faut modifier le premier (le port sur l'hôte)) et ajuster la redirection réalisée ci-précédemment au port de la machine.

### Installation de docker :

Télécharge + lance l'installation de Docker :
`curl -sSL https://get.docker.com | sh`

Ajout de l'utilisateur pi au groupe Docker afin que Pi puisse lancer les commandes Docker :
`sudo usermod -aG docker <username>` avec dans notre cas l'utilisateur Pi, ce qui donne donc `sudo usermod -aG docker pi`.

Relancement de la machine pour mettre à jour les permissions :
`sudo reboot now` pour redémarrer la machine maintenant.

Installation des librairies nécessaires à l'installation de Docker-compose:
`sudo apt-get install libffi-dev libssl-dev` pour installer les librairies nécessaires à l'installation de Python.
`sudo apt-get install -y python python-pip` afin d'installer Python et son installer Pip.
`sudo apt-get remove python-configparser` pour enlever python-configparser.

Installation de docker-compose :
`sudo pip install docker-compose` -> Installation de docker-compose via l'installer Pip

Et boom ! Docker et docker-compose sont installés et utilisables sur la machine !

### Récupération de la conf :

Pour récupérer la conf, il suffit de download/clone ce repo et de copier la conf dans le dossier que vous désirez.

### Récupération de la donnée :

Création du dossier "point de montage" sur lequel on montera la partition issue de la deuxième rasp :
`mkdir /home/pi/Docker/infraProject/webServers/app/Share`

Pour regarder les partages disponibles sur l'adresse 192.168.0.44 (celle de la raspberry hôte des données) :
`smbclient -L 192.168.0.44`

Pour monter une partition avec le système de fichier `smb3` et les options -o pour préciser l'identifiant et le mot de passe de la partition que à laquelle on veut accèder, ainsi que l'id du groupe / utilisateur que l'on veut donner. Enfin, on précise l'adresse de la machine hôte du répertoire des données (192.168.0.44) ainsi que le nom que l'on a récupéré avant (USBShare) puis le chemin /!\ ABSOLU /!\ jusqu'au point de montage.
`sudo mount -t smb3 -o username=pi,password=aledaled,gid=33,uid=33 //192.168.0.44/USBShare /home/pi/Docker/infraProject/webServers/app/Share/`

### Ajout d'une Machine :

Pour ajouter une machine au projet, il suffit de dupliquer un des dossiers dans `webApp` contenant le Dockerfile, puis changer le nom. Enfin, ajouter la création du Docker dans le `docker-compose.yml` (il suffit de dupliquer une des créations de server et adapter les noms) et finalement ajouter une ligne dans le fichier de configuration de haproxy (`haproxy/haproxy.cfg`) comme les précédentes (en changeant le nom du server).
