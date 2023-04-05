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
