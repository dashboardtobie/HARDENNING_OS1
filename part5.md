# Part V : OpenSSH Server

## 1. Basics

🌞 **Afficher l'identifiant du processus serveur OpenSSH en cours d'exécution**

- listez tous les programmes en cours d'exécution (avec une commande `ps`)
- mettez en évidence uniquement la ligne qui concerne le serveur SSH (y'en a qu'une)
```
[dash@localhost ~]$ ps -aux | grep "sshd"
root        1587  0.0  1.9  16772  9216 ?        Ss   17:30   0:00 sshd: /usr/sbin/sshd -D [listener] 0 of 10-100 startups
```

🌞 **Changer le port d'écoute du serveur OpenSSH**
```
[dash@localhost ~]$ sudo nano /etc/ssh/sshd_config
```
dans le fichier on ajoute `port 2222`

- prouvez que votre changement a pris effet
```
[dash@localhost ~]$ sudo systemctl restart sshd
[dash@localhost ~]$ sudo ss -tulnp | grep sshd
tcp   LISTEN 0      128    192.168.133.129:2222      0.0.0.0:*    users:(("sshd",pid=1921,fd=3))
```
- prouvez que vous pouvez toujours vous connecter à la machine en SSH, sur ce nouveau port
```
[dash@localhost ~]$ sudo firewall-cmd --add-port=2222/tcp --permanent
[dash@localhost ~]$ sudo firewall-cmd --reload
```
```
┌─[✗]─[dashboard@parrot]─[~]
└──╼ssh dash@192.168.133.129 -p 2222
dash@192.168.133.129's password: 
Last login: Tue Feb  4 23:40:48 2025 from 192.168.133.128
[dash@localhost ~]$
```
- expliquez pourquoi on considère parfois utile de changer le port d'écoute par défaut du serveur SSH


## 2. Authentication modes

### A. Key-based authentication

🌞 **Configurer une authentification par clé**

- vous devez pouvoir vous connecter sur votre utilisateur
- sans saisir de password
- en utilisant une paire de clés
```
┌─[✗]─[dashboard@parrot]─[~]
└──╼ $ssh-keygen -t rsa -b 4096 -f ~/.ssh/id_rsa
Generating public/private rsa key pair.
Enter passphrase (empty for no passphrase): 
Enter same passphrase again: 
Your identification has been saved in /home/dashboard/.ssh/id_rsa
Your public key has been saved in /home/dashboard/.ssh/id_rsa.pub
The key fingerprint is:
SHA256:qogDURM2Ad9fRUS1n27CXGpG7BfdfocHEF8/p6yw/Jk dashboard@parrot
The key's randomart image is:
+---[RSA 4096]----+
|..=o     +=.o   .|
| ooo     .   + ..|
| ....   .   o ..o|
|.    . .   . + ++|
| .    . S . o O o|
|.      . . B = = |
|.     .   o X = =|
|.. . .     + * .o|
|... .       E    |
+----[SHA256]-----+
┌─[dashboard@parrot]─[~]
└──╼ $cat ~/.ssh/id_rsa.pub | ssh dash@192.168.133.129 -p 2222 "mkdir -p ~/.ssh && cat >> ~/.ssh/authorized_keys && chmod 600 ~/.ssh/authorized_keys && chmod 700 ~/.ssh"
dash@192.168.133.129's password: 
┌─[✗]─[dashboard@parrot]─[~]
└──╼ $ssh dash@192.168.133.129 -p 2222
Last login: Wed Feb  5 00:32:40 2025 from 192.168.133.128
[dash@localhost ~]$ 
```

🌞 **Désactiver la connexion par password**
```
[dash@localhost ~]$ sudo nano /etc/ssh/sshd_config
```
On ajoute les paramètres suivants `PasswordAuthentication no` et `PubkeyAuthentication yes`
```
[dash@localhost ~]$ sudo systemctl restart sshd
```

🌞 **Désactiver la connexion en tant que `root`**
```
[dash@localhost ~]$ sudo nano /etc/ssh/sshd_config
```
On ajoute le paramètre `PermitRootLogin no`
```
[dash@localhost ~]$ sudo systemctl restart sshd
```

## 3. Cert-based authentication


🌞 **Configurer une authentification par certificat**

- j'ai dit par certificat, pas par simple clé
- pareil, faites-le avec votre utilisateur pour les tests
Au niveau du client
```
┌─[✗]─[dashboard@parrot]─[~]
└──╼ $ssh-keygen -t rsa -b 4096 -f ~/.ssh/id_rsa_cert
Generating public/private rsa key pair.
Enter passphrase (empty for no passphrase): 
Enter same passphrase again: 
Your identification has been saved in /home/dashboard/.ssh/id_rsa_cert
Your public key has been saved in /home/dashboard/.ssh/id_rsa_cert.pub
The key fingerprint is:
SHA256:o5Qiu6+2nF0XRjiFazZkyHXAmrAVf0ElKsb2Cc7HGRo dashboard@parrot
The key's randomart image is:
+---[RSA 4096]----+
|   ..=o+*..      |
|  ..+.== o       |
|   +E=*.o        |
|  .=oB=O         |
|  . =oO.S        |
|   o + o o       |
|  .   o .        |
| ..+ . .         |
| .*+o            |
+----[SHA256]-----+
┌─[✗]─[dashboard@parrot]─[~]
└──╼ $cat ~/.ssh/id_rsa_cert.pub | ssh dash@192.168.133.129 -p 2222 "cat > /tmp/id_rsa_cert.pub"
```
Au niveau du serveur
```
┌─[✗]─[dashboard@parrot]─[~]
└──╼ssh dash@192.168.133.129 -p 2222
Last login: Wed Feb  5 02:01:13 2025 from 192.168.133.128
[dash@localhost ~]$ ssh-keygen -t rsa -b 4096 -f /etc/ssh/ssh_ca
Generating public/private rsa key pair.
Enter passphrase (empty for no passphrase): 
Enter same passphrase again: 
Saving key "/etc/ssh/ssh_ca" failed: Permission denied
[dash@localhost ~]$ sudo ssh-keygen -t rsa -b 4096 -f /etc/ssh/ssh_ca
Generating public/private rsa key pair.
Enter passphrase (empty for no passphrase): 
Enter same passphrase again: 
Your identification has been saved in /etc/ssh/ssh_ca
Your public key has been saved in /etc/ssh/ssh_ca.pub
The key fingerprint is:
SHA256:GPDmQFql1bZmk4d/dPzmA0JZX5umu/n875PQT9t56jE root@localhost.localdomain
The key's randomart image is:
+---[RSA 4096]----+
|    +.o.         |
|   + =  o    .  .|
|  . o +. +  o.. +|
|     + oB .o. o= |
|      ooS+.. .+. |
|          ...+ .+|
|           .. E+*|
|             .oO=|
|             +==X|
+----[SHA256]-----+
[dash@localhost ~]$ sudo chmod 600 /etc/ssh/ssh_ca
[dash@localhost ~]$ sudo ssh-keygen -s /etc/ssh/ssh_ca -I "client-cert" -n dash -V +52w /tmp/id_rsa_cert.pub
Signed user key /tmp/id_rsa_cert-cert.pub: id "client-cert" serial 0 for dash valid from 2025-02-05T02:30:00 to 2026-02-04T02:31:39
[dash@localhost ~]$ sudo nano /etc/ssh/sshd_config
TrustedUserCAKeys /etc/ssh/ssh_ca.pub
[dash@localhost ~]$ sudo systemctl restart sshd
[dash@localhost ~]$ cat /tmp/id_rsa_cert-cert.pub | ssh dashboard@192.168.133.128  "cat > /home/dashboard/.ssh/id_rsa_cert-cert.pub"
dashboard@192.168.133.128's password: 
[dash@localhost ~]$ 
```
On revient sur le client
```
┌─[dashboard@parrot]─[~]
└──╼ $mv ~/.ssh/id_rsa_cert-cert.pub ~/.ssh/id_rsa_cert.pub
┌─[dashboard@parrot]─[~]
└──╼ $chmod 600 ~/.ssh/id_rsa_cert.pub
┌─[dashboard@parrot]─[~]
└──╼ $nano ~/.ssh/config
 GNU nano 7.2                                                             /home/dashboard/.ssh/config                                                              Modified  
Host rocky_server
    HostName 192.168.133.129
    User dash
    IdentityFile ~/.ssh/id_rsa_cert
    CertificateFile ~/.ssh/id_rsa_cert.pub
┌─[dashboard@parrot]─[~]
└──╼ $ssh rocky_server -p 2222
Load key "/home/dashboard/.ssh/id_rsa_cert.pub": error in libcrypto
sign_and_send_pubkey: signing failed for RSA-CERT "/home/dashboard/.ssh/id_rsa_cert" from agent: agent refused operation
Last login: Wed Feb  5 02:47:33 2025 from 192.168.133.128
[dash@localhost ~]$ 
```
## 4. Further hardening

🌞 **Proposer au moins 5 configurations supplémentaires qui permettent de renforcer la sécurité du serveur OpenSSH**

> Je vous recommande fooooortement de vous inspirer de ressources d'Internet pour ça. Regardez par exemple le guide de l'ANSSI à ce sujet (obsolète, mais la plupart des principes sont toujours valides), ou encore le guide CIS sur le sujet, ou l'excellent guide Mozilla sur le sujet, . Il existe d'autres ressources de confiance, à votre meilleur moteur de recherches !

## 5. fail2ban

> Un outil extrêmement récurrent dans le monde Linux : un premier rempart contre les attaques de bruteforce.

🌞 **Installer fail2ban sur la machine**

🌞 **Configurer fail2ban**

- en cas de multiples tentatives de connexion échouées sur le serveur SSH, l'utilisateur sera banni
- précisément : après 7 tentatives de connexion échouées en moins de 5 minutes
- c'est l'adresse IP de la personne qui fait des connexions échouées de façon répétée qui est blacklistée

🌞 **Prouvez que fail2ban est effectif**

- faites-vous ban
- montrez l'état de la jail fail2ban pour voir quelles IP sont ban
- levez le ban avec une commande adaptée
