# virtu-relative
TODO : étendre aux systèmes physiques

Remarque : ce qui suit est aussi vrai pour les hôtes physiques, mais comme je contruis plus de VMs, j'ai choisi de consigner mes notes ici.
voir aussi : https://github.com/lenainjaune/libvirt
## Linux VM
L'expérience a montré que certains paquets sont incontournables :
```sh
# TODO : ajouter numlockx (pour verrouiller numpad AVANT login) ?
root@host:~# apt install -y vim htop locate less aptitude wget gawk man sshfs rsync tree curl net-tools gnupg2 rfkill util-linux nmap tcpdump
```
Pour compiler :
```sh
root@host:~# apt install -y build-essential
```
voir aussi les paquets de qui permettent la résolution par nom [ici](https://github.com/lenainjaune/network_howto#acc%C3%A9s-r%C3%A9seau-par-nom)

## Créer un réseau privé avec passerelle NAT
Voir le ticket glpi/front/ticket.form.php?id=294 en attente de centraliser
