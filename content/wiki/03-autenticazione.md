---
title: "Autenticazione"
date: 2023-05-11T11:51:00+02:00
---

Il bastione, `saragozza`, e tutti i nodi di proxmox (`pve{n}`) consentono
l'accesso tramite il sistema LDAP interno, sia tramite password che tamite
chiavi ssh. L'utente `root` su tutte queste macchine ha a disposizione le chiavi
ssh di tutti gli utenti disponibili.

Le macchine di proxmox inoltre non sono accedibili in alcun modo se non
passando tramite il bastione. Di conseguenza, è sempre necessario fare SSH Jump
a `saragozza` per collegarsi ad un nodo.
Possiamo dare degli alias alle macchine usando la SSH Config. In questo modo,
ci possiamo collegare facendo jump usando un nome simobilico (i.e. `pve1`).
Oltretutto, la config specifica anche una serie di `LocalForward` per tutte le
porte dei servizi presenti su `saragozza` e i vari `pve{n}`.

Questo è un esempio di una config SSH consigliata.

```
Host adm-saragozza
  HostName 130.136.3.2
  LocalForward 8200 localhost:8200
  LocalForward 3000 localhost:3000
  User user

Host adm-pve1
  ProxyJump adm-saragozza
  HostName 130.136.3.3
  User user
  LocalForward 8006 localhost:8006

Host adm-pve2
  ProxyJump adm-saragozza
  HostName 130.136.3.4
  User user
  LocalForward 8006 localhost:8006

Host adm-pve3
  ProxyJump adm-saragozza
  HostName 130.136.3.5
  User user
  LocalForward 8006 localhost:8006

Host adm-pve4
  ProxyJump adm-saragozza
  HostName 130.136.3.6
  User user
  LocalForward 8006 localhost:8006

Host adm-pve5
  ProxyJump adm-saragozza
  HostName 130.136.3.7
  User user
  LocalForward 8006 localhost:8006
```

Bisogna ricordarsi di sostituire ogni occorrenza di `user` con il nome del
proprio utente su LDAP.
Ad esempio, se il mio nome utente è `luca`, posso usare:

```sh
$ sed -i 's/user/luca/g' ~/.ssh/config
```
