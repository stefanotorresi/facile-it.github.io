---
authors: ["sergio"]
comments: true
date: "2015-03-16"
draft: false
share: true
categories: [Italiano, Software testing, Test coverage, TDD]
title: "Software testing: Coverage vs Efficacia"

languageCode: "it-IT"
type: "post"
aliases:
  - "/software-testing-coverage-vs-efficacia"
---
Controllare il tasso di coverage dei test è un'attività frequente tra gli sviluppatori.

Numerosi sono i fattori che hanno reso il code coverage popolare:

* è una metrica facile da comprendere;
* si misura senza difficoltà;
* è oggettiva e imparziale;
* è universale (applicabile a tutti i paradigmi di programmazione).

Ma al di là di questi vantaggi, ***possiamo affermare che una test suite con un'alta percentuale di coverage sia realmente efficace?***

### Quando un test è efficace?
Una test suite è considerata efficace se consente di rilevare una grande quantità di failure; d'altronde, ciò è proprio la finalità ultima del software testing.
La massima efficacia è realizzabile solo applicando testing esaustivo ma, siccome ciò è spesso impraticabile, ci si accontenta di test più semplici con un'efficacia inferiore.
È importante notare che, a differenza del coverage, il livello d'efficacia viene stabilito soggettivamente dallo sviluppatore.

### Quanti tipi di coverage?
La copertura può essere misurata in molti modi differenti. Essa è sempre un rapporto tipicamente espresso in forma percentuale. 

La formula per il calcolo del coverage è la seguente:

![](/images/software-testing-coverage-vs-efficacia/CodeCogsEqn.gif)

Notare il termine generico *obiettivo*: diverse tipologie di coverage hanno obiettivi differenti.
Qui di seguito è presente un elenco esemplificativo e non esaustivo delle diverse tipologie di coverage:

* Statement coverage
* Branch coverage (o decision coverage)
* Condition coverage
* Function/Method coverage

Nel corso di quest'articolo si fa riferimento sempre allo Statement coverage, talvolta chiamato semplicemente "coverage".

--------------
### Un esempio pratico

#### Funzione under-test

``` python
def weightedAverage(array):
	sum = weightSum = 0     # Bug, dovrebbe essere: sum = weightSum = 0.0
	for (value, weight) in array:
		sum += value        # Bug, dovrebbe essere: sum += value * weight
		weightSum += weight
	return sum / weightSum
```
Le specifiche della funzione *weightedAverage* sono le seguenti:

* Calcolare la media ponderata di un array contenente coppie di valori.  
Ad esempio: `[(10, 2), (11, 4), (5, 1)]`
* Restituire sempre un risultato di tipo float.
* Non effettuare divisione tra interi per non ridurre la precisione del risultato.

I commenti già presenti nel codice evidenziano due errori importanti:

* La media non è calcolata correttamente: l'istruzione `sum += value` dovrebbe essere `sum += value * weight`
* L'output della funzione non è di tipo float se i dati di input sono interi.  
L'errore è risolvibile modificando `sum = weightSum = 0` in `sum = weightSum = 0.0`

#### Test suite #1

``` python
def testWeightedAverage():
	expected = 25.0
	inputArray = [(24, 1), (26, 1)]
	actual = weightedAverage(inputArray)
	if expected == actual:
		print 'Okay'
	else:
		print 'Failure!'
```

La test suite #1 ha statement coverage 100%, tuttavia non consente di rilevare i due errori menzionati in precedenza. Sebbene la coverage sia alta, l'efficacia è molto bassa.

#### Test suite #2

``` python
def testWeightedAverageDifferentWeight():
	expected = 26.0
	inputArray = [(24, 1), (27, 2)]
	actual = weightedAverage(inputArray)
	if expected == actual:
		print 'Okay'
	else:
		print 'Failure!'
```
La test suite #2 ha statement coverage 100%, ma non consente di verificare correttamente che l'output della funzione sia di tipo float.
Il primo errore viene rilevato, il secondo no.
È importante notare che questo problema viene risolto aggiungendo l'asserzione `type(actual) is float`.  
Le asserzioni sono parte fondamentale del testing; date loro la giusta importanza! Ogni buona asserzione in più consente di migliorare sensibilmente l'efficacia dei vostri test case.

#### Test suite #3

``` python
def testWeightedAverageFloatResult():
	expected = 27.2
	inputArray = [(24, 1), (28, 4)]
	actual = weightedAverage(inputArray)
	if expected == actual:
		print 'Okay'
	else:
		print 'Failure!'
```
La test suite #3 ha statement coverage 100% ed è efficace perché ci consente di rilevare tutti gli errori.

### Conclusioni
Abbiamo visto che percentuali di coverage alte non sempre implicano test efficaci.
Tuttavia test suite efficaci hanno necessariamente bisogno di coverage rate alti.
In temini logici possiamo quindi affermare che:

![](/images/software-testing-coverage-vs-efficacia/CodeCogsEqn--1-.gif)

Coverage rate bassi sono un campanello d'allarme che ci consente di stabilire la scarca efficacia dei test.
Possiamo infatti affermare che:

![](/images/software-testing-coverage-vs-efficacia/CodeCogsEqn--2-.gif)
