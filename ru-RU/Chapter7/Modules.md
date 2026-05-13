# Глава 7. Модули (Modules)

Когда вы размещаете некоторые классы вместе в Module, вы тем самым говорите следующему разработчику, который будет смотреть на ваш дизайн, что о них следует думать вместе. Если ваша модель рассказывает историю, то Modules — это главы.

*Domain-Driven Design: Tackling Complexity in the Heart of Software*
— Eric Evans

Распространённый вопрос при построении Application в соответствии с Domain-Driven Design — где размещать код. Особенно если вы используете PHP framework, важно понимать рекомендуемый способ размещения кода, где размещать Infrastructure code и как должны быть структурированы различные концепции внутри модели.

В Domain-Driven Design для этого существует tactical pattern: modules. В наши дни все структурируют код с помощью модулей. Во всех языках есть какой-либо инструмент для группировки классов и языковых определений. В Java есть packages. В Ruby есть modules. В PHP есть namespaces.

Domain-Driven Design делает ещё один шаг вперёд в сторону packaging и grouping ваших классов и придаёт этим building blocks семантический смысл. Более того, modules рассматриваются как часть модели. Поскольку они являются частью модели, важно подобрать наилучшие имена, сгруппировать вместе Domain objects, тесно связанные друг с другом, и сохранить несвязанные Domain objects decoupled. Modules не следует рассматривать как способ разделения кода; их следует рассматривать как способ разделения значимых концепций модели.

## Общий обзор

Как объяснялось в Главе 1, *Getting Started with Domain-Driven Design*, наш Domain внутренне организован в Subdomains. Каждый Subdomain в идеале моделируется и реализуется одним Bounded Context, хотя иногда требуется больше одного. При хорошем дизайне каждый Bounded Context представляет собой независимую систему, которая будет разрабатываться и поддерживаться отдельной командой. Мы рекомендуем реализовывать каждый Bounded Context как полноценное Application. Это означает, что два Bounded Context не будут находиться в одном и том же code Repository. Благодаря этому их можно развёртывать независимо, они могут иметь разные циклы разработки или даже быть реализованы на разных языках. Внутри ваших Bounded Contexts вы будете использовать modules для группировки Domain objects, имеющих сильную связь друг с другом.

## Использование Modules в PHP

До PHP 5.3 modules не поддерживались полностью. Однако с появлением PHP 5.3 мы можем использовать PHP namespaces для реализации module pattern. По историческим причинам мы покажем, как namespaces использовались до PHP 5.3, но вам следует стремиться использовать версию PHP, поддерживающую PHP namespaces. Лучшим выбором всегда будет последняя стабильная версия PHP.

## Пространства имён первого уровня

Распространённый подход — использовать namespace первого уровня, идентифицирующий вашу компанию. Это поможет избежать конфликтов с third-party libraries. Если вы используете PSR-0, для namespace будет существовать реальная папка; если вы используете PSR-4, она не требуется. Мы подробнее рассмотрим это чуть позже. Но сначала давайте посмотрим на conventions namespacing в PHP.

## Namespacing в стиле PEAR

До PHP 5.3 из-за отсутствия конструкции namespace использовались namespaces в стиле PEAR. PEAR — это акроним от PHP Extension and Application Repository, и в старые добрые времена это был Repository reusable components. Он всё ещё существует, но уже не слишком удобен, и сейчас им пользуется не так много людей — особенно после появления Composer и Packagist. PEAR как источник reusable components нуждался в способе избежать конфликтов имён классов, поэтому контрибьюторы начали добавлять prefixes к именам классов, имитируя namespaces. До сих пор существуют проекты, использующие такую форму namespaces (например PHPUnit и Zend Framework 1). Пример namespaces в стиле PEAR:

Следующий пример представляет namespaces в стиле PEAR:

![Cool](https://github.com/eatae/dddinphp/blob/main/share/image--008.jpg)

Имя класса для Entity Bill при использовании namespaces в стиле PEAR выглядело бы так: `BuyIt_Billing_Domain_Model_Bill_Bill`. Однако это выглядит довольно уродливо и не соответствует одной из главных мантр Domain-Driven Design: каждое имя класса должно быть названо в терминах Ubiquitous Language. По этой причине мы настоятельно не рекомендуем использовать такой подход.

## Namespacing PSR-0 и PSR-4

Namespaces появились вместе с выходом PHP 5.3 наряду с другими важными возможностями. Это было серьёзное изменение, и группа разработчиков наиболее важных frameworks сформировала PHP-FIG — акроним от PHP Framework Interop Group — в попытке стандартизировать и унифицировать общие аспекты создания frameworks и libraries. Первой PHP Standard Recommendation (PSR), выпущенной этой группой, стал стандарт autoloading, который, если кратко, предлагал отношение один-к-одному между классом и PHP-файлом с использованием namespaces. Сегодня PSR-4 — упрощённая версия PSR-0, всё ещё сохраняющая связь между классами и физическими PHP-файлами, — является предпочтительным и рекомендуемым способом структурирования кода. Мы считаем, что именно его следует использовать для реализации modules в проекте.

Возвращаясь к той же структуре папок, показанной в предыдущем разделе, давайте посмотрим, что изменяется при использовании PSR-0. Имя класса для Entity Bill, при использовании namespaces и PSR-0, станет просто `Bill`, а fully qualified class name будет выглядеть как `BuyIt\Billing\Domain\Model\Bill\Bill`.

Как вы можете видеть, это позволяет нам именовать Domain objects в терминах Ubiquitous Language, и именно такой способ структурирования и организации кода является предпочтительным. Если вы используете Composer, а вам следует его использовать, то вам нужно настроить некоторые параметры autoloading в вашем файле composer.json:

```json
...
"autoload": {
    "psr-0": {
        "BuyIt\\": "src/BuyIt/"
    }
},
"autoload-dev": {
    "psr-0": {
        "BuyIt": "tests/BuyIt/"
    }
},
...
```

Если вы всё ещё не используете PSR-4 или ещё не мигрировали с PSR-0, мы настоятельно рекомендуем это сделать. Вы сможете избавиться от папки namespace первого уровня, и структура вашего кода будет лучше соответствовать Ubiquitous Language:

![Cool](https://github.com/eatae/dddinphp/blob/main/share/image--010.jpg)

Однако для предотвращения конфликтов с third-party libraries всё ещё рекомендуется добавлять namespace первого уровня в ваш файл composer.json:

```json
...
"autoload": {
    "psr-4": {
        "BuyIt\\": "src/"
    }
},
"autoload-dev": {
    "psr-4": {
        "BuyIt\\": "tests/"
    }
},
...
```

Если вы предпочитаете иметь namespace первого уровня, но при этом использовать PSR-4, потребуется внести некоторые небольшие изменения:

![Cool](https://github.com/eatae/dddinphp/blob/main/share/image--012.jpg)

```json
...
"autoload": {
  "psr-4": {
    "BuyIt\\": "src/BuyIt/"
  }
},
"autoload-dev": {
  "psr-4": {
    "BuyIt\\": "tests/BuyIt/"
  }
}, ...
```

Как вы, возможно, заметили в примерах, мы разделили папки src и tests. Это было сделано для оптимизации файла autoloading, генерируемого Composer, а также для уменьшения объёма памяти, необходимого для хранения classmap. Кроме того, это поможет вам настраивать параметры whitelisting и blacklisting при генерации отчётов code coverage для unit testing. Если вы хотите узнать больше о конфигурации autoloading в Composer, ознакомьтесь с документацией.

А как насчёт PHAR-файлов?

Они тоже могут использоваться, однако мы не рекомендуем этого делать. В качестве упражнения составьте список преимуществ и недостатков использования PHAR-файлов для моделирования modules.

Ограниченные Контексты и Applications

Если взять пример вымышленной компании под названием BuyIt, работающей в e-commerce Domain, то может иметь смысл создать отдельное application для каждого из различных Ограниченных Контекстов, решающих конкретные области Domain.

Если некоторые из различных Ограниченных Контекстов — это Order Management, Payment Management, Catalog Management и Inventory Management, мы рекомендуем иметь отдельное application для каждого из них:

![Cool](https://github.com/eatae/dddinphp/blob/main/share/image--014.jpg)

Каждое application предоставляет любой набор delivery mechanisms, который необходим. С распространением тренда microservices всё больше людей строят Bounded Contexts, которые в итоге предоставляют REST APIs внешнему миру. Однако Bounded Context — это нечто большее, чем просто API. Помните, что API — это лишь один из множества delivery mechanisms; Bounded Context также может предоставлять web interface для взаимодействия.

## Могут ли два Bounded Contexts находиться в одном Application? И что насчёт обратного варианта?

Лучший вариант — один Subdomain, один Bounded Context и одно application. Если один Bounded Context реализован двумя applications, поддержка и deployment становятся немного сложнее. А в случае application, реализующего два Bounded Contexts, процесс deployment, время выполнения тестов и проблемы при merge могут замедлить разработку.

Обратите внимание, что каждое имя Bounded Context представляет значимую концепцию в нашем e-commerce Domain и названо в терминах Ubiquitous Language:

* `Catalog` — хранит весь код, связанный с описаниями продуктов, комбинациями продуктов и так далее.
* `Inventory` — хранит весь код, связанный с управлением складскими остатками продуктов.
* `Orders` — хранит весь код, связанный с системами обработки заказов. Он будет содержать finite-state machine, отвечающую за обработку заказов.
* `Payments` — хранит весь код, связанный с платежами, счетами и waybills.

## Структурирование кода в Modules

Давайте немного глубже рассмотрим один из Bounded Contexts. Возьмём, например, контекст `Orders` и изучим детали структуры. Как следует из названия, этот Bounded Context отвечает за представление всех процессов, через которые проходит заказ — от его создания до доставки клиенту, который его приобрёл. Более того, это независимое Application, поэтому оно содержит папку исходного кода (`source code folder`) и папку тестов (`tests folder`). Папка исходного кода содержит весь код, необходимый для работы данного Bounded Context: Domain code, Infrastructure code и Application layer.

Следующая диаграмма должна проиллюстрировать организацию:

![Cool](https://github.com/eatae/dddinphp/blob/main/share/image--016.jpg)

Весь код имеет vendor namespace, названный в терминах имени организации (в данном случае — `BuyIt`), и содержит две подпапки: `Domain`, в которой находится весь Domain code, и `Infrastructure`, в которой находится Infrastructure layer, тем самым изолируя всю Domain logic от деталей Infrastructure layer. Следуя такой структуре, мы явно показываем, что собираемся использовать Hexagonal Architecture в качестве базовой архитектуры. Ниже приведён пример альтернативной структуры, которую также можно использовать:

![Cool](https://github.com/eatae/dddinphp/blob/main/share/image--018.jpg)

Приведённый выше стиль структуры использует дополнительную подпапку для хранения Services, определённых внутри Domain Model. Хотя такая организация может иметь смысл, мы предпочитаем её не использовать, поскольку такой способ разделения кода обычно больше сфокусирован на архитектурных элементах, чем на значимых концепциях модели. Мы считаем, что такой стиль легко может привести к появлению своего рода Service layer поверх Domain Model, что само по себе не обязательно плохо. Помните, что Domain Services используются для описания операций в Domain, которые не принадлежат ни Entities, ни Value Objects. Поэтому далее мы будем придерживаться предыдущей организации кода.

Возможно размещать код непосредственно внутри подпапки `Domain/Model`. Например, может быть обычной практикой помещать туда общие interfaces и Services, такие как `DomainEventPublisher` или `DomainEventSubscriber`.

Если бы нам нужно было моделировать контекст `Order Management`, у нас, вероятно, была бы Entity `Order` вместе с её Repository и всей информацией о состоянии. Поэтому нашей первой попыткой было бы разместить все эти элементы непосредственно внутри подпапки `Domain/Model`. На первый взгляд это может показаться самым простым способом:

![Cool](https://github.com/eatae/dddinphp/blob/main/share/image--020.jpg)

## Рекомендации по дизайну

Рассмотрим несколько базовых правил и типичных проблем, на которые следует обращать внимание при реализации modules:

* Namespaces должны быть названы в терминах Ubiquitous Language.
* Не называйте namespaces на основе patterns или building blocks (`Value Objects`, `Services`, `Entities` и так далее).
* Создавайте namespaces таким образом, чтобы то, что находится внутри, было как можно слабее связано с другими namespaces.
* Рефакторите namespaces так же, как и ваш код. Перемещайте их, переименовывайте, группируйте, выделяйте и так далее.
* Не используйте коммерческие названия продуктов, поскольку они могут измениться. Придерживайтесь Ubiquitous Language.

Мы поместили Entities `Order` и `OrderLine`, Events `OrderLineWasAdded` и `OrderWasCreated`, а также `OrderRepository` в одну и ту же подпапку `Domain/Model`. Такая структура может быть нормальной, но только потому, что у нас пока ещё простая модель. Но что насчёт Entity `Bill` и её Repository? Или Entity `Waybill` и соответствующего Repository? Давайте добавим все эти элементы и посмотрим, как они впишутся в реальную структуру кода:

![Cool](https://github.com/eatae/dddinphp/blob/main/share/image--022.jpg)

Хотя такой стиль организации кода может быть приемлемым, в долгосрочной перспективе он может стать непрактичным и довольно сложным в сопровождении. Каждый раз, когда мы будем выполнять итерации и добавлять новые возможности, модель будет становиться ещё больше, а подпапка будет поглощать всё больше кода. Нам нужно разделить код таким образом, чтобы мы могли получить представление о модели с одного взгляда. Никаких технических соображений — только concerns Domain.

Чтобы добиться этого, мы можем разделить модель с использованием Ubiquitous Language, находя значимые концепции, которые помогут нам логически группировать элементы в терминах Domain.

Для этого мы можем попробовать следующий подход:

![Cool](https://github.com/eatae/dddinphp/blob/main/share/image--024.jpg)

Таким образом код становится более организованным с концептуальной точки зрения. И, как указывает Eric Evans в *Blue Book*, modules — это способ коммуникации, поскольку они дают нам представление о том, как Domain Model работает внутренне, а также помогают повысить cohesion и снизить coupling между концепциями. Если посмотреть на предыдущий пример, можно увидеть, что концепции `Order` и `OrderLine` тесно связаны, поэтому они находятся в одном module. С другой стороны, `Order` и `Waybill`, хотя и разделяют один и тот же context, являются разными концепциями, поэтому они находятся в разных modules. Modules — это не просто способ группировать связанные концепции модели, но также способ выразить часть дизайна модели.

## Следует ли помещать Repositories, Factories, Domain Events и Services в отдельные подпапки?

Технически их действительно можно разместить в отдельных подпапках, однако делать это настоятельно не рекомендуется. В таком случае мы смешаем технические concerns и concerns Domain — помните, что главная задача module заключается в группировке связанных концепций Domain model и их decoupling от несвязанных концепций. Modules не разделяют код как таковой, а разделяют значимые концепции.

## Modules в Infrastructure Layer

До этого момента мы обсуждали, как структурировать и организовывать код в Domain layer, но почти ничего не сказали об Infrastructure layer. А поскольку мы используем Hexagonal Architecture для инверсии зависимости между Domain layer и Infrastructure layer, нам потребуется место, куда можно поместить все реализации interfaces, определённых в Domain layer. Возвращаясь к примеру billing context, нам нужно место для реализаций `BillRepository`, `OrderRepository` и `WaybillRepository`.

Очевидно, что их следует размещать в папке `Infrastructure`, но где именно? Предположим, что мы решили использовать Doctrine ORM для реализации persistence layer. Как нам разместить Doctrine-реализации наших Repositories внутри папки `Infrastructure`? Давайте сделаем это напрямую и посмотрим, как это будет выглядеть:

![Cool](https://github.com/eatae/dddinphp/blob/main/share/image--026.jpg)

Мы могли бы оставить всё как есть, однако, как мы уже видели на примере Domain layer, такая структура и организация быстро деградирует и через несколько итераций модели превратится в беспорядок. Каждый раз, когда модель будет расти, ей, вероятно, потребуется ещё больше Infrastructure, и в итоге мы начнём смешивать различные технические concerns: persistence, messaging, logging и многое другое. Наша первая попытка избежать запутанного хаоса Infrastructure-реализаций — определить отдельный module для каждой технической concern внутри Bounded Context:

![Cool](https://github.com/eatae/dddinphp/blob/main/share/image--028.jpg)

Это выглядит значительно лучше и гораздо более пригодно для долгосрочной поддержки, чем наша первая попытка. Однако нашим namespaces всё ещё не хватает некоторой связи с Ubiquitous Language. Давайте рассмотрим другой вариант:

![Cool](https://github.com/eatae/dddinphp/blob/main/share/image--030.jpg)

Намного лучше. Это соответствует организации нашей Domain Model, но уже внутри Infrastructure layer — к тому же всё выглядит более простым для поиска. Если вы заранее знаете, что у вас всегда будет только один persistence mechanism, вы можете придерживаться именно такой структуры и организации. Она достаточно проста и удобна в сопровождении.

Но что делать, если вам приходится работать с несколькими persistence mechanisms? В наши дни довольно распространено иметь relational persistence mechanism и какую-либо shared in-memory persistence, такую как Redis или Riak, либо иметь локальную in-memory implementation для возможности тестирования кода. Давайте посмотрим, как это вписывается в текущий подход:

![Cool](https://github.com/eatae/dddinphp/blob/main/share/image--032.jpg)

Мы рекомендуем именно такой подход. Однако все реализации Repository находятся внутри одного и того же module. Это может показаться немного странным при наличии большого количества различных технологий. Если вам это кажется полезным, вы можете создать дополнительный module, чтобы сгруппировать связанные реализации по используемой базовой технологии:

![Cool](https://github.com/eatae/dddinphp/blob/main/share/image--034.jpg)

Этот подход похож на организацию unit testing. Однако существуют классы, конфигурации, шаблоны и прочие элементы, которые невозможно соотнести с Domain Model. Именно поэтому внутри Infrastructure могут появляться дополнительные modules, связанные с конкретными технологиями.

Где следует размещать Doctrine mapping files или Twig templates?

![Cool](https://github.com/eatae/dddinphp/blob/main/share/image--036.jpg)

Как вы можете видеть, чтобы Doctrine работал, нам необходим `EntityManagerFactory` и все mapping files. Мы также можем включать любые другие Infrastructure objects, необходимые в качестве базовых классов. Поскольку они не связаны напрямую с нашей Domain Model, такие ресурсы лучше размещать в отдельном module. То же самое относится и к Delivery Mechanisms (`API`, `Web`, `Console Commands` и так далее). Более того, вы можете использовать разные PHP frameworks или libraries для каждого delivery mechanism:

![Cool](https://github.com/eatae/dddinphp/blob/main/share/image--038.jpg)

В предыдущем примере мы использовали Laravel Framework для обслуживания API, Symfony Console Component как точку входа для командной строки, а Silex и Slim — для web delivery mechanism. Что касается пользовательского интерфейса (User Interface), его следует размещать внутри каждого delivery mechanism. Однако если существует возможность переиспользовать UI между различными delivery mechanisms, можно создать модуль с именем `UI` на одном уровне с `Persistence` или `Delivery`. В целом, мы рекомендуем сопротивляться тому, как фреймворки диктуют организацию вашего кода. Фреймворки должны подчиняться вам, а не наоборот.

## Смешивание различных технологий

В больших business-critical приложениях довольно часто встречается смесь нескольких технологий. Например, в web-приложениях с интенсивным чтением данных обычно присутствует некоторый денормализованный источник данных (Solr, Elasticsearch, Sphinx и так далее), который обслуживает все операции чтения приложения, в то время как традиционная RDBMS, такая как MySQL или Postgres, главным образом отвечает за все операции записи. Когда возникает такая ситуация, обычно появляется вопрос: можно ли направить операции чтения в search engine, а операции записи — в традиционный RDBMS datasource. Наш общий совет здесь заключается в том, что подобные ситуации являются признаком (smell) CQRS, поскольку нам требуется независимо масштабировать чтения и записи приложения. Поэтому если у вас есть возможность использовать CQRS, скорее всего, это будет лучшим выбором.

Но если по какой-либо причине вы не можете использовать CQRS, потребуется альтернативный подход. В такой ситуации может пригодиться использование паттерна Proxy из Gang of Four. Мы можем определить реализацию Repository в терминах паттерна Proxy:

```php
namespace BuyIt\Billing\Infrastructure\FullTextSearching\Elastica;

use BuyIt\Billing\Domain\Model\Order\OrderRepository;

use BuyIt\Billing\Infrastructure\Domain\Model\Order\Doctrine\
    DoctrineOrderRepository;

class ElasticaOrderRepository implements OrderRepository
{
    private $client;
    private $baseOrderRepository;

    public function __construct(
        Client $client,
        DoctrineOrderRepository $baseOrderRepository
    ) {
        $this->client = $client;
        $this->baseOrderRepository = $baseOrderRepository;
    }

    public function find($id)
    {
        return $this->baseOrderRepository->find($id);
    }

    public function findBy(array $criteria)
    {
        $search = new \Elastica\Search($this->client);

        // ...

        return $this->toOrder($search->search());
    }

    public function add($anOrder)
    {
        // Сначала пытаемся добавить Order в Elastic index

        $ordersIndex = $this->client->getIndex('orders');

        $orderType = $ordersIndex->getType('order');

        $orderType->addDocument(
            new \ElasticaDocument(
                $anOrder->id(),
                $this->toArray($anOrder)
            )
        );

        $ordersIndex->refresh();

        // После этого пытаемся сохранить Order в RDBMS store

        $this->baseOrderRepository->add($anOrder);
    }
}
```

Этот пример предоставляет наивную реализацию с использованием DoctrineOrderRepository и Elastica client — клиента для взаимодействия с Elasticsearch server. Обратите внимание, что для некоторых операций мы используем RDBMS datasource, а для других — Elastica client. Также обратите внимание, что операция `add` состоит из двух частей. Первая пытается сохранить `Order` в Elasticsearch index, а вторая пытается сохранить `Order` в relational database, делегируя операцию Doctrine implementation. Учтите, что это лишь пример и один из возможных способов реализации. Вероятно, его можно улучшить — например, сейчас вся операция `add` выполняется синхронно. Вместо этого мы могли бы поставить операцию в очередь некоторого messaging middleware, который, например, сохранял бы `Order` в Elasticsearch. Возможностей и улучшений здесь существует очень много — всё зависит от ваших потребностей.

## Modules в Application Layer

Мы уже рассмотрели Domain и Infrastructure modules, поэтому теперь давайте посмотрим на Application layer. В Domain-Driven Design мы рекомендуем использовать Application Services как способ отделить клиента как от самой Domain Model, так и от необходимого знания о том, как с ней взаимодействовать. Как вы увидите в главе 11, *Application*, Application Service создаётся вместе со своими dependencies, выполняется с DTO request и возвращает DTO response.

Он также может использовать output dependency для возврата результата:

![Cool](https://github.com/eatae/dddinphp/blob/main/share/image--040.jpg)

Мы рекомендуем создавать modules вокруг Application Services. Каждый module будет содержать свой request и response. Если вы используете Data Transformer как output dependency, следуйте тому же подходу Infrastructure, как и в случае с UI.

## Итоги

Modules — это способ группировки и разделения концепций внутри нашего приложения. Modules должны называться в соответствии с Ubiquitous Language. Мы не должны забывать, что modules — это способ коммуникации высокоуровневых концепций, который помогает нам поддерживать низкую связанность (low coupling) и высокую согласованность (high cohesion). Мы увидели, что можем создавать meaningful modules даже в старых версиях PHP, используя префиксы. Сегодня же стало легко строить modules, следуя соглашениям namespacing PSR-0 и PSR-4.




