# ProjetInfraB2

Notre ProjetInfra est un système de stockage accessible en ligne depuis une interface web.
Pour ce projet nous utilisons deux Rspberry pi 4 et une clé USB ou disque dure externe (DDE) pour le stockage.

##Stockage

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

##Web
