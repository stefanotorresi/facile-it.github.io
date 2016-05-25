---
authors: ["elviro"]
comments: true
date: "2015-06-05"
draft: false
share: true
categories: [Italiano, Swift, Software testing, Xcode]
title: "Swift testing avanzato: stubbing e test asincroni"

languageCode: "it-IT"
type: "post"
aliases:
  - "/swift-testing-avanzato-stubbing-e-test-asincroni"
---

[In un precedente articolo](http://engineering.facile.it/xcode-testing-in-pratica/) abbiamo visto le impostazioni di base in Xcode per la scrittura dei **test unitari**: abbiamo evidenziato inoltre **l'importanza e l'utilità intrinseca dei test**, attraverso un semplice esempio riguardante un caso d'uso tipico. Nel presente articolo vedremo alcune tecniche un po' più avanzate: 

- implementeremo uno **Stub Object** in Swift;
- analizzeremo un altro caso di test *asincrono*;

## Lo *Stub Object*

Uno *Stub Object* (per il resto dell'articolo, *stub*) rappresenta un'istanza di una certa classe, la quale *mima* una vera classe presente nella nostra *code base*:  l'istanza si comporta esattamente come una equivalente istanza della classe mimata, tranne alcune differenze, ad esempio **alcuni metodi possono essere sovrascritti** per poter fornire **un determinato output** utile per i test. Nell'implementare uno *stub* non è generalmente consentito modificare dettagli di logica interni relativi alla classe che stiamo mimando, ma **è possibile sovrascrivere metodi pubblici**, in modo che essi ritornino i valori che vogliamo, oppure che svolgano una particolare procedura necessaria per i test. Tanto per fare un esempio pratico potremmo *stubbare* una classe che ci fornisce la data precisa in un certo istante, in modo da ottenere una data diversa da usare nei test, oppure un client che chiede a un server delle informazioni su un utente, in modo da far ritonare al client *stub* delle informazioni arbitrarie.

Gli *stub* fanno parte di una classe di strumenti che sono usati nell'ambito dello *Unit Testing* per verificare che determinati oggetti rispettino precisi **contratti** stipulati tra essi: il classico articolo di Martin Fowler [Mocks Aren't Stubs](http://martinfowler.com/articles/mocksArentStubs.html) è solitamente considerato un buon punto di riferimento per comprendere i possibili strumenti usati a tal scopo. L'idea è che, dal punto di vista dei nostri test unitari, un oggetto risulterà essere *correttamente implementato* se avrà rispettato la sua parte del *contratto* stipulato con altri oggetti, con la seguente conseguenza: 

> se l'oggetto A è correttamente implementato e l'oggetto B rispetta il contratto stipulato con l'oggetto A, allora anche l'oggetto B è correttamente implementato 

Gli *stub* permettono di **disaccoppiare le logiche di funzionamento degli oggetti** perché, se abbiamo verificato il corretto funzionamento dell'oggetto A (attraverso appropriati test unitari), possiamo usare uno *stub* di tale oggetto per verificare il corretto funzionamento dell'oggetto B nei suoi confronti, eliminando qualsiasi dipendenza di B nei confronti della logica interna dell'oggetto A. L'obiettivo è scrivere quindi dei **test veramente unitari** per l'oggetto B, anche se questo dipende dall'oggetto A. Il classico talk [Integration Tests are a scam](https://vimeo.com/80533536) di J.B. Rainsberger fornisce un punto di vista particolarmente "radicale" sull'argomento.

Il classico caso in cui uno *stub* risulta utile è quello del client che fa una richiesta al server: se la logica di implementazione del server è sotto il nostro controllo possiamo **testare che il server rispetti il contratto con i suoi client** generando degli *stub* di questi per ogni possibile richiesta; d'altra parte, per verificare che i client siano in grado di **gestire correttamente ogni possibile risposta del server**, possiamo generare uno *stub* del server che fornisca ogni possibile risposta: in questo modo possiamo testare in maniera disaccoppiata client e server.

## Un caso d'uso: *Location Services*

Il caso d'uso che tratteremo è relativo ai **servizi di localizzazione**, uno strumento frequentemente utilizzato dagli sviluppatori iOS e in generale da chi sviluppa su **smartphone**: per tali sviluppatori, quello dei servizi di localizzazione è un tema molto importante, **fonte di complessità e sfide** che caratterizzano tipicamente il mondo *mobile*. Non è infatti possibile ottenere l'attuale posizione GPS in un preciso momento: l'ultima posizione GPS disponibile si basa su una regressione, effettuata dal sistema, dei dati ricevuti da diversi sensori - antenne wifi, cellulare e ovviamente GPS, e spesso anche i sensori di movimento - in diversi momenti; **il sistema può inviare notifiche alla nostra app in qualsiasi momento**, ed essa deve essere in grado di "digerire" correttamente i dati ricevuti, interpretando anche eventuali errori. Scrivere dei test unitari per verificare che la nostra app geolocalizzata funzioni correttamente può presentare non poche difficoltà: non possiamo infatti affidarci ai "veri" sensori del dispositivo, perché probabilmente i test verranno eseguiti sul simulatore, oppure magari su un vero iPhone la cui ricezione GPS in un certo istante potrà essere più o meno buona, ma essa rappresenterà un solo caso possibile rispetto a tanti. Vogliamo essere in grado di **testare ogni possibile risposta** del sistema GPS del dispositivo, dall'assenza totale di segnale al cambiamento frequente di posizione, alla **mancata autorizzazione** da parte dell'utente per l'accesso ai servizi di localizzazione.

Supponiamo ad esempio che una certa classe, diciamo un `UIViewController`, esegua delle **azioni che dipendono dalla posizione ricevuta**. L'idea è quella di testare che il nostro `UIViewController` si comporti "bene" rispetto alle notifiche del sistema di localizzazione. Il progetto Xcode contenente il codice mostrato in questo articolo è disponibile su [GitHub](https://github.com/broomburgo/AsyncTestingStubbing): si consiglia di scaricare il progetto e tenerlo a riferimento per il resto dell'articolo.

Invece di usare direttamente `CLLocationManager` per ottenere la posizione GPS del dispositivo, implementeremo una semplice classe che fungerà da *wrapper*, chiamata `LocationCoordinator`: un'istanza di tale classe potrà essere configurata con due semplici funzioni `onUpdate` e `onFailure`, in modo da evitare l'implementazione dei vari metodi di callback del `protocol` `CLLocationManagerDelegate`. L'idea è quella di passare un'istanza di `LocationCoordinator` al nostro `UIViewController`: quest'ultimo quindi imposterà le due funzioni di callback indicate sopra, in modo da modificare il suo stato e i dati mostrati all'utente.

Di seguito è riportato il codice della classe `LocationCoordinator`:

```
import Foundation
import CoreLocation

public class LocationCoordinator: NSObject {
    
    public let locationManager: CLLocationManager
    
    public override init() {
        locationManager = CLLocationManager()
        super.init()
        locationManager.delegate = self
        locationManager.requestWhenInUseAuthorization()
    }
    
    private var updated: (CLLocation -> ())?
    public func onUpdate(value: CLLocation -> ()) {
        updated = value
    }
    
    private var failed: (NSError -> ())?
    public func onFailure(value: NSError -> ()) {
        failed = value
    }
}

extension LocationCoordinator: CLLocationManagerDelegate {
    
    public func locationManager(manager: CLLocationManager!, didUpdateLocations locations: [AnyObject]!) {
        let location = locations.last as! CLLocation
        if let updated = self.updated {
            updated(location)
        }
    }
    
    public func locationManager(manager: CLLocationManager!, didFailWithError error: NSError!) {
        if let failed = self.failed {
            failed(error)
        }
    }
    
    public func locationManager(manager: CLLocationManager!, didChangeAuthorizationStatus status: CLAuthorizationStatus) {
        switch status {
        case .AuthorizedWhenInUse:
            locationManager.startUpdatingLocation()
        default:
            break
        }
    }
}
```

Come si può vedere, la classe è configurata per richiedere l'autorizzazione a leggere la posizione GPS del dispositivo quando l'app è in uso, ma non sarà necessario fornire questa autorizzazione per i test: creeremo infatti uno *stub* che genererà posizioni arbitrarie, senza usare i sensori di sistema. Si ricorda inoltre che usando iOS SDK >= 8.0, per usare i servizi di localizzazione sarà necessario inserire nel file Info.plist una chiave che descrive il motivo per il quale l'app chiede l'autorizzazione ad accedere a tali servizi:

![](/images/swift-testing-avanzato-stubbing-e-test-asincroni/locationUsageDescription.png)

Sebbene l'aggiunta della chiave `NSLocationWhenIsUseUsageDescription` non sia necessaria per eseguire i test, senza questa chiave l'app non potrà funzionare in iOS8 nel caso in cui provassimo ad avviarla normalmente.

La classe di cui vogliamo testare il corretto funzionamento è indicata nel progetto con il nome di `LocationViewController`, e il suo scopo è mostrare le attuali coordinate GPS dell'utente o mostrare un messaggio in caso di errore: un sua istanza è, in ogni instante, caratterizzata da un `LocationState` che rappresenta in quale stato essa si trova rispetto alla ricerca della posizione GPS:

```
public enum LocationState {
    case Searching
    case Found
    case Error(CLError)
}
```

## Implementiamo i test

Per testare la classe verificheremo che, a seconda dei diversi possibili risultati ottenibili dalla ricerca della posizione GPS, un'istanza di essa **aggiorni correttamente il suo stato attuale**. È quindi necessario fare in modo che un'istanza del `LocationCoordinator` ignori le notifiche ricevute da `CLLocationManager`, in modo da forzare posizioni ed errori comodi per i nostri test; per farlo dichiariamo una sottoclasse di `LocationCoordinator` chiamata `STUB_LocationCoordinator` che modifichi la *parent class* nel seguente modo:

- ignora le notifiche del `locationManager`:
```
locationManager.stopUpdatingLocation()
locationManager.delegate = nil
```

- dichiara 4 funzioni per forzare posizione GPS e/o errori:
```
forceLocation
forceError
forceRandomDelayedLocations
forceDelayedErrorLocationUnknown
```

Ciascuna di queste funzioni genera le stesse notifiche che normalmente genererebbe `CLLocationManager`. Le prime due funzioni, `forceLocation` e `forceError` forzano immediatamente una certa posizione GPS o un certo errore, mentre la funzione `forceRandomDelayedLocations` genera una serie di posizioni casuali dopo un certo ritardo e a intervalli regolari, e la funzione `forceDelayedErrorLocationUnknown` forza un errore del tipo `.LocationUnknown` dopo un certo ritardo.

Prima di procedere con i test osserviamo che nel file `AsyncTestingStubbingTests.swift` c'è una sezione *utility* nella quale sono implementate alcune funzioni comode per **rendere  il codice dei test più espressivo**. Ad esempio la funzione `mainViewController` ritorna l'istanza del LocationViewController utilizzata nell'interfaccia grafica nell'app. Il *testing framework* integrato in Xcode è chiamato `XCTest`, e per quanto adeguatamente completo nelle sue funzionalità, esso è principalmente basato su funzioni del tipo `XCTAssert` che verificano che una qualche condizione sia vera, e se non lo è stampano in console un messaggio di errore passato alla stessa funzione `XCTAssert` nel momento in cui essa è chiamata: questo può portare alla scrittura di **test verbosi e poco espressivi**. Ci sono diverse librerie facilmente integrabili nei progetti Xcode che semplificano la scrittura dei nostri *assert*, come ad esempio [Nimble](https://github.com/Quick/Nimble), ma come si può vedere dalle poche funzioni di utility implementate nel progetto associato a questo articolo, basta poco per migliorare consistentemente la leggibilità dei nostri test.

A questo punto possiamo procedere all'implementazione dei test. Le funzioni `testLocation` e `testError` testano semplicemente che lo stato del `mainViewController` sia corretto prima e dopo la generazione di una posizione GPS e di un errore arbitrari. Tuttavia questi test non permettono di rappresentare correttamente un caso d'uso tipico, perché **le notifiche sulla posizione GPS posso arrivare in qualsiasi momento**, e il `LocationViewController` deve essere in grado di *reagire* alle notifiche nel momento in cui queste si presentano. Per simulare la cosa useremo due test *asincroni*, cioè test il cui risultato non è ottenuto immediatamente, ma dopo un certo tempo: l'idea è quella di "mettere in pausa" i test per un certo numero di secondi, finché non si verifica una certa condizione oppure non scade il tempo: in quest'ultimo caso il test verrà considerato **fallito** da `XCTest` perché la condizione attesa non si è verificata entro il tempo richiesto.

Riportiamo ad esempio il codice della funzione `testDelayedLocation`:

```
func testDelayedLocation() {
        if let vc = mainViewController() {
            let locationExpectation = expectationWithDescription("locationExpectation")
            let coordinator = STUB_LocationCoordinator()
            vc.showLocationWithCoordinator(coordinator)
            vc.locationState.assertState(.Searching)
            coordinator.forceRandomDelayedLocations(0.25, times:3)
            after(0.5) {
                vc.locationState.assertState(.Found)
                after(0.25) {
                    vc.locationState.assertState(.Found)
                    locationExpectation.fulfill()
                }
            }
            vc.locationState.assertState(.Searching)
            waitForExpectationsWithTimeout(1, handler: nil)
        }
        else {
            fail("this will never happen")
        }
    }
```

La funzione di utility `after` semplicemente esegue del codice dopo un certo numero di secondi: in base all'implementazione di `forceRandomDelayedLocations` presente in `STUB_LocationCoordinator`, il `LocationViewController` dovrebbe  ricevere alcune posizioni GPS a intervalli regolari dopo un certo ritardo, e per verificarlo inseriamo due *assert*, dopo mezzo secondo e poi ancora un quarto di secondo. Per realizzare il test asincrono  definiamo una `XCTestExpectation`, quindi una "aspettativa", chiamata `locationExpectation`: in coda al codice di test chiamiamo la funzione `waitForExpectationsWithTimeout` che fa continuare la funzione di test - senza che essa *ritorni* - fino alla "realizzazione" delle aspettative, tenendo conto di un certo tempo di timeout. Quindi, per "realizzare" un'aspettativa, chiamiamo la funzione `fulfill` su `locationExpectation`, ma solo dopo aver verificato che lo stato del `LocationViewController` sia quello corretto.

Possiamo estendere ulteriormente la *test suite* implementando altri test che simulano ogni tipo di errore possibile, oppure che simulano una situazione in cui posizioni GPS e errori si alternano: una volta compresa la struttura di base per realizzare *stub* e test asincroni sarà facile migliorare la [coverage](http://engineering.facile.it/software-testing-coverage-vs-efficacia/) dei nostri test.