---
authors: ["lorenzo"]
comments: true
date: "2015-03-02"
draft: false
share: true
categories: [Italiano, PHP, PHP internals]
title: "var_dump() aggiunge proprietà pubbliche a DateTime"

languageCode: "it-IT"
type: "post"
aliases: 
  - "/var_dump-aggiunge-proprieta-pubbliche-a-datetime"
---
PHP, fin dalla versione 5.2.0, introduce gli oggetti `\DateTime` per operare con date ed intervalli, fornendo finalmente un alternativa alle vecchie funzioni procedurali.

Recentemente mi sono reso conto di uno strano comportamento che si verifica quando vengono chiamate `var_dump`, `print_r`, `var_export` o `debug_zval_dump` su un istanza di `\DateTime`.

Considerando il seguente codice e il suo output:

```
$date = new \DateTime();
var_dump(isset($date->date)); // OUTPUT: bool(false)
```

ci rendiamo conto del fatto che non esiste alcuna proprietà `$date` all'interno dell'istanza di `\DateTime`.

Se più avanti nel codice scrivessimo

```
var_dump($date);
```

otterremmo come output:

```
class DateTime#1 (3) {
  public $date =>
  string(26) "2015-03-01 23:18:42.000000"
  public $timezone_type =>
  int(3)
  public $timezone =>
  string(11) "Europe/Rome"
}
```

il che ci mostra chiaramente una proprietà pubblica `$date`, contenente il timestamp attuale.

Controllando nuovamente l'esistenza della proprietà pubblica `$date` scopriremmo infatti che ora è presente ed ha il valore settato:

```
var_dump(isset($date->date)); // OUTPUT: bool(true)
echo $date->date; // OUTPUT: 2015-03-01 23:21:05.000000
```

Questo comportamento è dovuto alla funzione interna `static HashTable *date_object_get_properties(zval *object)` che alla [linea 2176](https://github.com/php/php-src/blob/968a9f48071bcc099b4e978fc99fd09b6f69d172/ext/date/php_date.c#L2176-L2177) di `php_date.c` imposta la proprietà `$date`. Dal momento che `var_dump` e le altre funzioni citate sopra chiamano proprio quella funzione durante l'esecuzione, ci imbattiamo in questo comportamento.

```
	/* first we add the date and time in ISO format */
	ZVAL_STR(&zv, date_format("Y-m-d H:i:s.u", sizeof("Y-m-d H:i:s.u")-1, dateobj->time, 1));
	zend_hash_str_update(props, "date", sizeof("date")-1, &zv);
```

È importante perciò sapere e tenere a mente che alcune proprietà visibili con var_dump(), se non documentate, non dovrebbero essere utilizzate in quanto potrebbero risultare in comportamenti non attesi!
