---
authors: ["elviro"]
comments: true
date: "2015-09-15"
draft: false
share: true
categories: [Italiano, Software complexity, Functional programming, OOP, Structured programming]
title: "No Country For If Else"
eng: "no-country-for-if-else"

languageCode: "it-IT"
type: "post"
aliases:
  - "/no-country-for-if-else"
---

C'è un **ospite indesiderato** che ci accompagna sempre mentre scriviamo codice e realizziamo progetti software: si tratta del **codice già esistente**, e dobbiamo tener conto della sua complessità man mano che la *code base* aumenta di dimensioni. Un'elevata complessità del codice può rendere le seguenti attività particolarmente difficili:

- comprendere il significato di codice vecchio, scritto da altri o da se stessi;
- tracciare le cause di bug, cioè errori, nel codice;
- eseguire modifiche a una certa procedura;
- aggiungere funzionalità a strutture già esistenti;

Anche approcciando lo sviluppo di nuovo software con [metodologie agili](https://en.wikipedia.org/wiki/Agile_software_development), dobbiamo comunque fare i conti con il codice esistente, e per farlo dobbiamo almeno essere in grado di **comprenderlo senza sforzi eccessivi**. Dunque quando parlo di *complessità* mi riferisco in particolare alla difficoltà con la quale una programmatore è in grado di *ragionare* sul codice. La prefazione del noto testo accademico [Structure and interpretation of computer programs](http://deptinfo.unice.fr/~roy/sicp.pdf) contiene la seguente frase:

> programs must be written for people to read, and only incidentally for machines to execute

Non potrei essere più d'accordo: il tempo speso da un programmatore a scrivere nuovo codice rappresenta solo **una minima parte** del suo tempo totale di lavoro; molto tempo è passato a leggere il codice esistente, per correggerlo, estenderlo, modificarlo o semplicemente comprenderlo. Generando codice più comprensibile, sul quale sia più semplice ragionare, possiamo valutare più facilmente la **correttezza** di quanto abbiamo scritto o, in caso di comportamento non atteso, trovare agevolmente l'errore. Ovviamente tutto questo non deve prescindere dalle tecniche che abitualmente utilizziamo per testare la correttezza del software *a posteriori*: d'altronde, nessuna disciplina tecnico/scientifica è esente da verifiche empiriche, e in effetti l'approccio empirico all'ingegneria del software è preso in considerazione anche in ambito accademico, come dimostra l'[esistenza](http://static.springer.com/sgw/documents/1525357/application/pdf/10664_JournalMetrics_Flyer.pdf) della rivista [Empirical Software Engineering](http://link.springer.com/journal/10664). La mia personale posizione è nel mezzo: il testing è importante, ma è anche importante ragionare *a priori* sul proprio design, sulla sua correttezza, e sulla presenza di eventuale **complessità accidentale**.

## Indice
- [L'importanza dell'astrazione](#l-importanza-dell-astrazione)
- [Misurare la complessità](#misurare-la-complessita)
- [Esempio: soluzione strutturata](#esempio-soluzione-strutturata)
- [Esempio: soluzione OO](#esempio-soluzione-oo)
- [Esempio: soluzione funzionale](#esempio-soluzione-funzionale)
- [Una verifica empirica](#una-verifica-empirica)
- [Conclusioni](#conclusioni)

<a name="l-importanza-dell-astrazione"></a>
## L'importanza dell'astrazione

Come abbiamo visto in un [precedente articolo](http://engineering.facile.it/programmazione-funzionale-perche-preoccuparsi/), più che della complessità intrinseca legata alla logica del nostro software, dobbiamo preoccuparci della [complessità accidentale](http://shaffner.us/cs/papers/tarpit.pdf) che introduciamo adottando **soluzioni non ottimali e inutilmente complicate**: l'utilizzo eccessivo di strutture di controllo *if-else-for-switch* tende a rendere il codice difficile da comprendere, da mantenere e da testare; si veda ad esempio il noto [Arrow Anti-Pattern](http://c2.com/cgi/wiki?ArrowAntiPattern). È da notare che l'introduzione della [programmazione *strutturata*](https://en.wikipedia.org/wiki/Structured_programming), cioè quella basata proprio sulle strutture di controllo appena citate, ha rappresentato **un grande passo avanti** nello sviluppo del software a cavallo tra gli anni '60 e '70: essa ha permesso di programmare a un più alto livello di astrazione rispetto allo stile precedente, rimuovendo la necessità di gestire manualmente l'ordine di esecuzione del codice con il [famigerato](https://www.cs.utexas.edu/users/EWD/ewd02xx/EWD215.PDF) comando `goto`.

I paradigmi di sviluppo cambiano quando i programmatori si ritrovano a dover gestire progetti sempre più corposi e complessi: per evitare di essere imbrigliati dalla eccessiva complessità del proprio codice, o del codice scritto altri che ci troviamo comunque a mantenere, dobbiamo lavorare a **più alti livelli di astrazione**. Quanto alti? Una buona risposta a questa domanda potrebbe essere la seguente:

>  a un elevato livello di astrazione possiamo dire al sistema di fare ciò che vogliamo, senza specificare come farlo

A un adeguato livello di astrazione possiamo concentrarci sul *cosa* lasciando che sia il sistema a decidere il *come*. Ovviamente non esiste un "massimo" livello di astrazione, e in quanto programmatori ci troviamo sempre a lavorare **a metà strada** tra le cariche elettriche in movimento in un microprocessore, e le necessità di business della nostra azienda: dobbiamo tuttavia essere in grado di identificare i casi nei quali stiamo lavorando a un livello di astrazione troppo basso, poiché a tale livello può aumentare molto il rischio di fare errori e introdurre complessità accidentale. Ad esempio, nel momento in cui usiamo una struttura di controllo come *if-else* all'interno di una procedura relativamente grande, stiamo *manualmente* prendendo una decisione in base allo stato del nostro sistema: molto meglio sarebbe sviluppare un elemento software, ad esempio un oggetto, che sia in grado di **prendere quella decisione al posto nostro**; il nostro ruolo, a quel punto, sarà *dichiarare* la funzionalità di quell'elemento. Mescolare il *cosa* con il *come* è una delle principali cause di difficoltà nel comprendere cosa fa un blocco di codice, perché può portare all'oscuramento dell'**intento** di una procedura, legato alla logica di business, con i **dettagli di implementazione**, che sono concetti separati e spesso del tutto indipendenti (uno stesso intento può essere realizzato con diverse implementazioni a più basso livello).

Vediamo un semplice esempio. Supponiamo di avere la seguente funzione:

```swift
func <A> optionalValue1(value: A, cond1: Bool, cond2: Bool) -> A? {
	var x: A?
	if cond1 == true {
		if cond2 == true {
			x = nil
		}
		else {
			x = value
		}
	}
	else {
		if cond2 == true {
			x = value
		}
		else {
			x = nil
		}
	}
	return x
}
```
Anche se la funzione è piuttosto semplice, non è immediatamente evidente cosa stia succedendo; vediamolo passo-passo:

- è dichiarata una variabile `x` di tipo `A?`;
- si verifica la prima condizione:
	- se la prima condizione è vera, si verifica la seconda condizione:
		- se la seconda condizione è vera, la variabile è impostata a `nil`;
		- se la seconda condizione è falsa, la variabile è impostata a `value`;
	- se la prima condizione è falsa, si verifica la seconda condizione:
		- se la seconda condizione è vera, la variabile è impostata a `value`;
		- se la seconda condizione è falsa, la variabile è impostata a  `nil`;
- è ritornata la variabile `x`;

Basta pensarci un attimo per capire che, perché a `x` venga assegnato `value` le due condizioni devono essere semplicemente diverse tra loro. Possiamo inoltre pensare a un'altra semplificazione: non è necessario dichiarare la variabile `x`, basta ritornare `value` se le due condizioni sono diverse, altrimenti ritornare `nil`:

```swift
func <A> optionalValue2(value: A, cond1: Bool, cond2: Bool) -> A? {
	return cond1 != cond2 ? value : nil
}
```
Abbiamo semplificato molto la funzione, rendendola più facile da comprendere: ora **basta un'occhiata** per capire cosa fa. Ma in un caso più complesso, con tante diverse condizioni e percorsi possibili, può non bastare semplificare le condizioni. 

<a name="misurare-la-complessita"></a>
## Misurare la complessità

Un'unità di misura presa spesso in considerazione nel valutare la complessità di un metodo o una funzione è la [complessità ciclomatica](https://en.wikipedia.org/wiki/Cyclomatic_complexity) (CC): essa rappresenta il livello di complessità generato dall'uso di molteplici strutture di controllo in un metodo o una funzione. È possibile calcolare il numero di CC per una procedura strutturata rappresentando quest'ultima con un [grafo diretto](https://en.wikipedia.org/wiki/Directed_graph) e contando il numero di archi e nodi. Ad esempio, possiamo rappresentare la funzione `optionalValue1` con il seguente grafo:

```
[(value,cond1,cond2)]
|
|-> [var x] -> [cond1 == true ?]
               |
               |-y-> [cond2 == true ?]
               |     |
               |     |-y-> [x = nil] --------|
               |     |-n-> [x = value] ------|
               |                             |
               |-n-> [cond2 == true ?]       |
                     |                       |
                     |-y-> [x = value] ------|
                     |-n-> [x = nil] --------|
                                             |-> [return x]
```

Per un singolo grafo connesso, il numero di CC è calcolabile in base alla seguente formula:

```
CC = [numero archi] - [numero nodi] + 2
```

Per la funzione `defaultValue1` si può vedere che CC = 4. Apparentemente la versione semplificata `defaulValue2` ha una complessità ciclomatica inferiore, ma in realtà, come indicato anche nella [pubblicazione originale](http://www.literateprogramming.com/mccabe.pdf) di T.J.McCabe, CC vale solo per una procedura *completamente strutturata*, cioè una procedura con un solo punto di ingresso e un solo punto di uscita; inoltre CC dovrebbe tener conto di **tutti i casi possibili** quando si verifica una condizione, e `if cond1 != cond2` ha appunto 4 casi possibili. Una trattazione delle possibili evoluzioni di CC tenendo conto di molteplici punti di ingresso e uscita per un modulo è disponibile [qui](http://www.acis.pamplin.vt.edu/faculty/tegarden/wrk-pap/SQJ.PDF). Quindi non terremo conto di CC nel resto dell'articolo per i seguenti motivi:

- la complessità che ci interessa è quella relativa alla "comprensibilità" di una funzione, e spesso pattern non strutturati (come `guard clause`,`return` anticipati o *conditional expression* come quella prodotta dall'operatore ternario `?:`) rendono una funzione o un metodo più semplici da capire;
- l'obiettivo è scrivere un programma componendo tante piccole funzioni la cui complessità sia la **minima possibile** (nessuna struttura di controllo), quindi avrà poco valore misurare il numero di CC per ogni funzione;

Mi è parso comunque importante citare la complessità ciclomatica in questo articolo, ma più che *misurare* la complessità accidentale, siamo interessati a **rimuoverla del tutto**, sostituendo le decisioni condizionali tipiche della programmazione strutturata con qualcos'altro. È da notare che una *espressione* condizionale rappresenta un concetto **più semplice** rispetto a una *istruzione* condizionale: nel primo caso, l'intera espressione ritorna semplicemente un valore in base a una o più condizioni; nel secondo caso, una o più istruzioni potrebbero essere eseguite o meno in base allo stato del sistema.

Immaginiamo di avere una funzione di questo tipo:

```swift
func getCorrectValueConsideringConditions <A> (#cond1: Condition<A>, cond2: Condition<A>, cond3: Condition<A> ...) -> A? {
	/// do stuff
}
```
In questo caso possiamo avere un gran numero di condizioni legate a un valore di tipo `A`, e potrebbe essere necessario valutare ciascuna di queste con diversi *if-else* e/o *switch* annidati. Un modo per risolvere questo problema può consistere nell'*astrarre* il concetto stesso di *condizione*, trasformandolo in una *relazione statica* tra un qualche parametro e un valore di tipo `A`: dovremmo quindi realizzare un sistema che sia in grado di *risolvere* un qualunque numero di relazioni, possibilmente in un **qualunque ordine**, e in base ad esse dedurre il valore risultante.

Vediamo un esempio pratico.

<a name="esempio-soluzione-strutturata"></a>
## Esempio: soluzione strutturata

Una persona si reca all'ufficio di collocamento per cercare lavoro: in base ad alcuni attributi (preferenze, giovane/anziano, numero di figli di età inferiore ai 18 anni) essa può essere inviata a un certo sportello per selezionare un impiego tra i vari disponibili, oppure mandata fuori dall'ufficio nel caso in cui il set di attributi non generi una lista di lavori accettabili. Ecco la procedura completa (non è importante il realismo, è solo un esempio):

Un persona **p1** arriva allo sportello **d1** per cercare un impiego; **p1** ha una lista di preferenze lavorative, ma potrebbe accettare anche altri lavori fuori lista; **p1** ha inoltre una lista di lavori che di sicuro non accetterebbe mai;

- se **p1** è giovane si ottiene da **p1** una lista **l1** di preferenze;
	- se **l1** è troppo piccola, si chiede a **p1** di ingrandire la lista con preferenze secondarie;
	- si inseriscono le preferenze nel sistema e si ottiene una lista **l2** di posizioni disponibili;
	- se la lista **l2** è vuota, si chiede a **p1** se vuole visualizzare la lista completa delle posizioni;
	- se **p1** sceglie una posizione, **p1** è inviato allo sportello **d2** per proseguire con la procedura;
- se **p1** è anziano, si chiede a **p1** di scegliere dalla lista **l3** dei lavori adatti per gli anziani;
	- se **p1** sceglie una posizione, **p1** è inviato allo sportello **d3** per proseguire con la procedura;
- se **p1** ha figli, in ogni caso la lista delle posizioni disponibili sarà influenzata dal numero di figli:
	- se **p1** ha 1 figlio con età inferiore ai 18 anni, si presenta a **p1** la lista **l4** dei lavori adatti al suo caso;
	- se **p1** ha 2 figli con età inferiore ai 18 anni, si presenta a **p1** la lista **l5** dei lavori adatti al suo caso;
	- nei due casi precedenti, se **p1** sceglie una posizione, **p1** è inviato allo sportello **d4**;
	- se **p1** ha 3 figli o più, **p1** è inviato allo sportello **d5** per proseguire con la procedura;

Un approccio "strutturato" al problema può consistere nello scrivere una funzione in cui l'intera procedura è espressa con una serie di *if-else* e uno *switch* (nel caso del numero di figli).

Il progetto relativo a questo articolo è disponibile su [GitHub](https://github.com/broomburgo/NoCountryForIfElse): suggerisco di controllare il codice nel progetto man mano che si prosegue nella lettura dell'articolo; i file `.swift` sono semplici file di testo, e possono essere visualizzati con qualsiasi editor. Nel file `common.swift` sono indicate alcune strutture e funzioni comuni tra tutte le soluzioni al problema proposto. In particolare, indicheremo una "persona" con `struct Person`, in sostanza un *value object* immutabile che contiene i vari attributi considerati nel problema:

```swift
struct Person {
    
    let name: String
    let isYoung: Bool
    let childrenCount: Int
    let likedJobsMain: [String]
    let likedJobsSecondary: [String]
    let dislikedJobs: [String]
}
```

L'idea è quella di scrivere una funzione `placeNameForPerson` che ritorni il nome della *destinazione successiva* per la persona in ingresso al problema: tale destinazione può essere un altro sportello `d2,d3,d4,d5` oppure `outside` nel caso in cui la persona lasci l'ufficio di collocamento senza lavoro. La funzione `placeNameForPerson_structured` ritorna appunto il nome del luogo di destinazione della persona in ingresso, e richiede anche un secondo input, un `DeskWithJobs` cioè uno sportello con diverse liste di lavori disponibili. Nel caso strutturato, l'intera logica di esecuzione è indicata nella funzione `placeNameForPerson_structured`, ed è appunto basata su verifiche condizionali annidate, legate alle caratteristiche dei parametri in ingresso.

Si vede subito che è piuttosto difficile capire cosa succeda nella funzione semplicemente leggendola: probabilmente, in un "vero" software, una funzione di questo tipo sarebbe annotata con **un gran numero di commenti**. Inoltre, si vede facilmente che per tracciare un bug in una funzione del genere sarebbe necessario un debugging passo-passo, perché il gran numero di condizioni espresse rende la funzione sostanzialmente non testabile in maniera efficiente. Infine, se ci trovassimo a dover modificare la procedura, modificando delle condizioni o introducendone altre, avremmo vita dura.

Il problema di fondo è in realtà dovuto al fatto che, con un approccio strutturato, stiamo **mescolando l'intento con l'implementazione**: in questo caso l'intento è associare gruppi di condizioni a specifici sportelli "target", ma nella soluzione strutturata tali relazioni sono "nascoste" nel codice e devono essere dedotte da esso; se ad esempio ci venisse chiesto qual è lo sportello per una persona anziana con 2 figli, dovremmo necessariamente seguire il flusso di codice:

- poiché la prima condizione è relativa al numero di figli, entriamo subito nel branch `childrenCount != 0`;
- a questo punto entriamo nel `case 2` per il numero di figli;
- c'è una condizione in base alla quale il codice verifica che c'è un lavoro disponibile;
- a scelta positiva lo sportello di destinazione è il "d4";
- dov'è la condizione "giovane/anziano"? C'è un bug nel codice? Analizzando il ramo `childrenCount == 0` possiamo verificare che tale condizione è effettivamente analizzata, ma è semplicemente irrilevante per l'altro ramo;

È una strada relativamente lunga per capire una cosa semplice, ed è da notare che se volessimo fare delle modifiche, dovremmo sempre scorrere nuovamente l'intera procedura per verificare che abbiamo coperto tutti i casi possibili, e che non ci siano conflitti o ambiguità.

Possiamo fare di meglio.

<a name="esempio-soluzione-oo"></a>
## Esempio: soluzione OO

Progettare una soluzione a oggetti è spesso un'operazione **tanto razionale quanto creativa**: in genere è possibile immaginare moltissimi design OO per risolvere uno stesso problema. Come detto, il nostro intento è di *astrarre* il concetto di *condizione*; basandoci su questo approccio, proviamo fissare qualche punto, in modo da identificare le responsabilità e i comportamenti da assegnare alla classi:

- vogliamo essere in grado di rappresentare ciascuna condizione come se fosse un *check* separato dagli altri, quindi avremo certamente qualcosa che assomiglia a un tipo *Check* caratterizzato da metodi che verificano se una certa persona passa un controllo o no;
- vogliamo rappresentare separatamente i check relativi a un attributo della persona, tipo il numero di figli, da quelli relativi alla lista di lavori tra cui scegliere;
- vogliamo costruire oggetti che siano in grado di combinare più check in un unico check;
- vogliamo creare un oggetto che rappresenti una lista di check e sia in grado di valutare se la persona passi uno dei check della lista, uno e solo uno, oppure non passi alcun check;

Il file `oo.swift` contiene l'implementazione completa della soluzione a oggetti. È definito un `protocol PersonCheckType`, cioè un'interfaccia che dichiara un metodo `personIsValid` che verifica se una certa `Person` passa il check:

```swift
protocol PersonCheckType {
    func personIsValid(person: Person) -> Bool
}
```

Sono quindi dichiarate alcune classi di tipo `PersonCheckType` che permettono di verificare singolarmente ciascun attributo intrinseco di una `Person`.

Il tipo `JobsType` dichiara semplicemente un attributo `availableJobs` per raccogliere i lavori disponibili nei vari casi: da esso derivano altri *check*, a loro volta di tipo `PersonCheckType`, che verificano se una persona sia valida dal punto di vista dei lavori preferiti.

Infine, il tipo `NextDeskType` dichiara un metodo `nextDeskNameForPerson` in base al quale è possibile definire quale sia il nome del prossimo sportello per una persona; il metodo ritorna `String?` perché per una certa persona potrebbe non essere disponibile alcuno sportello:

```swift
protocol NextDeskType {
    func nextDeskNameForPerson(person: Person) -> String?
}
```

Dal tipo `NextDeskType` deriviamo le seguenti classi: 

- `CheckNode`, che rappresenta un nodo nella struttura decisionale del software;
- `CheckStructure`, che rappresenta l'intera struttura di nodi;

In effetti, l'implementazione di `nextDeskNameForPerson` in `CheckStructure` corrisponde esattamente alla soluzione del problema posto: trovare il nome dello sportello di destinazione.

L'idea è quella di creare alcuni semplici **check unitari**, usando le varie classi di tipo `PersonCheckType`, e combinare questi check in oggetti concreti di classe `CheckNode`, definendo la nostra `CheckStructure`. In puro stile OO, abbiamo definito classi chiamate `MultipleCheck` e `ComposedCheck` per poter comporre più check, e `FailingCheck` per poter incapsulare un check di cui si richiede il fallimento. Nessuno dei metodi implementati presenta strutture decisionali che influenzino la *business logic*.

Come esempio si riporta l'implementazione della classe `ChildrenCountCheck`, che si occupa di verificare se una persona abbia un certo numero di figli:

```swift
class ChildrenCountCheck: PersonCheckType {
    
    let childrenCount: Int
    init(_ childrenCount: Int) {
        self.childrenCount = childrenCount
    }
    
    func personIsValid(person: Person) -> Bool {
        return person.childrenCount == childrenCount
    }
}
```

L'oggetto `structure` definito nel file `main.swift` presenta la lista delle condizioni, sotto forma di oggetti adeguatamente configurati; come si può vedere, la lista è ben leggibile e le condizioni sono chiare: siamo quindi riusciti a *dichiarare* l'intento in maniera semplice e separata dai dettagli di implementazione.

<a name="esempio-soluzione-funzionale"></a>
## Esempio: soluzione funzionale

Nello strutturare la soluzione OO abbiamo adottato un **principio di composizione**: oggetti dal comportamento semplice, adeguatamente preparati, composti tra loro per poter ottenere strutture più complesse. In particolare ciascuno dei nostri oggetti rappresenta uno specifico *comportamento unitario*, e i vari *comportamenti* sono combinati per ottenere un *comportamento composto*. **L'astrazione regge**, e riusciamo ad accettare il fatto che gli oggetti *check* siano scatole chiuse che incapsulano una singola decisione su una persona: creando una scatola più grande, con dentro scatole più piccole, possiamo rappresentare una condizione più complessa. Tuttavia ci sono due problemi tipici dell'approccio OO:

- ogni singola classe richiede molto codice solo per costruire l'astrazione di "decisione nella scatola";
- la composizione tra oggetti è basata sui metodi degli oggetti stessi, la qual cosa comporta, di nuovo, la scrittura di molto codice, che può offuscare l'intento;

È possibile ottenere più chiaramente lo stesso risultato, cioè creare una struttura decisionale complessa a partire da componenti semplici, usando uno stile di **programmazione funzionale**. Il codice funzionale è descritto nel file `functional.swift`. Partiamo sempre dall'immutabile `struct Person` e definiamo, per comodità, un altro semplice contenitore di dati, `struct DeskNode`, che accoppia il nome di un nodo al nome di uno sportello di destinazione. Per il resto, definiremo solo ed esclusivamente funzioni, modellizzando l'intero problema con una serie di trasformazioni di dati.

In questo caso `PersonCheck` è **un tipo di funzione**, che associa una `Person` a un `Bool`. Definiamo inoltre il tipo di funzione `PersonNode`, che associa un certo `DeskNode` a una `Person`, e che ha sostanzialmente lo scopo che aveva il metodo `nextDeskNameForPerson` nella soluzione a oggetti.

Rappresentiamo infine i vari "nodi" con una funzione del tipo `JobsNode`, che costruisce un `PersonNode` in base a un certo `PersonCheck`: in questo modo, per costruire una lista di `PersonNode` sarà sufficiente applicare i vari *check* (`PersonCheck`) ai rispettivi nodi (`JobsNode`).

Il metodo più spesso adottato in programmazione funzionale per comporre le funzioni è quello di definire degli **operatori personalizzati**. Vogliamo ad esempio comporre i vari `PersonCheck` come se componessimo dei semplici `Bool`, quindi ad esempio definendo degli operatori **AND** e **OR**. Ad imitazione dei classici `&&` e `||` definiremo rispettivamente gli operatori `<&>` e `<|>`, copiando regole di [precedenza e associatività](https://en.wikipedia.org/wiki/Operator_associativity) dalla libreria standard. Per migliorare la leggibilità e la chiarezza della composizione tra funzioni, definiremo anche un operatore di *applicazione di funzione*, nel seguente modo:

```swift
infix operator <*> {
    associativity left
    precedence 100
}
func <*> <A,B> (left: A -> B, right: A) -> B {
    return left(right)
}
```

Vediamo dalla definizione che l'operatore `<*>` in sostanza *applica* un argomento (posizionato a destra dell'operatore) a una certa funzione (posizionata a sinistra dell'operatore): esso rappresenta quindi un modo alternativo di applicare una funzione rispetto allo scrivere i suoi argomenti **tra parentesi**, ed è particolarmente adatto a migliorare la leggibilità del codice quando usiamo [funzioni di ordine superiore](https://en.wikipedia.org/wiki/Higher-order_function). Vediamo gli operatori all'opera nel file `main.swift`, quando definiamo i nostri nodi, ad esempio:

```swift
node("old", nextDeskName: "d3")
    <*> oldCheck
    <&> mustFail(childrenChecks)
    <&> checkExtendedJobs(availableJobsElderly)
```

La funzione sopraindicata costruisce un `PersonNode` chiamato *old* che ritorna lo sportello *d3* se la `Person` in ingresso rispetta un check composto formato dalle seguenti funzioni:

- `oldCheck`: la persona è anziana;
- `mustFail(childrenChecks)`: la persona non deve avere figli;
- `checkExtendedJobs(availableJobsElderly)`: l'insieme dei lavori adatti agli anziani e l'insieme dei lavori preferiti dalla persona devono avere una qualche intersezione;

A un primo impatto lo stile funzionale può risultare poco comprensibile, ma è sufficiente capire le astrazioni di base sulle quali è realizzata la composizione funzionale per poterne apprezzare **la potenza e l'espressività**. L'array `nodes`, nel file `main.swift`, contiene tutti i nodi del problema, rappresentati come funzioni: rispetto  alla soluzione a oggetti contenuta in `structure`, quella funzionale appare più chiara e leggibile.

<a name="una-verifica-empirica"></a>
## Una verifica empirica

Nel progettare le soluzioni "OO" e "funzionale" si è specificamente scelto di costruire strutture di verifica nelle quali l'ordine con il quale i check sono eseguiti fosse irrilevante: in questo modo rimuoviamo una nota causa di complessità accidentale, quella del "controllo", e cioè dell'**ordine** con il quale le operazioni sono eseguite (ne abbiamo parlato [qui](http://engineering.facile.it/programmazione-funzionale-perche-preoccuparsi/#cause-di-complessita)). Per fare ciò è tuttavia necessario che i check presenti nella lista **non siano ambigui**: al massimo un solo check deve poter passare per una certa persona. Probabilmente sarebbe possibile impostare un **metodo formale** per poter provare che un determinato set di condizioni sia non-ambiguo, e scrivere quindi un test unitario basato su tale prova, ma in questo caso un approccio *empirico* sembra più adatto, e **più semplice**.

Si è quindi scelto di procedere seguendo il metodo [QuickCheck](https://hackage.haskell.org/package/QuickCheck): si tratta di una libreria scritta per il linguaggio [Haskell](https://www.haskell.org) che permette di testare in modo automatico che le funzioni rispettino determinate *proprietà*. L'idea è quella di verificare che una funzione si comporti sempre nello stesso modo, testandola con **un gran numero di input casuali**: nel nostro caso la funzione è proprio la lista di check, e l'input è un valore di tipo `Person`. Per verificare che i check non siano ambigui possiamo semplicemente provarli con un gran numero di `Person` generate casualmente: se per un certo valore di `Person` due o più check risultano positivi, l'esecuzione del test si interrompe e sono stampati in console i nomi dei check incriminati. Se i check ambigui sono due, sarà sufficiente rendere uno di essi più specifico, ad esempio richiedendo che l'altro fallisca. Useremo qui il termine *QuickCheck* per indicare il test empirico implementato, ma in realtà la libreria originale fa molto di più: rimando alla pagina [linkata](https://hackage.haskell.org/package/QuickCheck) per tutte le informazioni.

Le funzioni `quickCheck_oo` e `quickCheck_functional` implementano la funzionalità richiesta: generano un certo numero di `Person` casuali, e verificano che il numero di `check` superato per ogni `Person` sia 0 o 1. Possiamo sfruttare immediatamente questo test aggiungendo un nuovo nodo. Supponiamo ad esempio che a un certo punto, dopo qualche tempo in cui il codice è stato usato con successo per risolvere il problema, si scelga di aggiungere una nuova condizione: è stato appositamente aperto uno sportello per persone anziane con un solo figlio. Si noti come modificare il codice della funzione `placeNameForPerson_structured` sarebbe piuttosto complicato: probabilmente dovremmo modificare interi branch decisionali, **aggiungendo la stessa condizione in diversi punti**, e la probabilità di sbagliare sarebbe molto alta. Con le altre due soluzioni, invece, si tratta semplicemente di aggiungere un nuovo check. Tuttavia, dopo aver aggiunto il nodo, se eseguiamo il programma vedremo che QuickCheck fallirà, stampando il seguente messaggio

```
ambiguous nodes: [1 child, 1 child old]
```

QuickCheck ci sta informando che i nodi chiamati "1 child" e "1 child old" sono **ambigui**: in effetti, se la `Person` ha un figlio ed è anziana, passeranno i check per entrambi i nodi. Per risolvere il problema sarà sufficiente introdurre una condizione di fallimento nel nodo "1 child": perché esso passi, deve fallire il check di anzianità.

Nel progetto è implementata anche una funzione `quickCheck_consistency` che esegue un test di sicurezza del tipo *sanity check*: verifica infatti che lo sportello di destinazione di un gran numero di `Person` generate casualmente rimanga lo stesso nei 3 metodi implementati.

<a name="conclusioni"></a>
## Conclusioni

Abbiamo visto tre metodi per risolvere uno stesso problema relativamente complesso, e in due di essi siamo riusciti a ottenere un sistema **più facilmente gestibile**, estendibile e modificabile in sicurezza, lavorando a un **più alto livello di astrazione**. In particolare il metodo "funzionale" ci ha permesso di scrivere codice più compatto, caratterizzato da un intento più evidente, al costo (basso) di un passo iniziale di astrazione in più, rappresentato dagli operatori personalizzati. Ma la chiave di lettura dell'intero processo è ancora una volta da ricercarsi nella già citata frase presente in *Structure and Interpretation of Computer Programs*, che riporto nuovamente:

> programs must be written for people to read, and only incidentally for machines to execute

Adottando uno stile più **dichiarativo**, in cui il codice proprio della *business logic* "dichiara l'intento", descrivendo il "cosa" e non il "come", è possibile scrivere software più chiaro e leggibile. Tale software sarà probabilmente anche più *testabile*, perché basato su **componenti atomici** a più basso livello, individualmente facili da testare, che sono combinati attraverso strutture di composizione, anch'esse agilmente testabili e provabili. L'obiettivo finale è quello di realizzare architetture software che siano in grado di **risolvere automaticamente i problemi**, descritti nel codice sfruttando gli **elementi costruttivi di base**. Abbiamo infine visto come un approccio misto tra "ragionato" e "empirico" possa garantire ottimi risultati; il ragionamento *a priori* e la verifica *a posteriori* sono entrambi strumenti utili, ma la loro unione risulta essere particolarmente potente: d'altronde, ogni disciplina scientifica richiede sia il ragionamento logico-matematico per la definizione delle teorie, sia la verifica sperimentale per poterne testare la validità.

Gli strumenti sono a disposizione, basta **iniziare ad usarli**.
