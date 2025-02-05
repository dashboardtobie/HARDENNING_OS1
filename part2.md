# Part II : Files and permissions

## 1. Listing POSIX permissions

🌞 **Déterminer les permissions des fichiers/dossiers...**

- le fichier qui contient la liste des utilisateurs
```
[dash@localhost ~]$ ls -l /etc/passwd
-rw-r--r--. 1 root root 1051 Feb  3 11:58 /etc/passwd
```
- le fichier qui contient la liste des hashes des mots de passe des utilisateurs
```
[dash@localhost ~]$ ls -l /etc/shadow
----------. 1 root root 989 Feb  3 12:29 /etc/shadow
```
- le fichier de configuration du serveur OpenSSH
```
[dash@localhost ~]$ ls -l /etc/ssh/sshd_config
-rw-------. 1 root root 3667 Apr 18  2024 /etc/ssh/sshd_config
```
- le répertoire personnel de l'utilisateur `root`
```
[dash@localhost ~]$ sudo ls -ld /root
dr-xr-x---. 3 root root 179 Feb  3 15:20 /root
```
- le répertoire personnel de votre utilisateur
```
[dash@localhost ~]$ ls -ld ~
drwx------. 2 dash dash 83 Feb  3 10:57 /home/dash
```
- le programme `ls`
```
[dash@localhost ~]$ ls -l /bin/ls
-rwxr-xr-x. 1 root root 140872 Apr 20  2024 /bin/ls
```
- le programme `systemctl`
```
[dash@localhost ~]$ ls -l /bin/systemctl
-rwxr-xr-x. 1 root root 305680 Apr  8  2024 /bin/systemctl
```

## 2. Extended attributes

🌞 **Lister tous les programmes qui ont le bit SUID activé**
```
[dash@localhost ~]$ find / -perm -4000 -type f 2>/dev/null
/usr/bin/chage
/usr/bin/gpasswd
/usr/bin/newgrp
/usr/bin/mount
/usr/bin/umount
/usr/bin/su
/usr/bin/crontab
/usr/bin/passwd
/usr/bin/sudo
/usr/sbin/unix_chkpwd
/usr/sbin/pam_timestamp_check
/usr/sbin/grub2-set-bootflag
```

🌞 **Rendre le fichier de configuration du serveur OpenSSH immuable**

- ça se fait avec les attributs étendus
- "immuable" ça veut dire qu'il ne peut plus être modifié DU TOUT : il est donc en read-only
```
[dash@localhost ~]$ sudo lsattr /etc/ssh/sshd_config
---------------------- /etc/ssh/sshd_config
[dash@localhost ~]$ sudo chattr +i /etc/ssh/sshd_config
```
- prouvez que le fichier ne peut plus être modifié
```
[dash@localhost ~]$ sudo lsattr /etc/ssh/sshd_config
----i----------------- /etc/ssh/sshd_config
[dash@localhost ~]$ sudo rm /etc/ssh/sshd_config
rm: cannot remove '/etc/ssh/sshd_config': Operation not permitted
```

## 3. Protect a file using permissions

🌞 **Restreindre l'accès à un fichier personnel**

- créer un fichier nommé `dont_readme.txt` (avec le contenu de votre choix)
- il doit se trouver dans un dossier lisible et écrivable par tout le monde
```
[dash@localhost ~]$ mkdir /tmp/folder
[dash@localhost ~]$ chmod 777 /tmp/folder
[dash@localhost ~]$ touch /tmp/folder/dont_readme.txt
```
- faites en sorte que seul votre utilisateur (pas votre groupe) puisse lire ou modifier ce fichier
- personne ne doit pouvoir l'exécuter
```
[dash@localhost ~]$ chmod 600 /tmp/folder/dont_readme.txt 
```
- prouvez que :
  - votre utilisateur peut le lire
  ```
  [dash@localhost ~]$ cat /tmp/folder/dont_readme.txt 
  bonjour
  ```
  - votre utilisateur peut le modifier
  ```
  [dash@localhost ~]$ echo "yo man" >> /tmp/folder/dont_readme.txt 
  [dash@localhost ~]$ cat /tmp/folder/dont_readme.txt 
  bonjour
  yo man
  ```
  - l'utilisateur `meow` ne peut pas y toucher
  ```
  [dash@localhost ~]$ sudo su -s /bin/bash - meow
  Last login: Mon Feb  3 15:35:05 CET 2025 on pts/0
  su: warning: cannot change directory to /home/meow: No such file or directory
  [meow@localhost dash]$ cat /tmp/folder/dont_readme.txt 
  cat: /tmp/folder/dont_readme.txt: Permission denied
  ```
  - l'utilisateur `root` peut quand même y toucher
  ```
  [root@localhost ~]# echo "dash" >> /tmp/folder/dont_readme.txt 
  [root@localhost ~]# cat /tmp/folder/dont_readme.txt 
  bonjour
  yo man
  dash
  ```

