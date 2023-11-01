---
title: "Storage (NUOVO)"
date: 2023-10-30T13:33+01:00
---

# Overview 

Il nuovo storage è un macchina con **TrueNas Core** e dei dischi (5 per il 
momento) in raid 6 (raid-z2) con **zfs**.

# Connect to the storage

Lo storage ha ip `130.136.0.200` e `130.136.0.201` avendo due interfacce di 
rete. Per raggiungerlo è necessario collegarsi ad un pve, che dovrebbe avere un 
ip nella sottorete `130.136.0.0/24`, e fare un forward locale della porta
80. Si può usare il seguente comando:
```bash
ssh adm-pve1 -L 8080:130.136.0.200:80
```
Successivamente per entrare nell'interfaccia web di TrueNas basta aprire un 
browser e digitare `localhost:8080` nella barra di ricerca.

# NFS share

Lo storage viene condiviso in rete tramite NFS. È possibile visionare le opzioni
nel menù `sharing -> Unix Share (NFS)`. Si raccomanda di dare accesso ad un 
particolare filesystem di zfs (spiegato nella parte dopo) solo agli ip 
interessati. Per montarlo sull'ip interessato invece basta utilizzare un `mount`
di Linux.

# Zfs pool

I 5 dischi persenti nello storage sono in raid 6, o più propriamente, in raid-z2.
Il raid è fatto grazie Zfs. In particolare l'opzione raind-z2 utilizza due 
dischi per il controllo della parità, permettendo quindi che 2 qualsiasi dischi
muoiano prima di perdere dati.

È presente in oltre un'ssd da 500GB partizionata manualmente in due: 
La prima partizione di 20GB è dedicata allo *slog*, mentre tutto il resto dello
spazio rimanente è dedicato alla cache *l2arc*.

Il nome della pool è banalmente `pool0` e si possono visionare le opzioni al 
riguardo nel menù `Storage -> Pools`.

Una *pool* in zfs è un insieme di dischi in una qualche forma di raid. I 5 
dischi presenti nello storage sono tutti contenuti nella stessa pool. Sulla pool
è possibile creare dei *filesystem* con zfs che non sono altro che spazi 
dedicati sulla pool in cui si possono settare delle quote.

## Status

Lo stato della pool si può comodamente vedere dall'apposito menù 
nell'interfaccia grafica `Storage -> Pool`. In alto al centro dovrebbe esserci
scritto *ONLINE*. Questo significa che la pool è funzionante e non ci sono 
problemi. Se al posto di ONLINE ci fosse scritto altro bisogna prendere le 
dovute misure per ripotare la pool al suo stato normale.

È inoltre possibile visionare lo stato dei singoli dischi che formano la pool
attraverso il menù `Status` accessibile tramite la rotella a destra sotto il 
pulsante ADD.

## Quotas

Grazie a zfs è possibile creare dei *filesystem* all'interno della pool che sono
semplicemente delle cartelle con una quota. Dovrebbe essere già presente una 
filesystem di nome `proxmox` che è dedicato ai dischi di tutte le vm sul 
cluster.

### Create a new filesystem with quota

Per creare un filesystem è possibile usare i tre puntini a estra del nome della
pool per accedere al menù `Add dataset`. Per inserire una quota è necessario
recarsi nella opzioni avanzate e inserire un valore per il capo `Quota for this 
dataset`.

### Expand a filesystem

Per espandere un filesystem già presente è necessario cliccare sui 3 puntini a
destra del suo nome, andare in `Edit options` e modificare il paramentro `Quota
for this dataset`.

## Configuration

Finchè si tratta di aggiungere dischi è tutto molto semplice perché 
l'interfaccia grafica permette di farlo. Nonostante ciò l'interfaccia grafica 
non permette di avere il pieno controllo. Per esempio non è possibile 
parizionare l'SSD in modo da utilizzare solo una piccola parte per lo slog e 
tutto il resto per l2arc. Per fare cioè quindi è necessario passare della shell.

### How to use the shell

Per acceder alla shell basta cliccare sulla sezione apposità del menù a destra.
La difficoltà sta che TrueNas Core è basato su FreeBSD e non Linux. La scelta di
utilizzare un sistema basato su FreeBSD risiede nel fatto che Zfs non è 
nativamente supportato nel kernel Linux, mentre è nativamente supportato in 
FreeBSD.

Tutti i comandi di zfs funzionano ovviamente:
```bash
zpool status        # Controllare lo stato della pool
zpool list
zpool iostat -v     # Controllare utilizzo cache
arc_summary         # Statistiche sulla l2arc
# ecc.
```

### Disks, gptid and partitions

Per fare una lista dei dischi su FreeBSD è possibile utilizzare il comando:
```bash
geom disk list
```

Per controllare lo schema delle partizione invece è possibile usare il comando
```bash
gpar show
```

### How to configure the ssd

Una volta individuata a quale numero corrisponde l'ssd è possibile partizionarlo
con il comando:
```bash
gpart add -t freebsd-zfs -s 20G ada{n}
```
Dove `{n}` è il numero del device dell'ssd.

In particolare la cofigurazione attuale è data dai seguenti due comandi:
```bash
gpart add -t freebsd-zfs -s 20G ada{n}
gpart add -t freebsd-zfs ada{n}
```
Il secondo, non avendo un size specificato, occupa tutto lo spazio rimanente con
la partizione che si crea. Forse non è necessario specificare il tipo tanto zfs
lo sovrascrive.

Per inserire le partizione come slog e cache in zfs bisogna innanzitutto 
indentificarle univocamente con il comando:
```bash
glabel status
```
Il codice univoco con cui riferirsi alle partizioni si trova sulla sinistra, 
mentre il nome più classico sulla destra. I nomi sulla destra sono formati dal
nome del disco (ada{n}) più il numero della partizione. Per esempio la seconda
partizione del disco 5 risulta essere `ada5p2`.

Una volta identificati univocamente le partizioni è possibile aggiungerle alla 
pool:
```bash
zpool add {{ pool_name }} {{ log | cache }} {{ codice_univoco }}
```

In particolare i comandi specifici saranno simili a:
```bash
zpool add pool0 log gptid/8d780d3b3-7713-11ee-bfea-001517d9ecd5
zpool add pool0 cache gptid/937f9a33-7713-11ee-bfea-001517d9ecd5
```

Per controllare che l'aggiunta sia stat efficace è possibile usare il comando
```bash
zpool status
```

## Scrub and smart

Per controllare che i dati non si siano corrotti all'insaputa di zfs è necesario
eseguire periodicamente un scrub della pool. In pratica lo scrub controlla dato
per dato se è ancora tutto integra. Questo controllo rallenta i dischi ed è 
quindi consigliato eseguirlo durante le ore notturne.

In particolare TrueNas mette a disposizione in automatico delle Tasks, che si 
trovano nell'apposito menù di sinistra, che permettono di eseguire lo scrub e i
controlli SMART periodicamente.

Al momento lo scub è programmato per ogni domentica alle 00:00.

I controlli SMART sono usati per lo stato dei dischi. Nonostante infatti un 
disco posso continuare a funzionare per zfs, magari ha molti settori rotti che
rallentano le sue prestazioni. I controlli SMART servono appunto per 
identificare i dischi con molti settori rotti o con troppi errori interni di 
lettura. **Spesso se le prestazioni della pool calano è perché un disco ha 
troppi settori danneggiati ed è necessario sostituirlo**.

## Replace a disk

Il rimpiazzamento di disco dovrebbe essere possibile anche dall'interfaccia 
grafica sotto il menù `Status` che si trova cliccando la rotella di fiacno al 
nome della pool nell menù `Storage -> Pools`.

I dischi possono essere rimossi senza spegnere la macchina. In particolare lo 
storage attual ha quattro dischi frontali e uno laterale. Per quelli frontali
sono previsti dei led che indicano le operazioni del disco. È quindi possibile 
capire a quale disco fisico un determinato `ada{n}` è associato semplicemente
leggendolo e vedono quale led rimane sempre acceso:
```bash
dd if=/dev/ada{n} of=/dev/null
```
Si noti che il disco laterale non ha led, quindi per controllare qale sia 
all'interno del sistema è necessario identificare prima gli altri e andare per
esclusione.
