---
authors: ["algatux"]
comments: true
date: "2015-10-05"
draft: false
image: "/images/cover.jpg"
menu: ""
share: true
categories: [Italiano, Security, Encryption, RSA, AES, Rjindael]
title: "Protezione di dati sensibili usando la crittografia"

languageCode: "it-IT"
type: "post"
aliases:
  - "/protezione-di-dati-sensibili-usando-la-crittografia"
---
La protezione dei dati è uno dei problemi più sentiti e ricorrenti degli ultimi anni: che si tratti di informazioni private di utenti, o dell'azienda per cui lavoriamo, il problema è sempre lo stesso. 

**Come proteggere questi dati da eventuali malintenzionati che vorrebbero -  e potrebbero(!!) -  riuscire a entrarne in possesso ?**

Prima di arrivare alla soluzione di questo problema però, è doveroso suddividere la casistica in almeno due rami. Nel mondo del web i dati possono essere "ottenuti" in due momenti differenti: 1) mentre transitano sulla rete sotto forma di pacchetti, oppure 2) successivamente al loro immagazzinamento (es. database o file) all'interno di uno o più server.

In questo articolo esploreremo il secondo caso, e verrà proposta una possibile soluzione: come proteggere quei dati che parsimoniosamente abbiamo accumulato "in casa"?.

## Mettere i dati in cassaforte

In caso di dati immagazzinati, ad esempio, all'interno di un database; è certamente più difficile raggiungere il nostro tesoro, in quanto già al "sicuro" all'interno di un ambiente protetto, del quale ci fidiamo maggiormente rispetto alla volatilità della rete.
Tuttavia è meglio non sentirsi troppo sicuri: più i nostri dati saranno preziosi, più eventuali interessati saranno stimolati a ricercare ogni via per ottenerli.
Una qualsiasi falla, sia hardware che software, per quanto limitata nel tempo potrebbe esporre le nostre ricchezze ad attacchi di vario genere, permettendo a chiunque (o quasi) di ottenerle.

Una possibile ed efficace tecnica di protezione in questi casi è **rendere inutili o illeggibili i dati** per coloro che li hanno ottenuti in maniera illegittima, attraverso il loro immagazzinamento in forma cifrata.

Quella che proporrò di seguito è una possibile soluzione al problema di come immagazzinare e cifrare questi dati, in modo da ottenere un livello di sicurezza adeguato e mantenere una buona elasticità, tanto da permettere differenti implementazioni e da coprire quanti più possibili casi d'uso.

La soluzione stessa prevede che la prima parte della problematica esposta (protezione dei dati durante il transito) sia risolta, e si baserà sull'uso combinato di due tipologie differenti di algoritmi di cifratura.

## Algoritmi Crittografici Simmetrici e Asimmetrici

Con *algoritmi crittografici* si intendono tutti quei **processi e procedure finalizzati ad ottenere un dato "offuscato" in modo da non essere comprensibile/intelligibile** da persone non autorizzate a leggerlo.

### Algoritmi Simmetrici

Questa tipologia di algoritmi si basa sull'uso di chiavi di cifratura dette **simmetriche** che permettono di cifrare e decifrare dati **utilizzando la medesima chiave crittografica**.
Un esempio è [AES](https://en.wikipedia.org/wiki/Advanced_Encryption_Standard) (Advanced Encryption Standard), evoluzione dell'algoritmo [Rijndael](https://en.wikipedia.org/wiki/Rijndael_key_schedule) le cui caratteristiche si possono riassumere in un buon livello di sicurezza ed un'ottima velocità, sia che venga implementato via hardware che software.

![AES](/images/protezione-di-dati-sensibili-usando-la-crittografia/aes.png)

### Algoritmi Asimmetrici
Gli algoritmi asimmetrici si distinguono dai precedenti per l'utilizzo di **due chiavi crittografiche distinte**, dette pubblica e privata, per effettuare le operazioni di cifratura e decifratura.
Il nome deriva dal metodo con il quale le due chiavi devono essere utilizzate. La chiave pubblica viene **liberamente scambiata**, mentre la chiave privata **rimane a conoscenza solo di coloro che devono poter leggere i dati** o i messaggi scambiati.
La particolarità che caratterizza queste chiavi è l'impossibilità di ricavare, ad esempio, la chiave privata essendo in possesso della chiave pubblica. A meno di particolari exploit dell'algoritmo, l'unico metodo per ottenere tale chiave è il **brute force** sui dati stessi.
In questo caso l'esempio per eccellenza è [RSA](https://en.wikipedia.org/wiki/RSA_(cryptosystem)), uno standard di fatto nella sicurezza riguardante la trasmissione di dati: è infatti largamente utilizzato per cifrare le comunicazioni che avvengono tra client e server.

![RSA](/images/protezione-di-dati-sensibili-usando-la-crittografia/rsa.png)

## La soluzione combinando RSA e AES
Ora che abbiamo ripassato le basi tecnologiche, possiamo pensare a come implementare una soluzione. 

Immaginiamo di avere una piattaforma che permetta ai vari **utenti** di collegarsi con le proprie credenziali e di effettuare svariate operazioni, tra cui immagazzinare i propri documenti privati (foto, documenti, password, contratti, etc.) che necessitano di un più alto livello di sicurezza, e magari di poter condividere tra loro questi dati in un futuro prossimo.

Partendo da queste semplici condizioni, e predisponendo il tutto alla futura "condivisibilità" dei documenti, una possibile soluzione è la seguente:

* utilizzeremo AES per cifrare i singoli documenti;
* per permettere la condivisione, utilizzeremo una tabella (di scambio) del nostro database; al suo interno verranno tracciati il proprietario del documento e gli utenti che potranno accedervi;
* ogni singolo utente avrà il suo portachiavi, contenente le chiavi RSA che fungeranno da "lucchetto" per la chiave del singolo documento;
* sarà necessario offuscare anche la chiave privata di ogni utente, in modo da permetterne l'accesso solo al proprietario;

### Gestire gli utenti
Alla creazione di un nuovo utente sarà nostra premura **generare una nuova coppia di chiavi** RSA, che andrà legata all'utente. 
Prima di memorizzare queste chiavi nel database (assieme alle credenziali dell'utente o in un'altra tabella), sarà necessario offuscare la chiave privata. È buona prassi, in questi casi, far scegliere una **masterkey** all'utente, da utilizzare per cifrare a sua volta la chiave privata con algoritmo simmetrico.
La masterkey scelta sarà importantissima in quanto andrà richiesta ogni qual volta dovremo decifrare i nostri dati (purtroppo la user experience è sacrificata a favore di un livello di sicurezza nettamente maggiore).
**N.B.** È assolutamente sconsigliato memorizzare la masterkey (in qualsiasi forma) sul server: l'attaccante potrebbe facilmente accedervi e tentare di forzarla sbloccando di conseguenza tutte le altre chiavi.

### Cifrare il documento
**Ogni volta** in cui un utente caricherà un file o inserirà un testo da proteggere, sarà necessario generare una nuova chiave simmetrica **univoca**. Useremo la chiave per cifrare il documento utilizzando AES e lo memorizzeremo all'interno della nostra base dati.

### Associare il file all'utente
Come ultimo passo dovremo associare i dati cifrati all'utente che li sta memorizzando. È il momento di utilizzare la "tabella di scambio". Inseriremo un record contenente i riferimenti all'utente e al file, insieme alla chiave univoca legata a quest'ultimo. La chiave verrà però prima cifrata utilizzando la chiave pubblica dell'utente.

Abbiamo ottenuto un sistema simile a quello visualizzato di seguito:

![](/images/protezione-di-dati-sensibili-usando-la-crittografia/system.png)

### Condividere!
Ogni volta in cui un utente proprietario di un documento (o che ha ottenuto il diritto di leggerlo) vorrà condividerlo con un secondo utente sarà sufficiente operare come segue:

* richiedere la masterkey dell'utente;
* decifrare la chiave privata dello stesso utente utilizzando la chiave appena ottenuta;
* decifrare la chiave legata al documento (recuperata dalla tabella di scambio);
* effettuare una copia del record di scambio, sostituendo l'utente con quello di destinazione e cifrando nuovamente la chiave di scambio con quella pubblica del destinatario;

### E ora?

Ora che abbiamo ultimato (almeno teoricamente) il nostro sistema di protezione dei dati **siamo definitivamente al sicuro?** **Assolutamente no!** Tralasciando il buon esercizio tecnico svolto fin ora, dobbiamo ricordarci che **abbiamo affidato buona parte della sicurezza del nostro sistema all'utente**.
Il punto debole infatti risiede nella masterkey che resta nelle mani dei nostri utenti: se questi ultimi dovessero condividerla, o perderla, sarebbe impossibile garantire la sicurezza o il recupero dei dati immagazzinati.
Inoltre una chiave - che è scelta dell'utente - troppo semplice sarebbe facilmente aggirabile tramite brute force o altre tecniche. 

In conclusione, tengo ad esortarvi a rispettare tutte le [regole generali riguardanti la composizione delle password](https://en.wikipedia.org/wiki/Password#Factors_in_the_security_of_a_password_system), impendendo agli utenti di sceglierne una troppo semplice.
