# Глава 12. Интеграция Ограниченных Контекстов (Integrating Bounded Contexts)

Каждое корпоративное приложение обычно состоит из нескольких областей, в которых работает компания. Такие области, как биллинг, инвентарь, управление доставкой, каталог и так далее, — распространённые примеры. Самым простым способом управления всеми этими аспектами может показаться монолитная система. Но можно задаться вопросом: действительно ли всё должно быть именно так? Что если трение между командами, работающими над этими отдельными областями, можно уменьшить, разделив это большое монолитное приложение на более мелкие независимые части? В этой главе мы рассмотрим, как это сделать, так что приготовьтесь к инсайтам и эвристикам вокруг стратегического проектирования.

### Работа с распределёнными системами
Работать с распределёнными системами сложно. Разделение системы на независимые автономные части имеет свои преимущества, но также увеличивает сложность. Например, координация и синхронизация распределённых систем — нетривиальная задача и, как следствие, требует внимательного подхода. Как сказал Мартин Фаулер в книге PoEAA, первый закон распределённых систем всегда таков: «Не распределяйте».

## Интеграция через хранилище данных

Одной из самых распространённых техник интеграции различных частей приложения всегда было использование общего хранилища данных вместе с общей кодовой базой. Обычно это называют монолитным приложением, и зачастую оно заканчивается единым хранилищем данных, содержащим данные всех аспектов приложения.

Рассмотрим приложение электронной коммерции. Общее хранилище данных будет содержать все аспекты (например, таблицы внутри реляционной базы данных), связанные с каталогом, биллингом, инвентарём и так далее. Сам по себе такой подход не является неправильным — например, в небольших линейных приложениях, где сложность невысока. Однако в сложных доменах могут возникнуть некоторые проблемы. Если данные разделяются между множеством таблиц, затрагивающих различные аспекты приложения, транзакции будут серьёзно влиять на производительность.

Ещё одна менее техническая проблема связана с Ubiquitous Language. Главное преимущество разделения Bounded Context заключается в наличии собственного Ubiquitous Language для каждого из них. Благодаря этому модели разделяются по своим контекстам. Смешивание всех моделей внутри одного контекста может привести к неоднозначности и путанице.

Возвращаясь к системе электронной коммерции, представим, что мы хотим ввести понятие футболки. В контексте каталога футболка будет продуктом со свойствами вроде цвета, размера, материала и, возможно, красивых фотографий. Однако в системе инвентаря нас это особо не интересует. Здесь продукт имеет другое значение, где важны такие свойства, как вес, местоположение на складе или размеры. Смешивание обоих контекстов приведёт к переплетению понятий и усложнит проектирование. В терминах Domain-Driven Design такое смешение концепций называется Shared Kernel.

### Shared Kernel
Выделите некоторую часть доменной модели, которую команды соглашаются совместно использовать. Разумеется, вместе с этой частью модели сюда входит и связанный с ней код или структура базы данных. Этот явно разделяемый материал имеет особый статус и не должен изменяться без консультации с другой командой. Часто интегрируйте работоспособную систему, хотя и несколько реже, чем темп CONTINUOUS INTEGRATION внутри самих команд. Во время этих интеграций запускайте тесты обеих команд.
Eric Evans — *Domain-Driven Design: Tackling Complexity in the Heart of Software*

Мы не рекомендуем использовать Shared Kernel, поскольку несколько команд могут столкнуться при его разработке, что приводит не только к проблемам сопровождения, но и становится источником трения. Однако если вы всё же решили использовать Shared Kernel, изменения должны заранее согласовываться всеми вовлечёнными сторонами. Концептуально у этого подхода есть и другие проблемы — например, люди начинают воспринимать его как место, куда складывают всё, что больше нигде не подходит, и со временем он бесконечно разрастается. Более удачный способ борьбы с растущей сложностью монолита — разбить его на разные автономные части, взаимодействующие через REST, RPC или системы обмена сообщениями. Это требует чёткого определения границ, при котором каждый контекст, скорее всего, получит собственную инфраструктуру — хранилища данных, серверы, middleware для сообщений и так далее — а также, возможно, собственную команду.

Как можно догадаться, это может привести к определённому дублированию, но это компромисс, на который мы готовы пойти ради уменьшения сложности. В Domain-Driven Design такие независимые части называются Bounded Context.

## Отношения интеграций (Integration Relationships) 

### Заказчик - Поставщик (Customer - Supplier)

Когда между двумя Bounded Context существует однонаправленная интеграция, где один выступает поставщиком (upstream), а другой — клиентом (downstream), возникают команды разработки Customer - Supplier.

- Установите чёткие отношения customer/supplier между двумя командами. Во время сессий планирования дайте downstream-команде роль клиента по отношению к upstream-команде. Согласовывайте и планируйте задачи, связанные с требованиями downstream-команды, чтобы все понимали обязательства и график. Совместно разрабатывайте автоматические acceptance-тесты, которые будут проверять ожидаемый интерфейс. Добавьте эти тесты в набор тестов upstream-команды и запускайте их как часть её continuous integration. Такое тестирование позволит upstream-команде вносить изменения без страха побочных эффектов для downstream.
Eric Evans — *Domain-Driven Design: Tackling Complexity in the Heart of Software*

Команды разработки Customer - Supplier — наиболее распространённый способ интеграции Bounded Context и обычно представляют собой взаимовыгодную ситуацию, когда команды тесно взаимодействуют.

### Раздельные пути (Separate Ways)

Продолжая пример с электронной коммерцией, представим отчётность по доходам для старой легаси-финансовой системы ритейлера. Интеграция может оказаться невероятно дорогой, и усилия на её реализацию просто не будут стоить результата. В стратегическом проектировании Domain-Driven Design это называется Separate Ways.

Интеграция всегда дорога. Иногда выгода от неё невелика. Поэтому объявите BOUNDED CONTEXT не связанным с другими вообще, позволяя разработчикам находить простые специализированные решения в рамках этой небольшой области.
Eric Evans — *Domain-Driven Design: Tackling Complexity in the Heart of Software*

### Конформист (Conformist)

Снова рассмотрим пример электронной коммерции и интеграцию со сторонним сервисом доставки. Оба домена отличаются моделями, командами и инфраструктурой. Команда, поддерживающая сторонний сервис доставки, не будет участвовать в планировании вашего продукта и не станет предоставлять решения для системы электронной коммерции. У этих команд нет тесных отношений. Мы можем принять и адаптироваться к их доменной модели. В стратегическом проектировании такая интеграция называется Conformist Integration.

Устраните сложность перевода между BOUNDED CONTEXTS, полностью следуя модели upstream-команды. Хотя это ограничивает стиль downstream-дизайнеров и, вероятно, не даёт идеальной модели для приложения, выбор CONFORMITY значительно упрощает интеграцию. Кроме того, вы будете использовать общий UBIQUITOUS LANGUAGE с командой-поставщиком. Поставщик находится в ведущей позиции, поэтому полезно сделать коммуникацию максимально простой для него. Альтруизма может оказаться достаточно, чтобы они делились с вами информацией.
Eric Evans — *Domain-Driven Design: Tackling Complexity in the Heart of Software*

### Реализация интеграции Bounded Context

Чтобы упростить объяснение, предположим, что Bounded Context находятся в отношениях Customer - Supplier.

## Современный RPC

Под современным RPC мы подразумеваем RPC через RESTful-ресурсы. Bounded Context предоставляет наружу чёткий интерфейс взаимодействия. Он публикует ресурсы, которыми можно управлять через HTTP-глаголы. Можно сказать, что Bounded Context предоставляет набор сервисов и операций. В стратегическом проектировании это называется Open Host Service.

### Open Host Service
- Определите протокол, предоставляющий доступ к вашей подсистеме как к набору SERVICES. Откройте этот протокол так, чтобы все, кому требуется интеграция, могли им пользоваться. Расширяйте и улучшайте протокол для новых требований интеграции, за исключением случаев, когда отдельная команда имеет специфические потребности. Тогда используйте отдельный translator, расширяющий протокол для этого частного случая, чтобы общий протокол оставался простым и согласованным.
Eric Evans — *Domain-Driven Design: Tackling Complexity in the Heart of Software*

Давайте рассмотрим пример из приложения Last Wishes, поставляемого вместе с GitHub-организацией этой книги.

Приложение представляет собой веб-платформу, позволяющую людям сохранять свои последние завещания перед смертью. Существует два контекста: один отвечает за обработку завещаний — Will Bounded Context, а другой отвечает за начисление очков пользователям системы — Gamification Context. В контексте Will пользователь может иметь бейджи, связанные с количеством очков, набранных в Gamification Context. Это означает, что нам необходимо интегрировать оба контекста, чтобы показывать бейджи пользователя внутри Will Context.

Gamification Context — полноценное event-driven приложение, построенное на собственном event sourcing engine. Это full-stack Symfony-приложение, использующее FOSRestBundle, BazingaHateoasBundle, JMSSerializerBundle, NelmioApiDocBundle и OngrElasticsearchBundle для предоставления REST API уровня 3 и выше (известного как Glory of REST) согласно Richardson Maturity Model. Все события, возникающие в этом контексте, проецируются в Elasticsearch для формирования данных, необходимых представлениям. Мы будем предоставлять количество очков пользователя через endpoint вида `http://gamification.context.host/api/users/{id}`.

Мы также будем получать пользовательскую projection из Elasticsearch и сериализовывать её в формат, заранее согласованный с клиентом:

```php
namespace AppBundle\Controller;

use FOS\RestBundle\Controller\Annotations as Rest;
use FOS\RestBundle\Controller\FOSRestController;
use Nelmio\ApiDocBundle\Annotation\ApiDoc;

class UsersController extends FOSRestController
{
    /**
     * @ApiDoc(
     *     resource = true,
     *     description = "Finds a user given a user ID",
     *     statusCodes = {
     *         200 = "Returned when the user have been found",
     *         404 = "Returned when the user could not be found"
     *     }
     * )
     *
     * @Rest\View(
     *     statusCode = 200
     * )
     */
    public function getUserAction($id)
    {
        $repo = $this->get('es.manager.default.user');
        $user = $repo->find($id);

        if (!$user) {
            throw $this->createNotFoundException(
                sprintf(
                    'A user with an ID of %s does not exist',
                    $id
                )
            );
        }

        return $user;
    }
}
```

Как мы объясняли в главе 2, *Architectural Styles*, чтение рассматривается как инфраструктурная задача, поэтому нет необходимости оборачивать его в поток Command / Command Handler.

Итоговое представление пользователя в формате JSON+HAL будет выглядеть так:

```json
{
    "id": "c3c587c6-610a-42df",
    "points": 0,
    "_links": {
        "self": {
            "href":
            "http://gamification.ctx/api/users/c3c587c6-610a-42df"
        }
    }
}
```

Теперь мы в хорошей позиции для интеграции обоих контекстов. Нам лишь нужно написать клиент в Will Context для потребления endpoint, который мы только что создали. Следует ли нам смешивать обе доменные модели? Непосредственное использование Gamification Context будет означать адаптацию Will Context к Gamification, что приведёт к интеграции типа Conformist. Однако разделение этих аспектов выглядит оправданным. Нам нужен слой, гарантирующий целостность и консистентность доменной модели внутри Will Context, а также механизм перевода points (Gamification) в badges (Will). В Domain-Driven Design такой механизм перевода называется Anti-Corruption Layer.

### Антикоррупционный слой (Anti-Corruption Layer)
- Создайте изолирующий слой, предоставляющий клиентам функциональность в терминах их собственной доменной модели. Этот слой взаимодействует с другой системой через её существующий интерфейс, практически не требуя изменений в самой системе. Внутри слой при необходимости выполняет перевод между двумя моделями в обоих направлениях.
Eric Evans — *Domain-Driven Design: Tackling Complexity in the Heart of Software*

Итак, как же выглядит слой Anti-Corruption? В большинстве случаев сервисы будут взаимодействовать с комбинацией Adapters и Facades. Сервисы инкапсулируют и скрывают низкоуровневую сложность, стоящую за этими преобразованиями. Facades помогают скрывать и инкапсулировать детали доступа, необходимые для получения данных из модели Gamification. Adapters выполняют перевод между моделями, часто используя специализированные Translators.

Давайте посмотрим, как определить User Service внутри модели Will, который будет отвечать за получение бейджей, заработанных конкретным пользователем:

```php
namespace Lw\Domain\Model\User;

interface UserService
{
    public function badgesFrom(UserId $id);
}
```

Теперь посмотрим на реализацию на стороне Infrastructure. Для процесса трансформации мы будем использовать adapter:

```php
namespace Lw\Infrastructure\Service;

use Lw\Domain\Model\User\UserId;
use Lw\Domain\Model\User\UserService;

class TranslatingUserService implements UserService
{
    private $userAdapter;

    public function __construct(UserAdapter $userAdapter)
    {
        $this->userAdapter = $userAdapter;
    }

    public function badgesFrom(UserId $id)
    {
        return $this->userAdapter->toBadges($id);
    }
}
```

А вот HTTP-реализация для UserAdapter:

```php
namespace Lw\Infrastructure\Service;

use GuzzleHttp\Client;

class HttpUserAdapter implements UserAdapter
{
    private $client;

    public function __construct(Client $client)
    {
        $this->client = $client;
    }

    public function toBadges($id)
    {
        $response = $this->client->get(
            sprintf('/users/%s', $id),
            [
                'allow_redirects' => true,
                'headers' => [
                    'Accept' => 'application/hal+json'
                ]
            ]
        );

        $badges = [];

        if (200 === $response->getStatusCode()) {
            $badges =
                (new UserTranslator())
                    ->toBadgesFromRepresentation(
                        json_decode(
                            $response->getBody(),
                            true
                        )
                    );
        }

        return $badges;
    }
}
```

Как видите, Adapter также выступает в роли Facade для Gamification Context. Мы сделали это таким образом, потому что получение ресурса User со стороны Gamification довольно прямолинейно. Adapter использует UserTranslator для выполнения преобразования:

```php
namespace Lw\Infrastructure\Service;

use Lw\Infrastructure\Domain\Model\User\FirstWillMadeBadge;
use Symfony\Component\PropertyAccess\PropertyAccess;

class UserTranslator
{
    public function toBadgesFromRepresentation($representation)
    {
        $accessor = PropertyAccess::createPropertyAccessor();

        $points = $accessor->getValue($representation, 'points');

        $badges = [];

        if ($points > 3) {
            $badges[] = new FirstWillMadeBadge();
        }

        return $badges;
    }
}
```

Translator специализируется на преобразовании points, приходящих из Gamification Context, в badges.

Мы показали, как интегрировать два Bounded Context, команды которых находятся в отношениях Customer-Supplier. Gamification Context предоставляет интеграцию через Open Host Service, реализованный RESTful-протоколом. С другой стороны, Will Context потребляет этот сервис через Anti-Corruption layer, отвечающий за перевод модели из одного домена в другой и обеспечивающий целостность Will Context.

## Очереди сообщений (Message Queues)

RESTful-ресурсы — не единственный способ интеграции между Bounded Context. Как мы увидим далее, messaging middleware позволяет создавать слабо связанные интеграции между различными Context.

Продолжая пример приложения Last Wishes, мы только что реализовали однонаправленные отношения между двумя командами для управления points и badges внутри соответствующих Context. Однако мы намеренно оставили вне рассмотрения одну важную функциональность: награждение пользователя каждый раз, когда он создаёт wish.

Мы могли бы использовать ещё один Open Host Service с pull-стратегией. Will Context периодически запрашивал бы Gamification Context, чтобы синхронизировать badges (например, через scheduler вроде Cron). Такое решение ухудшит пользовательский опыт и приведёт к напрасной трате ресурсов.

Лучший подход — использовать messaging middleware. При таком решении Context смогут отправлять сообщения в middleware (обычно очередь сообщений). Заинтересованные стороны смогут подписываться, просматривать и потреблять информацию по требованию в слабосвязанной форме. Для этого нам нужен специализированный, общий и единый язык коммуникации, чтобы все участники понимали передаваемую информацию. Это называется Published Language.

### Published Language
- Используйте хорошо документированный общий язык, способный выражать необходимую доменную информацию как общее средство коммуникации, при необходимости выполняя перевод в этот язык и из него.
  Eric Evans — *Domain-Driven Design: Tackling Complexity in the Heart of Software*


Размышляя о формате этих сообщений и внимательнее изучив нашу Domain Model, мы понимаем, что у нас уже есть всё необходимое: глава 6, *Domain-Events*. Нет необходимости определять новый способ коммуникации между Bounded Context. Вместо этого мы можем использовать Domain Events для определения общего языка между Context. Определение чего-то, что произошло и важно для Domain Experts, идеально соответствует тому, что нам нужно: формальный Published Language.

В нашем примере мы можем использовать RabbitMQ как messaging middleware. Это, вероятно, один из самых надёжных и устойчивых AMQP-протоколов обмена сообщениями. Мы также будем использовать популярные PHP-библиотеки php-amqplib и RabbitMQBundle.

Начнём с Will Context, поскольку именно он генерирует Events при регистрации пользователя или создании wish. Как мы уже видели в главе 6, *Domain-Events*, хорошей практикой является хранение Domain Events в постоянном хранилище, поэтому будем считать, что это уже реализовано. Нам нужен publisher сообщений, который будет извлекать и публиковать сохранённые Domain Events из Event Store в messaging middleware. Интеграцию с RabbitMQ мы уже делали в главе 6, *Domain-Events*, так что теперь нам остаётся реализовать код в Gamification Context. Мы будем слушать Events, генерируемые Will Context. Поскольку мы используем Symfony Framework, мы воспользуемся Symfony-пакетом RabbitMQBundle.

Мы определяем двух consumers сообщений для событий User Registered и Wish Was Made:

```php
namespace AppBundle\Infrastructure\Messaging\PhpAmqpLib;

use Lw\Gamification\Command\SignupCommand;
use OldSound\RabbitMqBundle\RabbitMq\ConsumerInterface;
use PhpAmqpLib\Message\AMQPMessage;

class PhpAmqpLibLastWillUserRegisteredConsumer
    implements ConsumerInterface
{
    private $commandBus;

    public function __construct($commandBus)
    {
        $this->commandBus = $commandBus;
    }

    public function execute(AMQPMessage $message)
    {
        $type = $message->get('type');

        if ('Lw\Domain\Model\User\UserRegistered' === $type) {
            $event = json_decode($message->body);
            $eventBody = json_decode($event->event_body);

            $this->commandBus->handle(
                new SignupCommand($eventBody->user_id->id)
            );

            return true;
        }

        return false;
    }
}
```

Обратите внимание, что в данном случае мы обрабатываем только сообщения типа `Lw\Domain\Model\User\UserRegistered`:

```php
namespace AppBundle\Infrastructure\Messaging\PhpAmqpLib;

use Lw\Gamification\Command\RewardUserCommand;
use Lw\Gamification\Domain\Model\AggregateDoesNotExist;
use OldSound\RabbitMqBundle\RabbitMq\ConsumerInterface;
use PhpAmqpLib\Message\AMQPMessage;

class PhpAmqpLibLastWillWishWasMadeConsumer
    implements ConsumerInterface
{
    private $commandBus;

    public function __construct($commandBus)
    {
        $this->commandBus = $commandBus;
    }

    public function execute(AMQPMessage $message)
    {
        $type = $message->get('type');

        if ('Lw\Domain\Model\Wish\WishWasMade' === $type) {
            $event = json_decode($message->body);
            $eventBody = json_decode($event->event_body);

            try {
                $points = 5;

                $this->commandBus->handle(
                    new RewardUserCommand(
                        $eventBody->user_id->id,
                        $points
                    )
                );
            } catch (AggregateDoesNotExist $e) {
                // Noop
            }

            return true;
        }

        return false;
    }
}
```

Снова отметим: нас интересуют только события `Lw\Domain\Model\Wish\WishWasMade`.

В обоих случаях мы используем Command Bus, который обсуждали в главе *Application*. Вкратце его можно описать как магистраль, отделяющую Command от Receiver. То, когда и как выполняется Command, не зависит от того, кто её инициировал.

Gamification Context использует Tactician (и TacticianBundle) — простой Command Bus, который можно расширять и адаптировать под свою систему. Теперь мы почти готовы начать потребление Events из Will Context.

Единственное, что ещё осталось сделать, — определить конфигурацию RabbitMQBundle в файле `config.yml` Symfony:

```yaml
services:
    last_will_user_registered_consumer:
        class:
            AppBundle\Infrastructure\Messaging\
                PhpAmqpLib\PhpAmqpLibLastWillUserRegisteredConsumer
        arguments:
            - @tactician.commandbus

    last_will_wish_was_made_consumer:
        class:
            AppBundle\Infrastructure\Messaging\
                PhpAmqpLib\PhpAmqpLibLastWillWishWasMadeConsumer
        arguments:
            - @tactician.commandbus

old_sound_rabbit_mq:
    connections:
        default:
            host: "%rabbitmq_host%"
            port: "%rabbitmq_port%"
            user: "%rabbitmq_user%"
            password: "%rabbitmq_password%"
            vhost: "%rabbitmq_vhost%"
            lazy: true

    consumers:
        last_will_user_registered:
            connection: default
            callback: last_will_user_registered_consumer

            exchange_options:
                name: last-will
                type: fanout

            queue_options:
                name: last-will

        last_will_wish_was_made:
            connection: default
            callback: last_will_wish_was_made_consumer

            exchange_options:
                name: last-will
                type: fanout

            queue_options:
                name: last-will
```

Наиболее удобная конфигурация RabbitMQ — вероятно, паттерн Publish / Subscribe. Все сообщения, публикуемые Will Context, будут доставляться всем подключённым consumers. В конфигурации RabbitMQ exchange это называется fanout.

Exchange представляет собой агент, отвечающий за доставку сообщений в соответствующие очереди:

```bash
> php app/console rabbitmq:consumer --messages=1000 \
  last_will_user_registered

> php app/console rabbitmq:consumer --messages=1000 \
  last_will_wish_was_made
```

С помощью этих двух команд Symfony запустит обоих consumers, и они начнут слушать Domain Events. Мы указали лимит в 1000 сообщений на обработку, поскольку PHP — не лучшая платформа для выполнения долгоживущих процессов. Также может быть хорошей идеей использовать что-то вроде Supervisor для мониторинга и периодического перезапуска процессов.

# Итоги

Хотя мы рассмотрели лишь небольшую часть темы, стратегическое проектирование находится в самом сердце Domain-Driven Design. Это важнейшая часть, помогающая разрабатывать более качественные и семантически выразительные модели. Мы рекомендуем использовать messaging middleware для интеграции Bounded Context, поскольку это естественным образом ведёт к более простым, слабосвязанным и event-driven архитектурам.
