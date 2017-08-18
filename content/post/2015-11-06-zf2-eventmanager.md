---
title: EventManager
subtitle: Componente usado para implementar o padrão observer
date: 2015-11-06
tags: ["php", "zf2"]
---
O `EventManager` é um componente usado para implementar o padrão `observer` `( aspect-oriented design )` e programação orientada a eventos em aplicações `Zend Framework 2`.
Um `EventManager` pode ser criado por simplesmente instanciar um novo objeto, exemplo:
```php
$eventManager = new EventManager();
```

<!--more-->

#### Attaching Event Listeners

Você usa o método `attach()` para anexar ouvintes de eventos para o `event manager.`
Esse método leva três parâmetros, o nome do evento para anexar, uma que pode ser chamado para executar quando o evento é acionado, e a prioridade do evento.

<br/>
##### Parâmetros:

##### Nome do evento
O nome do evento é o identificador exclusivo para este evento.
Será usado para disparar o evento em um momento posterior.


##### callback
O parâmetro `callback` será chamado quando o evento nomeado é acionado.
No `callback` será passado um único parâmetro que será uma instância da `EventInterface`,
geralmente um evento que é uma implementação concreta dessa interface que é fornecido com o framework.
O `EventInterface` fornece informações sobre o evento disparado através dos métodos:

> `getName()`

>`getTarget()`

> `getParams()`

um parâmetro adicional passado para o evento pode ser acessado usando o método `getParams()`.


##### Prioridade
O parâmetro opcional prioridade é usado para determinar a ordem em que os ouvintes de evento são chamados quando vários ouvintes são registrados para o mesmo evento.
Prioridades podem ser positivas ou negativas, e que a prioridade padrão é 1.
se os ouvintes compartilham a mesma prioridade, eles são disparados na ordem em que foram registrados.

Exemplo de uso no arquivo `Module.php` para disparar um método de validação

```PHP
public function init(ModuleManager $moduleManager)
{
    $sharedEvents = $moduleManager->getEventManager()->getSharedManager();
    $sharedEvents->attach('Zend\Mvc\Controller\AbstractActionController', MvcEvent::EVENT_DISPATCH, array($this, 'validaAuth'), 100);
}
```
<br/>

Exemplo de `listener`

```PHP
$logger = new Logger();
$eventManager->attach('insert', function(EventInterface $event) use ($logger) {
     $params = $event->getParams();
     $logger->log('Usuário ' . $params['id'] . ' adicionou ' . $params['itemId']);
}, 100);
```

O `listener` ainda precisa de um gatilho `trigger` que é o responsável por executar a ação

<br/>
<br/>

#### Triggering Events

Evento são acionados através do `Event Manager` usando o método `trigger()`.
O método `trigger` leva 4 parâmetros: o nome do evento, um alvo opcional,
uma matriz opcional de argumentos, e um callback opcional.

<br/>

##### Parâmetros:

##### Nome do evento
O nome do evento é o identificador exclusivo para este evento.
em ordem para os `listeners` serem disparados, este deve ser idêntico ao nome usado
para anexar um `listener` no método `attach()`


##### alvo
O parâmetro `$target` normalmente será a instância do objeto atual,
ou o nome da função de dispara de fora um objeto ( ou uma `static function` ).


##### array de argumentos
O parâmetro argumento é um `key/value` array com argumentos que serão passados para o
evento e está disponível no `listener` através do método `getParams()`.


##### callback
O `callback` pode e será chamado depois que o `listener` registrado for concluído.
O `callback` será passado os resultados retornados a partir desse `listener`.



<br/>
#### Usando o componente EventManager

##### Estrutura do projeto:

- zf2-componentization
    - src
        - Components
            - EventManager
                - CreditCard.php
                - ListenerAggregate.php
    - vendor
    - index.php
    - composer.json


<br/>


Configurando o `composer.json`, depois somente baixar o
[composer](https://getcomposer.org/download/) e executar:

`composer.json`
```php
{
    "name": "Your project",
    "description": "Componentização ZF2",
    "license": "MIT",
    "config": {
        "process-timeout": 3000
    },
    "require": {
        "php": ">=5.5",
        "zendframework/zendframework": "2.3"
    },
    "autoload": {
        "psr-0": {
            "Components": "src/Components/"
        }
    }
}
```
<br/>

Configurando o autoloader para o framework e para nosso namespace Components

`index.php`
```PHP
require 'vendor/zendframework/zendframework/library/Zend/Loader/StandardAutoloader.php';

$loader = new Zend\Loader\StandardAutoloader([
    'autoregister_zf' => true
]);
$loader->registerNamespace('Components', 'src/Components');
$loader->register();
```
<br/>

Criamos a seguinte estrutura na classe `CreditCard` onde ela implementa a interface
`EventManagerAwareInterface` onde possui o método com a instância de
`EventManager`, possui um atributo `$event` onde ele recebe a instância
de `EventManager` com a configuração do `Identifiers` que estabelece o nome nome da classe
que o evento está chamando, ou seja o relacionamento do `EventManager` que está sendo criado
com uma classe, no nosso caso a classe `CreditCard`, no próximo exemplo será criado o `trigger` que será disparado

`CreditCard.php`
```PHP
namespace Components\EventManager;

use Zend\EventManager\EventManager as Event;
use Zend\EventManager\EventManagerAwareInterface;
use Zend\EventManager\EventManagerInterface;

class CreditCard implements EventManagerAwareInterface
{
    /**
     * @var
     */
    protected $event;

    /**
     * @param EventManagerInterface $eventManager
     * @return $this
     */
    public function setEventManager(EventManagerInterface $eventManager)
    {
        $eventManager->setIdentifiers([
            __CLASS__,
            get_called_class()
        ]);
        $this->event = $eventManager;
        return $this;
    }

    /**
     * @return mixed
     */
    public function getEventManager()
    {
        if (null == $this->event)
            $this->setEventManager(new Event());

        return $this->event;
    }
}
```
<br/>

Criamos o método `getCreditCardFlag` onde ele resgata o `EventManager`
e criamos o `trigger`, `getCreditCardFlag` recebe um parâmetro que será o número
do cartão que será retornado no `callback` atravéz do método `getParam()` a badeira
do cartão de crédito.

`CreditCard.php`
```PHP
<?php

namespace Components\EventManager;

use Zend\EventManager\EventManager as Event;
use Zend\EventManager\EventManagerAwareInterface;
use Zend\EventManager\EventManagerInterface;

class CreditCard implements EventManagerAwareInterface
{
    /**
     * @var
     */
    protected $event;

    const BRAND_VISA = 'visa';
    const BRAND_MASTERCARD = 'mastercard';
    const BRAND_DISCOVER = 'discover';
    const BRAND_AMEX = 'amex';
    const BRAND_DINERS_CLUB = 'diners_club';
    const BRAND_JCB = 'jcb';
    const BRAND_SWITCH = 'switch';
    const BRAND_SOLO = 'solo';
    const BRAND_DANKORT = 'dankort';
    const BRAND_MAESTRO = 'maestro';
    const BRAND_FORBRUGSFORENINGEN = 'forbrugsforeningen';
    const BRAND_LASER = 'laser';

    /**
     * @param EventManagerInterface $eventManager
     * @return $this
     */
    public function setEventManager(EventManagerInterface $eventManager)
    {
        $eventManager->setIdentifiers([
            __CLASS__,
            get_called_class()
        ]);
        $this->event = $eventManager;
        return $this;
    }

    /**
     * @return mixed
     */
    public function getEventManager()
    {
        if (null == $this->event)
            $this->setEventManager(new Event());

        return $this->event;
    }


    /**
     * @param $number
     */
    public function getCreditCardFlag($number)
    {
        $this->getEventManager()->trigger(
            __FUNCTION__,
            $this,
            ['flag' => $this->getBrand($number)]
        );
    }

    /**
     * @param $number
     * @return int|string
     */
    public function getBrand($number)
    {
        foreach ($this->getSupportedBrands() as $brand => $val) {
            if (preg_match($val, $number)) {
                return $brand;
            }
        }
    }

    /**
     * @return array
     */
    public function getSupportedBrands()
    {
        return array(
            static::BRAND_VISA => '/^4\d{12}(\d{3})?$/',
            static::BRAND_MASTERCARD => '/^(5[1-5]\d{4}|677189)\d{10}$/',
            static::BRAND_DISCOVER => '/^(6011|65\d{2}|64[4-9]\d)\d{12}|(62\d{14})$/',
            static::BRAND_AMEX => '/^3[47]\d{13}$/',
            static::BRAND_DINERS_CLUB => '/^3(0[0-5]|[68]\d)\d{11}$/',
            static::BRAND_JCB => '/^35(28|29|[3-8]\d)\d{12}$/',
            static::BRAND_SWITCH => '/^6759\d{12}(\d{2,3})?$/',
            static::BRAND_SOLO => '/^6767\d{12}(\d{2,3})?$/',
            static::BRAND_DANKORT => '/^5019\d{12}$/',
            static::BRAND_MAESTRO => '/^(5[06-8]|6\d)\d{10,17}$/',
            static::BRAND_FORBRUGSFORENINGEN => '/^600722\d{10}$/',
            static::BRAND_LASER => '/^(6304|6706|6709|6771(?!89))\d{8}(\d{4}|\d{6,7})?$/',
        );
    }
}
```

<br/>

Ficando da seguinte maneira o arquivo de inicialização `index.php`, é instânciado `CreditCard`
resgatando o `EventManager` e criando um `listener` com o método `attach()`, é passado o nome
do método que possui o `trigger()` o segundo parâmetro é o `callback` com o retorno do
`EventManager` passamos a `key` `flag` para o método `getParam()` que nos retorna o nome
da bandeira do número de cartão de crédito que foi passado como parâmetro para `getCreditCardFlag`

`index.php`
```PHP
require 'vendor/zendframework/zendframework/library/Zend/Loader/StandardAutoloader.php';

$loader = new Zend\Loader\StandardAutoloader([
    'autoregister_zf' => true
]);
$loader->registerNamespace('Components', 'src/Components');
$loader->register();

$creditCard = new \Components\EventManager\CreditCard();
$creditCard->getEventManager()->attach('getCreditCardFlag', function($e) {
    var_dump($e->getParam('flag'));
});
$creditCard->getCreditCardFlag(4444333322221111); // visa
```
<br/>

Retorno:

    visa

<br/>
<br/>

#### Short Circuiting

Quando vários `listeners` são registrados é possível parar todos os `listener` disparando um
`stopping event propagation` a partir de um `listener`.
Isso é feito usando o `stopPropagation()` dentro do `listener`.

```PHP
// email the user
$ventManager->attach('item-added', function(EventInterface $event) use ($email) {
     $params = $event->getParams();
     $result = $email->emailUser($params['userId']);
     if ($result) {
          $event->stopPropagation(true);
     }
}, 100);

// log the item added
$eventManager->attach('item-added', function(EventInterface $event) use ($logger) {
     $params = $event->getParams();
     $logger->log('User ' . $params['userId'] . ' added item ' . $params['itemId']);
});
```
<br/>

Em nosso projeto vamos criar um método novo na classe `CreditCard`, onde usamos outro tipo de `trigger`
`triggerUntil` que significa que até quando uma determinada condição não estiver satisfeita não é retornado
o resultado, isso é util quando queremos que a depender do resultado possa ser tomado outro fluxo
dentro da aplicação, utilizamos um `callback` para fazer uma condição, quando o valor passado não for
esperado é emitido uma mensagem e o `trigger` é finalizado.

`CreditCard.php`
```PHP
    /**
     * @param $number
     * @return mixed
     */
    public function brandSuported($number)
    {
        $arg = compact('number');
        $result = $this->getEventManager()->triggerUntil(
            __FUNCTION__,
            $this,
            $arg,
            function ($v) use ($number) {
                if ($this->getBrand($number) != self::BRAND_MASTERCARD) {
                    return true;
                }
            }
        );

        if ($result->stopped()) {
            echo " | Somente aceitamos mastercard | ";
            return $result->last();
        }

        echo " Processo de transa&ccedil;&atilde;o em andamento ... ";
    }
```
<br/>

Ficando da seguinte maneira a `index.php`, usamos o novo método criado na classe
`CreditCard`, `brandSuported()`, passamos dois números de cartão de crédito para simular o exemplo,
um de bandeira visa e outro e bandeira mastercard, note que também alteramos o segundo
parâmetro do `listener` para `*` pois estamos chamando mais de um método, então
o `listener` escuta qualquer método, também pode ser passado em formato de `array`
`['getCreditCardFlag', 'brandSuported']`

`index.php`
```PHP
require 'vendor/zendframework/zendframework/library/Zend/Loader/StandardAutoloader.php';

$loader = new Zend\Loader\StandardAutoloader([
    'autoregister_zf' => true
]);
$loader->registerNamespace('Components', 'src/Components');
$loader->register();

$creditCard = new \Components\EventManager\CreditCard();
$creditCard->getEventManager()->attach('*', function($e) {
    var_dump( $e->getParam('flag') );
});
$creditCard->getCreditCardFlag(4444333322221111); // visa

// Short Circuiting
$creditCard->brandSuported(4444333322221111);     // visa
$creditCard->brandSuported(5266736406590700);     // mastercard
```
<br/>

Retorno:

    visa | Somente aceitamos mastercard | Processo de transação em andamento ...

<br/>
<br/>

#### Aggregate Listeners

O `ListenerAggregateInterface` fornece um método alternativo para anexar vários
`listeners` através de uma única classe.

`Aggregate listeners` permitem que vários `listeners` possam ser anexados,
mas o mais importante desprendido de um único lugar.
Os `listeners` também são acionados dentro do escopo da classe que implementa a interface,
que pode ser conveniente para os `listeners` que precisam fazer uso de serviços externos.
`Aggregate listeners` são uma excelente escolha se você precisa separar `listeners`,
como eles permitem que você guarde o `Zend\Stdlib\Callbackhandler`
totalmente dentro do `aggregate listener`, o que torna a remoção mais fácil.

`listener` estão ligados utilizando o método `attach()`, passando uma instância
`EventManagerInterface` como o parâmetro único.
`listener` são separadas usando o método de `detach()`, que aceita uma instância
`EventManagerInterface` como seu parâmetro.

```PHP
class MailAggregateListener implements ListenerAggregateInterface
{
    protected $listeners = [];
    protected $emailService;

    public function __construct(EmailService $emailService)
    {
        $this->emailService = $emailService;
    }

    public function handleEvent(EventInterface $event)
    {
        $this->emailService->emailEvent($event);
    }

    public function attach(EventManagerInterface $eventManager)
    {
        $listenTo = array(‘add-cart’, ‘remove-cart’, ‘empty-cart’, ‘checkout-cart’);
        foreach ($listenTo as $listen) {
            $this->listeners[$listen] = $this->attach($listen, [$this, ‘handleEvent’]);
        }
    }

    public function detach(EventManagerInterface $eventManager)
    {
        // just detach remove-cart listener
        $eventManager->detach($this->listeners[‘remove-cart’]);
    }
}
```

<br/>

Em nosso projeto vamos criar um arquivo chamado de `ListenerAggregate.php` dentro do diretório
`Components/EventManager/`, classe chamada de `ListenerAggregate` implementando `ListenerAggregateInterface`
onde implementaremos dois métodos comforme a interface nos provê `attach()` e `detach()` onde será feito a
implementação de criação dos `listeners` e a remoção dos mesmos.
Criamos a mais três métodos `executeProcessCreditCardOne()`, `executeProcessCreditCardTwo()` e `executeProcessCreditCardThree()`
onde são registrados no método `attach()`, como se trata de `ListenerAggregate` podem ser criados quantos métodos forem necessários
e atachados, o último parâmetro passado para o `attach` serve para especificar a prioridade de execução, quando negativo
a prioridade é menor, quando não especificado a prioridade é executado por ordem de registro.

`ListenerAggregate.php`
```PHP
namespace Components\EventManager;

use Zend\EventManager\EventManagerInterface;
use Zend\EventManager\ListenerAggregateInterface;

class ListenerAggregate implements ListenerAggregateInterface
{
    protected $listeners = [];
    protected $params;

    public function __construct($params)
    {
        $this->params = $params;
    }

    /**
     * @param EventManagerInterface $events
     */
    public function attach(EventManagerInterface $events)
    {
        $array = new \ArrayIterator($this->params);
        while($array->valid())
        {
            $this->listeners[] = $events->attach( $array->current()[0], [$this, $array->current()[1]], $array->current()[2] );
            $array->next();
        }
    }

    /**
     * @param EventManagerInterface $events
     */
    public function detach(EventManagerInterface $events)
    {
        foreach ( $this->listeners as $key => $listener)
        {
            if ( $events->detach($listener) )
                unset($this->listeners[$key]);
        }
    }

    /**
     * Process One
     */
    public function executeProcessCreditCardOne()
    {
        echo " | Execu&ccedil;&atilde;o do primeiro processamento de cart&atilde;o de cr&eacute;dito | ";
    }

    /**
     * Process Two
     */
    public function executeProcessCreditCardTwo()
    {
        echo " Execu&ccedil;&atilde;o do segundo processamento de cart&atilde;o de cr&eacute;dito | ";
    }

    /**
     * Process Three
     */
    public function executeProcessCreditCardThree()
    {
        echo " Execu&ccedil;&atilde;o do terceiro processamento de cart&atilde;o de cr&eacute;dito | ";
    }
}
```

<br/>

Na classe `CreditCard` vamos adicionar um novo método que possui os vários eventos que vamos registrar
com `attachAggregate()` no próximo exemplo.

```PHP
/**
 * @param $number
 */
public function processCreditCard($number)
{
    $this->getEventManager()->trigger(
        'processCreditCardOne',
        $this,
        ['number' => $number, 'status' => 'primeiro processamento'],
        function () {}
    );

    $this->getEventManager()->trigger(
        'processCreditCardTwo',
        $this,
        ['number' => $number, 'status' => 'segundo processamento'],
        function () {}
    );

    $this->getEventManager()->trigger(
        'processCreditCardThree',
        $this,
        ['number' => $number, 'status' => 'terceiro processamento'],
        function () {}
    );
}
```

<br/>

Na `index` instânciamos o `ListenerAggregate()` passando para seu construtor os nomes dos métodos e suas
prioridades, usamos o método `attachAggregate()` passando o objeto da classe `ListenerAggregate()` que foi
criada com os `listeners`, chamamos o método `processCreditCard()` onde ele possui vários `triggers`
ficando da seguinte maneira:

`index.php`
```php
require 'vendor/zendframework/zendframework/library/Zend/Loader/StandardAutoloader.php';

$loader = new Zend\Loader\StandardAutoloader([
    'autoregister_zf' => true
]);
$loader->registerNamespace('Components', 'src/Components');
$loader->register();

$creditCard = new \Components\EventManager\CreditCard();
$creditCard->getEventManager()->attach('*', function($e) {
    if (!empty($e->getParam('flag')))
        echo $e->getParam('flag');
});
$creditCard->getCreditCardFlag(4444333322221111); // visa

// Short Circuiting
$creditCard->brandSuported(4444333322221111);     // visa
$creditCard->brandSuported(5266736406590700);     // mastercard

// Aggregate Listeners
$listenerAggregate = new \Components\EventManager\ListenerAggregate([
    ['processCreditCardOne', 'executeProcessCreditCardOne', 100],
    ['processCreditCardTwo', 'executeProcessCreditCardTwo', 100],
    ['processCreditCardThree', 'executeProcessCreditCardThree', -100],
]);
$creditCard->getEventManager()->attachAggregate($listenerAggregate);
$creditCard->processCreditCard(5266736406590700);
```

<br/>

Retorno:

    visa | Somente aceitamos mastercard | Processo de transação em andamento ... | Execução do primeiro processamento de cartão de crédito | Execução do segundo processamento de cartão de crédito | Execução do terceiro processamento de cartão de crédito |

<br/>
<br/>

#### Share Event Manager

A interface `SharedEventManagerInterface` fornece uma maneira de anexar `listeners` para muitos `events managers`,
ou para um `event manager` não no âmbito da própria classe.

Significa que um módulo pode anexar um `listener` de eventos de outro módulo sem necessitar acesso direto a qualquer das ocorrências da classe do módulo.

O `SharedManagerInterface` é uma implementação concreta do `SharedEventManagerInterface` que é fornecido com o framework.

Anexando a `listeners` de eventos a um `SharedEventManager` é semelhante a um `EventManager` padrão.

O método `attach()` é usado mas com um parâmetro extra no início;
`$id` - o identificador para o evento acionado.

Cada instância de `EventManager` pode compor zero ou mais identificadores.

```PHP
$sharedEventManager = new SharedEventManager();
$sharedEventManager->attach(‘Product’, ‘add’, function(Event $event) {
     echo ‘Product Added!’;
});

$eventManager = new EventManager(‘Product’);
$eventManager->setSharedManager($sharedEventManager);
$eventManager->trigger(‘add’, new Product());
```
<br/>

Em nosso arquivo `index.php` vamos instânciar `SharedEventManager` e registar um `listener`
para `Components\EventManager\CreditCard` método `getCreditCardFlag`, ficando da seguinte maneira:

`index.php`
```php
require 'vendor/zendframework/zendframework/library/Zend/Loader/StandardAutoloader.php';

$loader = new Zend\Loader\StandardAutoloader([
    'autoregister_zf' => true
]);
$loader->registerNamespace('Components', 'src/Components');
$loader->register();

$creditCard = new \Components\EventManager\CreditCard();
$creditCard->getEventManager()->attach('*', function($e) {
    if (!empty($e->getParam('flag')))
        echo $e->getParam('flag');
});
$creditCard->getCreditCardFlag(4444333322221111); // visa

// Short Circuiting
$creditCard->brandSuported(4444333322221111);     // visa
$creditCard->brandSuported(5266736406590700);     // mastercard

// Aggregate Listeners
$listenerAggregate = new \Components\EventManager\ListenerAggregate([
    ['processCreditCardOne', 'executeProcessCreditCardOne', 100],
    ['processCreditCardTwo', 'executeProcessCreditCardTwo', 100],
    ['processCreditCardThree', 'executeProcessCreditCardThree', -100],
]);
$creditCard->getEventManager()->attachAggregate($listenerAggregate);
$creditCard->processCreditCard(5266736406590700);

// Shared Event Manager
$sharedEvent = new Zend\EventManager\SharedEventManager();
$sharedEvent->attach('Components\EventManager\CreditCard', 'getCreditCardFlag', function($e) {

    echo "<br/><br/>";
    echo "<strong>Name:</strong> ";
    echo $e->getName() . "<br/><br/>";

    echo "<strong>Target:</strong> ";
    echo get_class($e->getTarget());
    echo "<br/><br/>";

    echo "<strong>Bandeira:</strong> ";
    echo $e->getParam('flag');

});

$newCreditCard = new \Components\EventManager\CreditCard();
$newCreditCard->getEventManager()->setSharedManager($sharedEvent);
$newCreditCard->getCreditCardFlag(30083521152975);
```
<br/>

Será observado que o retorno terá sucesso somente por causa da linha
```PHP
$newCreditCard->getEventManager()->setSharedManager($sharedEvent);
```

<br/>

Retorno:

    visa | Somente aceitamos mastercard | Processo de transação em andamento ... | Execução do primeiro processamento de cartão de crédito | Execução do segundo processamento de cartão de crédito | Execução do terceiro processamento de cartão de crédito |

    Name: getCreditCardFlag

    Target: Components\EventManager\CreditCard

    Bandeira: diners_club

<br/>
<br/>

##### Referências

 - [The EventManager](http://framework.zend.com/manual/current/en/modules/zend.event-manager.event-manager.html)
 - [ZF2 Certification Study Guide](http://www.zend.com/en/services/certification/zf2-certification-study-guide)

----------------
O Código usado nesse projeto está no GitHub ==> <a href="https://github.com/luk4z7/zf2-componentization" target="blank">luk4z7</a>

-----------------

<br/>
<br/>
<br/>
<br/>
