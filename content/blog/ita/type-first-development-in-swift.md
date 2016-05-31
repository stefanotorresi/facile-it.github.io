---
authors: ["elviro"]
comments: true
date: "2015-05-05"
draft: false
share: true
categories: [Italiano, Swift, Functional programming, Type First Development]
title: "Type First Development in Swift"

languageCode: "it-IT"
type: "post"
aliases:
  - "/type-first-development-in-swift"
images: ['/images/logo.png']

---

Con *Type First Development* può intendersi un approccio allo sviluppo di nuove funzionalità o moduli di un software partendo dai **tipi** di dati coinvolti: non si tratta quindi di un pattern o una pratica codificata, ma solo di **un possibile punto di partenza** per iniziare il ragionamento. Ragionando esclusivamente sui tipi, prima ancora di pensare alle singole specifiche implementazioni dei vari blocchi di codice, è possibile costruire più facilmente una mappa dei vari di flussi di dati che attraversano il nostro software, e verificare immediatamente se stiamo scrivendo qualcosa di sensato, solido ed adeguatamente estendibile.

Ragionare sui tipi di dato è un classico approccio usato nel paradigma di programmazione noto come [*programmazione funzionale*](http://en.wikipedia.org/wiki/Functional_programming), e il presente articolo adotta tale paradigma in maniera piuttosto pura.

## Indice
- [Il *type system* di Swift](#il-type-system-di-swift)
- [Costruiamo un form: il tipo *Field*](#costruiamo-un-form-il-tipo-field)
- [Modificare un campo: tipi di funzione come tipi di dato](#modificare-un-campo-tipi-di-funzione-come-tipi-di-dato)
- [Funzioni generiche e composizione funzionale](#funzioni-generiche-e-composizione-funzionale)
- [Il tipo *Form*](#il-tipo-form)
- [Modifica di un form e ricerca di un campo](#modifica-di-un-form-e-ricerca-di-un-campo)
- [Conclusione](#conclusione)

<a name="il-type-system-di-swift"></a>
## Il *type system* di Swift

Il linguaggio **Swift**, usato nello sviluppo di software per Mac OSX e iOS, si presta molto bene all'approccio basato sui tipi illustrato in questo articolo, a causa di due particolari caratteristiche del linguaggio:

- Swift è [staticamente tipizzato](http://en.wikipedia.org/wiki/Type_system#Static_type-checking), cioè ogni parametro, costante o variabile che sia, è caratterizzato da uno specifico tipo, e se una funzione vuole in ingresso un valore di un certo tipo, chiamare la funzione con un valore di un altro tipo risulterà in un errore di compilazione;

- Swift è [fortemente tipizzato](http://en.wikipedia.org/wiki/Strong_and_weak_typing), cioè non esiste alcuna conversione implicita tra i tipi: ad esempio, in Swift `1` è diverso da `true` e i due valori non posso essere usati indifferentemente nello stesso contesto; 

Il *type system* di Swift è molto rigido, ma è in tale rigore che risiede la sua potenza: usare tipi errati provoca un errore di compilazione, e manipolando adeguatamente i tipi di dato nella costruzione di funzioni è possibile verificare gran parte della correttezza del proprio codice già nella fase di compilazione, evitando potenziali problemi in fase di esecuzione.

Swift usa molti termini ereditati da C, come `struct` e `enum`, ma il loro significato è completamente diverso, ad esempio:

- una `struct` in Swift può dichiarare metodi ed essere estesa con interfacce;
- `enum` in Swift è ciò che in altri linguaggi è spesso chiamato "sum type" o "[tagged union](http://en.wikipedia.org/wiki/Tagged_union)", e a ogni `case` possono essere assegnati dei valori associati, ad esempio `case Text(String)` rappresenta il caso `Text` e ha un valore `String` associato.

Per illustrare un modo per applicare l'approccio *Type First* nello sviluppo in Swift inizieremo a scrivere una libreria per gestire **form**, quindi liste strutturate di campi valorizzati, utili per raccogliere dati degli utenti tramite moduli da compilare, per effettuare sondaggi o inserire i parametri di query a un server.

<a name="costruiamo-un-form-il-tipo-field"></a>
## Costruiamo un form: il tipo *Field*

Un possibile tipo di dato da cui partire è **Field**, il campo, cioè il mattone costruttivo di base di un form.

```
struct Field {
    let id: String
    let name: String
    let value: String
    let visible: Bool
}
```

Questa definizione di Field è molto basilare, il ché non è un problema nelle prime fasi dello sviluppo, ma **sembra poco estendibile**, per i seguenti motivi:

- `value` è definito come `String` - ad esempio testo inserito dall'utente - ma un campo potrebbe contenere altri tipi di valori, ad esempio un checkbox si/no, oppure un preciso valore selezionato da una lista;
- `visibile` è definito come `Bool`, e vuole indicare se il campo è visibile oppure nascosto, ma se volessimo assegnare altri valori di visibilità, ad esempio "evidenziato" o "oscurato" dovremmo aggiungere altri parametri `Bool`, che alla fine potrebbero andare in contrasto l'uno con l'altro;

Possiamo risolvere immediatamente il problema, **senza cadere nell'eccessiva ottimizzazione preventiva**, semplicemente modificando il tipo di `value` e `visible` con degli `enum`: 

```
enum FieldValue: Equatable {
    case Text(String)
    case Empty
}

func == (lhs: FieldValue, rhs: FieldValue) -> Bool {
    switch (lhs, rhs) {
    case (.Empty, .Empty):
        return true
    case (.Text(let lhsText), .Text(let rhsText)):
        return lhsText == rhsText
    default:
        return false
    }
}

enum FieldVisibility: Equatable {
    case Visible
    case Hidden
}

func == (lhs: FieldVisibility, rhs: FieldVisibility) -> Bool {
    switch (lhs, rhs) {
    case (.Visible, .Visible), (.Hidden, .Hidden):
        return true
    default:
        return false
    }
}
```

Abbiamo fatto adottare il protocollo `Equatable` a entrambi i tipi, in modo da poter confrontare `value` e `visibility` di due campi per verificare se sono uguali: il vantaggio degli `enum` è che **possiamo facilmente aggiungere nuovi casi** senza rompere il codice esistente, ci basterà gestire i nuovi `case` via via che vengono aggiunti.

Un'altra piccola modifica utile corrisponde a ridefinire il tipo del parametro `id`: a tale parametro deve essere assegnata **una chiave univoca**, diversa per ogni campo, quindi può convenire definire un tipo `UniqueKey`:

```
typealias UniqueKey = String
```

In Swift, la keyword `typealias` ci permette di definire un *alias* di un tipo, cioè un termine che corrisponde esattamente a quel tipo, e permette di:

- **migliorare la leggibilità**, la chiarezza e la compattezza del codice;
- **facilitare la manipolazione dei tipi**, come vedremo a breve;

Ogni volta in cui una funzione richiederà `UniqueKey` sapremo che in quel campo sarà necessario inserire una stringa univoca, che identifica un particolare campo.

Definiamo nuovamente `Field` incorporando i nuovi tipi:

```
struct Field {
    let id: UniqueKey    
    let name: String
    let value: FieldValue
    let visibility: FieldVisibility
    init (_ id: Key, _ name: String, _ value: FieldValue, _ visibility: FieldVisibility) {
        self.id = id
        self.name = name
        self.value = value
        self.visibility = visibility
    }
}
```

Abbiamo aggiunto anche un costruttore di convenienza che ci permetterà di creare un `Field` semplicemente passando i 4 dati richiesti.

<a name="modificare-un-campo-tipi-di-funzione-come-tipi-di-dato"></a>
## Modificare un campo: tipi di funzione come tipi di dato

La prima e più semplice operazione che vogliamo poter effettuare su `Field` è quella di modificarlo in qualche modo: ogni operazione di questo tipo, in Swift, può essere indicata con `Field -> Field`, cioè una trasformazione che prende un `Field` in ingresso e produce un `Field` in uscita:

```
typealias FieldChange = Field -> Field
```

L'operatore `->` ha lo stesso significato che ha già in [Haskell](http://en.wikipedia.org/wiki/Haskell_(programming_language)): definisce il tipo di una funzione che prende in ingresso un dato del tipo indicato a sinistra dell'operatore, e ritorna un dato del tipo indicato a destra dell'operatore (in questo caso entrambi i dati sono di tipo `Field`).

È da evidenziare che `FieldChange` non rappresenta un tipo di "dato" in senso stretto, ma **un tipo di "funzione**": in effetti, definendo `FieldChange`, possiamo trattare le funzioni di quel tipo come se fossero dati.

Per fare un esempio potremmo definire la funzione `setVisibility`, che prende in ingresso un valore di `FieldVisibility` e ritorna, appunto, un `FieldChange`, cioè un'altra funzione.

```
func setVisibility (visibility: FieldVisibility) -> FieldChange {
    return { field in
        return Field (
            field.id,
            field.name,
            field.value,
            visibility
        )
    }
}

/// alcuni esempi
let setHidden = setVisibility(.Hidden)
let visibleField1 = Field ("","", .Empty, .Visible)
let isTrue1 = visibleField1.visibility == .Visible
let hiddenField1 = setHidden(visibleField1)
let isTrue2 = hiddenField1.visibility == .Hidden
```

Negli esempi mostrati, `setHidden` è una funzione ma dal punto di vista sintattico è indistinguibile da un qualsiasi altro valore, e il suo tipo (cioè `FieldChange`) è dedotto dal compilatore di Swift in base alla definizione della funzione.

Una funzionalità che vogliamo certamente introdurre nella libreria corrisponde al poter operare una trasformazione su un campo in base a un qualche parametro definito in un altro campo: ad esempio, un particolare campo mostra un check che permette di mostrare/nascondere altri campi. Definiamo quindi un nuovo tipo:

```
typealias FieldChangeGenerator = Field -> FieldChange
```

`FieldChangeGenerator` è il tipo di una funzione dalla quale otteniamo, in base allo stato un certo campo origine, una certa trasformazione di tipo `FieldChange`, cioè qualcosa che prende un `Field` in ingresso e ritorna un `Field` in uscita: si tratta quindi di una funzione che ritorna un'altra funzione, di tipo diverso. Vorremmo ad esempio poter scrivere una funzione del tipo `setVisibilityIfOrigin`, che imposta un valore di visibilità in base allo stato di un campo origine: ok, ma quale valore di visibilità? Dipende dal campo origine, e possiamo risolvere il problema definendo un nuovo tipo di funzione, che prende in ingresso un campo, e ritorna un valore di visibilità:

```
typealias FieldCondition = Field -> FieldVisibility
```

La nostra funzione `setVisibilityIfOrigin` semplicemente prenderà ingresso un dato di tipo `FieldCondition`:

```
func setVisibilityIfOrigin (condition: FieldCondition) -> FieldChangeGenerator {
    return { origin in
        let newVisibility = condition(origin)
        return setVisibility(newVisibility)
    }
}

/// alcuni esempi

let setVisibleIfNotEmpty = setVisibilityIfOrigin { field in
    switch field.value {
    case .Empty:
        return .Hidden
    default:
        return .Visible
    }
}

let nonEmptyField = Field ("", "", .Text("something"), .Visible)

let anotherVisibleField = setVisibleIfNotEmpty(nonEmptyField)(invisibleField)

let isTrue3 = anotherVisibleField.visibility == .Visible
```

<a name="funzioni-generiche-e-composizione-funzionale"></a>
## Funzioni generiche e composizione funzionale

In realtà la nostra `FieldCondition` sembra essere un po' limitata: ritorna un valore di `FieldVisibility`, mentre una vera *condition* **dovrebbe ritornare un valore generico**, a seconda dei casi: ad esempio, potrebbe cambiare il **valore** di un campo bersaglio in base allo stato di un campo origine. L'ideale sarebbe scrivere qualcosa del genere:

```
typealias FieldCondition<T> = Field -> T
/// errore di compilazione!
```

Sfortunatamente in Swift non è possibile scrivere dei `typealias` generici, ma possiamo facilmente aggirare l'ostacolo costruendo una `struct` generica che contiene un solo parametro, al quale è assegnata una funzione di tipo `Field -> T`:

```
struct FieldCondition<T> {
    let apply: Field -> T
    init(_ apply: Field -> T) {
        self.apply = apply
    }
}
```

Se prima abbiamo scritto semplicemente una funzione `setVisibilityIfOrigin`, l'idea è riuscire ora a scrivere **una funzione generica** `setTargetIfOrigin<T>` che ritorna un `FieldChangeGenerator` che trasforma un campo bersaglio rispetto a un certo parametro, definito in base allo stato di un campo origine: come scrivere questa funzione? **Ragioniamo sui tipi**:

- `FieldCondition<T>` è un contenitore per una funzione di tipo `Field -> T`;
- `FieldChangeGenerator` è un alias per `Field -> FieldChange`;
- per completare la catena abbiamo bisogno di una trasformazione del tipo `T -> FieldChange` perché unendo `Field -> T` e `T -> FieldChange` otteniamo appunto `Field -> FieldChange`;

Possiamo scrivere quindi la funzione generica di cui sopra:

```
func setTargetIfOrigin<T> (condition: FieldCondition<T>, generate: T -> FieldChange) -> FieldChangeGenerator {
    return { field in
        return generate(condition.apply(field))
    }
}
```

Se tutto sembra molto astratto è perché **lo è**: stiamo gradualmente costruendo degli strumenti di manipolazione dei dati (e delle funzioni, che sono anch'esse dati) operando su astrazioni successive.

Nella funzione `setTargetIfOrigin<T>` c'è un parametro `generate` che è definito come `T -> FieldChange`: cosa è `T`? È semplicemente lo stesso tipo di dato con il quale viene passata la condizione in `condition`; se passiamo una condizione sulla visibilità, ad esempio, `T` sarà `FieldVisibility`: **una funzione generica si specializza nel momento in cui è definito il tipo concreto da assegnare ai tipi generici**. Volendo fare un esempio concreto, possiamo ridefinire la funzione `setVisibilityIfOrigin`:

```
func setVisibilityIfOrigin (condition: FieldCondition<FieldVisibility>) -> FieldChangeGenerator {
    return setTargetIfOrigin (condition) { visibility in setVisibility(visibility) }
}
```

Come si può vedere, la funzione chiama `setTargetIfOrigin`, passando una condizione del tipo `FieldCondition<FieldVisibility>`: il secondo parametro dovrà essere quindi una funzione del tipo `FieldVisibility -> FieldChange`, e possiamo usare la funzione `setVisibility` definita prima. Vediamo qualche altro esempio:

```
let setVisibleIfNotEmpty = setVisibilityIfOrigin (FieldCondition { origin in
    switch origin.value {
    case .Empty:
        return .Hidden
    default:
        return .Visible
    }
})

let copyValue = setTargetIfOrigin (FieldCondition { $0.value}) { value in
    return { target in
        return Field(target.id,target.name,value,target.visibility)
    }
}

let field1 = Field("field1","",.Text("1"),.Visible)
let field2 = Field("field2","",.Text("2"),.Visible)

let newField1 = copyValue(field2)(field1)

let isTrue4 = newField1.value == .Text("2")
```

Può essere interessante spendere qualche parola per la funzione `copyValue` definita nell'esempio:

- la condizione è chiaramente del tipo `FieldCondition<FieldValue>` perché la *closure* passata ritorna il `value` del campo;
- nella seconda *closure* passata, il valore è usato per costruire un nuovo campo, uguale al bersaglio tranne proprio per quel valore;

Quindi `copyValue`, che ricordiamo è di tipo `FieldChangeGenerator`, è una trasformazione che prende il valore di un campo origine e lo assegna a un campo bersaglio.

Nel caso in cui avessimo una serie di campi e volessimo applicare una certa trasformazione a un solo campo particolare, avremmo bisogno di un check sul campo bersaglio. Come al solito, pensiamo prima al **tipo** di questo check: sarà qualcosa che prende un `FieldChange` e ritorna un altro `FieldChange`, che potrebbe essere uguale a quello in ingresso, oppure diverso in base a una certa condizione applicata al campo bersaglio.

```
typealias FieldCheck = FieldChange -> FieldChange
```

Per quanto riguarda la condizione, possiamo usare lo stesso FieldCondition<T> di prima, ma in questo caso lo specializzeremo direttamente in `FieldCondition<Bool>` perché quello che ci interessa sapere è se un certo campo bersaglio è interessato oppure no da una certa trasformazione.

```
typealias FieldConditionBool = FieldCondition<Bool>
```

Definiamo quindi una funzione `checkTarget` che prende in ingresso una condizione, e ritorna un `FieldCheck`:

```
func checkTarget (condition: FieldConditionBool) -> FieldCheck {
    return { change in
        return { field in
            if condition.apply(field) {
                return change(field)
            }
            else {
                return field
            }
        }
    }
}
```

Nel definire `checkTarget` possiamo vedere un altro caso in cui ragionare solo sui tipi ci aiuta a capire cosa poi dobbiamo effettivamente fare nel codice:

- la funzione deve ritornare un dato di tipo `FieldCheck`, che è una funzione che prende in ingresso un dato di tipo `FieldChange`, quindi la prima riga della funzione è appunto `return { change in`;
- `FieldCheck` è una funzione che ritorna un dato di tipo `FieldChange`, che come definito prima corrisponde a una funzione che prende in ingresso un `Field`, quindi la seconda riga è `return { field in`;
- `FieldChange` deve ritornare un `Field`, ma a questo punto possiamo applicare la nostra condizione sul campo in ingresso: se la condizione è verificata allora ritorniamo il campo trasformato, mentre se non lo è "facciamo uscire" il campo esattamente come è "entrato";

Come si può vedere, abbiamo iniziato a implementare funzioni ragionando quasi esclusivamente sui tipi coinvolti. Vediamo alcuni esempi:

```
func ifTargetId (id: UniqueKey) -> FieldCheck {
    return checkTarget (FieldCondition { $0.id == id })
}

let visibleField = ifTargetId("field2")(setHidden)(field1)
let hiddenField = ifTargetId("field2")(setHidden)(field2)

let isTrue5 = visibleField.visibility == .Visible
let isTrue6 = hiddenField.visibility == .Hidden

let fields = [field1,field2]
let newFields = fields.map(ifTargetId("field2")(setHidden))

let isTrue7 = newFields[0].visibility == .Visible
let isTrue8 = newFields[1].visibility == .Hidden
```

<a name="il-tipo-form"></a>
## Il tipo *Form*

Abbiamo implementato quindi alcuni strumenti per manipolare i singoli campi: a questo punto proviamo a unire il tutto definendo un `Form` come contenitore strutturato di campi, insieme ad alcune funzioni di manipolazione. Intanto **pensiamo al tipo da assegnare al nostro form**: una possibilità è quella di definire alcuni blocchi costruttivi che ne descrivono la struttura:

```
/// sezione: è una semplice lista di campi
struct Section {
    let fields: [Field]
}

/// step: un gruppo di sezioni, ad esempio una pagina
struct Step {
    let sections: [Section]
}

/// form: un insieme di pagine
struct Form {
    let steps: [Step]
}
```

Ci sono però due problemi con questo approccio:

- stiamo prematuramente fornendo al nostro form **una struttura rigida**: se in futuro decidessimo di voler creare una sottostruttura rispetto a `Section` sarebbe probabilmente molto complesso modificare il tutto;
- nello scrivere le logiche di ricerca e manipolazione dei campi ci ritroveremmo a dover gestire le `struct` appena definite: stiamo quindi automaticamente **accoppiando** un particolare modello di dati con la logica di manipolazione dei form, che **dovrebbero essere adattabili a più modelli, senza cambiare la logica**;

Per evitare questi problemi gli approcci possibili sono molti: un'opzione, ad esempio, è quella definire un tipo di dato corrispondente a un contenitore *ricorsivo* per un valore generico, che chiameremo `Node<T>`. Un valore di tipo `Node<T>` definisce due casi:

- *branch*, cioè una lista di valori di tipo T;
- *root*, cioè una lista di altri `Node<T>`

```
enum Node<T> {
    case Branch([T])
    case Root([Node<T>])
}
```

Possiamo quindi definire un `Form` come una versione specializzata di `Node<T>` dove `T` è `Field`, e definiremo un paio di funzioni di convenienza per generare un *branch* di `Field` o una *root* di `Form`:

```
typealias Form = Node<Field>

func branch (fields: [Field]) -> Form {
    return Node.Branch(fields)
}

func root (nodes: [Form]) -> Form {
    return Node.Root(nodes)
}
```

<a name="modifica-di-un-form-e-ricerca-di-un-campo"></a>
## Modifica di un form e ricerca di un campo

Come accadeva per `Field`, la prima operazione che vogliamo poter effettuare su `Form` è quella modificarlo; definiamo quindi il tipo `FormChange` in maniera a simile a come avevamo definito `FieldChange`:

```
typealias FormChange = Form -> Form
```

Supponiamo ad esempio di avere un form, e voler applicare una modifica a un campo presente nel form, ottenendo un nuovo form con il campo modificato: abbiamo bisogno di una funzione che potremmo chiamare `changeFromFieldChange` che prende in ingresso un `FieldChange` e ritorna un `FormChange`; grazie alla struttura ricorsiva del tipo `Form` definito, possiamo scrivere in poche righe una funzione che attraversa un form nei suoi vari livelli e lo ricostruisce applicando la modifica passata in ingresso a ogni campo, tenendo eventualmente conto di una certa condizione per verificare se un certo campo è interessato dalla modifica o meno.

```
func changeFromFieldChange (fieldChange: FieldChange) -> FormChange {
    return { form in
        switch form {
        case .Branch (let fields):
            return branch(fields.map(fieldChange))
        case .Root (let subforms):
            return root(subforms.map(changeFromFieldChange(fieldChange)))
        }
    }
}
```

Come si può vedere, la funzione verifica se un form è del tipo `.Branch` o `.Root`:

- nel primo caso, ricostruisce il *branch* applicando la funzione `FieldChange` in ingresso a ogni campo nel *branch*;
- nel secondo caso, ricostruisce il *root* applicando ricorsivamente la stessa funzione `changeFromFieldChange`;

Per migliorare la leggibilità possiamo definire una nuova funzione `updateField` identica a `changeFromFieldChange` che dichiara **in maniera più espressiva** il tipo di operazione che vogliamo poter effettuare su un form, e cioè aggiornare un certo campo in base a una certa trasformazione:

```
let updateField = changeFromFieldChange
```

Vediamo quindi un esempio:

```
let form = branch([field1,field2])

let newForm = updateField(ifTargetId("field1")(setHidden))(form)
```

Come si può vedere dall'esempio, otteniamo `newForm` componendo alcune funzioni definite in precedenza con `updateField`: se l'id del campo bersaglio è "field1", nascondiamo il campo.

Come verificare se il campo con id "field1" è effettivamente nascosto? Come ultima operazione vogliamo definire una funzione `fieldsWithCondition` che ci permetta di cercare uno o più campi in un form in base a una certa condizione: la condizione sarà ancora di tipo `FieldConditionBool`, cioè da un campo otteniamo un valore `true` o `false`. Per scrivere la funzione ragioniamo sul tipo di dato ritornato: esso deve corrispondere a una funzione che prende un form e ritorna un array di campi; chiameremo questo tipo `FieldGet`.

```
typealias FieldGet = Form -> [Field]

func fieldsWithCondition (condition: FieldConditionBool) -> FieldGet {
    return { form in
        switch form {
        case .Branch(let fields):
            return fields.filter(condition.apply)
        case .Root(let subforms):
            return subforms.map(fieldsWithCondition(condition)).reduce([], combine: +)
        }
    }
}
```

La funzione `fieldsWithCondition` opera ricorsivamente, come è ovvio che sia considerando la natura ricorsiva del tipo di dato `Node<T>`. Vediamo qualche applicazione:

```
func fieldsWithId (id: UniqueKey) -> FieldGet {
    return fieldsWithCondition (FieldConditionBool { $0.id == id })
}

let allFields: FieldGet = fieldsWithCondition (FieldConditionBool { _ in true })

let twoFields = allFields(form)

let isTrue9 = count(twoFields) == 2

let isTrue10 = fieldsWithId("field1")(form)[0].visibility == .Visible
```

La funzione `fieldsWithId` prende una `UniqueKey` e permette di ottenere tutti i campi con quell'id (presumibilmente l'array risultante conterrà un solo elemento); `allFields` genera semplicemente un array con tutti i campi: da notare che la *closure* `{ _ in true }` ignora il valore in ingresso e ritorna sempre `true`.

Usando `fieldsWithId` possiamo infine verificare che il campo con id "field1" sia effettivamente nascosto, in base alle manipolazioni precedenti:

```
let isTrue11 = fieldsWithId("field1")(newForm)[0].visibility == .Hidden
```

<a name="conclusione"></a>
## Conclusione

Per concludere, abbiamo visto come partendo da **tipi**, che siano essi relativi a singoli valori o a funzioni, è stato possibile iniziare a scrivere una libreria composta da semplici tipi di base e funzioni elementari che vengono composte via via a livelli di astrazione più alti. Definendo tipi specifici per molte astrazioni siamo riusciti a **gestire meglio la complessità** inerente nel manipolare e comporre funzioni, al punto da poter **trattare funzioni complesse come semplici dati** da passare ad altre funzioni. In un articolo successivo vedremo come estendere la libreria qui iniziata, abbracciando completamente la composizione funzionale e usando operatori specificamente definiti per migliorare la chiarezza del codice e rendere più semplice la manipolazione e combinazione delle funzioni.

> **N.B.** *Nello scrivere questo articolo si è scelto specificamente di non definire operatori su misura per comporre le funzioni, per evitare un ulteriore appesantimento legato alla presenza di codice dalla sintassi poco familiare. La chiamata di una funzione in Swift è c-like: si scrive il nome della funzione e poi si inseriscono i dati di ingresso tra due parentesi tonde; questo purtroppo può portare a diverse chiamate in parentesi annidate; inoltre, può non risultar chiara una chiamata del tipo `updateField(ifTargetId("field1")(setHidden))(form)`: come accennato, in un articolo futuro vedremo come usare operatori personalizzati per rendere il tutto più semplice e leggibile.*

> **N.B.** *La chiamata `subforms.map(fieldsWithCondition(condition)).reduce([], combine: +)`, nella funzione `fieldsWithCondition`, è piuttosto inefficiente dal punto di vista della performance: l'inefficienza è dovuta al fatto che a ogni chiamata vengono generati array intermedi dovuti alle varie mappature dei dati; non è in realtà necessario creare questi array intermedi: basta utilizzare procedure "lazy", che vedremo in un articolo futuro.*

## Riferimenti

* Tomas Petricek - [Why type-first development matters](http://tomasp.net/blog/type-first-development.aspx)
* Michael Feathers - [A Type Driven Approach to Functional Design](http://www.infoq.com/presentations/Type-Functional-Design)
* Johannes Weiß - [Type Driven Development in Swift](https://speakerdeck.com/johannesweiss/type-driven-development-in-swift)
* Luca Cardelli - [Typeful Programming](http://www.lucacardelli.name/Papers/TypefulProg.pdf)

Il codice mostrato nell'articolo è disponibile su [GitHub Gist](https://gist.github.com/broomburgo/ba8e5dd6cd509fdfd781).
