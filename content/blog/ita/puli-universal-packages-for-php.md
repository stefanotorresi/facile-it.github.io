---
authors: ["sergio"]
comments: true
date: "2015-12-15"
draft: false
image: ""
menu: ""
share: true
categories: [Italiano, "PHP", "Open Source"]
title: "Puli: Universal Packages for PHP"

languageCode: "it-IT"
type: "post"
aliases:
  - "/puli-universal-packages-for-php"
images: ['/images/logo.png']

---

## Composer e l'ecosistema PHP
L'ecosistema PHP è notevolmente cambiato negli ultimi anni grazie all'introduzione di *Composer*.
Esso ha definito degli standard per risolvere problemi comuni come l'autoloading e la gestione di pacchetti e dipendenze.
In questi ultimi anni si è assistito ad un proliferare di numerose componenti software riusabili che hanno significativamente modificato le strategie di sviluppo.

Nonostante Composer si sia evoluto molto, in alcuni contesti le sue funzionalità risultano essere limitanti e incomplete.
I framework e i pacchetti più complessi hanno bisogno di una gestione personalizzata delle dipendenze. Generalmente sono necessarie operazioni di configurazione e registrazione. Ciò è specialmente vero per pacchetti core dei framework come i *template engines*.
Composer, inoltre, possiede una cattiva gestione delle risorse non PHP. I file di configurazione devono essere gestiti manualmente oppure è necessario adottare standard strettamente legati a ciascun framework.

## La soluzione: Puli
A partire dal gennaio 2014 è stato sviluppato **Puli, Universal Packages for PHP**, da Bernhard Schussek.
Puli mira a risolvere i problemi d'integrazione menzionati in precedenza creando un **layer d'astrazione aggiuntivo tra Composer e l'applicazione**. Tale layer consente di gestire le risorse secondo convenzioni indipendenti dai framework. Ciò significa che Puli può essere usato anche dalle applicazioni che non ne fanno uso.
Attualmente Puli è in Beta, la versione stabile non è stata ancora annunciata ma è prevista per l'inizio del 2016.
**Puli è stato ideato per affiancare Composer**.

Puli lavora ad un livello più alto per fornire le seguenti funzionalità:

 - <a href="#puli-path">Gestione dei path virtuali per risorse non PHP</a>;
 - <a href="#override-puli-path">Overriding dei path virtuali</a>;
 - <a href="#binding-type">Interazione semplificata dei pacchetti con l'uso di Binding Type</a>;
 - <a href="#url-generation">Generazione automatica di URL per risorse pubbliche</a>;
 - <a href="#assets-install">Installazione automatica delle risorse pubbliche nelle document root</a>.

## Le componenti di Puli
Puli offre svariate funzionalità organizzate nei seguenti componenti:

 - Resource repository
 - Discovery service
 - Public resources manager

### Configurazione
Nell'offire questi servizi, Puli legge e scrive un proprio file di configurazione `puli.json` presente nella root directory del pacchetto, allo stesso modo del `composer.json`.

Tale configurazione può essere modificata manualmente oppure utilizzando l'**interfaccia CLI di Puli**;
Tale strumento può essere installato da Packagist tramite il pacchetto `puli/cli`.
Al pari di Composer, sono disponibili archivi Phar per un uso immediato.

### Installazione
Affiché le applicazioni possano usare i servizi offerti da Puli è necessario procedere all'installazione.
Per applicazioni che non fanno uso di framework è sufficiente abilitare il *Composer Plugin* `puli/composer-plugin`.
Quest'azione viene svolta da Composer con il comando:
```
$ composer require puli/composer-plugin:^1.0
```

Tale plugin consente di generare atomaticamente la cosiddetta **Puli Factory Class**: componente chiave da cui è possibile accedere a tutti i servizi offerti da Puli.

A questo punto è possibile usare Puli nel modo seguente:
```php
require './vendor/autoload.php';

// La costante PULI_FACTORY_CLASS viene 
// definita al "composer install"
$factoryClass = PULI_FACTORY_CLASS;
$factory = new $factoryClass();

$puliRepository = $factory->createRepository();
$puliDiscovery  = $factory->createDiscovery($puliRepository);
$puliGenerator  = $factory->createUrlGenerator($puliDiscovery);
```

### Resource repository
Il resource repository fornisce un **file system virtuale** per accedere alle risorse non PHP.
Ciò consente di localizzare facilmente i file senza conoscere la loro effettiva posizione.

#### Accesso alle risorse con Composer
Con Composer era frequente scrivere codice di questo tipo:
```php
$translationFileName = realpath(__DIR__ . '/../vendor/acme/blog/resources/translation-it.xml');
$translationContent = file_get_contents($translationFileName);
```

Tale operazione fa uso di dettagli implementativi: il layout delle directory fornito da Composer. La soluzione non è riusabile nè configurabile.

<a name="puli-path"></a>
#### Accesso alle risorse con Puli
Ciascun pacchetto compatibile con Puli può registrare diversi **Path Mapping** utilizzati per tradurre i **percorsi virtuali (Puli path)** in **percorsi reali**.
Tali Path Mapping sono elencati nel `puli.json` e normalmente sono modificati con il comando Puli CLI `map`.
Ad esempio, il maintainer di acme/blog può eseguire questo comando per definire un nuovo mapping:
```
$ puli map /acme/blog resources
```

Tale operazione crea un'associazione tra il prefisso del path virtuale `/acme/blog` e la directory `resources`.

Un ipotetico utilizzatore del pacchetto acme/blog potrà quindi accedere alle risorse nel seguente modo:
```php
$puliPath = '/acme/blog/translation-it.xml';
$translationContent = $puliRepository->get($puliPath)->getBody();
```

È importante notare come i Path mapping siano definiti dal maintainer del pacchetto. Le altre componenti si limitano ad usarli per accedere alle risorse.

<a name="override-puli-path"></a>
#### Resource overriding
È tuttavia presente un'eccezione a questa regola, tale concetto è alla base del **Resource overriding**.
Immaginiamo di dover lavorare con due pacchetti: A e B.
Il pacchetto B è dipendente dal pacchetto A.
B, tuttavia, vuole personalizzare alcune risorse fornite da A.
Il maintainer di B può quindi rimpiazzare (*override*) tali risorse aggiungengo nuovi Path Mapping.

<a name="binding-type"></a>
### Discovery service
Il componente discovery semplifica l'interazione tra pacchetti: consente di condividere le risorse senza codice boilerplate attraverso l'uso di **Binding Type**.

Per il componente Discovery i pacchetti sono siddivisi in due categorie:

 - **Resource providers**: pacchetti che offrono risorse;
 - **Resource consumers**: pacchetti che richiedono risorse.

I Binding Types sono definiti dai resource consumers. Ciascun Binding Type è idendificato da un nome (es. `acme/translator/messages`).
I resource providers possono quindi associare una o più risorse a tali Binding Type.
**L'associazione avviene legando un Puli Path ad un Binding Type**.

I *Binding Type* sono quindi un'interfaccia per la condivisione di risorse. 

#### Esempio
Esaminiamo, con un esempio, l'interazione tra due pacchetti: `acme/md2html` e `acme/blog`.

`acme/md2html` è un componente per tradurre contenuto Markdown verso pagine Html. Md2html è un sistema configurabile: la resa dell'output avviene attraverso template Twig modificabili.

`acme/blog` fornisce tools per la gestione di articoli da blog. Siccome questi ultimi possono essere scritti in Markdown, `acme/blog` fa uso di `acme/md2html`.

Il componente `acme/blog` può personalizzare i template di Md2html in questo modo:
```php
$md2html = new Md2Html();
$md2html->useHeaderTemplate(__DIR__ . '/resources/templates/markdown-header.twig');

$htmlContent = $md2html->translate('## markdown content');
```

Questo approccio ha due svantaggi:

 - `acme/blog` ha la diretta responsabilità di configurare correttamente Md2html utilizzando l'API fornita;
 - Gli utenti di `acme/blog` non possono modificare facilmente le personalizzazioni apportate a Md2html.

Tali problemi possono essere risolti con Puli.

Il mantainer di Md2html decide di rendere il suo pacchetto compatibile con Puli.

Definisce quindi il Binding Type: `acme/md2html/header-template` con il seguente comando:
```
$ puli type --define acme/md2html/header-template
```

Il maintainer di `acme/blog` può quindi usare tale Binding Type nel seguente modo:
```
$ puli map /acme/blog resources
$ puli bind /acme/blog/templates/markdown-header.twig acme/md2html/header-template
```

Così `acme/md2html` può leggere il nuovo template `markdown-header.twig` in modo trasparente.

Gli utilizzatori di acme/blog che desiderassero modificare tale template hanno due strategie:

 - Modificare l'associazione al Binding Type `acme/md2html/header-template`
 - Sovrascrivere il Puli Path `/acme/blog/templates/markdown-header.twig` con il meccanismo dell'overriding esaminato in precedenza.

<a name="url-generation"></a>
### Public resources manager
Puli gestisce anche le risorse pubbliche, gli assets. In particolare, è in grado di generare path e URL da usare nei template o nei fogli di stile.

Come primo passo, è necessario **registrare i server web che servono tali risorse**.
È possibile aggiungere un nuovo server con il comando:
```
$ puli server --add localhost public_html
```
In questo esempio, `localhost` è il nome del server, mentre `public_html` è la sua *document root*.

Ora è possibile pubblicare delle risorse sul server `localhost` con il comando:
```
$ puli publish /acme/blog/public localhost /blog
```
Ciò mappa il Puli Path `/acme/blog/public` alla directory `blog` della document root del server `localhost`.

Puli ora ha tutte le informazioni per generare i persorsi delle risorse pubbliche:
```php
$puliPath = '/acme/blog/public/images/logo.png';
$url = $puliGenerator->generateUrl($puliPath);
echo $url; // -> "/blog/images/logo.png"
```

L'ultima operazione può essere eseguita in un template Twig con l'utilizzo dell'estensione `puli/twig-extension`:
```twig
<img src="{{ resource_url('/acme/blog/public/images/logo.png') }}" />
```
<a name="assets-install"></a>
#### Installazione delle risorse
Puli può installare automaticamente le risorse pubbliche nella document root dei web server.

Il comando è il seguente:
```
$ puli publish --install
Installing /acme/blog/public into public_html via symlink...
```

Gli autori di Puli hanno annunciato che **tale funzionalità sarà rimossa da Puli CLI in favore di estensioni Gulp o altri tool simili**.

### Conclusioni
Abbiamo visto come Puli offra numerose funzionalità aggiuntive rispetto al solo utilizzo di Composer.
Puli è un progetto giovane, nel corso del tempo saranno aggiunte ulteriori funzionalità che lo renderanno ancora più potente.
Per approfondire, la documentazione ufficiale è molto chiara e completa ed è disponibile all'indirizzo: [http://docs.puli.io/](http://docs.puli.io/).
