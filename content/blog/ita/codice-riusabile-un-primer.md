---
authors: ["elviro"]
comments: true
date: "2015-12-09"
draft: false
share: true
categories: [English, Swift, Functional Programming, Code Reuse]
title: "Codice riusabile: un primer"

languageCode: "it-IT"
type: "post"
aliases:
  - "/codice-riusabile-un-primer"
---

[L'ultima volta](http://engineering.facile.it/optionals-in-objective-c-ita/) abbiamo visto una possibile implementazione del tipo [Optional](https://developer.apple.com/library/prerelease/ios/documentation/Swift/Conceptual/Swift_Programming_Language/TheBasics.html#//apple_ref/doc/uid/TP40014097-CH5-ID330) in Objective-C; l'obiettivo primario dell'articolo era quello di importare in Objective-C uno strumento frequentemente utilizzato in Swift, ma usare una classe come `Optional` può essere considerato un'applicazione di un concetto molto più generale: il **riutilizzo del codice**. In effetti, `Optional` non è legato a un particolare dominio, e può essere riutilizzato più e più volte in molti progetti: questo è esattamente ciò che accade in Swift. Ma a pensarci bene, questo è ciò che accade per una grande varietà di *classi* in Objective-C (e di *tipi* in Swift): ad esempio, `NSArray` e `Array` sono entrambi costrutti che espongono una specifica interfaccia, possiedono una certa implementazione, e vengono riutilizzati continuamente in metodi e funzioni. `NSArray` e `Array` non sono legati a un particolare dominio, e possiedono due importanti caratteristiche:

- sono **generici**, quindi non sono legati a un particolare sottotipo: un array di numeri e un array di stringhe funzioneranno esattamente allo stesso modo per quanto riguarda le funzioni strettamente associate agli array, come il conteggio degli elementi, rimuovere o aggiungere un elemento, mappare, filtrare, ridurre e così via;
- sono **componibili**, quindi possono essere combinati tra loro e con altri oggetti più specifici, mantenendo comunque lo stesso **comportamento predicibile**: un array di *clienti*, ciascuno con il suo array *prodotti acquistati*, può essere mappato in un array di array di *prodotti*, che può essere appiattito in un array di *prodotti*, che può essere ridotto a un numero che rappresenta il costo totale;

`Array`, come `Optional`, è un ottimo esempio di codice perfettamente riutilizzabile, ma ci sono molti altri costrutti che soddisfano le regole di cui sopra, cioè *genericità* e *componibilità*. Gli oggetti specifici di dominio, invece, tipicamente non soddisfano queste regole, anche quando un oggetto sembra essere sufficientemente generico da poter essere riutilizzato. Una classe `Cliente`, ad esempio, potrebbe sembrare un buon candidato, ma cercare di usare la stessa classe in due diversi progetti si rivelerebbe **una pessima idea**: probabilmente saremo costretti a **specializzare** la classe per ciascun progetto, portando a divergenza tra le implementazioni, o usare diversi livelli di **indirezione**, perché stiamo inserendo a forza un oggetto in un dominio che non gli appartiene.

C'è un'altra cosa da considerare: se davvero vogliamo riutilizzare del codice, dovremo anche stare attenti alle nostre **scelte di design**, perché non tutti i design permettono l'utilizzo di codice generico. Potrei dimenticarmi dell'esistenza dei tipi `Optional` e `Array`, e creare classi che non espongono mai i loro componenti di base; ad esempio, potrei creare una classe che rappresenta una collezione di oggetti opzionali, senza mai esporre nell'interfaccia il fatto che sto utilizzando `Array` e `Optional` nell'implementazione: ne risulterebbe una certa **complicazione** della firma dei metodi, ma è possibile. Ma, a pensarci bene, è una buona idea? **Ne dubito**. Creare una nuova e specifica classe, non componibile, per ogni possibile necessità produrrà migliaia di linee di codice **boilerplate**, interfacce complicate, e nomi di classi assurdamente lunghi. Se vogliamo scrivere codice riutilizzabile, oltre a seguire le due regole già viste, dobbiamo anche prestare attenzione al modo in cui architettiamo le nostre app, e il principio di design da seguire in questo caso è il principio di **composizione**, che può essere sintetizzato con la seguente frase:

> Costrutti e comportamenti specifici del dominio dovrebbero essere realizzati componendo blocchi costruttivi atomici e generici.

Ciò significa sostanzialmente che, invece di creare di volta in volta una specifica implementazione per ovviare alle nostre necessità, dovremmo costruire le nuove funzionalità aggregando oggetti atomici già definiti. Sembra un'idea complessa, e forse dovremmo fermarci un attimo e chiederci se scrivere codice riutilizzabile sia davvero una buona idea. La mia risposta è un definitivo **si**, per i seguenti motivi:

- scrivendo componenti riutilizzabili possiamo **incapsulare** uno specifico comportamento, evitando di dover riscrivere di continuo sostanzialmente lo stesso codice;
- potremmo disporre di **linee guida** per progettare l'architettura delle nostre app, perché non avremmo bisogno di definire sempre nuove interfacce per rappresentare gli stessi comportamenti;
- un componente riutilizzabile è molto facile da **testare**, perché piccolo e atomico, quindi usandolo potremmo confidare maggiormente sulla correttezza del nostro codice;

Questi stessi vantaggi si ottengono, infatti, proprio usando oggetti come `Array` e `Optional`. Ma, come ho detto, esistono molti altri componenti di questo tipo, e in effetti definire le loro interfacce è un problema a sé, di non facile soluzione: lo trovo tuttavia un problema molto interessante. Seguendo le regole *generico* e *componibile* abbiamo già una linea guida per definire nuovi componenti riusabili, ma per chiudere il cerchio abbiamo bisogno di una terza regola, non meno importante:

- un oggetto riusabile deve essere **semplice**, cioè deve avere una unica responsabilità, lineare e facile da descrivere;

`Optional` segue questa regola: è un contenitore generico per un oggetto che potrebbe esserci, oppure no. `Array` segue questa regola: rappresenta una collezione ordinata di oggetti, ai quali posso accedere in un tempo costante. Proviamo a creare un nuovo componente riutilizzabile applicando le regole.

Consideriamo questo caso d'uso reale (piuttosto frequente): diversi oggetti devono essere notificati quando un certo oggetto cambia il suo stato, o compie qualche azione. Un modo per risolvere il problema è rappresentato dall'[observer pattern](https://en.wikipedia.org/wiki/Observer_pattern), che consiste nel far implementare ai nostri oggetti un'interfaccia che indica un comportamento di tipo [publish-subscribe](https://it.wikipedia.org/wiki/Publish/subscribe). Tuttavia, far implementare agli oggetti un'interfaccia **non è riusabile**: ogni volta in cui useremo questo pattern finiremo per **riscrivere** codice molto simile. Vogliamo invece incapsulare il comportamento alla base di questo pattern in un oggetto generico, e riutilizzare quell'oggetto. Quanto segue è una possibile implementazione di tale oggetto, quella che personalmente uso in produzione, ma ce ne sono molte altre: in effetti, l'idea di trovare un modo generico e componibile di rappresentare e manipolare *flussi* di segnali osservabili ha fatto nascere un intero paradigma di programmazione, chiamato [functional reactive programming](https://en.wikipedia.org/wiki/Functional_reactive_programming). Ma nel nostro caso siamo interessati a creare un ben più semplice oggetto **Signal**.

La classe `Signal` che vogliamo implementare deve rappresentare un *aggiornamento* per un certo oggetto: l'aggiornamento può riguardare qualsiasi cosa, per esempio un nuovo valore per un attributo, o il fatto che una certa azione è stata compiuta. Useremo il linguaggio Swift, grazie al quale potremo godere anche di ottima sicurezza sulla manipolazione dei **tipi** coinvolti. Ecco in sintesi cosa chiediamo alla classe `Signal`:

- deve avere un sottotipo **parametrico**, che sarà appunto il tipo del valore segnalato ad ogni aggiornamento;
- deve dichiarare un metodo `observe`, che prende in ingresso una *closure* che rappresenta l'azione da compiere a ogni aggiornamento;
- deve dichiarare un metodo `send`, che prende in ingresso un nuovo valore del tipo sottostante;
- deve dichiarare semplici metodi di composizione, che seguano le convenzioni classiche per i nomi, come `map` per generare un nuovo `Signal` da uno esistente trasformando il valore osservato, e `forwardTo` per fare in modo che un altro `Signal`, quando produce un nuovo valore, *attivi* il segnale di partenza;

Dal punto di vista semantico, possiamo descrivere un `Signal` come un contenitore di un valore che esisterà a un certo punto nel futuro, e continuerà ad aggiornarsi indefinitamente.

Segue l'intera implementazione della classe `Signal`:

``` swift
public enum SignalContinuation
{
  case Continue
  case Stop
}

public class Signal<Subtype>
{
  private var observers: [(Subtype -> SignalContinuation)] = []

  public init() {}

  public func observe (observeFunction: Subtype -> SignalContinuation)
  {
    observers.append(observeFunction)
  }

  public func send (value: Subtype)
  {
    var continuations: [(Subtype -> SignalContinuation)] = []
    while observers.count > 0
    {
      let observer = observers.removeFirst()
      let continuation = observer(value)
      switch continuation
      {
      case .Continue:
        continuations.append(observer)
      case .Stop: break
      }
    }
    observers = continuations
  }

  public func forwardTo (otherSignal: Signal<Subtype>) -> Signal
  {
    observe { action in
      otherSignal.send(action)
      return .Continue
    }
    return self
  }

  public func forwardTo <OtherSubtype> (
    otherSignal: Signal<OtherSubtype>,
    mappingFunction: Subtype -> OtherSubtype) -> Signal
  {
    observe { action in
      otherSignal.send(mappingFunction(action))
      return .Continue
    }
    return self
  }

  public func map <OtherSubtype> (transform: Subtype -> OtherSubtype) -> Signal<OtherSubtype>
  {
    let mappedSignal = Signal<OtherSubtype>()
    forwardTo(mappedSignal, mappingFunction: transform)
    return mappedSignal
  }
}
```

L'implementazione è semplice, ma potente. Un problema affrontato di frequente quando si implementa l'*observer pattern* è la gestione delle disiscrizioni; è responsabilità di chi osserva smettere di farlo, e in questa implementazione di `Signal` la cosa è gestita direttamente nella *closure* passata al metodo `observe`: la *closure* deve ritornare un valore di tipo `SignalContinuation` che può essere appunto `.Continue` o `.Stop`. Un altro problema è la gestione della memoria: dobbiamo assicurarci che quando la memoria di un osservatore è rilasciata, questo debba anche smettere di osservare i segnali, o il messaggio verrà inviato a un puntatore non valido, con conseguente crash dell'app. Swift definisce alcuni descrittori di *memory ownership*, e `weak` fa al caso nostro: sarà sufficiente inserire una *guard clause* all'inizio della *closure* passata al metodo `observe`; se l'oggetto è diventato `nil`, la *closure* ritornerà `.Stop`. L'esempio seguente mostra un possibile utilizzo della classe `Signal`, incluso il meccanismo appena descritto:

``` swift
class Sender
{
  let signal = Signal<Int>()

  func sendNew(value: Int)
  {
    signal.send(value)
  }
}

class Receiver<Type: CustomStringConvertible>
{
  func startObserving(signal: Signal<Type>)
  {
    signal.observe { [weak self] value in
      guard let this = self else { return .Stop }
      this.printNewValue(value)
      return .Continue
    }
  }

  func printNewValue(value: Type)
  {
    print(value)
  }
}

let sender = Sender()
let receiver = Receiver<Int>()

receiver.startObserving(sender.signal)

sender.signal.send(3)
sender.signal.send(5)
sender.signal.send(10)
sender.signal.send(20)
/// this will print 3, 5, 10, 20 on console
```

Nell'esempio possiamo osservare l'applicazione dei concetti introdotti all'inizio dell'articolo: invece di creare una nuova interfaccia per lo stesso comportamento, stiamo direttamente usando e riusando l'oggetto `Signal`. Un esempio più complesso potrebbe essere rappresentato dall'aggiunta di un *resonator*, cioè di un oggetto che possiede un segnale che risuona con un altro, quindi:

``` swift
class DoublingResonator
{
  let signal = Signal<Int>()

  func resonateWith(otherSignal: Signal<Int>)
  {
    otherSignal.forwardTo(signal) { $0*2 }
  }
}

let resonator = DoublingResonator()
resonator.resonateWith(sender.signal)

let receiver = Receiver<Int>()

receiver.startObserving(resonator.signal)

sender.signal.send(3)
sender.signal.send(5)
sender.signal.send(10)
sender.signal.send(20)
/// this will print 6, 10, 20, 40 on console
```

Ci sono molte altre opzioni per comporre segnali, ma finché non ne avremo bisogno sarà meglio mantenere la classe **semplice**: gradualmente nel tempo potremo aggiungere nuove funzionalità, e fintanto che queste saranno sufficientemente generiche e propriamente testate, saremo in grado di usarle in tutti i nostri progetti.

Per concludere, definire le giuste astrazioni per conseguire riusabilità del codice non è un problema di semplice soluzione: molte pubblicazioni accademiche affrontano il problema (il classico articolo [Software Reuse](http://www.biglever.com/papers/Krueger_AcmReuseSurvey.pdf) di Charles W. Krueger fornisce una buona panoramica delle tecniche considerate), e il motivo per il quale la [teoria delle categorie](https://it.wikipedia.org/wiki/Teoria_delle_categorie) ha trovato molte applicazioni in programmazione funzionale è perché essa offre un eccellente insieme di astrazioni per affrontare diverse classi di problemi. Tuttavia sono convinto che i vantaggi legati allo scrivere codice riutilizzabile siano molti, e che poter basare un design sulla composizione di oggetti atomici sia **un degno obiettivo** da perseguire.