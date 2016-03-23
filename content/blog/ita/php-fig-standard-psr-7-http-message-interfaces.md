---
author: "michele"
comments: true
date: "2015-03-23"
draft: false
share: true
categories: [PHP, PSR, PHP Coding standards, Open Source]
title: "PHP coding standard: PSR-7  HTTP message interfaces"
type: "post"

languageCode: "it-IT"

---
Le specifiche PSR-7 descrivono una [proposta di standardizzazione](https://github.com/php-fig/fig-standards/blob/master/proposed/http-message.md) delle **interfacce per i messaggi HTTP**.

Come sappiamo, il protocollo HTTP, attraverso le specifiche redatte dal W3C, definisce una serie di regole di comunicazione che vengono implementate dalle applicazioni client e server che lo adottano.

A prima vista niente di nuovo sotto il Sole dunque, ma cerchiamo insieme di comprendere l'insieme di problematiche che sono al centro di questo nuovo dibattito che riguarda gli standard di codifica del linguaggio PHP.

#### Una questione di interoperabilità tra framework

I principali framework PHP, pur fornendo dei layer di astrazione del protocollo HTTP maturi e funzionali, impongono la scrittura di codice da essi dipendente.

Questo vuol dire che **tutto il codice** che implementa le logiche applicative, anche se ben definito e isolato, **dovrà essere adattato per ciascun framework**, a discapito di un'evidente similarità operativa, come per esempio:

* il recupero delle informazioni dalle variabili del server per costruire la richiesta
* il parsing degli header e del body della richiesta
* l'aggiunta di uno o più header alla risposta
* l'impostazione dello status code

Immaginiamo di voler scrivere un applicativo che abbia questi requisiti:

* aggiunge un header personalizzato alla risposta, `X-GREETINGS`
* l'header deve contenere un messaggio personalizzato in base all'orario della richiesta
* l'header deve essere aggiunto solo in base agli headers inviati ed il verbo HTTP (ad es. solo per le richieste GET)

Decidiamo di scrivere le linee di codice necessarie con **Symfony2** e **Zend Framework 2**, per poi muoverci verso **una sola soluzione ipoteticamente riutilizzabile** in ciascuno dei due contesti.

Diamo per scontato di avere a disposizione un servizio che dato un orario determini con che formula salutarci:

```php
<?php

namespace Acme\Utils;

class GreetingSentence
{
  public static function compose(\DateTime $time, $name)
  {  		  
            $msg = "Ciao $name";  
			$hour = (int)$time->format('h');				

			if ($hour >= 20 && $hour < 6) {
				return $msg + ' buonanotte!';
			} else if ($hour >= 6 && $hour < 13) {
				return $msg + ' buongiorno!';
			} else if ($hour >= 13 && $hour < 18) {
				return $msg + ' buon pomeriggio!';	
			} else if ($hour >= 18 && $hour < 20) {
				return $msg + ' buonasera!';
			} else {
            	return $msg + ' c\'è vita su Marte!';
            }						
  }
}

```

###### Symfony 2
Ecco come implementeremmo la funzionalità richiesta in Symfony2:
```
<?php 

namespace Acme\HelloYouBundle\Controller;

use Symfony\Component\HttpFoundation\Request;
use Symfony\Component\HttpFoundation\Response;
use Acme\Utils\GreetingSentence;

class MainController 
{	
	/**
	 * @param string $name
	 * @param Request $request
     */
	public function greetAction(Request $request)
	{
        $response = new Response();

        if ($request->isMethod('GET')) {
            $name = $request
                ->server
                ->get('HTTP_USER_AGENT'');

            $requestTimeStamp = $request
                ->server
                ->get('REQUEST_TIME');

            $requestTime = new \DateTime();
            $requestTime->setTimestamp($requestTimeStamp);

            $msg = GreetingSentence::compose($requestTime, $name);		
			$response->headers->set('X-GREETINGS', $msg);
        }

        return $response;
    }
} 
```

###### Zend Framework 2
Questa è invece l'implementazione in Zend Framework 2:

```
<?php

namespace Acme\HelloYouModule\Controller;

use Zend\Mvc\Controller\AbstractActionController;
use Zend\View\Model\ViewModel;
use Acme\Utils\GreetingSentence;

class MainController extends AbstractActionController
{
	$response = $this->getResponse();

	public function greetAction()
	{
    	if ($this->getRequest()->getMethod() == 'GET') {
          $name = $this
              ->getRequest()
              ->getServer('HTTP_USER_AGENT');

          $requestTimestamp = $this
              ->getRequest()
              ->getServer('REQUEST_TIME');

          $requestTime = new \DateTime();
          $requestTime->setTimestamp($requestTimeStamp);

          $msg = GreetingSentence::compose($requestTime, $name);			
          $response->setHeader('X-GREETINGS', $msg);
        }
	}
    
    $response->send();
}
```
Risulta evidente come siamo stati a costretti ad implementare due pezzi di codice quasi identici. Vediamo ora **come uno standard come quello proposto dal PSR-7 possa aiutarci**.

#### Middleware: ovvero componenti web riutilizzabili
Potremmo voler implemenare la nostra soluzione come un **middleware**, ovvero un componente che **esiste durante il ciclo di vita di una comunicazione HTTP**, cioè dall'accettazione della richiesta fino all'evasione della risposta.

Negli esempi precedenti abbiamo aggiunto il codice di gestione all'interno dei controller, e questa implementazione carica l'Action della responsabilità di esaminare la richiesta e comporre la risposta; questa operazione dovrà essere **ripetuta in ogni controller** della nostra applicazione.

Per fortuna i maggiori framework espongono astrazioni molto complete del ciclo di vita di una richiesta HTTP, che permettono di implementare il nostro middleware.

* in Symfony2 l'interfaccia [HttpKernelInterface](https://github.com/symfony/HttpKernel/blob/master/HttpKernelInterface.php) definisce un componenente capace di calcolare per una data una richiesta (Request) una determinata risposta (Response).

* in Zend Framework l'interfaccia che definisce un componente capace di associare una richiesta ad una risposta si chiama [DispatchableInterface](https://github.com/zendframework/zf2/blob/master/library/Zend/Stdlib/DispatchableInterface.php).

Tuttavia se volessimo utilizzare lo stesso codice in Zend Framework 2 e Symfony2, dovremmo wrappare il codice necessario attorno a entrambe le interfacce oppure importare i componenti dell'uno nell'altro.

A questo punto possiamo cominciare a capire realmente il significato di questo nuovo processo di standardizzazione proposto dal [**FIG**](http://www.php-fig.org/).

#### Scrivere middleware usando le interfacce proposte nello standard PSR-7

La proposta dello standard PSR-7 si riassume nella descrizione di una serie di [interfacce](https://github.com/php-fig/http-message/tree/master/src).

Attualmente Matthew Weier O'Phinney, membro attivo del FIG e main contributor di Zend Framework 2 sta realizzando un'[implementazione](https://github.com/phly/http/tree/master/src) completa di queste interfacce.

A questo punto possiamo scrivere il codice di un middleware, aderendo alle specifiche PSR-7, come una semplice classe capace di interpretare una richiesta e costruire una risposta secondo le nostre necessità:

```
<?php

use phly\http\ServerRequestFactory;
use phly\http\ServerRequest;
use phly\http\Response;

class AcmeLifeCycle
{
    /**
    * @param ServerRequest|null $request
    * @param Response|null $response
    *
    * @return Response
    */
    public function doSomething(ServerRequest $request = null, Response $response = null)
    {
        if (is_null($request)) {
            /** @var ServerRequest $request */
            $request = ServerRequestFactory::fromGlobals();
        }

        /** @var Response $response */
        if (is_null($request)) {
            /** @var Response $response */
            $response = new Response();
        }

        if ($request->getMethod() === 'GET') {
            $serverParams = $request->getServerParams();
            $name = $serverParams['HTTP_USER_AGENT'];

            $requestTimeStamp = $serverParams['REQUEST_TIME'];

            $requestTime = new \DateTimeImmutable();
            $requestTime
                ->setTimestamp($requestTimeStamp);

            $msg = GreetingSentence::compose($requestTime, $name);

            //Questo metodo è un costruttore di copia
            $response = $response->withHeader('X-GREETINGS', $msg);
        }

        return $response;
    }
}
```
Da notare che sia la classe `Response` che `ServerRequest` implementano lo stesso trait `Message`.

La classe `Message` è stata volutamente concepita affinchè sia **immutabile** ed i vari metodi `->withX()` restituiranno solo copie modificate dell'oggetto inizialmente istanziato.

Ovviamente questo middleware adesso come adesso è tutt'altro che utilizzabile all'interno di un framework, poichè ciascun vendor usa un modello diverso per richieste e risposte, ma è molto probabile che **in un futuro** non troppo lontano le varie implementazioni **convergeranno** su qualcosa di simile a quella appena vista e proposta da O'Phinney.

#### Conclusioni

Se pensiamo agli strumenti forniti di base dal linguaggio PHP (e non ai framework) nel contesto di una comunicazione HTTP, ci rendiamo conto che di fatto dovremmo ridurci a lavorare con le sole variabili globali d'ambiente contenenti tutti i valori necessari (`$_SERVER`, `$_GET`, `$_POST`, `$_FILES`, ...).

Invece, grazie a PSR-7, potremmo **standardizzare la rappresentazione degli oggetti coinvolti nel ciclo richiesta\riposta** (cioè i messaggi HTTP), e poter finalmente sviluppare componenti **middleware indipendenti** dal framework utilizzato, il cui unico presupposto esistenziale è quello di **gestire messaggi HTTP tramite il linguaggio PHP**.

Sicuramente tenere [a portata di github](https://github.com/php-fig/fig-standards) gli sviluppi di questa proposta vorrà dire tenersi aggiornati sui **futuri sviluppi** dei più blasonati framework e componenti PHP, tenendo conto, **come già accaduto per le precedenti proposte** [PSR-0](https://github.com/php-fig/fig-standards/blob/master/accepted/PSR-0.md) e [PSR-4](https://github.com/php-fig/fig-standards/blob/master/accepted/PSR-4-autoloader.md) (autoloading) e [PSR-3](https://github.com/php-fig/fig-standards/blob/master/accepted/PSR-3-logger-interface.md) (logging), che, quasi sicuramente, i principali vendor le implementeranno nelle loro prossime versioni.

#### Altre risorse consultabili

* Blog di Matthew Weier O'Phinney: https://mwop.net/blog/2015-01-08-on-http-middleware-and-psr-7.html
* Specifiche PSR-7: https://github.com/php-fig/fig-standards/blob/master/proposed/http-message.md
* Interfacce proposte dalla PSR-7: https://github.com/php-fig/http-message
* Implementazione di Matthew Weier O'Phinney: https://github.com/phly/http/tree/master/src
* Michael Dowling spiega la streaming interface: http://mtdowling.com/blog/2014/07/03/a-case-for-higher-level-php-streams/
