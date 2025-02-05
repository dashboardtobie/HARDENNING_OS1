# Part IV : Storage and partitions

**Cette partie est dédiée aux partitions**, quelques subtilités autour, et on en profite pour parler des **options de montage**.

## 1. Existing partitions

🌞 **Déterminer la liste des partitions du système**

- on dit aussi la liste des partitions qui sont actuellement "montées"
```
[dash@localhost ~]$ df -hT
Filesystem          Type      Size  Used Avail Use% Mounted on
devtmpfs            devtmpfs  4.0M     0  4.0M   0% /dev
tmpfs               tmpfs     229M     0  229M   0% /dev/shm
tmpfs               tmpfs      92M  2.9M   89M   4% /run
/dev/mapper/rl-root xfs       6.2G  1.5G  4.8G  24% /
/dev/sda1           xfs       960M  225M  736M  24% /boot
tmpfs               tmpfs      46M     0   46M   0% /run/user/1001
```

🌞 **Identifier la partition qui est montée sur `/`**

- la partition est identifiée par un chemin dans `/dev`
```
[dash@localhost ~]$ df -hT /
Filesystem          Type  Size  Used Avail Use% Mounted on
/dev/mapper/rl-root xfs   6.2G  1.5G  4.8G  24% /
```

## 2. Mount options

🌞 **Déterminer les options de montage de la partition `/`**

- expliquer chaque option de montage configurées pour `/`
```
[dash@localhost ~]$ findmnt -o OPTIONS /
OPTIONS
rw,relatime,seclabel,attr2,inode64,logbufs=8,logbsize=32k,noquota
```

🌞 **Monter une partition de type `tmpfs` sur le dossier `/tmp`**

- il doit être impossible de lancer un programme s'il est stocké sur cette partition
  - ça se fait en spécifiant une option de montage
- prouvez que la modification est effective :
  - je veux voir une copie d'une programme existant dans `/tmp`
  - puis une commande pour lui mettre les full droits pour tout le monde : `777` (ou `rwxrwxrwx`)
  - preuve que votre utilisateur ne peut pas l'exécuter

```
[dash@localhost ~]$ sudo nano tmpfs /etc/fstab
```
On ajoute `tmpfs /tmp tmpfs defaults,noexec,nosuid,nodev 0 0` dans la fichier.
```
[dash@localhost ~]$ sudo mount /tmp
[dash@localhost ~]$ mount | grep '/tmp'
tmpfs on /tmp type tmpfs (rw,nosuid,nodev,noexec,relatime,seclabel,inode64)
[dash@localhost ~]$ cp /bin/ls /tmp/
[dash@localhost ~]$ chmod 777 /tmp/ls
[dash@localhost ~]$ ls -l /tmp/ls
-rwxrwxrwx. 1 dash dash 140872 Feb  3 19:12 /tmp/ls
[dash@localhost ~]$ /tmp/ls
-bash: /tmp/ls: Permission denied
```
