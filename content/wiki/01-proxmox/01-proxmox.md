---
title: "Proxmox"
date: 2023-03-30T11:29:58+02:00
---

# Proxmox Installation

Questa guida delinea come installare Proxmo su un nuovo nodo del cluster. Il nodo
prenderà nome `pve{n}`.

## Resettare il BIOS

Togliere la batteria CMOS, attendere 30 sec e rimetterla.
Su alcuni modelli, è necessario cortocircuitare i due pin dove viene inserita
la batteria o mettere un jumper altrove sulla scheda madre (tutto a alimentatore
staccato). Consultate il manuale della specifica scheda madre per le istruzioni.

## Impostazioni del bios

La posizione di queste opzioni nel bios varia da macchina a macchina. Per i
computer

- **Settings > Restore defaults**
- **Settings > Advanced > Power Management Set UP > Restore After AC Power Loss = (Power On)**
- **Boot mode = UEFI**
- **Exit > Save and Exit**

## Installazione di Proxmox

Configurazione generale:

- Keyboard Layout: U.S. English
- Mail: admstaff at cs.unibo.it
- Password: (la stessa degli altri PVE)

Configurazione del network:

- FQDN: `pve{n}.students.cs.unibo.it`
- IP: `130.136.3.{2+n}/24`
- Gateway: `130.136.3.254`
- DNS: `130.136.1.110 1.1.1.1`

## Post installazione

1. Entrare come root

2. Per verificare il collegamento ad internet si possono pingare altri nodi del
   cluster e macchine esterne (ad esempio `1.1.1.1`).

3. Verificare il funzionamento dell'interfaccia web da una vostra macchina: per
   collegarsi bisogna fare un forward SSH della porta 8006 sul PVE. Si può usare
   il comando:

   ```
   $ ssh -L 8006:localhost:8006 root@130.136.3.{2+n}
   ```

   Sulla vostra macchina, navigando a [https://localhost:8006](https://localhost:8006)
   dovreste vedere il login di Proxmox.

4. Bisogna eliminare la mirror di Proxmox a pagamento e il messaggio che ricorda
   di pagare l'abbonamento ogni login. Possiamo usare [proxmox-nag](https://github.com/foundObjects/pve-nag-buster).
   Seguire la guida sulla loro repository per come installarlo. Dovrebbe bastare un:

   ```
   $ bash <(curl -s https://raw.githubusercontent.com/foundObjects/pve-nag-buster/master/install.sh)
   ```

5. Fare un aggiornamento iniziale del sistema con:
   ```
   $ apt update
   $ sudo apt upgrade
   ```

## Inserire il nuovo nodo nel cluster

Per inserire il nuovo `pve{n}` nel cluster si deve lanciare il comando (sul nuovo nodo):

```
$ pvecm add <ip> --use_ssh 1
```

Dove `<ip>` è l'indirizzo di un nodo già presente nel cluster, ad esempio pve1: `130.136.3.3`

NOTA: la metà dei nodi del cluster dev'essere online per poter procedere con l'aggiunta.

# Proxmox Management

## ID Policy

Le macchine virtuali su proxmox vengono identificate tramite degli id numerici a 9 cifre.

Gli di per le nuove macchine devono rispettare questo schema (partendo da sinistra):

| Posizione cifra | Descrizione | Note |
|--|--|--|
| 0-1 | Critical level della macchina virtuale | Il valore 10 ha precedenza sul valore 90. <br> _Fare riferimento alla tabella sottostante per impostare correttamente il livello_|
| 2 | _\<unused\>_ ||
| 3 | Categoria della vm | <br> _Fare riferimento alla tabella sottostante per impostare correttamente la categoria_ |
| 4-5 | LDAP uidNumber dell'utente possessore della vm | _Fare riferimento alla wiki di ldap_ |
| 6-7-8 | Ultimo otteto dell'indirizzo IP della macchina| Fare riferimento a `netbox.students.cs.unibo.it` per scegliere un ip libero |

### Critical Level

| Valore | Descrizione |
|--|--|
| 10 | Riservato alle vm per i servizi principali di admstaff | 
| 20 | Riservato ai progetti di csunibo ospitati sui server di admstaff |
| 30 | Riservato ai progetti degli studenti ospitati sui server di admstaff |
| 40 | Kubernetes Cluster |
| 70 | Altro |
| 90 | Riservato ai templates | 

> Se non sai cosa scegliere probabilmente il livello 70 "Altro" fa per te.

### Categorie

| Categoira | Valore |
|--|--|
| Networking | 1 |
| General Purpose | 2 |
| Testing | 3 |
| Templates | 4 |

> Un possibile ID può dunque essere: 100102050 ovvero una vm per il networking creata dall'utente fil con ip 130.136.3.050


## Create Virtual Machine

Per creare una virtual machine bisogna clonare uno dei template presenti identificati dall'id `900x` (attualmente in pve1).
E' necessario cambiare l'ip della macchina una volta clonata. Nella sezione `Cloud-Init` della vm modificare la sezione `IP Config`.
> Per scegliere un ip valido e non già utilizzato utilizzare `netbox.students.cs.unibo.it`