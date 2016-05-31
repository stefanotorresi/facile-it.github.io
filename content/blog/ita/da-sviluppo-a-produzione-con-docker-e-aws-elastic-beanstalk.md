---
authors: ["lorenzo", "daniele"]
comments: true
date: "2015-10-19"
draft: false
image: ""
menu: ""
share: true
categories: [Italiano, Docker, AWS]
title: "Da sviluppo a produzione con Docker e AWS Elastic Beanstalk"
type: "post"
eng: "from-development-to-production-with-docker-and-amazon-ecs"
languageCode: "it-IT"
aliases:
  - "/da-sviluppo-a-produzione-con-docker-e-aws-elastic-beanstalk"
images: ['/images/logo.png']

---

## In locale funzionava
Questo articolo si rivolge a chi ha già una [conoscenza base di docker](https://docs.docker.com/articles/basics) e del suo funzionamento e sta cercando come avanzare al passo successivo, usandolo quotidianamente in sviluppo e in produzione.

Avere un ambiente di sviluppo/test **il più simile possibile** a quello di produzione aiuta molto nel garantire un **corretto funzionamento dopo il deploy**.  

In uno scenario tipico, lo sviluppatore ha installati sulla propria macchina locale tutti i servizi da cui dipende la sua applicazione, il che comporta quanto segue:

- nessun tipo di isolamento tra progetti che usano gli stessi servizi (versione, configurazione, dati);
- è difficile avere e mantenere in locale la stessa versione e la stessa configurazione dei servizi in produzione;
- condividere l'ambiente di sviluppo con colleghi e collaboratori è difficile se non impossibile;

Tutto questo conduce ad una delle peggiori frasi che io abbia mai sentito in tutta la mia esperienza lavorativa:

> It works on my machine _(in locale funzionava)_

![It works on my machine meme](/images/from-development-to-production-with-docker-and-amazon-ecs/wmm.jpg)

Potreste pensare che avrei potuto ottenere gli stessi risultati usando Vagrant o una classica macchina virtuale, ma questa soluzione non mi avrebbe dato i benefici di avere un layer di astrazione aggiuntivo senza dovermi preoccupare dell'overhead. Infatti posso avere molti più container che girano su una singola macchina di quelli che avrei avuto con la semplice virtualizzazione.

## Bookshelf: uno scaffale virtuale

Per snellire questo articolo ho preparato un'[applicazione demo](https://github.com/pennyphp/bookshelf) basata su [Penny PHP Framework](http://github.com/pennyphp/penny): è una semplice applicazione per l'archiviazione di libri, che consente all'utente di creare e visualizzare una lista di libri.

#### Download e dipendenze
Per prima cosa, scarichiamo l'applicazione dal suo repository:

```
git clone https://github.com/pennyphp/bookshelf
```

Le dipendenze in PHP sono gestite attraverso [composer](https://getcomposer.org/), e per soddisfarle basta digitare il comando:

```
composer install
```

Gli assets del frontend sono gestiti attraverso [Bower](http://bower.io) + [Grunt](http://gruntjs.com/); i due seguenti comandi scaricheranno e compileranno le dipendenze e produrranno gli assets direttamente nella cartella pubblica:

```
npm install
grunt dev
```

#### Avviare l'ambiente di sviluppo

Come potete vedere l'applicazione demo è distribuita con un ambiente di sviluppo docker che potete trovare nella cartella [docker/development](https://github.com/pennyphp/bookshelf/tree/master/docker/development).

Osservando il contenuto di `docker/development` possiamo trovare altre due cartelle:

- `nginx/`: questa directory contiene un Dockerfile che eredita l'immagine da [fntlnz/nginx](https://github.com/fntlnz/dockerfiles/tree/master/nginx) per crearne una nuova con la configurazione nginx necessaria;
- `fpm/`: questa directory contiene un Dockerfile che eredita l'immagine da [fntlnz/php](https://github.com/fntlnz/dockerfiles/tree/master/php) per crearne una nuova con la configurazione e le estensioni di php-fpm necessarie;

Dal momento che non abbiamo Elastic Beanstalk sulla nostra macchina locale (ne parleremo in seguito) e abbiamo bisogno di un modo per orchestrare i nostri container, lo faremo utilizzando **docker-compose**. Ho deciso di usare Docker Compose al posto del comando [**eb local**](http://docs.aws.amazon.com/elasticbeanstalk/latest/dg/eb3-local.html) (che consente di far girare l'ambiente di elastic beanstalk in locale) perché allo stato dell'arte compose è più facile da usare e mantenere in locale.

###### Il file `docker-compose.yml`
Per farlo dobbiamo creare un file `docker-compose.yml` nella nostra root di progetto.

```
cp docker/docker-compose.yml.development docker-compose.yml
```

A questo punto, nel nostro `docker-compose.yml` dovremmo avere quattro container da avviare:

- il container **Nginx**, che contiene un paragrafo server in ascolto sulla porta 80.
- il container **fpm**, che condivide un volume con la macchina host in modo da poter modificare il codice senza bisogno di ricostruire il container; inoltre, il container è collegato al container mysql per consentire agli script php di connettervisi;
- il container **mysql**, che conterrà i nostri dati di sviluppo;
- il container **redis**, usato come cache, principalmente da Doctrine;

Si noti come, dal momento che [i container sono collegati](https://docs.docker.com/userguide/dockerlinks/) è possibile accedere ad un servizio esposto usando il nome assegnato al container collegato; ad esempio, nel nostro caso il container fpm è [collegato al container mysql](https://github.com/pennyphp/bookshelf/blob/2e55738da9ff9e45fa44add9d97280635e95399d/docker/docker-compose.yml.development#L19-L20): è per questo che l'host configurato nella [configurazione di doctrine locale] (https://github.com/pennyphp/bookshelf/blob/2e55738da9ff9e45fa44add9d97280635e95399d/config/doctrine.local.php.dist#L13) è `mysql`.

Se non l'avete ancora fatto, dovrete buildare l'immagine `fpm` e scaricare le immagini `nginx`, `mysql` e `redis`; per farlo, digitate il comando:

```
$ docker-compose build
```

Ora che avete tutto ciò che vi serve potete avviare i container con:

```
$ docker-compose up
```

A questo punto i quattro container dovrebbero essere in esecuzione: potete verificare che tutto sia a posto con il comando `docker ps`.

Per proseguire, abbiamo bisogno di conoscere l'indirizzo ip del container nginx. Il port forwarding del container nginx è configurato come `80:80`, perciò è disponibile su **linux** agli indirizzi `127.0.0.1:80` e `localhost:80`, mentre su **OS X** all'indirizzo associato alla docker-machine; per identificarlo, usare il comando: 

```
docker-machine env <yourmachinename> | grep DOCKER_HOST
```

È ora di collegarci all'ip del nostro container nginx!

![Bookshelf screenshot](/images/from-development-to-production-with-docker-and-amazon-ecs/books.png)


Yay! Il nostro ambiente di sviluppo è in esecuzione!


#### Configurare e attivare l'ambiente di produzione

A questo punto ci serve un modo per rilasciare la nostra applicazione in produzione che possa: eseguire container Docker, scalare senza intoppi e possibilmente aver già installato altri interessanti componenti come, ad esempio, per il monitoring.

La scelta è caduta su [AWS Elastic Beanstalk](https://aws.amazon.com/elasticbeanstalk) ha tutto ciò che abbiamo elencato ed ha inoltre una tariffazione più competitiva con un [Free Tier](https://aws.amazon.com/free/) iniziale, sufficiente per far girare questa demo.

Prima di iniziare abbiamo bisogno di un account **Amazon Web Services**; se non ne avete ancora uno, potete crearlo [qui](https://aws.amazon.com/account)

Per configurare, rilasciare e gestire la nostra infrastruttura avremo bisogno del [comando eb](http://docs.aws.amazon.com/elasticbeanstalk/latest/dg/eb-cli3-install.html); per installarlo, digitare:

```
pip install awsebcli
```

Per avere accesso alla piattaforma dalla vostra riga di comando usando il comando **eb** dovrete configurare uno [**IAM ROLE**](http://docs.aws.amazon.com/elasticbeanstalk/latest/dg/AWSHowTo.iam.roles.aeb.html) ed associarlo ad uno [**IAM User**](http://docs.aws.amazon.com/IAM/latest/UserGuide/id_users_create.html). La creazione assistita di uno IAM User vi **darà due chiavi**, chiamate *AWS Access Key ID* e *AWS Secret Access Key*. Ci serviranno durante il prossimo passaggio.

A questo punto possiamo **initialize** il nostro progetto Bookshelf. Questo comando ci chiederà le due Access keys, oltre che alcune domande durante l'installazione.

```
eb init
```

Ora che il progetto è inizializzato dobbiamo **creare un nuovo ambiente**. Questo comando creerà effettivamente un'istanza **t2.micro EC2**, i gruppi di sicurezza, il load balancer, le notifiche cloudwatch ecc..

```
eb create bookshelf-production
```

Prima di rilasciare l'applicazione in produzione dobbiamo generare un [Token Github per composer](https://github.com/settings/tokens/new).
Questo è necessario per scaricare tutte le dipendenze senza intoppi.
Per aggiungere il token all'ambiente:

```
eb setenv COMPOSER_TOKEN=<your-token-here>
```

Ora potete verificare che il sistema sia pronto digitando:

```
eb status
```


Quando lo stato dell'applicazione diventa **Ready** potete a tutti gli effetti pubblicare l'applicazione con:

```
eb deploy
```

Il deployment creerà i container descritti in [Dockerrun.aws.json](https://github.com/pennyphp/bookshelf/blob/2e55738da9ff9e45fa44add9d97280635e95399d/Dockerrun.aws.json) e i files descritti in [dependencies.config](https://github.com/pennyphp/bookshelf/blob/2e55738da9ff9e45fa44add9d97280635e95399d/.ebextensions/dependencies.config).

Se vi state chiedendo cosa sia realmente il file **Dockerrun.aws.json**, basta dire che sta ad Elastic Beanstalk come `docker-compose.yml` sta all'ambiente locale.

I file contenuti in [.ebextensions](https://github.com/pennyphp/bookshelf/blob/2e55738da9ff9e45fa44add9d97280635e95399d/.ebextensions/) consentono di personalizzare e configurare il software da cui la vostra applicazione dipende. Il file **dependencies.config** è uno di questi. È proprio lui a risolvere le dipendenze di composer, a compilare gli asset del frontend con grunt e bower e a creare l'immagine PHP FPM usata nel [Dockerfile](https://github.com/pennyphp/bookshelf/blob/2e55738da9ff9e45fa44add9d97280635e95399d/docker/production/fpm/Dockerfile) di produzione. È importante notare che l'immagine viene nuovamente costruita solo se il Dockerfile viene modificato.


Come avrete notato, [Dockerrun.aws.json](https://github.com

La configurazione di default dell'applicazione Bookshelf riceve i parametri di connessione a Mysql e Redis [dalle variabili d'ambiente](https://github.com/pennyphp/bookshelf/blob/2e55738da9ff9e45fa44add9d97280635e95399d/config/doctrine.global.php) che sono:

```
MYSQL_HOST
MYSQL_PORT
MYSQL_USERNAME
MYSQL_PASSWORD
MYSQL_DATABASE
REDIS_HOST
REDIS_PORT
```
Ognuna di queste variabili d'ambiente può essere impostata usando il comando che abbiamo precedentemente usato per `COMPOSER_TOKEN`.

Potete ottenere i parametri di connessione per Mysql e Redis dopo aver creato [un'istanza RDS Mysql DB](http://docs.aws.amazon.com/AmazonRDS/latest/UserGuide/USER_CreateInstance.html) e un [Cluster ElastiCache Redis](http://docs.aws.amazon.com/opsworks/latest/userguide/other-services-redis-cluster.html)

A questo punto, con i container nginx e fpm in esecuzione e i database configurati, potete digitare `eb open` per aprire l'applicazione in produzione e verificare se tutto è ok!

La vostra infrastruttura, così com'è, è anche già pronta per scalare automaticamente; per muovere i primi passi in questa direzione potete dare un'occhiata al comando [`eb scale`](http://docs.aws.amazon.com/elasticbeanstalk/latest/dg/eb3-scale.html) e alla [documentazione di AWS](http://docs.aws.amazon.com/elasticbeanstalk/latest/dg/using-features.managing.as.html).

**Troubleshooting**: Se qualcosa dovesse andare storto potete accedere via ssh nella macchina Elastic Beanstalk EC2 con `eb ssh` e analizzare lo stato dei container usando strumenti che già conoscete come `docker logs`.

#### È fatta!

Ora potete risparmiare un sacco di tempo automatizzando il vostro workflow con docker ed ottenere un ambiente di sviluppo funzionante, auto-contenuto e condivisibile, pur mantenendolo molto simile alla vostra stabile ed efficiente infrastruttura in esecuzione nell'ambiente di produzione, sulle cui risorse avete pieno controllo.

![congratulations](/images/from-development-to-production-with-docker-and-amazon-ecs/good-job.jpg)



