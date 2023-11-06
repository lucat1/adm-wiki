---
title: "LDAP"
date: 2023-11-06T11:40+01:00
---

# Overview

L'implementazione di ldap usata è [Slapd](https://en.wikipedia.org/wiki/OpenLDAP)

# Configuration

Gli utenti sono salvati direttamente nel file ansible che configura slapd (presente nella repo accessibile solo ai membri di amdstaff).
Le password sono salvate tramite hashing con uno script (pwgen.pr) sempre presente nella repo di admstaff.

### N.B
> Per la sincronizzazione degli utenti su Proxmox e necessario accedere all'interfaccia di Proxmox e nella sezione Datacenter/Permissions/Realms
selezionare saragozza.students.cs.unibo.it e cliccare il tasto Sync
Questo permette all'utente di accedere ma non potrà eseguire nessuna azione. E' necessario spostarsi nella sezione Users, modificare l'utente
selezionato con il tasto Edit e aggiungerlo al gruppo `posix_adm-saragozza.students.cs.unibo.it`

# Network

I servizi collegati all'autenticazione LDAP sono:
- Vault
- Proxmox
- Netbox