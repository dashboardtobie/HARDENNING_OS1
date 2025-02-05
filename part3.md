# Part III : Networking

## 1. Listening ports

🌞 **Déterminer la liste des programmes qui écoutent sur port TCP**
```
[dash@localhost ~]$ sudo ss -tulnp | grep -v udp
Netid State  Recv-Q Send-Q Local Address:Port Peer Address:PortProcess                          
tcp   LISTEN 0      128          0.0.0.0:22        0.0.0.0:*    users:(("sshd",pid=814,fd=3))   
tcp   LISTEN 0      128             [::]:22           [::]:*    users:(("sshd",pid=814,fd=4))   
```

🌞 **Déterminer la liste des programmes qui écoutent sur port UDP**
```
[dash@localhost ~]$ sudo ss -ulnp
State            Recv-Q           Send-Q                     Local Address:Port                     Peer Address:Port          Process                                    
UNCONN           0                0                              127.0.0.1:323                           0.0.0.0:*              users:(("chronyd",pid=761,fd=5))          
UNCONN           0                0                                  [::1]:323                              [::]:*              users:(("chronyd",pid=761,fd=6))          
```
## 2. Firewalling

🌞 **Pour chacun des ports précédemment repérés...**

- montrez qu'il existe une règle firewall qui autorise le trafic entrant sur ce port
- ou pas ?
```
[dash@localhost ~]$ sudo nft list ruleset | grep "22"
		tcp dport 22 accept
[dash@localhost ~]$ sudo nft list ruleset | grep "323"
[dash@localhost ~]$
```

🌞 **Fermez tous les ports inutilement ouverts dans le firewall**

- principe du moindre privilège encore et encore !
- pas besoin qu'un port soit ouvert si aucun service n'écoute dessus
```
[dash@localhost ~]$ sudo iptables -A INPUT -p udp --dport 323 -j DROP
[dash@localhost ~]$ sudo iptables -L INPUT -v -n | grep 323
    0     0 DROP       17   --  *      *       0.0.0.0/0            0.0.0.0/0            udp dpt:323
```

🌞 **Pour toutes les applications qui sont en écoute sur TOUTES les adresses IP**

- dans Linux, ce sont les applications qui écoutent sur la pseudo-adresse IP `0.0.0.0` : ça signifie que toutes les adresses IP de la machine sont concernées
```
[dash@localhost ~]$ ss -tulnp 
tcp           LISTEN         0              128                          0.0.0.0:22                        0.0.0.0:*             users:(("sshd",pid=814,fd=3))            
tcp           LISTEN         0              128                             [::]:22                           [::]:*             users:(("sshd",pid=814,fd=4))            
```
- modifier la configuration de l'application pour n'écouter que une seule IP : celle qui est nécessaire
```
 On ajoute ajoute au fichier l'option `ListenAddress 192.168.133.129`
[dash@localhost ~]$ sudo nano /etc/ssh/sshd_config
[dash@localhost ~]$ sudo systemctl restart sshd
```
