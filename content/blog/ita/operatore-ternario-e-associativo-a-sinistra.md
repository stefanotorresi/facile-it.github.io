---
authors: ["lorenzo"]
comments: true
date: "2015-03-02"
draft: false
share: true
categories: [Italiano, PHP]
title: "L'operatore ternario è associativo a sinistra!"

languageCode: "it-IT"
type: "post"
aliases: 
  - "/operatore-ternario-e-associativo-a-sinistra"
images: ['/images/logo.png']

---
Gli [*operatori ternari*](http://it.wikipedia.org/wiki/Operatore_ternario) sono diffusi in molti linguaggi di programmazione e permettono di esprimire con una **sintassi breve** logiche condizionali. Per utilizzarli propriamente in PHP è però necessario conoscerne il comportamento.

Vediamo un esempio

```php
var_dump(true ? 'a' : 'b' ? 'c' : 'd'); // OUTPUT: string(1) "c"
```

Se state pensando che il risultato di questa espressione sia ovvio, vediamo cosa succede ad esempio in javascript

```javascript
console.log(true ? 'a' : 'b' ? 'c' : 'd'); // OUTPUT: a
```
Bene, mentre in PHP il risultato è dato da:

- true è vero, ritorna a
- a castato a bool è vero, **stampa c**


In javascript invece il ragionamento è diverso:

- true è vero, **stampa a**


Perchè questa differenza? Semplicemente perchè in PHP, **diversamente dalla maggioranza dei linguaggi**, l'operatore ternario è associativo a sinistra anzichè a destra.

Tenendone conto potremo evitare comportamenti inconsistenti nel nostro codice.
