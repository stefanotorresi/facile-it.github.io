---
authors: ["francesco"]
comments: true
date: "2015-04-15"
draft: false
share: true
categories: [Italiano, SOLID, PHP, Design patterns]
title: "Il principio OPEN/CLOSE le sue origini e i casi d'uso"

languageCode: "it-IT"
type: "post"
aliases:
  - "/principio-open-close-le-sue-origini-e-i-casi-d-uso"
---
## Definizione:
Il principio *OPEN/CLOSE* fa riferimento all'acronimo [SOLID](http://en.wikipedia.org/wiki/SOLID_(object-oriented_design)).  introdotto da  Michael Feathers che riporta alcune *buone pratiche* di programmazione ad oggetti ancora di forte attualità.

*OPEN/CLOSE* (la **O** dell'acronimo), nella sua definizione più generale indica che:  un modulo (un oggetto, una classe o un gruppo di funzioni) debba essere **aperto alle estensioni** ma **chiuso alle modifiche**.

Questo principio tuttavia non è altro che la formalizzazione e il raggruppamento di concetti e linee guida già presenti da tempo nella programmazione ad oggetti.


### Chiuso alle modifiche
***Chiuso alle modifiche che potrebbero aver effetto sul client***

Possiamo esprimere questa linea guida anche come  un evoluzione di Single responsibility: Un modulo prodotto con un buon design avrà una responsabilità precisa, definita e, quindi, non vi sarà la necessita di modificare il suo comportamento, **anche alla luce di nuove specifiche**.
Per chiarire ancora di più il concetto dobbiamo fare riferimento ad un altro famoso acronimo della programmazioni ad aggetti: GRASP e, in particolare, all'idea di [Protected Variations](http://www.martinfowler.com/ieeeSoftware/protectedVariation.pdf) (da ora in poi *PV*) e di [Low coupling](http://en.wikipedia.org/wiki/Loose_coupling).

In un articolo di [Craig Larman](http://en.wikipedia.org/wiki/Craig_Larman) si parla di *PV* come la necessità di:

*Nascondere l'implementazione ([information hiding](http://en.wikipedia.org/wiki/Information_hiding)) e le informazioni legate al design (hidden implementation) agli altri moduli in particolar modo al presentarsi di scelte difficili o cambiamenti molto frequenti.*

Questo garantisce, in generale, un livello di flessibilità superiore, si pensi a questo semplice esempio:


	/** Questa classe è esposta al client **/
    class ClasseEsterna 
    {
      private var $oggettoInterno;

      public function incrementa()       {
  		[...]			
        return $this->oggettoInterno->complessaFunzioneDiIncremento();
      }     
    }

Si può notare come :

* L'implementazione della *complessa* funzione interna è nascosta al client;
* All'evolvere della **complessaFunzioneDiIncremento** il nostro client sarà ==protetto== dalle modifiche;
* Abbiamo realizzato un *low coupling* fra la l'**oggettoInterno** e il client che utilizza la **ClasseEsterna**, promuovendo, di fatto, il ==riuso== di **oggettoInterno** in altri contesti.


### Aperto alle estensioni:
***Aperto ad essere esteso ed adattato***

Le modalità tramite le quali è possibile modificare il comportamento di un modulo per accogliere le nuove esigenze ed evoluzioni è tramite le estensioni.

In questo caso il concetto di estensione può essere visto sia in senso stretto (Ereditarietà) sia in senso lato come aggiunta di nuove classi, attributi metodi etc..

Sembrerebbe che questi due attributi siano in contrapposizione; Il modo classico per estendere il comportamento di un modulo è effettuare modifiche al modulo stesso.
Un modulo che non può cambiare è solitamente considerato qualcosa che ha un comportamento fisso e che non può evolvere.

Un esempio è il pattern chain of responsibility che vediamo di seguito.


### Chain of responsibility:

Questo pattern risolve un’ampia classe di situazioni nelle quali vi sia la necessità di eseguire operazioni **sequenziali** e **condizionali**.

La chain of responsibility è composta da 2 soggetti:

* **1** gestore.
* **n** anelli.

**GESTORE:**

Nella definizione teorica della catena questo soggetto non esiste!
Tuttavia, per avvicinarci ad  un implementazione di questo pattern, è necessario utilizzare un vigile della catena che è a conoscenza dello stato globale dell’esecuzione.
Il gestore ha la responsabilità di chiamare in causa i singoli anelli e di restituire il risultato dell’esecuzione globale.

```PHP

class Gestore
{

    /**
     * Il metodo execute chiama in causa tutti gli anelli della catena.
     * @param Object $oggetto
     * @return Object
     **/
    public function execute(Object $oggetto)
    {

        // [...]

        foreach ($listaAnelli as $anello) {
            $anello->handle($oggetto);
        }

        // [...]

        return $oggetto;

    }


}

```
**ANELLI:**

I singoli anelli implementano un interfaccia comune che solitamente espone un solo metodo pubblico esempio *handle* .**Non possono** tener conto del risultato di precedenti anelli, non sono a conoscenza dello stato globale dell’esecuzione e possono, quindi, prendere solo decisioni locali. 

```PHP
interface AnelloInterface {

 /**
  * Il metodo handle effettua delle operazioni sull'oggetto in esame.
  * @param Object $oggetto
  * @return void
  **/
 function handle(Object $oggetto);

}
```
*Come rispettiamo PV ? Ovvero, come proteggiamo il client da future evoluzione del sistema?*

Il client utilizzerà sempre **la stessa API** esposta dal gestore della catena. Le modifiche non interesseranno, infatti, il contratto fra client e gestore.

*Il sistema è aperto alle estensioni?*

All'evoluzione della catena verranno inseriti nuovi anelli che copriranno ulteriori casi d'uso o funzionalità. Sebbene i nuovi anelli dovranno implementare la stessa interfaccia potranno avere una logica interna completamente personalizzabile.

*Esiste una forte separazione fra i moduli del sistema?*

Il low coupling in questa caso è garantito da 2 aspetti:

* Il client non conosce gli elementi della catena perché ha una relazione diretta solamente con il gestore.
* È presente coesione fra l'interfaccia dei singoli anelli e il gestore. Purché si rispetti questo contratto è possibile esprimere anche logiche complesse. 

### Conclusione:

È necessario **scegliere le proprie battaglie** quando si parla di design, sia che si tratti di scelte macro-architetturali che piccole scelte sulla singola istanza.

I principi che non stati esposti non sono applicabili in tutte le situazioni, un bravo sviluppatore cercherà di utilizzare PV e low coupling in **aree strategiche del sistema**, **spesso soggette a cambiamento**.

Se non si riesce ad identificare correttamente queste aree non si farà altro che introdurre una maggiore complessità generale che, in certe situazioni può essere deleteria.

L'obbiettivo finale dovrebbe essere sempre minimizzare l'impatto sul sistema dei (naturali) cambiamenti; Open/Close PV e low coupling sono alcune delle strategie più efficaci.


#### Bibliografia

* Craig Larman - [Applying UML and Patterns – An Introduction to Object-Oriented Analysis and Design and Iterative Development (3rd ed.)](http://www.utdallas.edu/~chung/SP/applying-uml-and-patterns.pdf)
* Parnas, D.L. - ["On the Criteria To Be Used in Decomposing Systems into Modules".](https://www.cs.umd.edu/class/spring2003/cmsc838p/Design/criteria.pdf)
* Craig Larman - ["Protected Variation: The Importance of Being Closed"](http://www.martinfowler.com/ieeeSoftware/protectedVariation.pdf)
* SIGS Publications - [The Open-Closed Principle: C++ Report](http://www.objectmentor.com/resources/articles/ocp.pdf)
[Information Hiding](http://en.wikipedia.org/wiki/Information_hiding)
