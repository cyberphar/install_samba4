# Installer Samba4 en Active Directory  

## Pré-requis ##

Changer /etc/fstab pour ajouter les acl, user_xattr, barrier=1 dans la 4ème colonne  

Puis remonter la partition :  
```mount -o remount /```  

Spécifier des serveurs ntp dans /etc/ntp.conf  

```server 0.ubuntu.pool.ntp.org ```  

Changer le nom de la machine :  
```hostnamectl set-hostname ubndc01```  

## Installation ##  

Installer samba4:  
```
sudo apt install acl attr autoconf bind9utils bison build-essential debhelper dnsutils libgnutls30 \
docbook-xml docbook-xsl flex gdb libjansson-dev smbldap-tools libacl1-dev libaio-dev libarchive-dev \
libattr1-dev libblkid-dev libbsd-dev libcap-dev libcups2-dev libgnutls28-dev libgpgme-dev libjson-perl \
libldap2-dev libncurses5-dev libpam0g-dev libparse-yapp-perl libpopt-dev libreadline-dev nettle-dev \
perl perl-modules-5.28 pkg-config python-all-dev python-crypto python-dbg python-dev python-dnspython \
python3-dnspython python-gpg python3-gpg python-markdown python3-markdown python3-dev xsltproc xattr \
zlib1g-dev liblmdb-dev lmdb-utils libsystemd-dev python3-cryptography gnutls-bin cifs-utils \
php7.3-ldap samba krb5-user smbclient winbind
```  
Installation en mode interactive:  

```sudo samba-tool domain provision --use-rfc2307 --interactive```  

Remplir les paramètres demandés:  

Realm [EXAMPLE.COM]: EMPIREDESALEX.LOCAL  
Domain [EXAMPLE]: EMPIREDESALEX  
Server Role (dc, member, standalone) [dc]:  
DNS backend (SAMBA_INTERNAL, BIND9_FLATFILE, BIND9_DLZ, NONE) [SAMBA_INTERNAL]:  
DNS forwarder IP address (write 'none' to disable forwarding) [192.0.2.1]: none  
Administrator password:  
Retype password:  

Arrêt des services et redémarrage du post:  

```sudo systemctl disable smbd nmbd && sudo systemctl mask smbd nmbd \
sudo systemctl unmask samba-ad-dc && sudo systemctl enable samba-ad-dc \
sudo reboot
```
## Vérification ##  
Avant de démarrer Samba AD DC, il est important de vérifier la bonne configuration du DNS.
Le fichier /etc/resolv.conf devrait contenir
```
search empiredesalex.local  
nameserver 127.0.0.1
```
On vérifie les processus lancés par samba:  
```sudo samba-tool processes```

## Initialisation ##  
```
sudo mv --backup=t /etc/samba/smb.conf /etc/samba/smb.conf.old  
sudo samba-tool domain join empiredesalex.local DC -U administrator --realm=EMPIREDESALEX.LOCAL
``` 
## Rejoindre le domaine ##  
```sudo samba-tool domain join empiredesalex.local DC -U administrator --realm=EMPIREDESALEX.LOCAL```  
## Changer le mot de passe admin ##  
```sudo samba-tool user setpassword administrator```  
## Ajouter un enregistrement DNS ##  
```samba-tool dns add ubndc01 empiredesalex.local ubndc01 A 192.0.2.21 -U administrator```  
## Test Kerberos ##  
Demande d'un ticket:  
`kinit administrator@EXAMPLE.COM`  
Vérification de la liste:  
`klist`
## Test SMB ##  
Lister les partages définis localement sur le DC :  
```smbclient -L localhost```

## Voir l'état de la synchronisation ##  
```samba-tool drs showrepl```  

## Source ##  
> https://doc.ubuntu-fr.org/samba-active-directory 
