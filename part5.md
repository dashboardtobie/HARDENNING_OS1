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

```
[dash@localhost ~]$ sudo nano /etc/ssh/sshd_config
# Autoriser uniquement certains utilisateurs ou memnbres de groupe à se connecter par ssh
AllowUsers dash
AllowGroups sshusers

# Empeche une attaque ddos en limitant le nombre de connexion simultané
MaxStartups 2:30:10

# Activer le stictmode pour empêcher SSH d’utiliser des clés ou fichiers mal configurés
StrictModes yes

# Bloquer la redirection de ports qui peut être exploitée pour échapper aux restrictions réseau et faire du tunneling SSH malveillant
AllowTcpForwarding no
PermitTunnel no

# Éviter qu’une session SSH inutilisée reste ouverte indéfiniment et soit compromise
ClientAliveInterval 300
ClientAliveCountMax 2

[dash@localhost ~]$ sudo systemctl restart sshd
```

## 5. fail2ban

🌞 **Installer fail2ban sur la machine**
```
[dash@localhost ~]$ sudo yum install epel-release -y
[dash@localhost ~]$ sudo yum install fail2ban -y
```

🌞 **Configurer fail2ban**

- en cas de multiples tentatives de connexion échouées sur le serveur SSH, l'utilisateur sera banni
- précisément : après 7 tentatives de connexion échouées en moins de 5 minutes
- c'est l'adresse IP de la personne qui fait des connexions échouées de façon répétée qui est blacklistée
```
[dash@localhost ~]$ sudo systemctl enable --now fail2ban
Created symlink /etc/systemd/system/multi-user.target.wants/fail2ban.service → /usr/lib/systemd/system/fail2ban.service.
[dash@localhost ~]$ sudo cp /etc/fail2ban/jail.conf /etc/fail2ban/jail.local
[dash@localhost ~]$ sudo nano /etc/fail2ban/jail.local
 [sshd]
 enabled = true
 port = 2222
 filter = sshd
 logpath = /var/log/auth.log
 maxretry = 7
 findtime = 300
[dash@localhost ~]$ sudo systemctl restart fail2ban
[dash@localhost ~]$ sudo systemctl status fail2ban
● fail2ban.service - Fail2Ban Service
     Loaded: loaded (/usr/lib/systemd/system/fail2ban.service; enabled; preset: disabled)
     Active: active (running) since Wed 2025-02-05 11:26:21 CET; 8s ago
       Docs: man:fail2ban(1)
    Process: 17891 ExecStartPre=/bin/mkdir -p /run/fail2ban (code=exited, status=0/SUCCESS)
   Main PID: 17892 (fail2ban-server)
      Tasks: 5 (limit: 2665)
     Memory: 12.1M
        CPU: 128ms
     CGroup: /system.slice/fail2ban.service
             └─17892 /usr/bin/python3 -s /usr/bin/fail2ban-server -xf start

Feb 05 11:26:21 localhost.localdomain systemd[1]: Starting Fail2Ban Service...
Feb 05 11:26:21 localhost.localdomain systemd[1]: Started Fail2Ban Service.
Feb 05 11:26:21 localhost.localdomain fail2ban-server[17892]: Server ready
[dash@localhost ~]$ 
```

🌞 **Prouvez que fail2ban est effectif**

- faites-vous ban
```
PS C:\Users\th3dash> ssh dash@192.168.133.129 -p 2222
dash@192.168.133.129's password:
Permission denied, please try again.
dash@192.168.133.129's password:
Permission denied, please try again.
dash@192.168.133.129's password:
dash@192.168.133.129: Permission denied (publickey,gssapi-keyex,gssapi-with-mic,password).
PS C:\Users\th3dash> ssh dash@192.168.133.129 -p 2222
dash@192.168.133.129's password:
Permission denied, please try again.
dash@192.168.133.129's password:
Permission denied, please try again.
dash@192.168.133.129's password:
dash@192.168.133.129: Permission denied (publickey,gssapi-keyex,gssapi-with-mic,password).
PS C:\Users\th3dash> ssh dash@192.168.133.129 -p 2222
dash@192.168.133.129's password:
Permission denied, please try again.
dash@192.168.133.129's password:
ssh_dispatch_run_fatal: Connection to 192.168.133.129 port 2222: Connection timed out
PS C:\Users\th3dash>
```
- montrez l'état de la jail fail2ban pour voir quelles IP sont ban
```
[dash@localhost ~]$ sudo fail2ban-client status sshd
Status for the jail: sshd
|- Filter
|  |- Currently failed:	0
|  |- Total failed:	7
|  `- Journal matches:	_SYSTEMD_UNIT=sshd.service + _COMM=sshd
`- Actions
   |- Currently banned:	1
   |- Total banned:	1
   `- Banned IP list:	192.168.133.1
[dash@localhost ~]$ 
```
- levez le ban avec une commande adaptée
```
[dash@localhost ~]$ sudo fail2ban-client set sshd unbanip 192.168.133.1
1
[dash@localhost ~]$ sudo fail2ban-client status sshd
Status for the jail: sshd
|- Filter
|  |- Currently failed:	0
|  |- Total failed:	7
|  `- Journal matches:	_SYSTEMD_UNIT=sshd.service + _COMM=sshd
`- Actions
   |- Currently banned:	0
   |- Total banned:	1
   `- Banned IP list:	
[dash@localhost ~]$ 
```

