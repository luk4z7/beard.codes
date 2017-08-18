---
title: ServiceManager
subtitle: Ou mais conhecido como gerenciador de serviços
date: 2015-11-18
tags: ["php", "zf2"]
---
Agora vamos falar um pouco sobre `ServiceManager` peça fundamental para que possamos ter bom entendimento do ZF2.
O `Service Manager` em Zend Framework 2 é usado para encontrar e instanciar serviços (classes)
incluindo cumprindo com quaisquer dependências que um serviço possa exigir.

<!--more-->

Zend Framework 2 usa `Service Manager` em toda a estrutura do framework,
com `Service managers` especializados usados para encontrar tipos específicos de serviços,
incluindo:

- Controllers
- Plugins
- View Helpers
- Input Filters
- Filters
- Validators
- Form Elements
- Hydrators
- Application Services ( Criado pelos usuários )

<br/><br/>

#### Service Locator

O padrão de projeto `Service Locator` descreve um método de obtenção de um serviço através de uma camada de abstração.
`Service Manager` em Zend Framework 2 implementa a interface `ServiceLocatorInterface`.
A interface prescreve dois métodos, `has()` para determinar se um determinado serviço tenha sido registrado com o `Service Manager`,
e `get()` para retornar uma instância desse serviço.

```php
namespace Zend\ServiceManager;

interface ServiceLocatorInterface
{
    public function get($name);

    public function has($name);
}
```
<br/>

Quando é pedido para retornar um serviço,  a classe `Zend\ServiceManager\ServiceManager` vai criar instâncias de
objetos gradativamente do serviço solicitado, usando tipicamente `service factories` para criar os serviços
e satisfazer as suas dependências.
Os serviços devem ser criados com dependências definidas no construtor ( ou com setters ).
sempre que possível, a injeção de dependência deve ser usada para satisfazer dependências nos serviços criados pelo `ServiceManager`.

<br/>

#### Plugin Managers

Existem várias implementações do `ServiceLocatorInterface` que têm a tarefa de localizar diferentes serviços e plugins.

<br/>

#### Abstract Plugin Manager

O `AbstractPluginManager` é uma implementação de `Service Manager` para gerenciamento de plugins.
A classe abstrata fornece métodos para definir e obter plugins, juntamente com a criação de serviços das classes `invokable`, `factories` e objetos que pode ser chamados, como as `closures`.

<br/>

#### Controller Manager

O `Zend\Mvc\Controller\ControllerManager` estende de `AbstractPluginManager`, e é um gerenciador de plugin que é responsável pela recuperação e criação de classes do controlador.
Ele é usado internamente pelo roteador `MVC` para localizar e recuperar os controladores com base na chave fornecida pela rota.
É muito importante que os controladores estejam em um gerenciador de plug-in separado por razões de velocidade e segurança.

<br/>

#### Other Plugin Managers

Há também um número de `Plugin Managers` que têm a tarefa de criação e recuperação de tipos específicos de serviços e plugins.
Cada um destes `Plugins Managers` estende a `AbstractPluginManager`:

- ControllerPluginManager       -   Criação e recuperação de controller plugin
- ViewHelperPluginManager       -   Criação e recuperação de view helpers
- InputFilterPluginManager      -   Criação e recuperação de input filters
- FilterPluginManager           -   Criação e recuperação de filters
- ValidatorPluginManager        -   Criação e recuperação de validators
- FormElementManager            -   Criação e recuperação de form elements
- HydratorPluginManager         -   Criação e recuperação de Hydrators
- RoutePluginManager            -   Criação e recuperação de rotas
- SerializerAdapterManager      -   Usado para gerenciar instâncias de Serializer

<br/>

Vamos alterar um pouco a estrutura de nosso projeto e adicionar alguns arquivos para
exemplificar o uso de `AbstractPluginManager`, `AbstractPluginManagerFactory`, vamos
criar alguns plugins customizados.

##### Estrutura do projeto:

- zf2-componentization
    - src
        - Components
            - EventManager
                - CreditCard.php
                - ListenerAggregate.php
            - ServiceManager
                - Delegator
                    - CreditCardDelegator.php
                    - CreditCardDelegatorLazy.php
                - Factories
                    - AbstractCreditCardFactory.php
                    - CreditCardDelegatorFactory.php
                    - CreditCardParametersFactory.php
                - PluginManager
                    - Plugins
                        - Boleto1.php
                        - Boleto2.php
                    - Service
                        - PluginFactory.php
                    - Plugin.php
    - vendor
    - index.php
    - composer.json

<br/>

Primeiro passo vamos criar um arquivo com nome `Plugin.php`, em seguida uma classe chamada de `Plugin`
onde essa classe estende de `AbstractPluginManager` que por sua vez estende de
`ServiceManager` e implementa `ServiceLocatorAwareInterface`, `AbstractPluginManager`
possui um método abstrato chamado de `validatePlugin()` onde será necessário implementar.

`Plugin.php`
```PHP
namespace Components\ServiceManager\PluginManager;

use Zend\ServiceManager\AbstractPluginManager;

/**
 * Class Plugin
 * @package Components\ServiceManager\PluginManager
 */
class Plugin extends AbstractPluginManager
{
    /**
     * @param mixed $plugin
     */
    public function validatePlugin($plugin)
    {
        // TODO
    }
}
```
<br/>

Criamos agora nossa `factory`, um arquivo chamado de `PluginFactory.php` com uma classe chamada de
`PluginFactory` que será registrada mais tarde no `service_manager` nas `factories`, cumprindo seu papel de uma "fabrica",
ficando muito facil depois criar uma classe como um plugin, pois como ela herda de `ServiceManager` somente passamos
os serviços que queremos registrar via `setInvokableClass()`.

A constante `PLUGIN_MANAGER_CLASS` recebe a classe de plugin que será criado um serviço.

Em nosso exemplo sobrescrevemos o método `createService()` para evitar erros com serviços não registrados, como
em nosso exemplo não carregamos todo o framework e que são usados no método da classe `AbstractPluginManagerFactory`
como `Config`, `DiAbstractServiceFactory` e `Di`,

`PluginFactory.php`
```php
namespace Components\ServiceManager\PluginManager\Service;

use Zend\Mvc\Service\AbstractPluginManagerFactory;
use Zend\ServiceManager\ServiceLocatorInterface;

/**
 * Class PluginManager
 * @package Components\ServiceManager\PluginManager
 */
class PluginFactory extends AbstractPluginManagerFactory
{
    /**
     * Constante que recebe o namespace da classe a ser instânciada
     */
    const PLUGIN_MANAGER_CLASS = 'Components\ServiceManager\PluginManager\Plugin';

    /**
     * @param ServiceLocatorInterface $serviceLocator
     * @return \Zend\ServiceManager\AbstractPluginManager
     */
    public function createService(ServiceLocatorInterface $serviceLocator)
    {
        $pluginManagerClass = self::PLUGIN_MANAGER_CLASS;

        /**
         * @var $plugins \Zend\ServiceManager\AbstractPluginManager
         */
        $plugins = new $pluginManagerClass;
        $plugins->setServiceLocator($serviceLocator);

        return $plugins;
    }
}
```

<br/>

Criamos dois plugins, dois arquivos chamados de `Boleto1.php` e `Boleto2.php` com as
classes `Boleto1` e `Boleto2` somente com um construtor para simplificar o exemplo.

`Boleto1.php`
```PHP
namespace Components\ServiceManager\PluginManager\Plugins;

/**
 * Class Boleto1
 * @package Components\ServiceManager\PluginManager\Plugins
 */
class Boleto1
{
    /**
     * Boleto1 constructor.
     */
    public function __construct()
    {
        echo "BOLETO 1";
    }
}
```

`Boleto2.php`
```PHP
namespace Components\ServiceManager\PluginManager\Plugins;

/**
 * Class Boleto2
 * @package Components\ServiceManager\PluginManager\Plugins
 */
class Boleto2
{
    /**
     * Boleto2 constructor.
     */
    public function __construct()
    {
        echo " | BOLETO 2 | ";
    }
}
```

<br/>

Na `index.php` vamos instanciar `ServiceManager`
```PHP
$serviceManager = new Zend\ServiceManager\ServiceManager();
```
<br/>

Criar um array com nossa `factory` em seguida instanciar `ServiceManagerConfig`
e passar nosso array com as `factories` ou `invokables`.
```PHP
$config = [
    'factories' => [
        'Plugin' => 'Components\ServiceManager\PluginManager\Service\PluginFactory'
    ]
];

$serviceConfig = new \Zend\Mvc\Service\ServiceManagerConfig($config);
$serviceConfig->configureServiceManager($serviceManager);
```
<br/>

Depois que o serviço `Plugin` já está registrado somente usar o método `get()` para resgatar
a instância do serviço, configuramos mais dois serviços que são nossos plugins `Boleto1` e `Boleto2`,
depois novamente somente chamar os serviços dentro da aplicação.

```PHP
$plugin = $serviceManager->get('Plugin');
$plugin->setInvokableClass('Boleto1', 'Components\ServiceManager\PluginManager\Plugins\Boleto1');
$plugin->setInvokableClass('Boleto2', 'Components\ServiceManager\PluginManager\Plugins\Boleto2');

$boleto = $plugin->get('Boleto1');
$boleto = $plugin->get('Boleto2');
```
Retorno:

    BOLETO 1 | BOLETO 2 |

Lembrando que vamos analisar alguns termos visto nesse exemplo e como usá los, destacamos algumas como:
`invokables`, `factories`, `Config`, `ServiceConfig` etc ...

<br/>
<br/>

#### Invokables

A maneira mais simples de criar serviços através de um `service manager` é usando um `invokable`.

Um `invokable` é simplesmente uma classe que pode ser instanciada sem dependências
( ou com dependências que são cumpridas por um `initializer`).

`invokables` podem ser definidos usando a key "invokables" no array de configuração passado para o objeto `Config`

ex1:

```PHP
'controllers' => array(
    'invokables' => array(
        'Application\Controller\Index' => 'Application\Controller\IndexController'
    ),
),
```
ex2:
```php
$config = new Config( [
     ‘invokables’ => [
         ‘hydrator’ => ‘Zend\Stdlib\Hydrator\ClassMethods'
     ],
] );
$serviceManager = new ServiceManager( $config );
```

<br/>

##### setServices

Em nosso arquivo `index.php` vamos registrar um serviço usando o `setService`, no final do arquivo
vamos instanciar `ServiceManager`, registramos o serviço com `setService` instanciando a classe `CreditCard`,
depois chamamos o serviço com o método `get`, pode ser visto a estrutura da classe no retorno do `print_r`, no
segundo exemplo chamamos duas vezes o mesmo serviço atribuindo para váriaveis de nomes diferentes e fazemos uma comparação,
as váriaveis são idênticas pelo motivo que por default `ServiceManager` cria as instâncias compartilhadas.

`index.php`
```PHP
// Registrando serviços com setService
$serviceManager2 = new \Zend\ServiceManager\ServiceManager();
$serviceManager2->setService('CreditCardWithSetService', new Components\EventManager\CreditCard());
$servicoCreditCard = $serviceManager2->get('CreditCardWithSetService');
$servicoCreditCard2 = $serviceManager2->get('CreditCardWithSetService');
print_r( $servicoCreditCard );
var_dump( $servicoCreditCard === $servicoCreditCard2);
```
Retorno:

    Components\EventManager\CreditCard Object ( [event:protected] => ) bool(true)

<br/>

##### setInvokableClass

No exemplo acima aprendemos como criar serviços e chamá-los usando `setService` e `get`,
o grande problema do `setService` vem a ser o acoplamento que é criando quando
instânciamos o objeto passado como parâmetro, para isso existe uma outra maneira de registrar um serviço
com o `setInvokableClass`, quando o serviço somente é "invocado" pelo nome da classe

```PHP
$serviceManager2->setInvokableClass('CreditCardWithInvokableClass', '\Components\EventManager\CreditCard');
$serviceCreditCardWithInvokableClass = $serviceManager2->get('CreditCardWithInvokableClass');
print_r($serviceCreditCardWithInvokableClass);
```
<br/>

Da mesma maneira que foi feito com `setService` chamando dois serviços e os comparando, com o
`setInvokableClass` também obterá o mesmo resultado, para que possamos alterar isso quando necessitamos
de uma nova instância passamos o terceiro parâmetro para `setInvokableClass` como `false`, ficando
da seguinte maneiro nosso código:

```PHP
$serviceManager2->setInvokableClass('CreditCardWithInvokableClass', '\Components\EventManager\CreditCard', false);
$serviceCreditCardWithInvokableClass = $serviceManager2->get('CreditCardWithInvokableClass');
$serviceCreditCardWithInvokableClass2 = $serviceManager2->get('CreditCardWithInvokableClass');
print_r($serviceCreditCardWithInvokableClass);
var_dump( $serviceCreditCardWithInvokableClass === $serviceCreditCardWithInvokableClass2);
```
Retorno:

    Components\EventManager\CreditCard Object ( [event:protected] => ) bool(false)

<br/>
<br/>

#### Factories

`Factories` são classes ou funções que podem ser chamadas e retornam uma instância configurada do serviço solicitado.
O tipo `callables` deve retornar do serviço configurado, enquanto as classes devem implementar a interface `FactoryInterface`, e deve retornar o serviço configurado a partir do método prescrito `createService()`.

Ambos `callables` e classes podem obter a instância de `service manager` que os chamou passado como um parâmetro para o método chamado.
Isso permite que a `factory` possa cumprir quaisquer dependências do serviço, recuperando essas dependências pelo chamado do `service manager`, ou o `shared service manager` (através do método `getServiceLocator()` ).

As `factories` estão definidas na `key` : `factories` do array de configuração

ex1
```PHP
public function getServiceConfig()
{
   return [
      'factories' => [
         'Base\Service\ProcessJson' => function($sm) {
            $serializer = $sm->get('jms_serializer.serializer');
            return new Service\ProcessJson(null, null, $serializer);
         },
      ]
   ];
}
```

ex2
```PHP
$config = new Config([
     'factories' => [
          'UserService' => 'User\Factory\UserServiceFactory',
          // class implements the FactoryInterface

          'user' => function ( ServiceLocatorInterface $serviceManager ) {
               $service = $serviceManager->get('UserService');
               // defined above
               $user = new User($service); // constructor dependency injection
               return $user;
          }
     ],
]);
$serviceManager = new ServiceManager( $config );
```
<br/>

Vamos criar uma `factory` diretamente no arquivo `index.php`, usamos o método `setFactory` passando
o nome do serviço e logo em seguida uma classe ou um `callback`, vamos usar em nosso exemplo um `callback`,
dentro retornamos o serviço recem criado `CreditCardWithInvokableClass` a partir da instância de `ServiceManager`
que obtemos na variável `$sm`, muito útil para criarmos `containers` para resolver dependências, poderiamos criar um serviço
que automaticamente quando fosse chamado já resolveria alguma dependências para evitar a duplicação de código

```PHP
$serviceManager2->setFactory('CreditCardFactory', function($sm) {
    return $sm->get('CreditCardWithInvokableClass');
});
$creditCardFactory = $serviceManager2->get('CreditCardFactory');
var_dump($creditCardFactory);
```
Retorno:

    object(Components\EventManager\CreditCard)#41 (1) { ["event":protected]=> NULL }

<br/>

Continuando vamos dar outro exemplo de como trabalhar com `factories`, criaremos uma classe para armazenar nossa `factory`,
implementando `FactoryInterface`, o nome do arquivo será `CreditCardParametersFactory.php` dentro de `ServiceManager/Factories/`,
implemente o método `createService` que como o nome diz cria um serviço e nos retorna um `ServiceLocator` interface que possui o
método `get` que usamos muito, vamos criar outro método na classe `CreditCard` para depois terminar de implementar o método `createService`

`CreditCardParametersFactory.php`
```PHP
namespace Components\ServiceManager\Factories;

use Zend\ServiceManager\FactoryInterface;
use Zend\ServiceManager\ServiceLocatorInterface;

class CreditCardParametersFactory implements FactoryInterface
{
    /**
     * @param ServiceLocatorInterface $serviceLocator
     * @return mixed
     */
    public function createService(ServiceLocatorInterface $serviceLocator)
    {

    }
}
```

<br/>

Agora no arquivo `CreditCard.php` vamos criar o seguinte método no final da classe,
onde ele vai receber um array de parâmetros e somente exportar a informação.

`CreditCard.php`
```PHP
/**
 * @param array $array
 * @return mixed
 */
public function exportParameters($array = array())
{
    return var_export($array);
}
```
<br/>

Voltando para a classe `CreditCardParametersFactory` vamos finalizar o método `createService`,
como temos o `ServiceLocator` vamos resgatar o último serviço que criamos na `index.php`, `CreditCardWithInvokableClass`
e passamos um array com vários parâmetros para o método `exportParameters` que acabamos de criar na classe `CreditCard`
ficando da seguinte maneira:

`CreditCardParametersFactory.php`
```PHP
namespace Components\ServiceManager\Factories;

use Zend\ServiceManager\FactoryInterface;
use Zend\ServiceManager\ServiceLocatorInterface;

class CreditCardParametersFactory implements FactoryInterface
{
    /**
     * @param ServiceLocatorInterface $serviceLocator
     * @return mixed
     */
    public function createService(ServiceLocatorInterface $serviceLocator)
    {
        $card = $serviceLocator->get('CreditCardWithInvokableClass');
        $card->exportParameters(array(
            'firstName' => 'Bobby',
            'lastName' => 'Tables',
            'number' => '4444333322221111',
            'cvv' => '123',
            'expiryMonth' => '12',
            'expiryYear' => '2017',
            'email' => 'testcard@gmail.com'
        ));
        return $card;
    }
}
```

<br/>

Na `index.php` Criamos outro serviço chamado de `CreditCardAbstractFactory` e ao invés de
passamos um `callback`, vamos passar a nossa `factory` e depois somente chamar o serviço:

`index.php`
```PHP
$serviceManager2->setFactory('CreditCardAbstractFactory', 'Components\ServiceManager\Factories\CreditCardParametersFactory');
$serviceManager2->get('CreditCardAbstractFactory');
```
Retorno:

        array ( 'firstName' => 'Bobby', 'lastName' => 'Tables', 'number' => '4444333322221111', 'cvv' => '123', 'expiryMonth' => '12', 'expiryYear' => '2017', 'email' => 'testcard@gmail.com', )

<br/>
<br/>

#### Abstract Factories

`Abstract Factories` são `factories` que são capazes de múltiplos serviços.
`Abstract Factories` implementam o `AbstractFactoryInterface` e são consultados depois de `factories` e `invokables`,
se não houver nenhuma entrada para este serviço como uma `factory` ou `invokable`, `Abstract Factories` são registradas
perguntado se elas podem criar este serviço através do método `canCreateServiceWithName()`, se este método retorna `true`,
em seguida, o serviço é solicitado através do método `createServiceWithName()`.

Como o padrão de `factory`, o `service manager` que está sendo utilizado para solicitar este serviço é passado como um parâmetro para que as dependências do serviço possam ser cumpridas.
`Abstract Factories` são registradas usando a chave `abstract_factories` no array de configuração.

```PHP
class AbstractTableGatewayFactory implements AbstractFactoryInterface {
     public function createServiceWithName( ServiceLocatorInterface $serviceLocator, $name, $requestedName ) {
          // does this classname end with Table?
          return fnmatch ( '*Table', $requestedName );
     }

     public function canCreateServiceWithName ( ServiceLocatorInterface $serviceLocator, $name, $requestedName ) {
          $adapter = $serviceLocator->get('Zend\Db\Adapter\Adapter');
          return new TableGateway ( str_replace ( 'Table', '', $requestedName ), $adapter );
     }
}

$config = new Config ( [
     'abstract_factories' => [
         'AbstractTableGatewayFactory'
     ],
] );
$serviceManager = new ServiceManager ( $config );
$userTable = $serviceManager->get('userTable');
```

<br/>

Criaremos nossa classe para servir como uma `abstract factory`, nome do arquivo: `AbstractCreditCardFactory.php` no diretório:
`Components/ServiceManager/Factories/` e nome de classe: `AbstractCreditCardFactory`, implemeta `AbstractFactoryInterface` e
criamos os métodos que precisamos implementar `canCreateServiceWithName` que quando retorna `true` é criado o serviço com o
método `createServiceWithName`, em nosso exemplo somente verificamos se é suportado a bandeira que é passada junto com o
serviço concatenado, vamos analisar isso no `index.php`

`AbstractCreditCardFactory.php`
```PHP
namespace Components\ServiceManager\Factories;

use Components\EventManager\CreditCard;
use Zend\ServiceManager\ServiceLocatorInterface;
use Zend\ServiceManager\AbstractFactoryInterface;

class AbstractCreditCardFactory implements AbstractFactoryInterface
{
    public function canCreateServiceWithName(ServiceLocatorInterface $serviceLocator, $name, $requestedName)
    {
        $brand = explode('-', $requestedName);
        $reflaction = new \ReflectionClass('\Components\EventManager\CreditCard');

        if ( array_key_exists($brand[1], $reflaction->getConstants()) )
            return true;
    }

    /**
     * @param ServiceLocatorInterface $serviceLocator
     * @param $name
     * @param $requestedName
     * @return CreditCard
     */
    public function createServiceWithName(ServiceLocatorInterface $serviceLocator, $name, $requestedName)
    {
        return new CreditCard();
    }
}
```

<br/>

Como não estamos trabalhando ainda aqui nos exemplos com o `ServiceConfig` vamos usar o método `addAbstractFactory` para adicionar
a `abstract factory`, quando for chamado o serviço via `get` será identificado que não existe serviço criado com o nome passado, em seguida
é consultado a existência de uma `abstract factory`, como temos uma registrada é criado um serviço, retornando assim uma instância de `CreditCard`
onde já temos acesso a seus métodos.

`index.php`
```php
$serviceManager2->addAbstractFactory('Components\ServiceManager\Factories\AbstractCreditCardFactory');
$abstractCreditCardFactory = $serviceManager2->get('BrandSupported-BRAND_VISA');
$abstractCreditCardFactory->getCreditCardFlag(4444333322221111);
```

<br/>
<br/>

#### Aliases

`Aliases` podem ser usados para indicar um determinado nome de serviço para outro serviço que já tenha sido definido por um outro método.
`Aliases` podem ser registrados usando a `key` `aliases` do array de configuração.

ex
```PHP
$config = new Config ( [
     'aliases' => [
          'user' => 'UserService'
     ];
] );
$serviceManager = new ServiceManager( $config );
```
<br/>

Poderiamos trabalhar também com o método `setAlias` o retorno seria o mesmo de chamar o serviço como ele foi estabelecido.

```PHP
$serviceManager2->setFactory('CreditCardAbstractFactory2', 'Components\ServiceManager\Factories\CreditCardParametersFactory');
$serviceManager2->setAlias('FactoryWithAlias', 'CreditCardAbstractFactory2');
$serviceManager2->get('FactoryWithAlias');
```
Retorno:

    array ( 'firstName' => 'Bobby', 'lastName' => 'Tables', 'number' => '4444333322221111', 'cvv' => '123', 'expiryMonth' => '12', 'expiryYear' => '2017', 'email' => 'testcard@gmail.com', )

<br/>
<br/>

#### Shared Service Manager

`Zend\ServiceManager\ServiceManager` é uma implementação do `ServiceLocatorInterface` usado para armazenar os serviços personalizados, bem como internamente para armazenar serviços utilizados através do framework, é tipicamente escrito pelo desenvolvedor do software.

O `Shared Service Manager` está disponível a partir de qualquer um dos `Plugins Managers` especializados através do método `getServiceLocator()` do `AbstractPluginManager`.

No exemplo que trabalhamos chamando dois serviços iguais e comparando-os para checar seu valor e tipo sempre era retornado `true` a não ser
quando passamos o terceiro parâmetro para o `setInvokableClass` que seria o valor boolean `false`, por padrão esse valor é `true`, ou seja todos
os serviços são compartilhados, podemos usar também o método `setShared` passando o serviço e o valor boolean `false`, e a cada chamada para um serviço ele retornará uma nova instância.

```PHP
$serviceManager->setShared('Servico', false);
```

<br/>
<br/>

#### Peering Services

`Peering Services` é um recurso que serve para relacionar vários `services managers`, ou sejá através do método
`createScopedServiceManager()` é possivel criar um `service manager` a partir de uma instância de outro `service manager`,
uma herança é criada, existem duas constantes que definem isso: `SCOPE_PARENT` e `SCOPE_CHILD`, por default é criado como `SCOPE_PARENT`
ou seja o pai, em nosso arquivo `index.php` criamos um `service manager` chamado de `$sm1` e registrado um serviço qualquer.
`index.php`
```PHP
$sm1 = new \Zend\ServiceManager\ServiceManager();
$sm1->setInvokableClass('CreditCardTest1', '\Components\EventManager\CreditCard');
```
<br/>
Registramos outro `service manager`, agora chamado de `$sm2` com o escopo `SCOPE_PARENT`, o que acontece aqui agora é que
`$sm2` herda os serviços de `$sm1`

```PHP
$sm2 = $sm1->createScopedServiceManager(\Zend\ServiceManager\ServiceManager::SCOPE_PARENT);
print_r($sm2->get('CreditCardTest1'));
```
<br/>
Agora vamos criar por último um outro `service manager`, o `$sm3` com escopo `SCOPE_CHILD`, registramos mais um serviço,
pode ser analisado que nos é retornado um erro quando tentamos chamar o serviço criado por `$sm1`, não obtendo o mesmo efeito que no `$sm2`
que herda de `$sm1`, mas qual seria a finalidade então do escopo `SCOPE_CHILD` ?
```PHP
$sm3 = $sm1->createScopedServiceManager(\Zend\ServiceManager\ServiceManager::SCOPE_CHILD);
$sm3->setInvokableClass('CreditCardTest3', '\Components\EventManager\CreditCard');
```
<br/>
Para entendermos isso temos que acessar o serviço criado por `$sm3` usando `$sm1` e `$sm2`, resumindo
`$sm3` não possui acesso aos serviços criados com `$sm2` e `$sm1` pois se trata de um escopo `SCOPE_CHILD`,
mas como `$sm2` possui um escopo `SCOPE_PARENT` que seria o pai, ele possui acesso aos serviços criados por `$sm3`
que foi criado a partir de `$sm1` e possui acesso a `$sm1` que é de quem ele herda, `$sm1` também possui acesso aos serviços
de `$sm3` pois se `$sm2` possui acesso e foi criado a partir de `$sm1`, `$sm1` consequentemente vai ter acesso também,
ficando da seguinte maneira o final do arquivo:

`index.php`
```PHP
$sm1 = new \Zend\ServiceManager\ServiceManager();
$sm1->setInvokableClass('CreditCardTest1', '\Components\EventManager\CreditCard');

$sm2 = $sm1->createScopedServiceManager(\Zend\ServiceManager\ServiceManager::SCOPE_PARENT);
print_r($sm2->get('CreditCardTest1'));

$sm3 = $sm1->createScopedServiceManager(\Zend\ServiceManager\ServiceManager::SCOPE_CHILD);
$sm3->setInvokableClass('CreditCardTest3', '\Components\EventManager\CreditCard');
print_r($sm1->get('CreditCardTest3'));
print_r($sm2->get('CreditCardTest3'));
```

<br/>
<br/>

#### Initializers

`Initializers` são usados para executar tarefas adicionais de inicialização depois que um serviço foi criado.
O serviço recém-criado é transmitido para todos os `initializers` registrados após a criação.

Isto é útil para satisfazer as dependências usando a injeção de `setter`.

`Initializers` também são utilizados para verificar a instância da interface ( ambos definidos e enviados com a `factory` do usuário) para que essas dependências possam ser satisfeitas.

`Initializers` podem ser registrados usando a chave `initializers` no array de configuração, e pode ser qualquer classe que possa ser chamada ou que implementa o `InitializersInterface`

```PHP
interface UserServiceAwareInterface
{
     public function setUserService ( UserService $userService );
}

class ProductService implements UserServiceAwareInterface
{
     protected $userService;
     public function setUserService ( UserService $userService )
     {
          $this->userService = $userService;
     }
}

$config = new Config ( [
     'initializers' => [
          'userService' => function ( $service, ServiceLocatorInterface $serviceManager ) {
               if ( $service instanceof UserServiceAwareInterface ) {
                    $userService = $serviceManager->get( ‘UserService’ );
                    $service->setUserService ( $userService );
               }
          }
     ],
] );
$serviceManager = new ServiceManager ( $config );
```
<br/>

Vamos dar um exemplo de `initializer` em nosso projeto, criaremos uma propriedade `$timezone` em nossa classe `CreditCard` e também um `setter`
somente que o `setter` vamos estabelecer que a passagem do parâmetro seja uma instância de `DateTimeZone`:
`CreditCard.php`
```PHP
/**
 * @var
 */
protected $timezone;

/**
 * @param \DateTimeZone $timezone
 * @return $this
 */
public function setTimezone(\DateTimeZone $timezone)
{
    $this->timezone = $timezone;
    return $this;
}
```
<br/>

Na `index.php` o processo vai ser como nos outros exemplos, instância de `service manager` registrar um serviço com `setInvokableClass`
e o adicional seria o método `addInitializer` que vai ser executado quando o serviço for chamado, ou seja quando for gerado uma instância do
serviço chamado ele vai ser executado, diferentemente da `factory` que executa já gerando a instância, em nosso exemplo abaixo setamos
o timezone dentro do `initializer` que fara a injeção para a nossa propriedade `$timezone`, lembrando que as `factories` podem ser aninhadas
com os `initializer` lhe possibilitando executar diversas tarefas.

`index.php`
```PHP
$serviceManager3 = new \Zend\ServiceManager\ServiceManager();
$serviceManager3->setInvokableClass('InitializerCreditCard', '\Components\EventManager\CreditCard');
$serviceManager3->addInitializer(function($instance, $serviceManager) {
    if ( $instance instanceof \Components\EventManager\CreditCard )
    {
        $instance->setTimezone(new \DateTimeZone('Europe/London'));
    }
});
$initializer = $serviceManager3->get('InitializerCreditCard');
print_r($initializer);
```

Retorno com initializer:

    Components\EventManager\CreditCard Object ( [event:protected] => [timezone:protected] => DateTimeZone Object ( [timezone_type] => 3 [timezone] => Europe/London ) )


Retorno sem initializer:

    Components\EventManager\CreditCard Object ( [event:protected] => [timezone:protected] => )


<br/>
<br/>

#### ServiceConfig

`Service managers` e ( `plugin managers` ) podem ser configurados por meio de um classe que implementa
`Zend\ServiceMager\ConfigInterface` para o `service manager` como um argumento construtor.

A classe `Zend\ServiceManager\Config` é uma implementação concreta dessa interface que é fornecida com o
framework e pode ser usada para configurar `service managers`.

Existem várias maneiras que os `services managers` podem ser configurados para criar e recuperar serviços,
incluindo `factories` e ( `abstracts factories` ), `invokables` e `aliases`.
Além disso, `initializers` podem ser registrados para ser executado automaticamente bastando configurar uma injeção de dependência.

O objeto `Config` recebe um array, e usa esse array para configurar o `service manager` com `factories`, `abstract factories`, `invokables`, `aliases` e `initializers`.

O array é no formato de `key-value`, onde as chaves são o nome do serviço exclusivo que identifica esse serviço, e o valor é o meio de criar esse serviço.

Note que o nome do serviço registrado é sempre convertido, espaços e caracteres especiais são removidos, e a `string` é convertida para minúsculas, então `user-service` é o mesmo que `User\Service`, que são ambos convertidos para `UserService`

<br/>

No arquivo `index.php` vamos demonstrar uma configuração de alguns serviços que configuramos ao longo do estudo do `service manager` somente
que agora com um array de configuração que será passado para uma classe chamada `Zend\Mvc\Service\ServiceManagerConfig` que implementa
`ConfigInterface` implementado o método `configureServiceManager` que recebe `ServiceManager` e faz todas as configurações como
`setInvokableClass`, `setFactory`, `addAbstractFactory`, `setAlias`, `setShared`, `addInitializer`, todo aquele processo que fizemos
nos exemplos anteriores somente que agora de uma maneira que é totalmente transparente, esse é o processo que fazemos no método `getServiceConfig` na classe `Module`

`CreditCard.php`
```php
$serviceManager4 = new \Zend\ServiceManager\ServiceManager();
$config = [
    'abstract_factories' => [
        'Components\ServiceManager\Factories\AbstractCreditCardFactory'
    ]
    ,
    'factories' => [
        'CreditCardFactory' => function ($sm) {
            return new \Components\EventManager\CreditCard();
        }
    ],
    'invokables' => [
        'CreditCardInvokable' => '\Components\EventManager\CreditCard'
    ],
    'shared' => [
        'CreditCardInvokable' => false
    ]
];

$serviceConfig = new Zend\Mvc\Service\ServiceManagerConfig( $config );
$serviceConfig->configureServiceManager( $serviceManager4 );

print_r( $serviceManager4->get('CreditCardFactory') );
print_r( $serviceManager4->get('CreditCardInvokable') );
print_r( $serviceManager4->get('BrandSupported-BRAND_AMEX') );
```

Retorno de ambos os `print_r()`:

    Components\EventManager\CreditCard Object
    (
    [event:protected] => Zend\EventManager\EventManager Object
    (
        [events:protected] => Array
            (
            )

        [eventClass:protected] => Zend\EventManager\Event
        [identifiers:protected] => Array
            (
                [0] => Components\EventManager\CreditCard
            )

        [sharedManager:protected] => Zend\EventManager\SharedEventManager Object
            (
                [identifiers:protected] => Array
                    (
                    )

            )

    )

    [timezone:protected] =>
    )

<br/>
<br/>

#### Delegator

Incluido em Zend Framework 2.2.0.

`Delegator` é um recurso que nos permite substituir um serviço real, com um `delegator`, ou interagir com um objeto
produzido por uma `factory` antes de ser devolvidor por `Zend\ServiceManager`.

`Delegator` implementa a interface `DelegatorFactoryInterface`

O design pattern `delegator` é útil em casos em que você deseja quebrar um serviço real em um `delegator`,
ou geralmente interceptar ações que estão sendo executadas no `delegator` em um AOP ( Aspect-oriented programming ).

Vamos exemplificar um uso de `delegator` em nosso projeto, vamos criar um método `brand` em nossa classe `CreditCard` somente retornando um texto.

`CreditCard.php`
```php
/**
 * @return string
 */
public function brand()
{
    return self::BRAND_DINERS_CLUB;
}
```
<br/>

Criaremos um diretório `Components/ServiceManager/Delegator` e um arquivo chamado de `CreditCardDelegator.php`
essa será sua estrutura:

`CreditCardDelegator.php`
```php
use Zend\EventManager\EventManagerInterface;
use Components\EventManager\CreditCard;

class CreditCardDelegator extends CreditCard
{
    /**
     * @var
     */
    protected $realBrand;

    /**
     * @var
     */
    protected $eventManager;

    /**
     * CreditCardDelegator constructor.
     * @param CreditCard $realBrand
     * @param EventManagerInterface $eventManager
     */
    public function __construct(CreditCard $realBrand, EventManagerInterface $eventManager)
    {
        $this->realBrand = $realBrand;
        $this->eventManager = $eventManager;
    }

    /**
     * @return string
     */
    public function brand()
    {
        $this->eventManager->trigger('brand', $this);
        return $this->realBrand->brand();
    }
}
```

Essa é uma classe `delegator` onde estende de `CreditCard`, o construtor recebe o método original
`brand` e um `EventManager`, no método `brand` temos o retorno de nosso
método original `brand` e a `trigger` que executará antes de retornar a função.

<br/>

Agora somente implementarmos na `index.php` o `listener` e executar o método `brand`:

`index.php`

```php
$wrapperCreditCard = new Components\EventManager\CreditCard();
$eventManager = new Zend\EventManager\EventManager();
$eventManager->attach('brand', function() {
    echo "return string first !\n";
});

$delegator = new Components\ServiceManager\Delegator\CreditCardDelegator($wrapperCreditCard, $eventManager);
echo $delegator->brand();
```

Retorno:

    return string first ! diners_club

<br/>
<br/>

#### Delegator Factory

Vamos efetuar o mesmo processo somente que agora com uma classe `factory`, como mencionado `Delegator` implementa a interface `DelegatorFactoryInterface`, então criaremos um arquivo no diretório das
`factories` : `Components/ServiceManager/Factories/CreditCardDelegatorFactory.php`

`DelegatorFactoryInterface` precisa implementar o método `createDelegatorWithName` onde possui os seguintes parâmetros:

*  `$serviceLocator` Utilizado durante a criação do `delegator` para solicitar algum serviço
*  `$name` é o nome canônico do serviço que está sendo solicitado
*  `$requestedName` é o nome do serviço como inicialmente requerido para o `service locator`
*  `$callback` é um callable que é responsável de instanciar o serviço delegado (a instância do serviço real)

Estrutura do arquivo:

`CreditCardDelegatorFactory.php`
```php
namespace Components\ServiceManager\Factories;

use Zend\ServiceManager\DelegatorFactoryInterface;
use Zend\ServiceManager\ServiceLocatorInterface;
use Zend\EventManager\EventManager;
use Components\ServiceManager\Delegator\CreditCardDelegator;

class CreditCardDelegatorFactory implements DelegatorFactoryInterface
{
    /**
     * @param ServiceLocatorInterface $serviceLocator
     * @param string $name
     * @param string $requestedName
     * @param callable $callback
     * @return CreditCardDelegator
     */
    public function createDelegatorWithName(ServiceLocatorInterface $serviceLocator, $name, $requestedName, $callback)
    {
        $realBrand = call_user_func($callback);
        $eventManager = new EventManager();
        $eventManager->attach('brand', function () {
            echo "return string first, now with factory! \n ";
        });
        return new CreditCardDelegator($realBrand, $eventManager);
    }
}
```
Igualmente ao outro exemplo, criamos um `listener` que será disparado quando o método `brand` for executado.

<br/>

Implementação na `index.php`

```php
$serviceManager = new Zend\ServiceManager\ServiceManager();
$serviceManager->setInvokableClass('brand', 'Components\EventManager\CreditCard');
$serviceManager->setInvokableClass('brand-delegator-factory', 'Components\ServiceManager\Factories\CreditCardDelegatorFactory');
$serviceManager->addDelegator('brand', 'brand-delegator-factory');

$delegator = $serviceManager->get('brand');
echo $delegator->brand();
```
Usamos `setInvokableClass` para registrar o serviço e para registrar a `factory`, e finalmente adicionamos o `delegator` com o método
`addDelegator()`, lembrando que também poderia ser registrado o `delegator` atavés de um array de configuração:

```php
$config = array(
    'delegators' => array(
        'invokables' => array(
            'brand'                   => 'Components\EventManager\CreditCard',
            'brand-delegator-factory' => 'Components\ServiceManager\Factories\CreditCardDelegatorFactory',
        ),
        'brand' => array(
             'brand-delegator-factory'
        ),
    ),
);
```

retorno:

    return string first, now with factory! diners_club

<br/>
<br/>

#### Lazy Services

`Zend\ServiceManager` pode usar `delegator factory` para gerar referências (`lazy`) "preguiçosos" para seus serviços.

`Lazy Services` são proxies que são instanciados lentamente, e mantém uma referência à instância real do serviço de proxy.

Você pode querer inicializar lentamente um serviço quando ele é instanciado muito frequentemente, mas nem sempre usado.
Um exemplo típico é a conexão com o banco: é uma dependência de muitos outros elementos no seu aplicativo, mas isso não significa
que cada pedido vai executar consultas através dele.

Além disso, instanciar uma conexão com o banco de dados pode exigir algum tempo e consumir muitos recursos.

Usar um Proxy na conexão de banco de dados permitiria atrasar a sobrecarga até que o objeto seja realmente necessário.

`Zend\ServiceManager\Proxy\LazyServiceFactory` é um `delegator factory` capaz de gerar proxies lentamente no carregamento de seus serviços.

O `LazyServiceFactory` depende de `ProxyManager`, por isso vamos instalá-lo antes, somente adicionar a seguinte linha no arquivo `composer.json`
e executar `php composer.phar update`
```php
"ocramius/proxy-manager" : "0.3.*"
```
<br/>

Em nosso projeto vamos criar um arquivo para demonstrar como funciona um `lazy service`, dentro do diretório `Delegator` crie um arquivo com
nome de `CreditCardDelegatorLazy.php` que é projetado para ser lento em tempo de instanciação para fins de demonstração:

```php
namespace Components\ServiceManager\Delegator;

use Components\EventManager\CreditCard;

class CreditCardDelegatorLazy extends CreditCard
{
    /**
     * CreditCardDelegatorLazy constructor.
     */
    public function __construct()
    {
        sleep(10);
    }

    /**
     * @return string
     */
    public function brand()
    {
        return parent::brand();
    }
}
```

<br/>

Não podemos esquecer de registrar o namespace do `proxy-manager` que acabamos de baixar, ficando da seguinte maneira:

`index.php`
```php
$loader->registerNamespace('Components', 'src/Components');
$loader->registerNamespace('ProxyManager', 'vendor/ocramius/proxy-manager/src/ProxyManager');
$loader->register();
```

<br/>

Ainda no arquivo `index.php` vamos chamar nosso serviço `lazy`, configuramos o `service manager`
para gerar proxies em vez de serviços reais atravéz do array de configuração, logo criamos os serviços, as `factories` e o `delegator`,
finalizando, somente executar o método `brand` que terá um delay de 10 segundos para retornar a mensagem.

`index.php`
```php
$config = [
    'lazy_services' => [
        'class_map' => [
            'CreditCardDelegatorLazy' => 'Components\ServiceManager\Delegator\CreditCardDelegatorLazy'
        ],
    ],
];
$serviceManagerLazy = new Zend\ServiceManager\ServiceManager();
$serviceManagerLazy->setService('Config', $config);
$serviceManagerLazy->setInvokableClass('CreditCardDelegatorLazy', 'Components\ServiceManager\Delegator\CreditCardDelegatorLazy');
$serviceManagerLazy->setFactory('creditcard-delegator-factory-lazy', 'Zend\ServiceManager\Proxy\LazyServiceFactoryFactory');
$serviceManagerLazy->addDelegator('CreditCardDelegatorLazy', 'creditcard-delegator-factory-lazy');
$creditCardDelegatorLazy = $serviceManagerLazy->get('CreditCardDelegatorLazy');
echo $creditCardDelegatorLazy->brand();
```

Retorno, com delay de 10 segundos:

    diners_club

<br/>
<br/>

##### Referências

 - [Zend\ServiceManager Quick Start](http://framework.zend.com/manual/current/en/modules/zend.service-manager.quick-start.html)
 - [Delegator Factories in Zend Framework 2](http://ocramius.github.io/blog/zend-framework-2-delegator-factories-explained/)
 - [Getting Closer with PluginManager](https://samsonasik.wordpress.com/2014/01/29/zend-framework-2-getting-closer-with-pluginmanager/)
 - [ZF2 Certification Study Guide](http://www.zend.com/en/services/certification/zf2-certification-study-guide)

----------------
O Código usado nesse projeto está no GitHub ==> <a href="https://github.com/luk4z7/zf2-componentization" target="blank">luk4z7</a>

-----------------

<br/>
<br/>
<br/>
<br/>