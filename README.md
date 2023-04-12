# Installer Samba4 en Active Directory  

## Pré-requis ##

Changer /etc/fstab pour ajouter les acl, user_xattr, barrier=1 dans la 4ème colonne  

Puis remonter la partition :  
```mount -o remount /```  

Spécifier des serveurs ntp dans /etc/ntp.conf  

```server 0.ubuntu.pool.ntp.org ```  

Changer le nom de la machine :  
```hostnamectl set-hostname ```  

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

Realm [EXAMPLE.COM]: EMPIREDESALEX.COM  
Domain [EXAMPLE]: EMPIREDESALEX  
Server Role (dc, member, standalone) [dc]:  
DNS backend (SAMBA_INTERNAL, BIND9_FLATFILE, BIND9_DLZ, NONE) [SAMBA_INTERNAL]:  
DNS forwarder IP address (write 'none' to disable forwarding) [192.0.2.1]:  
Administrator password:  
Retype password:  

Arrêt des services et redémarrage du post:  

```sudo systemctl disable smbd nmbd && sudo systemctl mask smbd nmbd \
sudo systemctl unmask samba-ad-dc && sudo systemctl enable samba-ad-dc \
sudo reboot
```

## Rejoindre le domaine ##  
```sudo samba-tool domain join empiredesalex.com DC -U administrator --realm=EMPIREDESALEX.COM```
