---
authors: ["matteo"]
comments: true
date: "2015-04-01"
draft: false
share: true
categories: [Italiano, PHP, Symfony, Database]
title: "Gestione di ACL su insiemi di records"

languageCode: "it-IT"
type: "post"
aliases:
  - "/gestione-di-acl-su-insiemi-di-records"
---
Le **ACL** (*Access Control List*) sono un strumento molto potente per poter definire l'accesso a risorse con una granularità molto fine. Nel quotidiano abbiamo già modo di utilizzarle per definire i permessi per accedere a file su Unix o quali pacchetti far passare attraverso un firewall o ancora l'accesso a database.

In **Symfony** le *ACL* sono disponibili *out-of-the-box* nel caso di installazione completa e permettono la definizione delle regole di accesso a risorse tramite **ruoli** e **maschere**. Mentre i **ruoli** rappresentano dei sottoinsiemi degli utenti di una data applicazione (*amministratori*, *backoffice*, *business analyst*) e possono essere visti come delle etichette da assegnare ad un utente, le **maschere** sono la rappresentazione numerica delle azioni che possono essere effettuate dagli utenti aventi determinati ruoli. Per tornare all'esempio del filesystem Unix, i ruoli possono essere *Owner*, *Group* o *Others* mentre le maschere sono ad esempio 7 (lettura, scrittura e esecuzione) indicato per ciascun ruolo.

Avendo la possibilità di poter definire più di soli tre ruoli e più di sole tre azioni chiaramente le opportunità diventano molteplici dando quindi estrema flessibilità al sistema.

Di base **Symfony** persiste le informazioni definite tramite le *ACL* (chiamate **ACE** ovvero *Access Control Entry*) in tabelle specifiche tramite cui verificare se un utente può o meno effettuare una operazione: la verifica è immediata tramite le API esposte da **Symfony** fintanto che si tratta di operare su una singola risorsa ma la soluzione non scala certamente nel caso si debbano filtrare le risorse accessibili da un utente. Per una più approfondita trattazione su come utilizzare le API rese disponibili da **Symfony**, rimandiamo alla [relativa pagina della documentazione](http://symfony.com/it/doc/current/cookbook/security/acl.html)

Per ovviare a questo problema all'interno di un progetto di **Facile.it** si è pensato di adottare una soluzione che faccia uso di [**ElasticSearch**](https://www.elastic.co/products/elasticsearch) per poter restituire ad un utente il sottoinsieme dei record a cui può accedere quando scorre una lista di record. **ElasticSearch** era già stato scelto per poter migliorare la ricerca all'interno dei record delle differenti entità archiviate sul database relazionale, quindi si è trattato di operare due scelte che permettessero una gestione ottimale:

* dividere ciascuna entità persitita come documento su **ElasticSearch** in due parti, *metadata* e *data*;
* realizzare delle **annotation custom** per poter esprimere le **ACE** per ciascuna entità e campo delle entità.

Dividere il documento in due parti rappresenta il punto di partenza per ottenere il risultato cercato: ogni volta che **FOSElasticaBundle** procede con la copia di una entità su **ElasticSearch**, viene richiamata una callback (definibile con la configurazione del bundle) che si occupa di recuperare tutte le **ACE** dell'entità in questione e includerle nel campo *metadata* del documento.
Successivamente, quando si vorrà effettuare una query sull'insieme dei record persistiti su **ElasticSearch**, si dovrà solamente replicare le logiche (generiche) per l'applicazione delle **ACL** per così ottenere i soli risultati che l'utente può accedere. In questo modo vengono evitate complesse query su un database relazionale (che prevedono l'utilizzo di diverse join su più tabelle) come invece sarebbe richiesto da un approccio basato sul solo utilizzo di **MySQL**.

Tramite le **annotation** è invece possibile definire le varie **ACE** per ciascuna classe o attributo della classe così da poter configurare l'accesso alle entità direttamente dai file delle entità stesse, fornendo uno strumento familiare per chi normalmente non utilizza le **ACL**. Per creare una annotation custom rimandiamo anche in questo caso all'articolo presente nella [documentazione di **Doctrine**](http://doctrine-common.readthedocs.org/en/latest/reference/annotations.html) ricordando, però, che è importante gestire una propria **cache delle annotation** perchè questa non è fornita di base da **Doctrine**. Un comando apposito può facilitare la gestione dell'aggiornamento delle **ACE** di una entità ogni qualvolta venga effettuato un *deploy* dell'applicazione, integrando così le *annotation* create con le *API* di **Symfony** per le **ACL**.

Il file che definisce una entità di **Doctrine** potrà dunque presentarsi in questo modo.

	<?php
    namespace Facile\Ws\Bundle\Entity;

    use Doctrine\ORM\Mapping as ORM;
    use Facile\Ws\BunnyBundle\Annotation\EntityAce as ACE;

    /**
     * @ORM\Table(name="user")
     * @ORM\Entity
     * @ACE("ROLE", name="ROLE_ADMIN", mask="MASK_MASTER")
     * @ACE("ROLE", name="ROLE_USER", mask="MASK_VIEW")
     */
    class User
    {
        /**
         * @ORM\Column(name="id", type="smallint", nullable=false, options={"unsigned"=true})
         * @ORM\Id
         * @ORM\GeneratedValue(strategy="AUTO")
         * @ACE("ROLE", name="ROLE_USER", mask="MASK_VIEW")
         */
        protected $id;

        /**
         * @ORM\Column(name="timestamp", type="datetime", nullable=false)
         * @ACE("ROLE", name="ROLE_USER", mask="MASK_VIEW")
         */
        protected $timestamp;

        /**
         * @ORM\Column(type="string", length=25, unique=true)
         * @ACE("ROLE", name="ROLE_ADMIN", mask="MASK_MASTER")
         * @ACE("ROLE", name="ROLE_USER", mask="MASK_VIEW")
         */
        protected $username;

        ...
     }

