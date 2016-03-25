---
authors: ["luca"]
comments: true
date: "2015-03-09"
draft: false
share: true
categories: [Italiano, PHP, Open source, Doctrine, MySQL]
title: "MySQL has gone away? Come back!"

languageCode: "it-IT"
type: "post"
aliases: 
  - "/mysql-has-gone-away-come-back"
---

A ben più di uno sviluppatore sarà capitato di incappare nel comune errore [*MySQL server has gone away!*](http://dev.mysql.com/doc/refman/5.5/en/gone-away.html), magari seguito da un eccezione lanciata da una delle [**classi PDO**](http://php.net/manual/en/book.pdo.php), come ad esempio *PDOStatement::execute(): Error reading result set's header*.

Nella maggior parte dei casi, quando questo avviene in ambiente PHP, siamo connessi in maniera **persistente** (per fortuna!) ed a causa di una esecuzione **troppo lunga**, la connessione col server MySQL va in **timeout**. Lunghi tasks in batch, chiamate a ws non particolarmente rapidi, carichi elevati del server, sono alcuni degli scenari possibili.

Qualora l'errore sia noto e prevedibile (ad esempio avvenga alla stessa linea di codice ad ogni esecuzione) è possibile chiudere la connessione in maniera esplicita ed evitarlo, ma questa logica oltre che risultare **scomoda** e **ripetitiva** non è sempre applicabile e richiede una esplicita precauzione dello sviluppatore.

### Ma quanto è grave il problema?

Come illustrato, un errore di tipo *MySQL server has gone away* **non** ha alterato le informazioni presenti nel database, **non** implica un errore logico o di integrità e **non** ci impedisce di tentare nuovamente la query da cui è scaturito. Perciò si potrebbe dire che, da un punto di vista applicativo l'errore, se correttamente **intercettato e gestito**, non sia per nulla grave.

### Allora potrei ignorarlo?

Ignorare deliberatamente degli errori non può essere considerata una strategia vincente, perciò per prima cosa bisogna assicurarsi di aver fatto tutto quanto suggerito nella [documentazione](http://dev.mysql.com/doc/refman/5.5/en/gone-away.html) per **rimuovere le cause del problema**.

### E se non dovesse bastare?

Ritentare la query dovrebbe essere sempre possibile nonchè piuttosto semplice, a patto di saper riconoscere con precisione le eccezioni del **tipo corretto**.

A tal proposito è importante ricordare come la classe [*PDOException*](http://php.net/manual/en/class.pdoexception.php) non fornisca attraverso il metodo pubblico *getCode()* il **codice di errore del DB vendor** (es. [2006](http://dev.mysql.com/doc/refman/5.5/en/error-messages-client.html#error_cr_server_gone_error) per MySQL), ma lo inserisca piuttosto nel *message*.

    /** @var $e PDOException */
    echo $e->getMessage(); // 'SQLSTATE[HY000]: General error: 2006 MySQL server has gone away'`

### Come funziona?
Chiariamoci le idee con del codice:

```
$sql = 'SELECT count(*) FROM `TABLE`';
$retry = true;

while ($retry) {

    try {

        $nb = $pdo->query($sql)->fetchColumn();

    } catch (PDOException $e) {
		
        $retry = false;
        
        if($this->isMySQLHasGoneAwayException($e)) {
        
        	$this->reconectDB();
            $retry = true;
        }
    }        
}
# CODICE DEMO, NON FUNZIONANTE!
```

Nell'esempio  sopra riportato, una volta accertata la natura dell'eccezione si procede **riconnettendosi** al database (una **nuova** connessione!) e **ripetendo** la query, magari con un limite di tentativi per evitare ricursioni infinite!

Trattandosi di una **nuova connessione**, non è possibile utilizzare questa procedura nel caso l'errore avvenga durante una [**transazione**](http://en.wikipedia.org/wiki/Database_transaction) MySQL. Tutte le query inviate all'interno della connessione sarebbero ovviamente **perdute** e non sarebbe possibile recuperarle!

### Ma devo farlo ogni volta a mano?

**NO!**

In [**Facile.it**](http://www.facile.it/), utilizzando largamente l'ORM *Doctrine*, abbiamo pensato allo sviluppo di una piccola estensione, [**DoctrineMySQLComeBack**](https://github.com/facile-it/doctrine-mysql-come-back) che riteniamo utile distribuire in maniera **libera**.

Compatibile dalla versione 2.3 fino alla stabile 2.5, si occupa di **ritentare automaticamente** le SELECT in ambiente MySQL. Cosi non ci si dovrà più preoccupare del successo di ogni singola query per portare a termine le esecuzioni.

Le **istruzioni** per l'installazione e l'uso sono disponibili su [***GitHub***](https://github.com/facile-it/doctrine-mysql-come-back), e chiunque è libero di **contribuire** per migliorarne le funzionalità :)

[![Latest Stable Version](https://poser.pugx.org/facile-it/doctrine-mysql-come-back/v/stable.svg)](https://packagist.org/packages/facile-it/doctrine-mysql-come-back) [![Total Downloads](https://poser.pugx.org/facile-it/doctrine-mysql-come-back/downloads.svg)](https://packagist.org/packages/facile-it/doctrine-mysql-come-back) [![License](https://poser.pugx.org/facile-it/doctrine-mysql-come-back/license.svg)](https://packagist.org/packages/facile-it/doctrine-mysql-come-back)
