---
authors: ["nicola"]
comments: true
date: "2015-04-30"
draft: false
share: true
categories: [Italiano, Agile, Scrum, Backlog, Pair programming, Kanban]
title: "Un flusso di lavoro agile"

languageCode: "it-IT"
type: "post"
---
Parlare di **metodologie di lavoro** in azienda è una cosa piuttosto complessa, soprattutto perché non è possibile generalizzare un modo di lavoro che sia universalmente valido. Sento sempre più persone dire "con [SCRUM](http://it.wikipedia.org/wiki/Scrum_%28informatica%29) avrai risultati assicurati". Lo trovo piuttosto riduttivo.

In sostanza, sempre più persone hanno tradotto i [valori](http://agilemanifesto.org/iso/it/) ed i [princìpi](http://agilemanifesto.org/iso/it/principles.html) promossi dal manifesto agile in una serie di "ricette" pronte da seguire fedelmente per ottenere buoni risultati. Cosa c’è di "[agile](http://it.wikipedia.org/wiki/Metodologia_agile)" in tutto ciò? E’ come cucinare con un robot da cucina...
Ecco perché sono sempre più convinto che la vera ricetta del successo nei progetti sia qualcosa che si scopre "assaggiando" gli ingredienti che mettiamo nella ricetta (andrò avanti con la metafora della cucina, siete avvisati!).

Ad alcuni possono piacere le ricette semplici, come una bella grigliata. Come quei gruppi che apprezzano un manager molto presente, che dica a tutti cosa debbano fare, passo dopo passo. Qualcosa di "semplice" ma efficace.
Altri amano qualcosa di creativo, come una ricetta raffinata di un ristorante stellato. Un complesso artifizio di procedure di comunicazione per la gestione del lavoro. Penso ai gruppi che si coordinano da remoto con svariati tool. Li ammiro molto, io avrei difficoltà.
Infine, i gruppi che adorano il fast food. Quello sempre uguale ovunque tu sia, e che piace a quasi tutti i palati. Si perché, diciamocelo, piace a tutti. Come [SCRUM](http://it.wikipedia.org/wiki/Scrum_%28informatica%29)!
Quindi, uscendo dalla metafora, potremmo dire che [SCRUM](http://it.wikipedia.org/wiki/Scrum_%28informatica%29) è la ricetta perfetta perché piace a tutti (o quasi). Ma siamo sicuri che il famoso panino della nota catena americana sia anche il miglior modo di mangiare?

### Imparare dai propri errori
Il nostro gruppo di lavoro, ad esempio, ha passato mesi cercando di applicare la ricetta di [SCRUM](http://it.wikipedia.org/wiki/Scrum_%28informatica%29). Un mezzo disastro. Ci siamo chiesti dove stessimo sbagliando, finché non abbiamo definitivamente fatto indigestione. Piccoli o grandi conflitti interni al team ed una dose di stress altissima. Le nostre [retrospettive](http://en.wikipedia.org/wiki/Retrospective#Software_development) (retaggio dell’agile, non certo perché avessimo capito di cosa si trattasse veramente!) sembravano più sedute di terapia di gruppo, che momenti di condivisione e confronto. Col senno di poi siamo in grado di fare la lista delle cose che non andavano:

* il nostro cliente interno aveva **bisogni che cambiavano** ad una velocità tale da vanificare qualsiasi iterazione pianificata. Questo generava grande frustrazione nel gruppo che sentiva di disattendere il valore dell’abbracciare il cambiamento;
* il gruppo aveva appena perso delle pedine storiche che avevano lasciato in eredità un software costruito su un **dominio complesso**. Non c’era la seniority necessaria per fare delle stime attendibili. Ogni iterazione era un bagno di sangue **contro il tempo**, a danno della qualità del codice o della vita dei programmatori che lavoravano troppo;
* non avevamo capito **l’importanza dello SCRUM Master**, qualcuno che sappia proteggere i valori e le cerimonie di questa pratica. Passavamo da un’iterazione all’altra senza prenderci nemmeno il tempo di celebrare i successi. Fare i panini della nota catena, in casa, senza il particolare controllo qualità che la catena impone ai propri franchise è un esperimento fallito in partenza.

### Cambiare ricetta
Penso che essere agili sia, in primo luogo, non lasciarsi intrappolare da preconcetti. Ecco perché un giorno ho riunito il team ed ho detto che non mi riconoscevo in quel modo di lavorare!
Abbiamo iniziato a pensare ad **una ricetta diversa**, qualcosa che fosse più adatto al nostro palato. Guardando le caratteristiche del nostro gruppo e del contesto in cui lavoriamo abbiamo osservato che:

* ogni giorno le **priorità** sono potenzialmente rimesse in discussione;
vogliamo fare qualità e vogliamo che il progetto sia al passo con la tecnologia;
* abbiamo un team con una buona esperienza ma deve entrare nelle dinamiche di un **dominio complesso**;
* dobbiamo ancora scoprire **la nostra velocity, che comunque cambierà** man mano che prendiamo confidenza col dominio. Insomma, la velocity non sarebbe un parametro utile per le stime;
* **non possiamo rallentare** perché il cliente ha bisogno di noi;
abbiamo un elevato numero di bug;

Il nostro gruppo è composto da **5 programmatori ed un product specialist**, una persona che si occupa di tracciare i bisogni degli utenti, i bug e ci aiuta a far in modo che il prodotto sia bello e curato.
Ricevendo continue richieste, la prima cosa da fare è non perderne traccia. Abbiamo aperto un foglio di calcolo condiviso su Google Drive in cui scriviamo le storie che arrivano al product specialist, assegnamo un’etichetta che mostri al volo di cosa si tratti (frontend, fatture, bug, etc.) e facciamo una stima estremamente generica (è una storia piccola, epica, etc); di fatto si tratta del nostro **backlog**.

Ogni inizio del mese facciamo una **revisione del backlog** insieme al manager per capire se ci sono delle cose più prioritarie di altre e facciamo pulizia.
Uno dei capisaldi della nostra ricetta è la **prioritizzazione continua**. Ingrediente di base, dunque, il [Kanban](http://it.wikipedia.org/wiki/Kanban). 

Abbiamo aperto una board su [Trello](http://www.trello.com) dove far evolvere **le storie (i task)** da una colonna all’altra. Si parte dalla colonna "*prossime cose da fare*", che carichiamo man mano che si svuota, dal backlog o, molto più spesso, con le cose che scopriamo giorno dopo giorno parlando con il cliente (ecco la *prioritizzazione continua*!). Esempio tipico: "se metteste anche quella statistica nella pagina dei report delle vendite avremmo più controllo su quel target di clienti"... fatto!

Ogni storia riporta l’etichetta già indicata nel backlog, così un programmatore ha il polso di quale sia il dominio legato alla storia. Un nuovo arrivato avrebbe più difficoltà a lavorare sul dominio della contabilità. E’ meglio che si concentri su altre storie, col tempo prenderà confidenza anche con quel mondo.

Quando un programmatore è libero **"adotta" una storia**, la porta nella colonna con le cose che stiamo facendo e ci appiccica il proprio avatar. Durante lo sviluppo si occupa di tutto: la raccolta delle informazioni, verifica i mockup, prepara la demo con il cliente, etc. 
Quando ha finito ed è pronto a metterla in produzione la sposta nella colonna "*da provare in produzione*". Il task resta là finché un caso reale non ci permette di dire "funziona!". A quel punto va nelle cose "*fatte*".

In tutte le storie cerchiamo di metterci quanta **più qualità possibile**, facciamo i **test funzionali ed unitari** e teniamo sotto controllo il nostro tool di **continuous integration**.

Le storie su **nuovi domini** sono quelle più critiche, perché portano a fare molte nuove scelte di architettura. Io sono convinto che il **[pair programming](http://it.wikipedia.org/wiki/Pair_programming)** non sia una tecnica per raddoppiare la velocità, piuttosto un modo per **dimezzare la stupidità**! Si può sempre sbagliare, ma prendere una cantonata in due è raro. E’ più probabile che si faccia challenge a vicenda per le scelte importanti. Quindi in questi casi mettiamo insieme due programmatori e facciamo in modo che si confrontino.

Se una storia stenta ad andare in "*fatto*" siamo pronti a metterci anche in 2 o 3 pur di portare la storia a risultato, come ci insegna il [Kanban](http://it.wikipedia.org/wiki/Kanban).

A rotazione (ruotiamo ogni 2 settimane), uno o più programmatori nel team ci aiutano a gestire uno dei problemi che rischiano di compromettere i nostri piani. Li chiamiamo i **wallman**! Ci proteggono dalle **continue interruzioni** che altrimenti il team subirebbe, per gestire i bug o le varie richieste che vengono dal cliente.
Nel tempo "libero" (per fortuna oggi è tanto!) fanno una cosa estremamente importante per la salute del progetto: il **refactoring**! Il team, in pieno spirito [Lean](http://en.wikipedia.org/wiki/Lean_software_development), rilascia tante feature sperimentali a grande velocità. Alcune di queste sono dei piccoli accrocchi su pezzi di codice preesistenti. Quando la prova del mercato ci da un feedback positivo, un wallman ci aiuta a ripulire il codice. In questo modo, il team può concentrarsi sul rilasciare tanto valore per il business, consapevole che qualcuno copre le spalle.

Questa ricetta, come ogni grande ricetta, è una lenta e ragionata combinazione di ingredienti per soddisfare il nostro gusto. **A noi piace!** Non è detto che vada bene con tutti, ma sicuramente rispetta i valori fondamentali di collaborazione, creazione del valore e rispetto che sono propri del manifesto agile.

### La ricetta ideale
Un po' come nella cucina, dove ogni buona ricetta ha una parte dolce ed una nota acida, ogni combinazione di pratiche agili deve rispettare i princìpi fondamentali. Calateli nella vostra realtà! Tutto si migliora confrantandosi, facendo [retrospettive](http://en.wikipedia.org/wiki/Retrospective#Software_development). Alla fine arriva la giusta combinazione che vi farà dimenticare il fast food. E’ decisamente più gustoso un bel panino artigianale!
