# Part I : User management


## 1. Existing users

🌞 **Déterminer l'existant :**

- lister tous les utilisateurs créés sur la machine
```
[dash@localhost ~]$ compgen -u
root
bin
daemon
adm
lp
sync
shutdown

<snip>
```
- lister tous les groupes d'utilisateur
```
[dash@localhost ~]$ cat /etc/group
root:x:0:
bin:x:1:
daemon:x:2:
sys:x:3:
adm:x:4:
tty:x:5:
disk:x:6:
lp:x:7:
mem:x:8:
kmem:x:9:
wheel:x:10:it4

<snip>
```
- déterminer la liste des groupes dans lesquels se trouvent votre utilisateur
```
[dash@localhost ~]$ id -Gn dash
dash wheel
```

🌞 **Lister tous les processus qui sont actuellement en cours d'exécution, lancés par `root`**
```
[dash@localhost ~]$ ps -aux | grep "^root"
root           1  0.0  3.4 173220 15980 ?        Ss   10:38   0:00 /usr/lib/systemd/systemd --switched-root --system --deserialize 31
root           2  0.0  0.0      0     0 ?        S    10:38   0:00 [kthreadd]
root           3  0.0  0.0      0     0 ?        I<   10:38   0:00 [rcu_gp]
root           4  0.0  0.0      0     0 ?        I<   10:38   0:00 [rcu_par_gp]
root           5  0.0  0.0      0     0 ?        I<   10:38   0:00 [slub_flushwq]
root           6  0.0  0.0      0     0 ?        I<   10:38   0:00 [netns]
root           8  0.0  0.0      0     0 ?        I<   10:38   0:00 [kworker/0:0H-events_highpri]

<snip>
```

🌞 **Lister tous les processus qui sont actuellement en cours d'exécution, lancés par votre utilisateur**
```
[dash@localhost ~]$ ps -aux | grep "^dash"
dash         922  0.0  2.9  23808 13676 ?        Ss   10:39   0:00 /usr/lib/systemd/systemd --user
dash         924  0.0  1.5 108936  7204 ?        S    10:39   0:00 (sd-pam)
dash         931  0.0  0.8   7444  4096 tty1     Ss+  10:39   0:00 -bash
dash        1086  0.0  1.5  20328  7116 ?        S    10:58   0:00 sshd: dash@pts/0
dash        1087  0.0  0.9   7444  4224 pts/0    Ss   10:58   0:00 -bash
dash        1144  0.0  1.3  19376  6272 pts/0    R+   11:06   0:00 ps -aux
dash        1145  0.0  0.4   6408  2176 pts/0    S+   11:06   0:00 grep --color=auto ^dash
```
🌞 **Déterminer le hash du mot de passe de `root`**
```
[dash@localhost ~]$ sudo cat /etc/shadow | grep root
[sudo] password for dash: 
root:$6$.8fzl//9C0M819BS$Sw1mrG49Md8cyNUn0Ai0vlthhzuSZpJ/XVfersVmgXDSBrTVchneIWHYHnT3mC/NutmPS03TneWAHihO0NXrj1::0:99999:7:::
```
🌞 **Déterminer le hash du mot de passe de votre utilisateur**
```
[dash@localhost ~]$ sudo cat /etc/shadow | grep dash
dash:$6$KgPkoTD2MjP3E0fx$5e.5t.Ftuq5zZgSVzMXGCwB1Vp2lzvETvp0kAeXUYxbo4azaoqHzqAdj3a/StwIJskTqMYoNffeP4I9F27wN10:20122:0:99999:7:::
```
🌞 **Déterminer la fonction de hachage qui a été utilisée**
```
[dash@localhost ~]$ grep "^ENCRYPT_METHOD" /etc/login.defs 
ENCRYPT_METHOD SHA512
```
🌞 **Déterminer, pour l'utilisateur `root`** :

- son shell par défaut
- le chemin vers son répertoire personnel
```
[dash@localhost ~]$ getent passwd root | cut -d ":" -f 6,7
/root:/bin/bash
```
🌞 **Déterminer, pour votre utilisateur** :

- son shell par défaut
- le chemin vers son répertoire personnel
```
[dash@localhost ~]$ getent passwd dash | cut -d ":" -f 6,7
/home/dash:/bin/bash
```

🌞 **Afficher la ligne de configuration du fichier `sudoers` qui permet à votre utilisateur d'utiliser `sudo`**
```
[dash@localhost ~]$ sudo grep -E '^%wheel' /etc/sudoers
%wheel	ALL=(ALL)	ALL
```


## 2. User creation and configuration

🌞 **Créer un utilisateur :**

- doit s'appeler `meow`
- ne doit appartenir QUE à un groupe nommé `admins`
- ne doit pas avoir de répertoire personnel utilisable
- ne doit pas avoir un shell utilisable
```
[dash@localhost ~]$ sudo groupadd admins
[dash@localhost ~]$ sudo useradd -g admins -M -s /usr/sbin/nologin meow
```

🌞 **Configuration `sudoers`**
> Pour chaque point précédent, c'est une seule ligne de configuration à ajouter dans le fichier `sudoers` de la machine.
> prouvez que ces 3 configurations ont pris effet (vous devez vous authentifier avec le bon utilisateur, et faire une commande `sudo` qui doit fonctioner correctement)
- ajouter une configuration `sudoers` pour que l'utilisateur `meow` puisse exécuter seulement et uniquement les commandes `ls`, `cat`, `less` et `more` en tant que votre utilisateur.  

On ajoute les lignes suivantes dans le fichier `/etc/sudoers`
```
meow ALL=(dash) NOPASSWD: /bin/ls, /bin/cat, /bin/less, /bin/more
```
On teste :  
```
[dash@localhost ~]$ sudo su -s /bin/bash - meow
Last login: Mon Feb  3 12:56:16 CET 2025 on pts/0
su: warning: cannot change directory to /home/meow: No such file or directory
[meow@localhost dash]$ ls
ls: cannot open directory '.': Permission denied
[meow@localhost dash]$ sudo -u dash ls 
[meow@localhost dash]$ 
```
- ajouter une configuration `sudoers` pour que les membres du groupe `admins` puisse exécuter seulement et uniquement la commande `yum` en tant que `root`
```
%admins ALL=(ALL) NOPASSWD: /usr/bin/yum
```
On teste :
```
[meow@localhost dash]$ sudo -u root yum
usage: yum [options] COMMAND

List of Main Commands:

alias                     List or create command aliases

<snip>
```
- ajouter une configuration `sudoers` pour que votre utilisateur puisse exécuter n'importe quel commande en tant `root`, sans avoir besoin de saisir un mot de passe
```
dash ALL=(ALL) NOPASSWD: ALL
```
On teste :
```
[dash@localhost ~]$ sudo ls /root
anaconda-ks.cfg
```



## 3. Hackers gonna hack

🌞 **Déjà une configuration faible ?**

- l'utilisateur `meow` est en réalité complètement `root` sur la machine hein là. Prouvez-le.
```
[dash@localhost ~]$ sudo su -s /bin/bash - meow
Last login: Mon Feb  3 13:12:42 CET 2025 on pts/0
su: warning: cannot change directory to /home/meow: No such file or directory
[meow@localhost dash]$ sudo -u dash less /var/log/dnf.log
```
Une fois dans le fichier on fait un `!/bin/bash` et on reçoit un shell de l'utilisateur dash qui a des droits de root.

- proposez une configuration similaire, sans présenter cette faiblesse de configuration
  - vous pouvez ajouter de la configuration
  - ou supprimer de la configuration
  - du moment qu'on garde des fonctionnalités à peu près équivalentes !

Il faut tout simplement enlever less et more de la config et garder le reste:
```
meow ALL=(dash) NOPASSWD: /bin/ls, /bin/cat
%admins ALL=(ALL) NOPASSWD: /usr/bin/apt
dash ALL=(ALL) NOPASSWD: ALL
```

