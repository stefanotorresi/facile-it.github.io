---
authors: ["daniele"]
comments: true
date: "2015-07-22"
draft: false
image: ""
menu: ""
share: true
categories: [Italiano, "OSX", "Docker"]
title: "Si fa presto a dire Docker (e OS X)"

languageCode: "it-IT"
type: "post"
aliases:
  - "/si-fa-presto-a-dire-docker-e-os-x"
---
Di guide per l'installazione di Docker su OS X è pieno il web, ma **le soluzioni sono molte** e spesso scegliere non è facile.
In questo articolo vi mostrerò quello che è stato il mio personale percorso, nella speranza che possa essere utile a chiarirvi le idee e magari evitare qualche buco nell'acqua.

Da sviluppatore web, la mia esigenza è di conservare i sorgenti sulla macchina host e condividerli con la VM; per la natura di PHP, la condivisione deve anche essere molto veloce, poiché ad ogni richiesta i files verranno letti nuovamente dal disco. Nulla di complicato, quindi sono partito dalla cosa più semplice.

## boot2docker
Probabilmente la soluzione più nota e diffusa, [boot2docker](https://docs.docker.com/installation/mac/) è stata la mia prima scelta. Si installa con `homebrew` o `curl` e viene distribuita con una propria ISO, molto leggera e progettata appositamente per Docker, che gira su VirtualBox.
L'unica vera pecca è proprio il protocollo di condivisione nativo di VirtualBox, `vboxsf`, che risulta **molto lento** per progetti di medie dimensioni e rende il caricamento delle pagine lunghissimo.

## docker-osx
Decido di [provarlo](https://github.com/noplay/docker-osx) incuriosito da un collega che ne aveva creato una versione modificata per supportare la **condivisione NFS**, ma nonostante questo non lo trovo migliore di boot2docker. Inoltre il suo sviluppo è precedente, e il progetto è stato abbandonato quando Docker ha iniziato a supportare ufficialmente boot2docker: è in effetti lo stesso sviluppatore a sconsigliarne l'utilizzo da parte di nuovi utenti. 

## Vagrant e [parallels/boot2docker](https://github.com/Parallels/boot2docker-vagrant-box)
Leggo su un blog che è possibile avere prestazioni migliori utilizzando un sistema di virtualizzazione alternativo a VirtualBox, come ad esempio Parallels o VMWare Fusion. Per entrambi esistono delle ISO modificate di boot2docker e la macchina dev'essere configurata usando Vagrant. La mia scelta cade su **Parallels**, e finalmente riesco a creare **una macchina performante** con condivisione NFS.
Purtroppo il mantenimento della ISO sembra non stare al passo con la frequenza di rilascio di Docker (ero fermo alla versione 1.4.1 quando la 1.5 era disponibile da diverse settimane), quindi decido di cambiare nuovamente soluzione.

## Vagrant e Ubuntu
Dati i precedenti limiti e relativi fallimenti, tento una strada alternativa e [preparo un Vagrantfile](https://github.com/ildanno/parallels-docker-vagrantfile) contenente tutto quello che mi serve: una macchina Ubuntu che gira su Parallels, repository di Docker aggiornato, condivisione NFS e possibilità di eseguire comandi al boot.
Ma ancora una volta ho un problema fastidioso, che si verificava già con la soluzione precedente ma che (sbagliando) attribuivo a boot2docker. Ad ogni avvio della macchina, infatti, si crea **un conflitto** con OS X per l'utilizzo di **alcune risorse di rete** e devo risolverle manualmente riavviando il demone NAT di Parallels. Inoltre scopro che con vagrant+parallels lo spazio utilizzato dal disco virtuale non è più recuperabile una volta allocato e il mio SSD è ormai alle strette.

## VM custom 
Preso dallo sconforto scelgo di avere il **controllo completo**: installo **VMWare Fusion** per avere un sistema di virtualizzazione più performante di VirtualBox, ma senza gli inconvenienti di Parallels, e configuro una macchina con Ubuntu server. Posso montare share NFS e ridimensionare il disco virtuale. Mi sembra una soluzione molto artigianale, ma funziona esattamente come voglio io e credo finalmente di avere trovato la mia pace. Ma non è così.

## Docker Machine
Da alcune settimane è uscita una novità sul mercato, direttamente da casa Docker, che ho inizialmente ignorato. Si chiama [Docker Machine](https://docs.docker.com/machine/) e ad una prima occhiata non sembra fare cose molto diverse da boot2docker, tant'è che ne utilizza la stessa ISO. Però il fatto che venga rilasciato direttamente dalla casa madre mi fa ben sperare, vista anche l'evoluzione avuta da altri tools come [Docker Compose](https://docs.docker.com/compose/), e dopo qualche tempo decido di dargli una possibilità.

Mi rendo subito conto che in realtà Docker Machine ha **potenzialità molto maggiori** di quelle di boot2docker: è in grado di gestire più di una macchina, locale o in cloud, supporta diversi drivers e ha addirittura la possibilità di collegarsi ad un generico server remoto.
Creo quindi una macchina locale, utilizzando il driver di VMWare Fusion:

```
$ docker-machine create --driver vmwarefusion \
    --vmwarefusion-cpu-count 2 \
    --vmwarefusion-disk-size 40000 \
    --vmwarefusion-memory-size 4096 \
    dev2
```

In pochi secondi la macchina è pronta all'uso. Anche se la cartella `/Users` è già montata con il protocollo di condivisione file di VMWare (`vmhgfs`), decido comunque di configurare una condivisione NFS personalizzata.

Per prima cosa configuro OS X in modo da accettare connessioni dalla macchina su cui gira Docker (in questo caso verso la mia cartella `~/Sites`):
```
$ echo "/Users/dcontini/Sites -mapall=dcontini:staff $(docker-machine ip dev2)" | sudo tee -a /etc/exports
$ sudo nfsd restart
```

Ora configuro la macchina virtuale, avviando i tools NFS e, se necessario, creando il mount point:
```
$ docker-machine ssh dev2 -- sudo /usr/local/etc/init.d/nfs-client start
> Starting nfs client utilities.
$ docker-machine ssh dev2 -- sudo mkdir -p /var/www
$ docker-machine ssh dev2 -- sudo mount 172.16.153.1:/Users/dcontini/Sites /var/www
```

A questo punto ho finalmente la mia condivisione NFS e, se voglio, posso configurarne diverse e smontare la cartella `/Users`:
```
$ docker-machine ssh dev2 -- mount
.host:/Users on /Users type vmhgfs (rw,relatime)
172.16.153.1:/Users/dcontini/Sites on /var/www type nfs (rw,relatime,...)
```

Purtroppo ad ogni riavvio della macchina virtuale è necessario **rimontare manualmente la condivisione**, poiché Docker Machine esegue nuovamente il provisioning. Ma del resto, alla fine di questo viaggio, possiamo dire che si tratta del problema minore ;)

(Per i più pigri, invece, ho [pubblicato](https://github.com/ildanno/docker-machine-mount) **uno script** che automatizza l'operazione)
