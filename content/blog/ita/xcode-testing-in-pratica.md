---
authors: ["elviro"]
comments: true
date: "2015-04-09"
draft: false
share: true
categories: [Italiano, Swift, Software testing, Xcode]
title: "Xcode Testing in pratica"

languageCode: "it-IT"
type: "post"
aliases:
  - "/xcode-testing-in-pratica"
---

Tra i molti strumenti utili presenti in Xcode, il testing framework **XCTest** è certamente uno dei più rilevanti, non solo per l'importanza intrinseca dello Unit Testing in generale, ma soprattutto per la facilità con la quale è possibile scrivere ed eseguire test direttamente dall'IDE *out-of-the-box*, **senza la necessità di installare componenti di terze parti** o impostare una particolare configurazione per i progetti.

In effetti Xcode, al momento della creazione di un nuovo progetto, oltre a creare un target per il binario principale crea automaticamente anche un **target di test**, cioè un bundle aggiuntivo che può essere caricato nel bundle principale per poter fisicamente eseguire i test una volta avviata l'app. Nell'immagine seguente è possibile vedere come, in un progetto appena creato, sia già presente il test bundle, in questo caso chiamato *AwesomeAppTests.xctest*:

![](/images/xcode-testing-in-pratica/image_1.jpg)

Come mostrato nell'immagine, Xcode ha anche creato automaticamente il file *AwesomeAppTests.swift*, all'interno del quale potremo iniziare a scrivere i nostri test.

## Setup dell'ambiente di test
Al di là del meccanismo con il quale il testing avviene, è importante capire che il codice con il quale i test sono stati scritti è compilato in un bundle diverso, e per far sì che i test "vedano" il resto dell'app, è necessario seguire le [regole di access control tra i moduli in Swift](https://developer.apple.com/library/ios/documentation/Swift/Conceptual/Swift_Programming_Language/AccessControl.html); quindi classi, struct, funzioni, costanti e così via che abbiamo dichiarato e implementato nella nostra app, o libreria che sia, **dovranno essere marcati con la keyword *public***:

```        
public func sumOfInts (x: Int, y: Int) -> Int {
    return x + y
}
    
public class AwesomeItem {
    var name: String? = nil
}
``` 

In questo modo, qualsiasi modulo esterno che importerà il modulo costituito dalla nostra app potrà vedere la firma della funzione `sumOfInts` e della classe `AwesomeItem`. Per importare nei file di testing nel modulo dell'app sarà sufficiente scrivere `import AwesomeApp` all'inizio del file. Aggiungiamo al progetto un file .swift, inserendo il codice appena visto: useremo questo file per scrivere tutto il codice da testare.

Apriamo quindi il file *AwesomeAppTests.swift*, importiamo il modulo `AwesomeApp`, eliminiamo le due funzioni di test di esempio, che hanno il solo scopo di presentare la sintassi di base dei test all'utente, e scriviamo un test banale per la funzione `sumOfInts`; avremo quindi qualcosa del genere:

```
class AwesomeAppTests: XCTestCase {
    
    override func setUp() {
        super.setUp()
        // Put setup code here. This method is called before the invocation of each test method in the class.
    }
    
    override func tearDown() {
        // Put teardown code here. This method is called after the invocation of each test method in the class.
        super.tearDown()
    }
    
    func testSumOfInts() {
        /// test the sumOfIntsFunction
    }
}
```

Xcode ha identificato la funzione `testSumOfInts` come **una funzione di test** perché il suo nome inizia con "test", e ha posizionato un pulsante di avvio test proprio di fianco alla sua dichiarazione:

![](/images/xcode-testing-in-pratica/image_2.jpg)

Per far eseguire a Xcode questo test, e solo questo, sarà sufficiente cliccare su quel pulsante. In alternativa sarà possibile selezione "Test" dal menu "Product" per far eseguire a Xcode tutti i test che l'IDE ha riconosciuto. Xcode offre anche un **Test Navigator**, che mostra il nome di tutti i test implementati, raggruppati per *Test Case*: spostando il puntatore del mouse su un test apparirà un pulsante a forma di freccia che permetterà di avviare singolarmente i test, oppure tutti i test relativi a un particolare *Test Case*, o anche tutti i test scritti.

![](/images/xcode-testing-in-pratica/image_3.jpg)

Per poter eseguire i test, Xcode deve effettivamente avviare l'app: selezioniamo uno dei simulatori disponibili in modo che a ogni avvio di test Xcode apra automaticamente il simulatore scelto.

![](/images/xcode-testing-in-pratica/image_4.jpg)

## XCTestCase e XCTAssert
Nel framework **XCTest** un *Test Case* è rappresentato da una sottoclasse della classe `XCTestCase`: al momento dell'avvio dei test viene automaticamente generata un'istanza di ciascun Test Case che abbiamo implementato, e vengono eseguiti uno a uno tutti i test dichiarati, avviando le funzioni `testSomething()` che li implementano. Anche i metodi `setUp()` e `tearDown()` sono chiamati automaticamente nel corso dei test: come indicano i commenti inseriti da Xcode, il metodo `setUp()` è chiamato immediatamente prima di ogni test, e permette di inizializzare eventuali attributi d'istanza, o variabili globali; come è intuibile, il metodo `tearDown()` permette di ripristinare eventualmente lo stato iniziale dopo ciascun test. L'implementazione di questi due metodi è comunque del tutto opzionale: essi rappresentano semplicemente degli strumenti in più. 

Poiché **i nostri Test Case sono effettivamente delle classi**, possiamo implementare anche altri metodi e attributi per ciascuna classe, che possono facilmente essere richiamati nei singoli test, come nel seguente esempio:

```
class AwesomeAppTests: XCTestCase {
    
		var firstInt = 0
		var secondInt = 0
		var expectedSum = 0

    override func setUp() {
        super.setUp()
        
				self.firstInt = 1
				self.secondInt = 2
				self.expectedSum = 3
    }
    
    override func tearDown() {
        self.firstInt = 0
				self.secondInt = 0
				self.expectedSum = 0
        super.tearDown()
    }
   	
    func testSumOfInts() {
				let sum = sumOfInts(self.firstInt, self.secondInt)
        XCTAssert(sum == self.expectedSum)
    }
}
```

Come si vede dall'esempio, per testare la funzione `sumOfInts` si è impiegata la funzione `XCTAssert`: tale funzione rappresenta il blocco costruttivo di base per poter scrivere i nostri unit test, e prende in ingresso da 1 a 4 parametri: il primo parametro, l'unico obbligatorio, deve essere un'espressione la cui valutazione restituisce un valore Booleano true/false, mentre gli altri parametri, del tutto opzionali poiché caratterizzati da valori di default, rappresentano rispettivamente:

- un messaggio testuale che verrà mostrato in console nel caso in cui il test fallisca;
- il file all'interno del quale è presente il test fallito;
- la linea di codice in corrispondenza della quale è presente la funzione XCTAssert relativa al test fallito;

La libreria `XCTest` dichiara molte diverse funzioni del tipo `XCTAssert`, **i cui casi di utilizzo sono chiaramente dedotti dal nome delle funzioni**: ad esempio la funzione `XCTAssertNil` verifica che l'argomento, cioè il primo parametro, sia `nil`; la funzione `XCTAssertGreaterThan` prende due espressioni invece di una, la cui valutazione deve restituire un valore di tipo `Comparable` - cioè che può essere confrontato con altri valori tramite gli operatori maggiore/minore e derivati - e verifica che il risultato della prima espressione sia maggiore del risultato della seconda.

Sfruttando le altre funzioni `XCTAssert`, possiamo estendere `testSumOfInts` per includere altre verifiche su `sumOfInts`:

```
func testSumOfInts() {
        
        XCTAssertEqual(sumOfInts(self.firstInt, self.secondInt), sumOfInts(self.secondInt, self.firstInt))

        let sum = sumOfInts(self.firstInt, self.secondInt)
        XCTAssert(sum == self.expectedSum)
        if self.firstInt < 0 && self.secondInt < 0 {
            XCTAssertLessThan(sum, 0)
        }
        else {
            if self.firstInt < 0 {
                XCTAssertLessThan(sum, self.secondInt)
            }
            if self.secondInt < 0 {
                XCTAssertLessThan(sum, self.firstInt)
            }
            if self.firstInt >= 0 && self.secondInt >= 0 {
                XCTAssertGreaterThanOrEqual(sum, 0)
                XCTAssertGreaterThanOrEqual(sum, self.firstInt)
                XCTAssertGreaterThanOrEqual(sum, self.secondInt)
            }
        }
    }
}
```

## Test asincroni
La libreria XCTest include alcuni strumenti per eseguire **test su computazioni asincrone**, cioè eseguite da funzioni che non ritornano immediatamente un valore ma che potrebbero chiamare una funzione di callback a un certo punto nel futuro. Per testare una funzione asincrona è necessario:

1. creare un'istanza di `XCTestExpectation`, un oggetto che rappresenta l'**aspettativa** che ad un certo punto nel futuro accada qualcosa;
2. informare l'istanza del `XCTestCase`, generata automaticamente all'avvio dei test, che nel corso di un certo test è necessario che le *aspettative* vengano *soddisfatte*.

Per fare un esempio, aggiungiamo una semplice funzione pubblica a *AwesomeItem.swift* che permette di scaricare un'immagine presente a un certo URL, e chiama una funzione callback restituendo appunto una `UIImage` che rappresenti l'immagine scaricata: per semplicità non gestiremo gli errori, e nel caso in cui qualcosa vada storto otterremo semplicemente `nil` al posto dell'immagine (quindi l'oggetto effettivamente ottenuto sarà di tipo `UIImage?`, cioè un oggetto opzionale). Per poter manipolare `UIImage` dobbiamo importare `UIKit` in *AwesomeItem.swift*.

```
import Foundation
import UIKit

public func sumOfInts (a: Int, b: Int) -> Int {
    return a + b
}

public func downloadImage (imageURL: NSURL, callback: (UIImage?) -> ()) {
    let task = NSURLSession.sharedSession().downloadTaskWithRequest(NSURLRequest(URL: imageURL), completionHandler: { (tempLocalURL: NSURL!, response: NSURLResponse!, error: NSError!) -> Void in
        if let path = tempLocalURL?.path {
            let image = UIImage(contentsOfFile: path)
            NSFileManager.defaultManager().removeItemAtPath(path, error: nil)
            callback(image)
        }
        else {
            callback(nil)
        }
    })
}

public class AwesomeItem {
    var name: String? = nil
}
```

Che si riesca a meno a scaricare l'immagine, la funzione di callback deve comunque essere chiamata in un tempo ragionevole, e l'obbiettivo del test asincrono sarà proprio quello di verificare che la tale funzione venga chiamata, ignorando  l'oggetto ottenuto per l'immagine. Segue una possibile implementazione per questo test, nella quale creiamo una `XCTestExpectation` e impostiamo un'attesa di 5 secondi: è ovviamente possibile sincronizzare il tempo di attesa con il tempo di timeout della sessione di download creata, ma per semplicità imposteremo un tempo forfettario.

```
func testDownloadImageCallback() {
        
        let downloadImageCallbackExpectation = self.expectationWithDescription("downloadImageCallbackExpectation")
        
        let imageUrl = NSURL(string: "http://goo.gl/XYwppm")!
        
        downloadImage(imageUrl) { image in
            downloadImageCallbackExpectation!.fulfill();
        }
        
        self.waitForExpectationsWithTimeout(5.0, handler: nil)
    }
```

Se proviamo ad avviare il test cliccando sull'icona di avvio apparsa in Xcode proprio a sinistra della riga di dichiarazione della funzione, analogamente a quando mostrato prima, vedremo che Xcode rimarrà bloccato per 5 secondi prima di mostrare il messaggio `Asynchronous wait failed: Exceeded timeout of 5 seconds, with unfulfilled expectations: "downloadImageCallbackExpectation".`: come mai? Il fallimento del test ci informa subito del fatto che la funzione di download dell'immagine che abbiamo scritto ha qualche problema: è facile vedere che esso è dovuto al fatto che non abbiamo chiamato `resume()` sulla task di download creata. Correggiamo quindi la nostra funzione `downloadImage` nel seguente modo:

```
public func downloadImage (imageURL: NSURL, callback: (UIImage?) -> ()) {
    let task = NSURLSession.sharedSession().downloadTaskWithRequest(NSURLRequest(URL: imageURL), completionHandler: { (tempLocalURL: NSURL!, response: NSURLResponse!, error: NSError!) -> Void in
        if let path = tempLocalURL?.path {
            let image = UIImage(contentsOfFile: path)
            NSFileManager.defaultManager().removeItemAtPath(path, error: nil)
            callback(image)
        }
        else {
            callback(nil)
        }
    })
    task.resume()
}
```

**Il test ci ha immediatamente informati dell'errore**, e ci ha permesso di evitare una possibile sorgente di bug nel nostro codice, e diversi mal di testa. Grazie al modo *seamless* nel quale il framework XCTest è integrato in Xcode, risulta facile e veloce scrivere test che permettano di verificare immediatamente se il nostro codice funziona come dovrebbe, ed **è Xcode stesso a incoraggiare la scrittura di test** nel corso della realizzazione di un'app o di una libreria generando automaticamente un target di test ogni volta in cui si crea un nuovo progetto: non ci sono più scuse quindi per non iniziare a scrivere test completi ed efficienti che permettano di **realizzare software migliore**.

Il progetto Xcode contenente tutto il codice mostrato nell'articolo è disponibile su [GitHub](https://github.com/broomburgo/xcode-testing-in-pratica)