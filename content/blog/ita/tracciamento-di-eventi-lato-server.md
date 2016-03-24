---
authors: ["matteo"]
comments: true
date: "2015-04-21"
draft: false
share: true
categories: [Italiano, NewRelic, REST]
title: "Introduzione al tracciamento di eventi lato server"

languageCode: "it-IT"
type: "post"

---
L’adozione di strategie *DevOps* ha portato negli ultimi anni ad incrementare le aree coperte da **tool di monitoring** in maniera tale da avere un feedback in tempo reale dello stato dell’infrastruttura su cui si basa la propria applicazione, così da poter rispondere pro-attivamente a situazioni critiche. Su tale fronte infatti si sono visti comparire i tool più disparati che permettono la raccolta di informazioni, sia dei **server** o delle istanze su cui vengono eseguite le applicazioni (ad esempio CPU, memoria o disco), che le **applicazioni** stesse (ad esempio tempi medi di risposta, numero di query eseguite, tempo di esecuzione delle query, ecc. ecc.). 

Per entrambi i casi lo sviluppatore ha a disposizione sia soluzioni **open source** che soluzioni **SaaS**: la differenza sostanziale, a parità di feature principali, rimane la gestione dell’infrastruttura aggiuntiva, necessaria a supportare le operazioni per la raccolta dei dati inviati dai sensori predisposti a monitorare le metriche che interessano. Nonostante gli strumenti si facciano sempre più abbondanti (basti pensare a tutte le soluzioni recenti per il salvataggio di time series) e user-friendly, volersi affidare a provider terzi può essere un modo per concentrarsi maggiormente sul valore aggiunto che si vuole dare all’utente finale con la propria applicazione, senza doversi preoccupare di questioni di contorno.

Tra i principali prodotti SaaS presenti sul mercato troviamo *[NewRelic](http://newrelic.com/)*, *[AppDynamics](http://www.appdynamics.com/)*, *[ServerDensity](https://www.serverdensity.com)* e *[Ruxit](https://www.ruxit.com)*, ciascuno con i propri prezzi e le proprie peculiarità, ma tutti che permettono di avere un costante monitoraggio della propria applicazione, dal browser dell’utente fino alla query per salvare i dati, gratis o con costi mensili contenuti e proporzionali alla dimensione della propria infrastruttura. 

A **Facile.it** viene utilizzato **NewRelic** non solo per il monitoraggio base dei servizi che compongono il sito ma, grazie a **NewRelic Insights**, perché viene data la possibilità di salvare eventi personalizzati che possono dare maggiori informazioni sia agli sviluppatori che ai business analysts. Per poter accedere a **NewRelic Insights** bisogna avere un account a pagamento, ma è comunque possibile sperimentare l’eventuale utilità per le proprie esigenze grazie al periodo di prova di 14 giorni.

La raccolta dei dati può avvenire in due modi:

  * tramite ***custom attributes***
  * tramite ***eventi personalizzati***


#### Custom Attributes
I *custom attributes* sono delle coppie di chiave-valore da aggiungere alla transazione corrente (Web e Non-Web) già tracciata dagli strumenti di *NewRelic* per i principali linguaggi di programmazione (**PHP**, **NodeJs**, **Ruby**, **Python**, *Java*, *.NET*). Possiamo aggiungere ad esempio l’utente attualmente collegato che sta effettuando tale transazione, il numero di oggetti presenti nel carrello o ancora il loro valore.

Ad esempio, tramite l'estensione di *NewRelic* disponibile per **PHP**, è possibile aggiungere i dettagli dell'utente attualmente collegato chiamando la funzione `newrelic_add_custom_parameter ('userID', $userId)`.

#### Eventi personalizzati
Gli *eventi personalizzati* vanno inviati ad un **webservice REST** in formato **JSON**. Ci mettono in grado di tracciare un qualsiasi evento all’interno di una normale transazione già tracciata da NewRelic, come ad esempio i tempi di risposta di un servizio esterno utilizzato dall’applicazione.

Gli eventi personalizzati possono essere registrati richiamando la relativa funzione `newrelic_record_custom_event('DNDServiceCall',['business' => 'telefonia', 'responseTime' => 1500])` disponibile nelle ultime versioni dell'estensione **PHP** oppure tramite una semplice chiamata cURL come indicato dalla [documentazione](https://docs.newrelic.com/docs/insights/new-relic-insights/adding-querying-data/inserting-custom-events-insights-api) sui *custom events*.

Una volta che i dati vengono raccolti da *NewRelic* è possibile analizzarli tramite delle semplici query in un linguaggio simil-*SQL*, in questo caso chiamato **NRQL** (*NewRelic Query Language*). La sintassi base di una SELECT in SQL viene mantenuta, ma vengono forniti strumenti per meglio gestire le serie temporali, tramite **TIMESERIES**, **SINCE**, **UNTIL**: una trattazione più approfondita è disponibile alla relativa pagina della [documentazione](https://docs.newrelic.com/docs/insights/new-relic-insights/using-new-relic-query-language/nrql-reference). 

I risultati delle *query* vengono resi disponibili in formato **JSON**, oppure come widget da poter integrare poi in una dashboard, in maniera da tenere sempre sotto controllo le metriche più importanti per la conduzione del proprio business. Inoltre tramite [Data Explorer](https://docs.newrelic.com/docs/insights/new-relic-insights/using-insights-interface/exploring-your-data) è possibile consultare un campione degli eventi recentemente inviati dalla nostra applicazione a *NewRelic*, così da poter eventualmente raffinare le strutture dati con cui vengono raccolti.
