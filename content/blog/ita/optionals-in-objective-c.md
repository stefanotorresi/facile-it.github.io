---
authors: ["elviro"]
comments: true
date: "2015-11-09"
draft: false
share: true
categories: [Italiano, Swift, Objective-C, Functional programming, Monad]
title: "Optionals in Objective-C"
eng: "optionals-in-objective-c"

languageCode: "it-IT"
type: "post"
aliases:
  - "/optionals-in-objective-c-ita"
---

**Objective-C vivrà ancora per molto**. Nonostante Swift sia il nuovo punto di riferimento per lo sviluppo iOS e OS X, ci sono ragioni concrete per scegliere di continuare a sviluppare in Objective-C, almeno per un po':

- progetti esistenti basati su Objective-C richiedono ancora mantenimento e probabile aggiunta di nuove funzionalità, e anche se è tecnicamente possibile mescolare i linguaggi, la cosa può risultare **poco conveniente** per via della natura molto dinamica di Objective-C;
- Swift sta cambiando rapidamente, presenta ancora alcuni bug e **problemi di performance**, e il suo workflow manca ancora di alcune feature fondamentali per i professionisti, mentre Objective-C è un linguaggio maturo, con una community molto vivace;
- alcuni possono **preferire un linguaggio più dinamico**, e il supporto di Apple su Objective-C è [ancora forte](https://netguru.co/blog/objective-c-generics);

Personalmente ho la tendenza a preferire linguaggi più statici, e un approccio [type-first](http://engineering.facile.it/type-first-development-in-swift/) alla programmazione, ma di tanto in tanto mi piace lavorare in un ambiente più *dinamico*, quindi, sia per preferenza personale che per esigenze di business, non ho ancora messo Objective-C da parte. Ma dopo poche settimane di Swift, mi è mancata subito una delle sue funzionalità più potenti: gli [Optionals](https://developer.apple.com/library/ios/documentation/Swift/Conceptual/Swift_Programming_Language/TheBasics.html#//apple_ref/doc/uid/TP40014097-CH5-ID330).

## Indice
- [Il tipo `Optional`](#il-tipo-optional)
- [Optionals in Swift](#optionals-in-swift)
- [Torniamo a Objective-C](#torniamo-a-objective-c)
- [Esempio: un parser JSON](#esempio-un-parser-json)
- [Conclusione](#conclusione)

<a name="il-tipo-optional"></a>
## Il tipo `Optional`
Il tipo `Optional` è un tipo *generico* (o, più propriamente, *parametrico*), dipendente da un **tipo secondario**: è possibile ad esempio definire un `Optional<String>` oppure un `Optional<Int>`: la sintassi di Swift permette di scrivere i tipi appena indicati con `String?` e `Int?`. Ma cos'è esattamente un `Optional`?. È un tipo particolare che serve a rappresentare un dato che *potrebbe* esistere, ed essere quindi di un certo tipo secondario, o potrebbe non esistere, ed essere quindi **nil**: *inscatolando* il valore opzionale in un `Optional`, il compilatore di Swift saprà che quel valore potrà essere nil, e emettere un errore nei casi in cui stiamo usando un `Optional` dove ci si aspetta un valore **sempre** presente. Questa semplice funzionalità ci garantisce notevole rigore nella definizione dei nostri tipi di dato e funzione: ad esempio, in Swift non possiamo inizializzare un valore non opzionale con nil, e considerando che, per il compilatore, un valore non può essere usato prima di essere istanziato, se assegniamo il tipo `String` a un valore, siamo sicuri al 100% che quel valore sarà **sempre a comunque** una stringa.

Al contrario, in Objective-C un oggetto può essere sempre nil, e spesso siamo costretti a controllare l'effettiva presenza di un oggetto ogni volta in cui abbiamo bisogno di un'istanza che non sia nil. In realtà non si tratta di un grosso problema: è un classico **compromesso** dei linguaggi dinamici, e il fatto che posso sempre inviare un messaggio a un riferimento nil può essere anche considerato una *feature*: usando un linguaggio dinamico, ci si aspetta che progettiamo le nostre API tenendo sempre in considerazione il concetto di [late-binding](https://en.wikipedia.org/wiki/Late_binding). Inoltre, i nuovi [*nullability specifiers*](https://developer.apple.com/swift/blog/?id=25) di Objective-C aiutano il compilatore a emettere avvisi quando stiamo passando nil a un metodo o una *property* che richiedono invece che il parametro sia `nonnull`. Ma il problema rimane: abbiamo bisogno di molto *boilerplate* per verificare se qualcosa è nil, e ciò può portare a codice poco leggibile e comprensibile, soggetto ad errori. Swift non ha tipicamente bisogno di questo *boilerplate* grazie alla natura stessa del tipo `Optional`: esso ha le stesse caratteristiche del tipo `Maybe` in Haskell, o del tipo `Option` in Scala, cioè `Optional` è in realtà un **monad**.

Il concetto di *monad* è ereditato, in programmazione funzionale, dalla [teoria delle categorie](https://it.wikipedia.org/wiki/Teoria_delle_categorie), ed è facile trovare [molti](https://wiki.haskell.org/Monad) [testi](https://www.haskell.org/tutorial/monads.html) [introduttivi](http://learnyouahaskell.com/a-fistful-of-monads) sul tema, quindi non approfondirò ulteriormente: dirò solo che un *monad* è un *contesto computazionale*, quindi una *specifica* per il modo in cui una espressione deve essere valutata. Applicare una certa trasformazione a un *monad* avrà come risultato un'altra istanza dello stesso *monad* differente dalla prima, a seconda dello specifico tipo di *monad*. Ad esempio, applicare una trasformazione a un `Optional` risulterà nell'applicazione della stessa trasformazione al valore *contenuto*, se presente, o assolutamente niente se l'`Optional` contiene nil: in entrambi i casi, il risultato della trasformazione sarà un nuovo `Optional`, possibilmente con un tipo secondario diverso.

<a name="optionals-in-swift"></a>
## Optionals in Swift

Consideriamo il seguente codice Swift:

```Swift
func makeOptionalIntFrom(value: Int, ifTrue: Bool) -> Int?  
{
  return ifTrue ? value : nil
}

let optionalInt = makeOptionalIntFrom(3, ifTrue: true)  
print(optionalInt.dynamicType) /// prints 'Optional<Int>'

let toString: Int -> String = { "\($0)" }

let optionalString = optionalInt.map(toString)  
print(optionalString.dynamicType) /// prints 'Optional<String>'
```

Nell'esempio, `optionalInt` è un `Int` *contenuto* in un `Optional`, che quindi ha tipo `Optional<Int>` (in Swift è possibile abbreviarlo in `Int?`), e applicando ad esso la funzione `toString` otteniamo un'istanza di `Optional<String>`. Possiamo vedere che, per applicare effettivamente la funzione `toString`, abbiamo passato la funzione come argomento al metodo `map` del `Optional`: questa operazione è solitamente chiamata *lifting* di una funzione, perché la funzione `toString`, di tipo `Int -> String`, è *sollevata* (*lifted*) fino a diventare di tipo `Optional<Int> -> Optional<String>`.

Come possiamo vedere, non sono state usate istruzioni condizionali nel manipolare il valore intero *opzionale*: *sollevando* le trasformazioni con il metodo `map`, possiamo applicarle direttamente alle istanze di `Optional`, e possiamo anche concatenarle molto facilmente, ad esempio:

```Swift
func makeOptionalIntFrom(value: Int, ifTrue: Bool) -> Int?  
{
  return ifTrue ? value : nil
}

let optionalInt = makeOptionalIntFrom(3, ifTrue: true)  
print(optionalInt.dynamicType) /// prints 'Optional<Int>'

let doubled: Int -> Int = { $0*2 }

let toString: Int -> String = { "\($0)" }

let optionalDoubledString = optionalInt.map(doubled).map(toString)  
print(optionalDoubledString.dynamicType) /// prints 'Optional<String>'
```

Volendo esprimere l'operazione opposta, avremmo un problema: trasformare una stringa in un intero non è sempre possibile in Swift (ad esempio se non c'è alcun numero nella stringa), quindi l'operazione stessa risulterebbe in un `Optional`. Infatti:

```Swift
func makeOptionalStringFrom(value: String, ifTrue: Bool) -> String?  
{
  return ifTrue ? value : nil
}

let toInt: String -> Int? = { Int($0) }

let anotherOptionalString = makeOptionalStringFrom("3", ifTrue: true)

let anotherOptionalInt = anotherOptionalString.map(toInt).map(doubled)
/// errore di compilazione!
```

Il problema qui è che la funzione `toInt` è di tipo `String -> Optional<Int>`, e il *lifting* della funzione verso il *mondo opzionale* la trasformerebbe in una funzione del tipo `Optional<String> -> Optional<Optional<Int>>`, quindi un intero opzionale contenuto in un altro `Optional`; in base alla terminologia accettata, abbiamo bisogno di un'operazione di `flatMap`:

```Swift
let anotherOptionalInt = anotherOptionalString.flatMap(toInt).map(doubled)  
/// funziona
```

Quindi, per quanto riguarda il tipo `Optional`, `flatMap` è simile a `map`, ma deve essere usato per il *lifting* di funzioni che generano altri valori opzionali. Nel leggere il codice, `map` e `flatMap` possono sostanzialmente essere letti nello stesso modo: indicano entrambi una trasformazione, e il fatto che le funzioni passate a `flatMap` abbiano un tipo diverso può essere considerato un dettaglio di implementazione.

In realtà Swift tratta questi tipi di dato **in maniera differente** rispetto ad altri linguaggi: ad esempio, nei linguaggi *funzionali* `map` e `flatMap` sono di norma *funzioni libere*, mentre in Swift sono **metodi**. In generale, Swift incoraggia l'uso dei metodi, e del loro **concatenamento**, invece di funzioni libere, composte con operatori speciali: si tratta, più che altro, di una questione filosofica, ma nel trasportare queste feature in Objective-C l'approccio basato sui metodi tornerà utile poiché non c'è un modo semplice di rappresentare funzioni libere nel linguaggio, mentre i metodi hanno la classica sintassi alla quale siamo abituati.

<a name="torniamo-a-objective-c"></a>
## Torniamo a Objective-C

Implementiamo la classe `Optional` in Objective-C:

```ObjectiveC
///Optional.h

#import <Foundation/Foundation.h>

NS_ASSUME_NONNULL_BEGIN

@interface Optional : NSObject

+ (Optional*)with:(id _Nullable)value;

- (Optional*)map:(id(^)(id))mapBlock;
- (Optional*)flatMap:(Optional*(^)(id))flatMapBlock;

@end

NS_ASSUME_NONNULL_END 
```

```ObjectiveC
///Optional.m

#import "Optional.h"

@interface Optional ()

@property (strong, nonatomic, nullable) id value;

@end

@implementation Optional

+ (Optional*)with:(id _Nullable)value
{
  Optional* optional = [Optional new];
  optional.value = value;
  return optional;
}

- (Optional *)map:(id  _Nonnull (^)(id _Nonnull))mapBlock
{
  if (self.value != nil)
  {
    return [Optional with:mapBlock(self.value)];
  }
  return self;
}

- (Optional *)flatMap:(Optional* _Nonnull (^)(id _Nonnull))flatMapBlock
{
  if (self.value != nil)
  {
    return flatMapBlock(self.value);
  }
  return self;
}

@end
```

Come possiamo vedere, il metodo di classe `with:` crea un oggetto `Optional` inscatolando un altro oggetto, di tipo non specificato: poiché non esiste un vero concetto di *generic* in Objective-C, non sarà possibile realizzare un tipo `Optional` sicuro, dal punto di vista dei tipi, come in Swift, ma non sarà un grosso problema, e saremo comunque in grado di ereditare da Swift **potenti features**. Il metodo `map:` verifica se l'oggetto contenuto non sia nil, e in tal caso applica la trasformazione espressa con `mapBlock`: in questo caso i *nullability specifiers* di Objective-C ci hanno aiutato nello scrivere la firma dei metodi, infatti abbiamo chiaramente specificato che `mapBlock` accetta e ritorna oggetti che non siano nil; `flatMapBlock` ritorna invece un altro `Optional`.

Possiamo testare la cosa riscrivendo lo stesso esempio visto per Swift:

```ObjectiveC
///Test.h

#import <Foundation/Foundation.h>

@interface Test : NSObject

+ (void)testOptional;

@end
```

```ObjectiveC
///Test.m

#import "Test.h"
#import "Optional.h"

@implementation Test

+ (void)testOptional
{
  Optional* optionalInt = [self makeOptionalIntFrom:@3 ifTrue:YES];
  Optional* optionalDoubledString = [[optionalInt
                                      map:[self doubled]]
                                     map:[self toString]];
  NSLog(@"%@", optionalDoubledString);
}

+ (Optional*)makeOptionalIntFrom:(NSNumber*)fromInt ifTrue:(BOOL)ifTrue
{
  return [Optional with:ifTrue ? fromInt : nil];
}


+ (NSNumber*(^)(NSNumber*))doubled
{
  return ^NSNumber*(NSNumber* value)  {
    return @(value.integerValue*2);
  };
}

+ (NSString*(^)(NSNumber*))toString
{
  return ^NSString*(NSNumber* value)  {
    return [NSString stringWithFormat:@"%@", value];
  };
}

@end
```

Un volta applicate le trasformazioni, abbiamo bisogno di un metodo per "estrarre" l'oggetto all'interno del `Optional`: Swift permette questa operazione a livello di sintassi, mentre in Objective-C possiamo seguire le convenzioni standard in programmazione funzionale, e cioè definire un metodo `get`.

```ObjectiveC
///Optional.h

- (id _Nullable)get;
```

```ObjectiveC
///Optional.m

- (id)get
{
  return self.value;
}
```

Il metodo `get` ritorna un `id _Nullable`: ciò vuol dire che l'oggetto ritornato può (ovviamente) essere nil. Un pattern frequente quando si ha a che fare con oggetti nil consiste nel sostituirli con versioni di "default": può essere utile quando non ci interessano tanto i dati contenuti in un oggetto, ma non vogliamo correre i rischi associati al lavorare riferimenti *null*. Ancora una volta, con la classe `Optional` possiamo evitare istruzioni condizionali ed esprimere il meccanismo di "defaulting" in un modo più dichiarativo: dobbiamo semplicemente aggiungere un metodo `getOrElse:`, che ritorni l'oggetto contenuto se presente, o un oggetto di default passato al metodo stesso:

```ObjectiveC
///Optional.h

- (id)getOrElse:(id(^)())elseBlock;
```

```ObjectiveC
///Optional.m

- (id)getOrElse:(id  _Nonnull (^)())elseBlock
{
  if (self.value != nil)
  {
    return self.value;
  }
  return elseBlock();
}
```

Come si può vedere, il metodo `getOrElse:` **non** ritorna un oggetto `_Nullable`: in effetti non stiamo passando l'oggetto, ma un block che "produrrà" l'oggetto una volta invocato; in questo modo riusciamo a ottenere l'oggetto di default in maniera **lazy**, perché se l'oggetto contenuto è presente, non è necessario generare l'oggetto di default.

Per il resto dell'articolo lavoreremo su un esempio più concreto, e implementeremo diverse funzionalità per la classe `Optional`, rendendola via via più utile e potente.

<a name="esempio-un-parser-json"></a>
## Esempio: un parser JSON

Supponiamo di voler realizzare un'app che mostra informazioni sui film: possiamo ottenerle in formato JSON da un sito come [myapifilms](http://api.myapifilms.com/index.do). La rappresentazione JSON di ogni film è del tipo seguente:

```JSON
{
    "countries": [
        "USA"
    ],
    "directors": [
        {
            "name": "Frank Darabont",
            "nameId": "nm0001104"
        }
    ],
    "filmingLocations": [
        "St. Croix",
        "U.S. Virgin Islands"
    ],
    "genres": [
        "Crime",
        "Drama"
    ],
    "idIMDB": "tt0111161",
    "languages": [
        "English"
    ],
    "metascore": "80/100",
    "originalTitle": "",
    "plot": "Andy Dufresne is a young and successful banker whose life changes drastically when he is convicted and sentenced to life imprisonment for the murder of his wife and her lover. Set in the 1940s, the film shows how Andy, with the help of his friend Red, the prison entrepreneur, turns out to be a most unconventional prisoner.",
    "ranking": 1,
    "rated": "R",
    "rating": "9.3",
    "releaseDate": "19941014",
    "runtime": [
        "142 min"
    ],
    "simplePlot": "Two imprisoned men bond over a number of years, finding solace and eventual redemption through acts of common decency.",
    "title": "The Shawshank Redemption",
    "type": "Movie",
    "urlIMDB": "http://www.imdb.com/title/tt0111161",
    "urlPoster": "http://ia.media-imdb.com/images/M/MV5BODU4MjU4NjIwNl5BMl5BanBnXkFtZTgwMDU2MjEyMDE@._V1_SX214_AL_.jpg",
    "votes": "1,533,914",
    "writers": [
        {
            "name": "Stephen King",
            "nameId": "nm0000175"
        },
        {
            "name": "Frank Darabont",
            "nameId": "nm0001104"
        }
    ],
    "year": "1994"
}
```

Vogliamo definire la nostra rappresentazione interna del film con un *value object*, modificare in parte la struttura e magari ignorare alcuni elementi che non consideriamo rilevanti. Ecco un'interfaccia possibile per la classe `Movie`:

```ObjectiveC
NS_ASSUME_NONNULL_BEGIN

@interface Movie : NSObject

@property (copy, nonatomic, readonly) NSString* title;
@property (copy, nonatomic, readonly) NSNumber* rating;
@property (copy, nonatomic, readonly) NSNumber* year;
@property (copy, nonatomic, readonly) NSNumber* lengthInMinutes;

+ (Movie*)withJSONDict:(NSDictionary*)dict;

@end

NS_ASSUME_NONNULL_END
```

Il metodo di classe `withJSONDict:` crea un'istanza di `Movie` da un dizionario JSON come quello appena mostrato: come si può vedere dall'interfaccia dichiarata, abbiamo deciso di non permettere valori nil per le varie *property*; ma quando abbiamo a che fare con JSON **tutto può succedere**, come chiavi mancanti, valori null o di tipo diverso. Useremo il tipo `Optional` per parsare il dizionario JSON in modo chiaro, dichiarativo e a prova di errore.

Iniziamo con i titolo. Dal JSON vediamo che il titolo si trova in corrispondenza della chiave "title", e dovrebbe essere una stringa (`NSString` in Objective-C), quindi possiamo ottenere il titolo così:

```ObjectiveC
NSString* title = [[[[Optional
                        with:[dict objectForKey:@"title"]]

                       flatMap:^Optional*(id title) {
                         return [Optional with:[title isKindOfClass:[NSString class]] ? title : nil];
                       }]

                      flatMap:^Optional*(NSString* title) {
                        return [Optional with:title.length > 0 ? title : nil];
                      }]

                     getOrElse:^NSString*{
                       return @"NO TITLE";
                     }];
```

Il primo `flatMap` indica che l'oggetto "titolo" deve appartenere alla classe `NSString`. Conviene astrarre questa operazione inserendola direttamente nel costruttore, perché l'applicheremo ogni volta per tutti i dati:

```ObjectiveC
+ (Optional *)with:(id)value as:(Class)valueClass
{
  if ([value isKindOfClass:valueClass])
  {
    return [Optional with:value];
  }
  return [Optional with:nil];
}
```

Il secondo `flatMap:` indica che se la stringa trovata ha lunghezza 0, considereremo il titolo come "sconosciuto". In effetti questo step avrà l'effetto di "filtro", perché lascerà "passare" solo le stringhe non vuote; considerando ciò, aggiungiamo un metodo `filter:` alla classe `Optional`:

```ObjectiveC
///Optional.h

- (Optional*)filter:(BOOL(^)(id))filterBlock;
```

```ObjectiveC
///Optional.m

- (Optional*)filter:(BOOL (^)(id _Nonnull))filterBlock
{
  return [self flatMap:^Optional*(id value) {
    if (filterBlock(value))
    {
      return self;
    }
    else
    {
      return [Optional with:nil];
    }
  }];
}
```

Il metodo `filter:` accetta come parametro un block che ritorna un `BOOL` basato sul valore contenuto: internamente, `filter:` chiamerà comunque `flatMap:`, con un'espressione condizionale che verifichi se `filterBlock` riesce o fallisce.

Infine:

```ObjectiveC
NSString* title = [[[Optional  
                       with:[dict objectForKey:@"title"]
                       as:[NSString class]]

                      filter:^BOOL(NSString* string) {
                        return string.length > 0;
                      }]

                     getOrElse:^NSString*{ return @"NO TITLE"; }];
```

Quindi, nessuna espressione condizionale, e poche linee di codice, completamente dichiarative.

Per quanto riguarda i parametri `rating` e `year` abbiamo bisogno di un `map` che converta `NSString` in `NSNumber`:

```ObjectiveC
NSNumber* rating = [[[[Optional  
                         with:[dict objectForKey:@"rating"]
                         as:[NSString class]]

                        filter:^BOOL(NSString* string) {
                          return string.length > 0;
                        }]

                       map:^NSNumber*(NSString* stringValue) {
                         return [NSDecimalNumber decimalNumberWithString:stringValue];
                       }]

                      getOrElse:^NSNumber*{ return @0; }];
  movie.rating = rating;

  NSNumber* year = [[[[Optional
                       with:[dict objectForKey:@"year"]
                       as:[NSString class]]

                      filter:^BOOL(NSString* string) {
                        return string.length > 0;
                      }]

                     map:^NSNumber*(NSString* stringValue) {
                       return [NSDecimalNumber decimalNumberWithString:stringValue];
                     }]

                    getOrElse:^NSNumber*{ return @0; }];
```

Il parametro `lengthInMinutes` è un po' complicato; dal JSON possiamo vedere che la sua rappresentazione è la seguente:

```JSON
"runtime": [ 
    "142 min" 
]
```

Quindi ci aspettiamo un array, del quale siamo interessati solo al primo elemento (quindi l'array non deve essere vuoto), che deve essere una stringa; di questa stringa abbiamo bisogno solo della prima parte, rimuovendo ` min`. Fortunatamente abbiamo già tutti gli strumenti necessari per procedere:

```ObjectiveC
NSNumber* lengthInMinutes =
  [[[[[[Optional
        with:[dict objectForKey:@"runtime"]
        as:[NSArray class]]

       flatMap:^Optional*(NSArray* array) {
         return [Optional with:[array firstObject]];
       }]

      map:^NSString*(NSString* string) {
        return [string
                stringByReplacingOccurrencesOfString:@" min"
                withString:@""];
      }]

     filter:^BOOL(NSString* string) {
       return string.length > 0;
     }]

    map:^NSNumber*(NSString* stringValue) {
      return @([stringValue integerValue]);
    }]

   getOrElse:^NSNumber*{ return @0; }];
```

In realtà manca qualcosa: il metodo `[array firstObject]` ritorna il primo oggetto dell'array se questo ha almeno un elemento, oppure ritorna nil se l'array è vuoto: non c'è alcuna indicazione sul fatto che vogliamo che il primo elemento sia di tipo `NSString`, quindi dobbiamo aggiungere uno step `filter` per assicurarci che la classe dell'istanza trovata sia corretta. Quindi:

```ObjectiveC
NSNumber* lengthInMinutes =
  [[[[[[[Optional
         with:[dict objectForKey:@"runtime"]
         as:[NSArray class]]

        flatMap:^id(NSArray* array) {
          return [Optional with:[array firstObject]];
        }]

       filter:^BOOL(id value) {
         return [value isKindOfClass:[NSString class]];
       }]

      map:^NSString*(NSString* string) {
        return [string
                stringByReplacingOccurrencesOfString:@" min"
                withString:@""];
      }]

     filter:^BOOL(NSString* string) {
       return string.length > 0;
     }]

    map:^NSNumber*(NSString* stringValue) {
      return @([stringValue integerValue]);
    }]

   getOrElse:^NSNumber*{ return @0; }];
```

Come possiamo vedere, lavorando con la classe `Optional`, aggiungere una condizione significa semplicemente **aggiungere uno step** nella sequenza lineare di espressioni.

<a name="conclusione"></a>
## Conclusione
Implementare la classe `Optional` in Objective-C ci ha permesso di parsare un JSON, cosa spesso difficile da mantenere e soggetta a errori, in un modo semplice e lineare. In realtà ci sono **molti casi** in cui lavorare con gli *optionals* ci può dare gli stessi vantaggi: componendo operazioni di tipo `map`, `flatMap` e `filter` possiamo esprimere il nostro intento in maniera chiara e leggibile. Ma tutto questo non riguarda solo il tipo `Optional`: alcuni lettori avranno probabilmente notato che molti linguaggi (incluso Swift) permettono operazioni dell stesso tipo su liste e array, e il loro significato è identico: `map` e `flatMap`, in particolare, sono **concetti generici** legati ai *monad* (in effetti, anche il tipo `Array` è un *monad*), e riguarda la composizione dichiarativa di trasformazioni. Questo concetto è alla base della **programmazione funzionale**, e penso ci siano molti motivi per applicare concetti simili ad ambienti più *object-oriented*, perché aiuterebbero il programmatore a ragionare sul codice ed esprimere l'intento in maniera più dichiarativa.

Il codice per la classe `Optional` mostrata nell'articolo è reperibile su [GitHub Gist](https://gist.github.com/broomburgo/e318228a5f7d6a605e82).
