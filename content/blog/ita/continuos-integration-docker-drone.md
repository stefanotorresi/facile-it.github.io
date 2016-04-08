---
authors: ["lorenzo", "luca"]
comments: true
date: "2015-06-12"
draft: false
share: true
categories: [Italiano, Docker, Continuous integration]
title: "Continuous Integration con Docker e Drone"

languageCode: "it-IT"
type: "post"
aliases:
  - "/continuos-integration-docker-drone"
---
La **continuous integration** è una pratica che consiste nel **frequente allineamento**, su di una base comune definita *mainline*, delle copie di lavoro degli sviluppatori che collaborano al codice di un progetto.

Introdotta inizialmente da Grady Booch nel 1991, nella pubblicazione [Object Oriented Design: With Applications](http://books.google.com/books?id=w5VQAAAAMAAJ&q=continuous+integration+inauthor:grady+inauthor:booch&dq=continuous+integration+inauthor:grady+inauthor:booch&hl=en&sa=X&ei=0_TxU6TqIMOZyASJ3ICYCQ&ved=0CEQQ6AEwAg), la pratica è stata estesa e sviluppata all'interno dell'**extreme programming**, fino a sostenere la necessità di **allineare** le copie di lavoro **diverse volte al giorno**.

Il vantaggio principale nell'adottare la pratica è quello di **evitare l'integration hell** (o merge hell) **minimizzando il rischio** legato a copie di lavoro divergenti di difficile integrazione.

Il [software testing](http://engineering.facile.it/tag/software-testing/), pur non essendo indispensabile ai fini della **continuous integration**, ne è perfettamente complementare, dando allo sviluppatore garanzia di **integrità del funzionamento del software** prima e dopo l'integrazione sulla *mainline*.

Risale al 2000 un importante [articolo](http://martinfowler.com/articles/continuousIntegration.html) di [Martin Fowler](http://martinfowler.com/) che indica i **principi fondamentali** della CI:

* Mantieni un repository del codice sorgente
* Automatizza la build
* Rendi la build auto-testante
* Esegui commit alla baseline tutti i giorni
* Ogni commit fa partire una build automatica
* Ripara immediatamente le build fallite
* Fai in modo che la build sia veloce
* Esegui i test in un clone dell'ambiente di produzione
* Fai in modo che sia facile recuperare l'ultima build
* Tutti possono vedere lo stato delle build
* Automatizza il deploy

## Soluzioni per la CI ed il testing automatico

Negli ultimi anni si sono diffuse molte **soluzioni software** che permettono di automatizzare il processo di **build e testing** a partire da un semplice *push* sulla mainline di sviluppo. Ognuno di queste ha le sue peculiari caratteristiche che la rendono diverso dagli altri.

In Facile.it abbiamo stilato una lista di **requisiti** per trovare il sistema di CI più adatto al nostro **caso d'uso**:

* Possibilità di effettuare build in **ambienti molto diversi** (per sistema operativo, versioni software..) **identici a quelli di produzione**
* Possibilità di effettuare **contemporaneamente** build multiple, anche appartenenti allo **stesso progetto**
* Sistema di CI compatibile con diversi **servizi Git** (GitHub, GitLab, BitBucket..)
* Integrazione con **chat** e sistemi di **notifica** (Slack, IRC..)
* Invio **mail di alert** per build fallite
* Sistema di CI **estendibile via API**
* **Dashboard** che mostri lo stato delle build, per tenere gli **sviluppatori informati** riguardo lo stato delle proprie build e della mainline

Utilizzando in maniera intensiva [Docker](https://www.docker.com/) per lo sviluppo locale con ambienti **simili alla produzione**, una caratteristica interessante da avere consiste proprio nella possibilità di effettuare **build all'interno di container**.

La nostra scelta è ricaduta su [Drone](https://github.com/drone/drone), una piattaforma di continuous integration piuttosto giovane, ma sufficientemente stabile per un utilizzo in produzione.

## Drone: build attraverso container Docker

[Drone](https://github.com/drone/drone), rilasciato con licenza Apache 2.0, è una piattaforma di CI che automatizza le build **all'interno di container Docker**: la soluzione ideale in un ambiente dove **numerosi gruppi di lavoro** utilizzano **ambienti molto diversi** (per sistema operativo, versioni di interpreti di linguaggio, sistemi di database o caching) ma vogliono centralizzare i processi di Continuous Integration.

Come altre piattaforme di CI, Drone supporta diversi sistemi Git (GitHub, GitLab, BitBucket, Gogs et al.), diversi sistemi di [deploy](https://github.com/drone/drone/blob/v0.2.1/README.md#deployments) (Aws S3, SSH, Heroku, Swift et al.) e diversi sistemi di [notifica](https://github.com/drone/drone/blob/v0.2.1/README.md#notifications) (Webhook, Hipchat, Email et al.).

I container di build possono essere avviati in locale o **remoto**, dal momento che la [configurazione](https://github.com/drone/drone#setup) permette di indicare i **socket Docker** (UNIX, ma anche TCP) da utilizzare.

Drone supporta un [**sistema di plugin**](https://github.com/drone/drone-plugin-go) estremamente **flessibile**: un plugin riceve via riga di comando o input dal terminale un JSON contenente le **informazioni sulla build** corrente e pubblica il risultato in output.

I plugin vengono distribuiti a loro volta come **container Docker**, così possono condividere attraverso un volume-mount la **stessa copia del repository** su cui avviene la build.
L'*ENTRYPOINT* per il plugin nel Dockerfile **consiste nell'eseguibile** vero e proprio:
in questo modo è possibile scrivere **plugin in qualsiasi linguaggio** per fare qualsiasi tipo di operazione!

Per ultimo, ma comunque non di poco conto, il **monitor stato build** viene rilasciato in un progetto separato: [Drone Wall](https://github.com/drone/drone-wall) è semplicemente fantastico!
 
![Drone wall screenshot](/images/continuos-integration-docker-drone/drone-wall.jpg)

Quella che segue, è una guida per **l'installazione di Drone** (a sua volta in un container!), molto simile a quella  utilizzata in Facile.it.

## Prerequisiti
Per testare questa guida abbiamo utilizzato Docker 1.6.2 e [docker-compose 1.2.0](https://docs.docker.com/compose/install/).
Per chi non conoscesse Compose, basti sapere che è un **tool per definire convenientemente una configurazione complessa** (di solito multi-container) in un singolo file *yaml*, potendo poi lanciare tutti i container in essa definiti con un singolo conveniente comando anzichè con molti tediosi `docker run`. 

Come prima cosa è necessario clonare il gist, embeddato qui sotto per riferimento, lanciando il seguente comando:

```
git clone https://gist.github.com/de5d5861fa4d86f9598c.git
```

<script src="https://gist.github.com/fntlnz/de5d5861fa4d86f9598c.js"></script>

Una volta entrati nella cartella del gist, troviamo due file, *docker-compose.yml* e *nginx.conf*,  descritti in dettaglio di seguito:

## docker-compose.yml
Questo file contiene la configurazione dei container che ci permetterà di mettere insieme il nostro ambiente.

Al suo interno abbiamo **tre nodi** principali: *drone*, *nginx* e *wall*; analizziamoli singolarmente:

### DRONE
Come abbiamo detto in precedenza Drone supporta i vari servizi di hosting repository git, sia open source che PaaS. Per questa guida abbiamo scelto l'integrazione più semplice e immediata, quella con **GitHub**.

Nel nodo di configurazione di drone ci sono diverse variabili d'ambiente: due di queste ci serviranno a configurare l'**autenticazione tramite OAuth2** di GitHub (allo stato attuale, Drone non ha un sistema interno di gestione degli utenti).

Le variabili d'ambiente per GitHub sono `DRONE_GITHUB_CLIENT` e `DRONE_GITHUB_SECRET`; le chiavi necessarie vengono rilasciate da GitHub a seguito della [registrazione di una nuova applicazione](https://github.com/settings/applications/new).

La variabile d'ambiente `DRONE_REGISTRATION_OPEN` va settata a `false` quando si deciderà di non permettere la **registrazione di nuovi utenti**, ma è attualmente settata a `true` per permettere la registrazione del primo utente, che sarà anche il master dell'installazione.

Un'altra variabile d'ambiente molto importante è `DRONE_WORKER_NODES`: la sua importanza è data dal fatto che contiene il **path del socket di Docker** che permette la comunicazione con i container. Ogni ripetizione separata da virgola di `unix:///var/run/docker.sock` mette a disposizione delle build un nuovo worker per eseguire più build in parallelo.

Infine, l'ultima ma non meno importante variabile d'ambiente da impostare è `DRONE_SESSION_SECRET`, la chiave che servirà a codificare le sessioni. Una chiave generata [su Random.org](https://www.random.org/strings/?num=20&len=20&digits=on&upperalpha=on&loweralpha=on&unique=on&format=html&rnd=new) è perfetta per questo scopo.

In questo nodo ci sono inoltre **due volumi condivisi** che sono rispettivamente:

- `/data/drone:/var/lib/drone`: il path dove verrà scritto il database SQLite di Drone; eventualmente si può sostituire con un path più adeguato
- `/var/run/docker.sock:/var/run/docker.sock`: il path del socket di docker per permettere a Drone di lanciare nuovi container, poiché si trova anche lui all'interno di un container

Un'altra riga di questo nodo che è degna di nota è `privileged: true`: è infatti **necessario che il container contenente Drone giri in modalità privilegiata**, per disattivare quei controlli di sicurezza che altrimenti non permetterebbero al container di operare alla creazione, distruzione e modifica di altri container.

### WALL
In questo nodo è necessario impostare la variabile d'ambiente `API_TOKEN`, necessaria ad autorizzare la dashboard di visualizzazione delle build: il token si trova nel proprio profilo utente di Drone una volta avviato. Questa configurazione può essere quindi completata solo dopo aver avviato Drone per la prima volta.


### NGINX
Questo nodo fa il binding della porta 80 dell'host verso la porta 80 del container e condivide due volumi, rispettivamente:

- `nignx.conf`: condivide la configurazione del file `nginx.conf`
- `/var/log/nginx:/var/log/nginx`: condivide la cartella contenente i log di NGINX


`nginx.conf` va configurato per usare NGINX come proxy di Drone e Drone Wall, utilizzando [proxy_pass](http://nginx.org/en/docs/http/ngx_http_proxy_module.html#proxy_pass).

Al suo interno contiene già due *server blocks*, i quali a loro volta contengono le direttive `server_name`, rispettivamente `drone.local` per Drone e `wall.drone.local` per il Drone Wall.

## Up And Running
Ora che abbiamo completato la configurazione, possiamo semplicemente avviare i nostri container con questo comando, che va eseguito nella cartella contentente il clone del gist di sopra:

```
docker-compose up
```

Ora possiamo finalmente utilizzare drone puntando il nostro browser all'indirizzo [http://drone.local](http://drone.local) (o all'indirizzo configurato nel file `nginx.conf`)
