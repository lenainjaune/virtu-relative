RO en attente de la fin de migration

















































# virtu-relative
TODO : étendre aux systèmes physiques

Remarque : ce qui suit est aussi vrai pour les hôtes physiques, mais comme je contruis plus de VMs, j'ai choisi de consigner mes notes ici.
voir aussi : https://github.com/lenainjaune/libvirt
## Linux VM
L'expérience a montré que certains paquets sont incontournables :
```sh
# TODO : ajouter numlockx (pour verrouiller numpad AVANT login) ?
root@host:~# apt install -y vim htop locate less aptitude wget gawk man sshfs rsync tree curl net-tools gnupg2 rfkill util-linux nmap tcpdump binutils screen pv ntfs-3g
```
Pour compiler :
```sh
root@host:~# apt install -y build-essential gcc make gettext git
```
voir aussi les paquets qui permettent la résolution par nom [ici](https://github.com/lenainjaune/network_howto#acc%C3%A9s-r%C3%A9seau-par-nom)

## Créer un réseau privé avec passerelle NAT
Voir le ticket glpi/front/ticket.form.php?id=294 en attente de centraliser

## Booter avec un disque formaté en GPT
Pour accéder à un disque au format GPT, il faut booter par un BIOS UEFI, hors virt-manager ne le permet pas.

=> [ce fil](https://blog.wikichoon.com/2016/01/uefi-support-in-virt-install-and-virt.html) indique comment le faire avec la commande **virt-install** et [ce guide](https://www.golinuxcloud.com/virt-install-examples-kvm-virt-commands-linux/) explique comment construire la commande.

Au final :

root@host:~# virt-install --name "vm_name" --boot uefi --vcpus 2 --memory 8192 --os-type=Windows --network bridge=br0 --graphics=vnc --disk path=/path/vhd.qcow2 -v

Note : avant de comprendre pourquoi, il est impératif d'exécuter cette commande en **root**, sinon on aura une erreur "ERROR internal error: /usr/lib/qemu/qemu-bridge-helper --br=br0 --fd=33: failed to communicate with bridge helper: Transport endpoint is not connected
stderr=failed to parse default acl file `/etc/qemu/bridge.conf'" qui est lié à des droits

## Simuler un disque IDE
Avertissement : je ne maitrise pas ce qui suit, mais ça a marché dans mon cas.

De ce que je comprends, depuis qemu (?libvirt?) v6.0 on n'a plus par défaut la possibilité de choisir un HD IDE connecté à un bus IDE. 

J'ai eu le cas en [p1v](https://github.com/lenainjaune/p2v_vs_v2p_vs_p1v) depuis un notebook Samsung N140 avec un système Windows 7 Starter sur un disque physiquement connecté en IDE (BIOS date de 2009) virtuellement connecté en SATA qui se soldait au démarrage par un BSOD. Le conflit SATA vs IDE me semblait évident aussi j'ai cherché comment connecter le pHD par p1v a un vbus IDE. 

Les recherches ont révélées la cause probable qui viendrait du type de machine utilisé par la VM (voir dessous "Types de machines"). Par défaut dans les versions récentes de Qemu c'est **q35** or ce chipset ne supporte pas IDE ([source](https://github.com/dmacvicar/terraform-provider-libvirt/issues/667)).

Voici la balise **os** de la configuration du domaine que je teste :
```
root@goku:~# virt-install --boot hd --name "mon_domaine" ...
root@host:~# virsh edit mon_domaine
...
  <os>
    <type arch="x86_64" machine="pc-q35-5.2">hvm</type>
  </os>
...
```
=> ici BSOD et on voit que le type est q35


[Ce lien](https://bugzilla.redhat.com/show_bug.cgi?id=1437253#c5) indique que le chipset **I440FX** gère l'IDE à la différence de **Q35** et [ici](https://serverfault.com/questions/637917/how-can-i-change-qemu-kvm-machine-architecture-from-440fx-to-q35-with-virsh-edit) j'ai le nom complet du type de machine **pc-i440fx-2.1** que j'ai tout de suite testé (maintenant je sais lesquels je peux utiliser - voir "Types de machines") :
```
root@goku:~# virt-install --machine=pc-i440fx-2.1 --name "mon_domaine" ...
root@host:~# virsh edit mon_domaine
...
  <os>
    <type arch="x86_64" machine="pc-i440fx-2.1">hvm</type>
    <boot dev="hd"/>
  </os>
...
```
 => il y aura un bus IDE et il est possible de connecter des vHDs en IDE ; d'autre part on voit bien que le type est i4440fx.
 
On peut aussi le faire depuis **virt-manager** (donc sans passer par virt-install en CLI) en choisissant le système d'exploitation **Generic OS (generic)** lors de la création de la VM ([source](https://mangolassi.it/topic/19099/virt-manager-ide-disks/58?lang=fr)). Le type de machine sera aussi I440FX. Donc ça semble identique.

## Types de machines
Pour le paramètre **--machine**, la doc de [virt-install](https://manpages.org/virt-install) n'indique pas comment lister les valeurs supportées.

D'après ce que je comprends, ce paramètre indique le chipset de la VM.

```
root@goku:~# kvm -machine help
Supported machines are:
microvm              microvm (i386)
xenfv-4.2            Xen Fully-virtualized PC
xenfv                Xen Fully-virtualized PC (alias of xenfv-3.1)
xenfv-3.1            Xen Fully-virtualized PC
pc                   Standard PC (i440FX + PIIX, 1996) (alias of pc-i440fx-5.2)
pc-i440fx-X.Y        Standard PC (i440FX + PIIX, 1996)
...
pc-X.Y               Standard PC (i440FX + PIIX, 1996) (deprecated)
...
q35                  Standard PC (Q35 + ICH9, 2009) (alias of pc-q35-5.2)
pc-q35-X.Y           Standard PC (Q35 + ICH9, 2009)
...
...
isapc                ISA-only PC
none                 empty machine
xenpv                Xen Para-virtualized PC
```
# Réduire automatiquement la taille d'un disque QCOW2
[source](https://tuxfixer.com/how-to-shrink-openstack-qcow2-image-with-qemu-img/)
```sh
root@host:~# qemu-img convert -O qcow2 vm-bullseye.qcow2 vm-bullseye_shrinked.qcow2
```
# Windows VM
## Windows XP
Quand on active le Bureau à distance, j'ai constaté que parfois on ne peut pas se connecter tout de suite après le reboot. C'est juste que des fois il met plus de temps.

Apparemment le service concerné n'est pas **TermService** (démarré et en manuel par défaut) mais je n'en suis pas sûr.

TODO : trouver le service ou le programme concerné

J'ai également cherché une solution pour faire en sorte qu'un service soit exécuté en 1er mais je n'ai pas trouvé (recherche : "windows xp" make a service run first).

Service en démarrage auto par CLI ([source](https://www.windows-commandline.com/start-terminal-services-command-line/)) :
```bash
sc config TermService start= auto
```
