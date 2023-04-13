---
title: "Storage"
date: 2023-03-30T11:37:38+02:00
---

# Download del Software

Il software per il controllo dell'unità di storage è pensato per essere eseguito
su CentOS 6.3, che potete ancora torvare in giro online. Una volta installato
il sistema su una macchina nella stessa sottorete del server storage (è
consigliato usare una VM su proxmox) potete passare all'installazione del
software dal sito della DELL.

Su [questa pagina](https://www.dell.com/support/home/en-us/product-support/product/powervault-md3200i/drivers)
si trovano tutti gli aggiornamenti e strumenti per il Dell PowerVault serie MD3200.
In particolare, il software di management si trova nel cosiddetto "Resource DVD".
Questa è una foto di come risulta la pagina al momento della stesura:

![Screenshot della pagina del sito della DELL](screenshot-dell.png)

Una volta scaricata l'immagine e montata sulla macchina, ad esempio con:

```
$ mount -o loop file.iso /mnt
```

potete avviare (come `root`) il programma d'installazione. Una volta completata
la procedura troverete i software installati in `/opt/dell`, in particolare il
manager in `/opt/dell/mdstoragemanager/mdstoragemanager/client`.
Qui dentro, troverete il programma chiamato `SMclient`, che va eseguito come root.

## Configuare un nodo per accedere allo storage

È necessario collegarsi da proxmox alla vm con sopra il programma Dell per lo storage.
Lo storage si trova nella sottorete 130.136.0.100/24. Per accedere è quindi necessario
modificare il proprio ip.

```
$ sudo ip a add 130.136.0.{random}/24 dev eth0
```
Dove __random__ è un ip scelto a caso non usato da altri nodi o dal server. In 
particolare il server usa dal 100 al 105, mentre i nodi pve sono da 11 al 15. È
anche necessario settare `eth0` in modo che non sia down:
```
$ ip link set eth0 up
```
Per vedere se il cambio di ip funziona basta pingare lo storage `$ ping 130.136.0.100`.

Per avviare il software dell è necessario essere root e dare il comando
```
$ ./opt/dell/mdstoragemanager/mdstoragemanager/client/SMclient
```
Controllare che lo storage sul software dell abbia ip `130.136.0.100` se non fosse
eliminarlo e aggiungerlo nuovamente. Per farlo andare in "__Edit->Add storage array__"
e aggiungere gli ip:
```
130.136.0.100
130.136.0.101
```
Il messaggio __Attention needed__ va bene. Semplicemente indica un errore sulla 
batteria. (?)

È necessario ora entrare in ssh sulla vm che si vuole collegare allo storage. Poi bisogna
aggiungere al file `/etc/network/interfaces`

```
iface vmbr0:0 inet static
       address 130.136.0.1{n}/24 
```
Dove `{n}` è il numero del nodo (es pv4 n = 4). Per aggiornare `$ ifup -a`.

Bisogna contattare con il protocollo iscsi per avere la lista dei dischi sul nodo.
```
$ pvesm scan iscsi 130.136.0.102
```
Si noti che `130.136.0.102` è Semplicemente un indirizzo dello storage, ma andrebbe
bene un qualsiasi altro indirizzo tanto sono tutti collegati.

Installare i seguenti pacchetti:
```
$ apt install multipath-tools open-iscsi
```

Vedere il codice in fondo al file:
```
$  cat /etc/iscsi/initiatorname.iscsi
```

Aggiungere sul software dell:
"Configure->Configure hosts access manual". Name: pv{n}. Il codice è quello trovato con 
il comando `cat` sopra citato. "Yes, this host will share...". "Select exixting group->proxmox".

Copiare il seguente file da un nodo in cui è già presente: (Lasciato anche di seguito)
```
$ scp 130.136.3.3:/etc/multipath.conf /etc/multipath.conf
```

```
$ cat /etc/multipath.conf
```

[multipath file](multipath.conf)

Controllare il codice dopo il `wwid` e inserirlo al posto di `{code}` nel seguente comando
```
$ multipath -a {code}
```
Fare un `reboot` e vedere se i dischi si vedono con
```
$ lsblk
$ multipath -ll
```
