# Глава 10. Репозитории (Repositories)

Чтобы взаимодействовать с объектом Domain, вам нужно иметь ссылку на него. Один из способов добиться этого — создание. Альтернативно можно пройти по ассоциации. В Object-Oriented программах объекты имеют связи (references) с другими объектами, что делает их легко обходимыми и тем самым усиливает выразительность наших моделей. Но здесь есть подвох: вам нужен механизм для получения первого объекта — корня Aggregate.

Repository выступают как места хранения, где извлечённый объект возвращается точно в том же состоянии, в котором он был сохранён. В Domain-Driven Design каждый тип Aggregate обычно имеет связанный с ним уникальный Repository, который используется для его persistence и извлечения. Однако в случаях, когда требуется разделять иерархию объектов Aggregate, типы могут совместно использовать Repository.

После того как вы успешно извлекли Aggregate из Repository, каждое изменение, которое вы в него вносите, сохраняется, что устраняет необходимость снова обращаться к Repository.

## Определение

Martin Fowler определяет Repository следующим образом:

> Механизм между слоями domain и data mapping, действующий как in-memory коллекция domain-объектов. Клиентские объекты декларативно создают query specification и передают их Repository для выполнения. Объекты могут быть добавлены в Repository и удалены из него так же, как и из простой коллекции объектов, а mapping-код, инкапсулированный Repository, выполнит соответствующие операции за кулисами. Концептуально Repository инкапсулирует набор объектов, сохранённых в data store, и операции, выполняемые над ними, предоставляя более object-oriented представление persistence layer. Repository также поддерживает цель достижения чистого разделения и односторонней зависимости между слоями domain и data mapping.

## Repository — это не DAO

`Data Access Object` (DAO) — распространённый паттерн для сохранения Domain-объектов в базе данных. Легко спутать паттерн DAO с Repository. Существенное различие заключается в том, что Repository представляют коллекции, тогда как DAO ближе к базе данных и часто гораздо более ориентированы на таблицы. Обычно DAO содержит CRUD-методы для конкретного Domain-объекта. Давайте посмотрим, как может выглядеть обычный интерфейс DAO:

```php
interface UserDAO
{
    /**
     * @param string $username
     * @return User
     */
    public function get($username);

    public function create(User $user);

    public function update(User $user);

    /**
     * @param string $username
     */
    public function delete($username);
}
```

DAO interface может иметь несколько реализаций — от ORM-конструкций до обычных SQL-запросов. Главная проблема DAO в том, что их обязанности определены неясно. DAO обычно воспринимаются как gateway к базе данных, поэтому относительно легко сильно снизить cohesion, добавляя множество специфических методов для запросов к базе данных:

```php
interface BloatUserDAO
{
    public function get($username);

    public function create(User $user);

    public function update(User $user);

    public function delete($username);

    public function getUserByLastName($lastName);

    public function getUserByEmail($email);

    public function updateEmailAddress($username, $email);

    public function updateLastName($username, $lastName);
}
```

Как вы можете видеть, чем больше новых методов мы добавляем, тем сложнее становится unit-тестирование DAO, и тем сильнее он связывается с объектом `User`. Эта проблема будет расти со временем, пока множество участников продолжают увеличивать `Big Ball of Mud`.

## Repository, ориентированные на коллекции

Repository имитируют коллекцию, реализуя её общие характеристики интерфейса. Как коллекция, Repository не должен раскрывать намерения persistence behavior, такие как понятие сохранения в хранилище.

Базовый механизм persistence должен поддерживать эту потребность. Вам не должно требоваться вручную обрабатывать изменения объектов на протяжении их жизненного цикла. Коллекция ссылается на самые последние изменения объекта, что означает: при каждом доступе вы получаете актуальное состояние объекта.

Repository реализуют конкретный тип коллекции — `Set`. `Set` — это структура данных с инвариантом отсутствия дубликатов. Если вы попытаетесь добавить элемент, который уже присутствует в `Set`, он не будет добавлен. Это полезно в нашем случае, поскольку каждый Aggregate имеет уникальную identity, связанную с Root Entity.

Рассмотрим, например, следующую Domain Model:

```php
namespace Domain\Model;

class Post
{
    const EXPIRE_EDIT_TIME = 120; // seconds

    private $id;
    private $body;
    private $createdAt;

    public function __construct(PostId $anId, Body $aBody)
    {
        $this->id = $anId;
        $this->body = $aBody;
        $this->createdAt = new \DateTimeImmutable();
    }

    public function editBody(Body $aNewBody)
    {
        if ($this->editExpired()) {
            throw new RuntimeException('Edit time expired');
        }

        $this->body = $aNewBody;
    }

    private function editExpired()
    {
        $expiringTime = $this->createdAt->getTimestamp() +
            self::EXPIRE_EDIT_TIME;

        return $expiringTime < time();
    }

    public function id()
    {
        return $this->id;
    }

    public function body()
    {
        return $this->body;
    }

    public function createdAt()
    {
        return $this->createdAt;
    }
}
```

```php
class Body
{
    const MIN_LENGTH = 3;
    const MAX_LENGTH = 250;

    private $content;

    public function __construct($content)
    {
        $this->setContent(trim($content));
    }

    private function setContent($content)
    {
        $this->assertNotEmpty($content);
        $this->assertFitsLength($content);

        $this->content = $content;
    }

    private function assertNotEmpty($content)
    {
        if (empty($content)) {
            throw new DomainException('Empty body');
        }
    }

    private function assertFitsLength($content)
    {
        if (strlen($content) < self::MIN_LENGTH) {
            throw new DomainException('Body is too short');
        }

        if (strlen($content) > self::MAX_LENGTH) {
            throw new DomainException('Body is too long');
        }
    }

    public function content()
    {
        return $this->content;
    }
}
```

```php
class PostId
{
    private $id;

    public function __construct($id = null)
    {
        $this->id = $id ?: uniqid();
    }

    public function id()
    {
        return $this->id;
    }

    public function equals(PostId $anId)
    {
        return $this->id === $anId->id();
    }
}
```

Если бы мы захотели сохранять эту Entity `Post`, простой in-memory `Post Repository` можно было бы создать следующим образом:

```php
class SimplePostRepository
{
    private $post = [];

    public add(Post $aPost)
    {
        $this->posts[(string) $aPost->id()] = $aPost;
    }

    public function postOfId(PostId $anId)
    {
        if (isset($this->posts[(string) $anId])) {
            return $this->posts[(string) $anId];
        }

        return null;
    }
}
```

И, как вы могли ожидать, он обрабатывается как коллекция:

```php
$id = new PostId();

$repository = new SimplePostRepository();

$repository->add(new Post($id, 'Random content'));

// later ...

$post = $repository->postOfId($id);

$post->editBody('Updated content');

// even later ...

$post = $repository->postOfId($id);

assert('Updated content' === $post->body());
```

Как вы можете видеть, с точки зрения коллекции нет необходимости в методе `save` внутри Repository. Изменения, затрагивающие объект, корректно обрабатываются underlying persistence layer.
Collection-oriented Repository — это Repository, которым не требуется повторно добавлять Aggregate, уже сохранённый ранее. В основном это относится к Repository, основанным на памяти, но существуют способы добиться этого и с `Persisted-Oriented Repository`. Мы рассмотрим это чуть позже; кроме того, мы разберём эту тему подробнее в главе 11, *Application*.

Первый шаг при проектировании Repository — определить collection-like interface для него. Интерфейс должен определять обычные методы коллекции, например:

```php
interface PostRepository
{
    public function add(Post $aPost);

    public function addAll(array $posts);

    public function remove(Post $aPost);

    public function removeAll(array $posts);

    // ...
}
```

Для реализации такого интерфейса вы также можете использовать abstract class. В целом, когда мы говорим об interface, мы имеем в виду общую концепцию, а не только конкретный PHP interface. Чтобы сохранить дизайн простым, не добавляйте методы, которые вам не нужны; определение интерфейса Repository и соответствующий Aggregate должны располагаться в одном Module.

Иногда `remove` физически не удаляет Aggregate из базы данных. Эта стратегия — когда Aggregate имеет поле статуса, обновляемое до значения «удалён» — известна как `soft delete`. Почему такой подход интересен? Он может быть полезен для аудита изменений и производительности. В таких случаях вы можете вместо этого помечать Aggregate как отключённый или логически удалённый. Интерфейс можно соответствующим образом обновить, удалив методы удаления или предоставив поведение disable в Repository.

Ещё одним важным аспектом Repository являются finder methods, например:

```php
interface PostRepository
{
    // ...

    /**
     * @return Post
     */
    public function postOfId(PostId $anId);

    /**
     * @return Post[]
     */
    public function latestPosts(DateTimeImmutable $sinceADate);
}
```

Как мы предлагали в главе 4, *Entities*, мы предпочитаем `Application-Generated Identities`. Лучшее место для генерации новой Identity для Aggregate — его Repository. Поэтому логичное место для получения глобально уникального ID для `Post` — это `PostRepository`:

```php
interface PostRepository
{
    // ...

    /**
     * @return PostId
     */
    public function nextIdentity();
}
```

Код, отвечающий за построение каждого экземпляра `Post`, вызывает `nextIdentity`, чтобы получить уникальный идентификатор `PostId`:

```php
$post = new Post($postRepository->nextIdentity(), $body);
```

Некоторые разработчики предпочитают размещать реализацию рядом с определением интерфейса как подпакет Module. Однако, поскольку мы хотим чёткое `Separation of Concerns`, мы рекомендуем вместо этого размещать её внутри слоя Infrastructure.

## In-Memory реализация

Как писал Robert C. Martin в Screaming Architecture:

> Хорошая архитектура программного обеспечения позволяет откладывать решения о framework, базах данных, web-server и других вопросах окружения и инструментах. Хорошая архитектура делает ненужным решение о Rails, Spring, Hibernate, Tomcat или MySql до гораздо более позднего этапа проекта. Хорошая архитектура также позволяет легко изменить своё мнение относительно этих решений. Хорошая архитектура делает акцент на use-case и отделяет их от второстепенных concerns.

На ранних стадиях вашего приложения быстрая in-memory implementation может оказаться очень полезной. Её можно использовать для развития других частей системы, откладывая решения по базе данных до правильного момента. In-memory Repository прост, быстр и лёгок в реализации.

Для нашего `Post Repository` достаточно in-memory hash map, чтобы предоставить всю необходимую функциональность:

```php
namespace Infrastructure\Persistence\InMemory;

use Domain\Model\Post;
use Domain\Model\PostId;
use Domain\Model\PostRepository;

class InMemoryPostRepository implements PostRepository
{
    private $posts = [];

    public function add(Post $aPost)
    {
        $this->posts[$aPost->id()->id()] = $aPost;
    }

    public function remove(Post $aPost)
    {
        unset($this->posts[$aPost->id()->id()]);
    }

    public function postOfId(PostId $anId)
    {
        if (isset($this->posts[$anId->id()])) {
            return $this->posts[$anId->id()];
        }

        return null;
    }

    public function latestPosts(\DateTimeImmutable $sinceADate)
    {
        return $this->filterPosts(
            function (Post $post) use ($sinceADate) {
                return $post->createdAt() > $sinceADate;
            }
        );
    }

    private function filterPosts(callable $fn)
    {
        return array_values(array_filter($this->posts, $fn));
    }

    public function nextIdentity()
    {
        return new PostId();
    }
}
```

## Doctrine ORM

Мы уже довольно много говорили о Doctrine ORM в предыдущих главах. Doctrine — это набор библиотек для хранения данных и object mapping. По умолчанию он поставляется вместе с популярным web framework Symfony и, помимо прочих возможностей, позволяет легко отделить ваше приложение от persistence layer благодаря паттерну `Data Mapper`.

ORM, в свою очередь, построен поверх мощного слоя абстракции базы данных, который позволяет взаимодействовать с базой через SQL dialect под названием `Doctrine Query Language (DQL)`, вдохновлённый знаменитым Java framework Hibernate.

Если мы собираемся использовать Doctrine ORM, первая задача — добавить зависимости в наш проект через Composer:

```bash
composer require doctrine/orm
```

### Object Mapping

Mapping между вашими Domain objects и базой данных можно рассматривать как деталь реализации. Жизненный цикл Domain не должен знать об этих деталях persistence. Поэтому mapping information должна определяться как часть слоя Infrastructure, вне Domain, и как реализация Repository.

#### Doctrine Custom Mapping Types

Поскольку наша Entity `Post` состоит из Value Object, таких как `Body` или `PostId`, хорошей идеей будет создание `Custom Mapping Types` или использование `Doctrine Embeddables`, как было показано в главе про Value Objects. Это значительно упростит object mapping:

```php
namespace Infrastructure\Persistence\Doctrine\Types;

use Doctrine\DBAL\Types\Type;
use Doctrine\DBAL\Platforms\AbstractPlatform;
use Domain\Model\Body;

class BodyType extends Type
{
    public function getSQLDeclaration(
        array $fieldDeclaration,
        AbstractPlatform $platform
    ) {
        return $platform->getVarcharTypeDeclarationSQL(
            $fieldDeclaration
        );
    }

    /**
     * @param string $value
     * @return Body
     */
    public function convertToPHPValue(
        $value,
        AbstractPlatform $platform
    ) {
        return new Body($value);
    }

    /**
     * @param Body $value
     */
    public function convertToDatabaseValue(
        $value,
        AbstractPlatform $platform
    ) {
        return $value->content();
    }

    public function getName()
    {
        return 'body';
    }
}
```

```php
namespace Infrastructure\Persistence\Doctrine\Types;

use Doctrine\DBAL\Types\Type;
use Doctrine\DBAL\Platforms\AbstractPlatform;
use Domain\Model\PostId;

class PostIdType extends Type
{
    public function getSQLDeclaration(
        array $fieldDeclaration,
        AbstractPlatform $platform
    ) {
        return $platform->getGuidTypeDeclarationSQL(
            $fieldDeclaration
        );
    }

    /**
     * @param string $value
     * @return PostId
     */
    public function convertToPHPValue(
        $value,
        AbstractPlatform $platform
    ) {
        return new PostId($value);
    }

    /**
     * @param PostId $value
     */
    public function convertToDatabaseValue(
        $value,
        AbstractPlatform $platform
    ) {
        return $value->id();
    }

    public function getName()
    {
        return 'post_id';
    }
}
```

Не забудьте реализовать магический метод `__toString` в Value Object `PostId`, так как Doctrine этого требует:

```php
class PostId
{
    // ...

    public function __toString()
    {
        return $this->id;
    }
}
```

Doctrine предлагает несколько форматов mapping, например YAML, XML или annotations. XML — наш предпочтительный выбор, поскольку он предоставляет мощное IDE autocompletion:

```xml
<?xml version="1.0" encoding="UTF-8"?>

<doctrine-mapping
    xmlns="http://doctrine-project.org/schemas/orm/doctrine-mapping"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="
        http://doctrine-project.org/schemas/orm/doctrine-mapping
        http://raw.github.com/doctrine/doctrine2/master/doctrine-mapping.xsd">

    <entity name="Domain\Model\Post" table="posts">

        <id name="id" type="post_id" column="id">
            <generator strategy="NONE" />
        </id>

        <field
            name="body"
            type="body"
            length="250"
            column="body"
        />

        <field
            name="createdAt"
            type="datetime"
            column="created_at"
        />
    </entity>
</doctrine-mapping>
```

#### Упражнение

*Запишите, как выглядел бы mapping в случае использования подхода `Doctrine Embeddables`. Обратитесь к главе *Value Objects* или *Entities*, если вам нужна помощь.*

### Entity Manager

`EntityManager` — центральная точка доступа к функциональности ORM. Его bootstrap довольно прост:

```php
use Doctrine\DBAL\Types\Type;
use Doctrine\ORM\EntityManager;
use Doctrine\ORM\Tools;

Type::addType(
    'post_id',
    'Infrastructure\Persistence\Doctrine\Types\PostIdType'
);

Type::addType(
    'body',
    'Infrastructure\Persistence\Doctrine\Types\BodyType'
);

$entityManager = EntityManager::create(
    [
        'driver' => 'pdo_sqlite',
        'path' => __DIR__ . '/db.sqlite',
    ],
    Tools\Setup::createXMLMetadataConfiguration(
        ['/Path/To/Infrastructure/Persistence/Doctrine/Mapping'],
        $devMode = true
    )
);
```

Не забудьте настроить его в соответствии с вашими потребностями и окружением.

### Реализация DQL

В случае этого Repository нам понадобится только `EntityManager`, чтобы извлекать Domain objects напрямую из базы данных:

```php
namespace Infrastructure\Persistence\Doctrine;

use Doctrine\ORM\EntityManager;
use Domain\Model\Post;
use Domain\Model\PostId;
use Domain\Model\PostRepository;

class DoctrinePostRepository implements PostRepository
{
    protected $em;

    public function __construct(EntityManager $em)
    {
        $this->em = $em;
    }

    public function add(Post $aPost)
    {
        $this->em->persist($aPost);
    }

    public function remove(Post $aPost)
    {
        $this->em->remove($aPost);
    }

    public function postOfId(PostId $anId)
    {
        return $this->em->find('Domain\Model\Post', $anId);
    }

    public function latestPosts(\DateTimeImmutable $sinceADate)
    {
        return $this->em->createQueryBuilder()
            ->select('p')
            ->from('Domain\Model\Post', 'p')
            ->where('p.createdAt > :since')
            ->setParameter(':since', $sinceADate)
            ->getQuery()
            ->getResult();
    }

    public function nextIdentity()
    {
        return new PostId();
    }
}
```

Если вы посмотрите некоторые примеры Doctrine, то можете заметить, что после вызова `persist` или `remove` следует вызывать `flush`. Но, как видно в нашем подходе, вызова `flush` нет. Выполнение `flush` и работа с транзакциями делегированы `Application Service`. Именно поэтому можно работать с Doctrine, предполагая, что `flush` всех изменений Entity произойдёт в конце запроса. С точки зрения производительности один вызов `flush` — лучший вариант.


## Репозиторий, ориентированный на сохранение данных (Persistence-Oriented Repository)

Бывают случаи, когда collection-oriented Repository плохо сочетаются с нашим механизмом persistence. Если у вас нет `unit of work`, отслеживание изменений Aggregate становится сложной задачей. Единственный способ сохранить такие изменения — явно вызвать `save`.

Определение интерфейса для persistence-oriented Repository похоже на определение collection-oriented аналога:

```php
interface PostRepository
{
    public function nextIdentity();

    public function postOfId(PostId $anId);

    public function save(Post $aPost);

    public function saveAll(array $posts);

    public function remove(Post $aPost);

    public function removeAll(array $posts);
}
```

В этом случае у нас появились методы `save` и `saveAll`, которые предоставляют функциональность, аналогичную предыдущим методам `add` и `addAll`. Однако важное различие заключается в том, как клиент их использует. В collection-oriented стиле методы `add` используются только один раз — когда Aggregate создаётся. В persistence-oriented стиле вы будете использовать `save` не только после создания нового Aggregate, но и после изменения существующего:

```php
$post = new Post(/* ... */);

$postRepository->save($post);

// later ...

$post = $postRepository->postOfId($postId);

$post->editBody(new Body('New body!'));

$postRepository->save($post);
```

Помимо этого различия, детали находятся только в реализации.

## Реализация на Redis

Redis — это in-memory key-value store, который может использоваться как cache или storage.

В зависимости от обстоятельств мы можем рассмотреть использование Redis как хранилища для наших Aggregate.

Для начала убедитесь, что у вас есть PHP client для подключения к Redis. Хороший вариант, который мы рекомендуем — Predis:

```bash
composer require predis/predis:~1.0
```

```php
namespace Infrastructure\Persistence\Redis;

use Domain\Model\Post;
use Domain\Model\PostId;
use Domain\Model\PostRepository;
use Predis\Client;

class RedisPostRepository implements PostRepository
{
    private $client;

    public function __construct(Client $client)
    {
        $this->client = $client;
    }

    public function save(Post $aPost)
    {
        $this->client->hset(
            'posts',
            (string) $aPost->id(),
            serialize($aPost)
        );
    }

    public function remove(Post $aPost)
    {
        $this->client->hdel('posts', (string) $aPost->id());
    }

    public function postOfId(PostId $anId)
    {
        if ($data = $this->client->hget('posts', (string) $anId)) {
            return unserialize($data);
        }

        return null;
    }

    public function latestPosts(\DateTimeImmutable $sinceADate)
    {
        $latest = $this->filterPosts(
            function (Post $post) use ($sinceADate) {
                return $post->createdAt() > $sinceADate;
            }
        );

        $this->sortByCreatedAt($latest);

        return array_values($latest);
    }

    private function filterPosts(callable $fn)
    {
        return array_filter(
            array_map(function ($data) {
                return unserialize($data);
            }, $this->client->hgetall('posts')),
            $fn
        );
    }

    private function sortByCreatedAt(&$posts)
    {
        usort($posts, function (Post $a, Post $b) {
            if ($a->createdAt() == $b->createdAt()) {
                return 0;
            }

            return ($a->createdAt() < $b->createdAt()) ? -1 : 1;
        });
    }

    public function nextIdentity()
    {
        return new PostId();
    }
}
```

## Реализация на SQL

В классическом примере мы можем создать простую PDO implementation для нашего `PostRepository`, используя обычные SQL-запросы:

```php
namespace Infrastructure\Persistence\Sql;

use Domain\Model\Body;
use Domain\Model\Post;
use Domain\Model\PostId;
use Domain\Model\PostRepository;

class SqlPostRepository implements PostRepository
{
    const DATE_FORMAT = 'Y-m-d H:i:s';

    private $pdo;

    public function __construct(\PDO $pdo)
    {
        $this->pdo = $pdo;
    }

    public function save(Post $aPost)
    {
        $sql = 'INSERT INTO posts ' .
            '(id, body, created_at) VALUES ' .
            '(:id, :body, :created_at)';

        $this->execute($sql, [
            'id' => $aPost->id()->id(),
            'body' => $aPost->body()->content(),
            'created_at' => $aPost->createdAt()->format(
                self::DATE_FORMAT
            )
        ]);
    }

    private function execute($sql, array $parameters)
    {
        $st = $this->pdo->prepare($sql);

        $st->execute($parameters);

        return $st;
    }

    public function remove(Post $aPost)
    {
        $this->execute(
            'DELETE FROM posts WHERE id = :id',
            [
                'id' => $aPost->id()->id()
            ]
        );
    }

    public function postOfId(PostId $anId)
    {
        $st = $this->execute(
            'SELECT * FROM posts WHERE id = :id',
            [
                'id' => $anId->id()
            ]
        );

        if ($row = $st->fetch(\PDO::FETCH_ASSOC)) {
            return $this->buildPost($row);
        }

        return null;
    }

    private function buildPost($row)
    {
        return new Post(
            new PostId($row['id']),
            new Body($row['body']),
            new \DateTimeImmutable($row['created_at'])
        );
    }

    public function latestPosts(\DateTimeImmutable $sinceADate)
    {
        return $this->retrieveAll(
            'SELECT * FROM posts WHERE created_at > :since_date',
            [
                'since_date' => $sinceADate->format(
                    self::DATE_FORMAT
                )
            ]
        );
    }

    private function retrieveAll($sql, array $parameters = [])
    {
        $st = $this->pdo->prepare($sql);

        $st->execute($parameters);

        return array_map(function ($row) {
            return $this->buildPost($row);
        }, $st->fetchAll(\PDO::FETCH_ASSOC));
    }

    public function nextIdentity()
    {
        return new PostId();
    }

    public function size()
    {
        return $this->pdo
            ->query('SELECT COUNT(*) FROM posts')
            ->fetchColumn();
    }
}
```

Поскольку у нас нет никакой mapping configuration, было бы очень полезно иметь метод инициализации schema внутри того же класса. Вещи, которые изменяются вместе, должны оставаться вместе:

```php
class SqlPostRepository implements PostRepository
{
    // ...

    public function initSchema()
    {
        $this->pdo->exec(<<<SQL
DROP TABLE IF EXISTS posts;

CREATE TABLE posts (
    id CHAR(36) PRIMARY KEY,
    body VARCHAR(250) NOT NULL,
    created_at DATETIME NOT NULL
) ENGINE=InnoDB
  DEFAULT CHARSET=utf8mb4
  COLLATE=utf8mb4_unicode_ci;
SQL
        );
    }
}
```

## Дополнительное поведение

```php
interface PostRepository
{
    // ...

    public function size();
}
```

Реализация может выглядеть следующим образом:

```php
class DoctrinePostRepository implements PostRepository
{
    // ...

    public function size()
    {
        return $this->em->createQueryBuilder()
            ->select('count(p.id)')
            ->from('Domain\Model\Post', 'p')
            ->getQuery()
            ->getSingleScalarResult();
    }
}
```

Добавление дополнительного поведения в Repository может быть очень полезным. Пример — возможность подсчитать количество элементов в данной коллекции. Вам может прийти в голову добавить метод с именем `count`; однако, поскольку мы пытаемся имитировать коллекцию, лучшим названием будет `size`.

Вы также можете помещать в Repository специфические вычисления, counters, read-optimized queries или сложные команды (`INSERT`, `UPDATE` или `DELETE`). Однако всё поведение всё равно должно соответствовать характеристикам коллекции Repository. Рекомендуется переносить как можно больше логики в stateless Domain Services, специфичные для Domain, вместо простого добавления этих обязанностей в Repository.

В некоторых случаях вам не потребуется весь Aggregate только ради доступа к небольшому объёму информации. Чтобы решить это, можно добавить методы Repository для доступа к таким данным как shortcuts. Следует удостовериться, что вы обращаетесь только к данным, которые могли бы быть получены через навигацию по Aggregate Root. Поэтому вы не должны предоставлять доступ к private и internal областям Aggregate Root, так как это нарушит установленное contractual agreement.

Для некоторых use case вам потребуются очень специфические запросы, представляющие композиции нескольких типов Aggregate, каждый из которых возвращает определённую информацию. Такие запросы могут выполняться и затем возвращаться как единый Value Object. Очень распространено, когда Repository возвращают Value Objects.

Если вы замечаете, что создаёте множество finder methods, оптимизированных под конкретные use case, это может быть распространённым code smell. Это может указывать на неправильно выбранную границу Aggregate. Однако если вы уверены, что границы определены правильно, возможно, пришло время изучить CQRS.

## Querying Repository

При сравнении Repository отличаются от коллекции, если учитывать их возможности querying. Repository работает с большим набором объектов, которые обычно не находятся в памяти в момент выполнения запроса. Нереалистично загружать в память все экземпляры Domain object и выполнять запрос по ним.

Хорошее решение — передавать criterion и позволять Repository самостоятельно обрабатывать детали реализации для успешного выполнения операции. Он может преобразовать criterion в SQL- или ORM-запросы либо выполнить итерацию по in-memory collection. Однако это не имеет значения, поскольку реализация сама занимается этими деталями.


## Паттерн Specification

Распространённой реализацией объекта-критерия является паттерн Specification. Спецификация — это простой предикат, который принимает объект Domain и возвращает boolean. Для заданного объекта Domain он вернёт true, если объект удовлетворяет спецификации, и false — в противном случае:

```php
interface PostSpecification
{
    /**
     * @return boolean
     */
    public function specifies(Post $aPost);
}
```

Нам лишь нужно добавить метод query в наш Repository:

```php
interface PostRepository
{
    // ...
    public function query($specification);
}
```

## In-Memory Implementation

В качестве примера, если бы мы хотели воспроизвести метод запроса latestPosts в нашем PostRepository, используя Specification для in-memory реализации, это выглядело бы следующим образом:

```php
namespace Infrastructure\Persistence\InMemory;

use Domain\Model\Post;

interface InMemoryPostSpecification
{
    /**
     * @return boolean
     */
    public function specifies(Post $aPost);
}
```

In-memory реализация поведения latestPosts могла бы выглядеть так:

```php
namespace Infrastructure\Persistence\InMemory;

use Domain\Model\Post;

class InMemoryLatestPostSpecification
    implements InMemoryPostSpecification
{
    private $since;

    public function __construct(\DateTimeImmutable $since)
    {
        $this->since = $since;
    }

    public function specifies(Post $aPost)
    {
        return $aPost->createdAt() > $this->since;
    }
}
```

Метод query для реализации нашего Repository мог бы выглядеть так:

```php
class InMemoryPostRepository implements PostRepository
{
    // ...

    /**
     * @param InMemoryPostSpecification $specification
     *
     * @return Post[]
     */
    public function query($specification)
    {
        return $this->filterPosts(
            function (Post $post) use($specification) {
                return $specification->specifies($post);
            }
        );
    }
}
```

Получить все последние posts из Repository так же просто, как создать специализированный экземпляр приведённой выше реализации:

```php
$latestPosts = $postRepository->query(
    new InMemoryLatestPostSpecification(new \DateTimeImmutable('-24'))
);
```

## SQL Implementation

Стандартная спецификация хорошо работает для in-memory реализаций. Однако, так как для SQL-реализации мы не загружаем заранее все объекты Domain в память, для таких случаев нам нужна более специализированная спецификация:

```php
namespace Infrastructure\Persistence\Sql;

interface SqlPostSpecification
{
    /**
     * @return string
     */
    public function toSqlClauses();
}
```

SQL-реализация этой спецификации могла бы выглядеть так:

```php
namespace Infrastructure\Persistence\Sql;

class SqlLatestPostSpecification implements SqlPostSpecification
{
    private $since;

    public function __construct(\DateTimeImmutable $since)
    {
        $this->since = $since;
    }

    public function toSqlClauses()
    {
        return "created_at >'" .
            $this->since->format('Y-m-d H:i:s') .
            "'";
    }
}
```

А вот пример того, как выполнять запрос к реализации SQLPostRepository:

```php
class SqlPostRepository implements PostRepository
{
    // ...

    /**
     * @param SqlPostSpecification $specification
     *
     * @return Post[]
     */
    public function query($specification)
    {
        return $this->retrieveAll(
            'SELECT * FROM posts WHERE ' .
            $specification->toSqlClauses()
        );
    }

    private function retrieveAll($sql, array $parameters = [])
    {
        $st = $this->pdo->prepare($sql);
        $st->execute($parameters);

        return array_map(function ($row) {
            return $this->buildPost($row);
        }, $st->fetchAll(\PDO::FETCH_ASSOC));
    }
}
```

## Managing Transactions

Domain Model — не то место, где следует управлять транзакциями. Операции, применяемые к Domain Model, должны быть независимы от механизма persistence. Распространённый подход к решению этой проблемы — размещение Facade в слое Application, тем самым группируя связанные use cases вместе. Когда метод Facade вызывается из слоя UI, бизнес-метод начинает транзакцию. После завершения Facade завершает взаимодействие, выполняя commit транзакции. Если что-то идёт не так, транзакция откатывается:

```php
use Doctrine\ORM\EntityManager;

class SomeApplicationServiceFacade
{
    private $em;

    public function __construct(EntityManager $em)
    {
        $this->em = $em;
    }

    public function doSomeUseCaseTask()
    {
        try {
            $this->em->getConnection()->beginTransaction();

            // Use domain model

            $this->em->getConnection()->commit();
        } catch (Exception $e) {
            $this->em->getConnection()->rollback();

            throw $e;
        }
    }
}
```

Проблема, возникающая с Facades, заключается в том, что нам приходится повторять один и тот же boilerplate-код снова и снова. Если унифицировать способ выполнения use cases, мы могли бы оборачивать их в транзакцию, используя паттерн Decorator:

```php
interface ApplicationService
{
    /**
     * @param $request
     * @return mixed
     */
    public function execute(BaseRequest $request);
}
```

```php
class SomeApplicationService implements ApplicationService
{
    public function execute(BaseRequest $request)
    {
        // do something
    }
}
```

Мы не хотим связывать наш слой Application с конкретной процедурой транзакций, поэтому вместо этого можем создать для неё простой интерфейс:

```php
interface TransactionalSession
{
    /**
     * @param callable $operation
     * @return mixed
     */
    public function executeAtomically(callable $operation);
}
```

Реализация паттерна Decorator, которая может сделать любой Application Service транзакционным, выглядит довольно просто:

```php
class TransactionalApplicationService implements ApplicationService
{
    private $session;
    private $service;

    public function __construct(
        ApplicationService $service,
        TransactionalSession $session
    ) {
        $this->session = $session;
        $this->service = $service;
    }

    public function execute(BaseRequest $request)
    {
        $operation = function() use($request) {
            return $this->service->execute($request);
        };

        return $this->session->executeAtomically(
            $operation->bindTo($this)
        );
    }
}
```

После этого мы могли бы также создать реализацию Doctrine transactional session:

```php
class DoctrineSession implements TransactionalSession
{
    private $entityManager;

    public function __construct(EntityManager $entityManager)
    {
        $this->entityManager = $entityManager;
    }

    public function executeAtomically(callable $operation)
    {
        return $this->entityManager->transactional($operation);
    }
}
```

Теперь у нас есть всё необходимое, чтобы выполнять наши use cases внутри транзакции:

```php
$useCase = new TransactionalApplicationService(
    new SomeApplicationService(
        // ...
    ),
    new DoctrineSession(
        // ...
    )
);

$response = $useCase->execute();
```

# Testing Repositories

Чтобы быть уверенными, что Repository будет работать в production, нам потребуется протестировать его реализацию. Для этого мы должны протестировать границы системы, удостоверившись, что наши ожидания корректны.

В случае теста Doctrine настройка будет немного более сложной:

```php
use Doctrine\DBAL\Types\Type;
use Doctrine\ORM\EntityManager;
use Doctrine\ORM\Tools;
use Domain\Model\Post;

class DoctrinePostRepositoryTest extends \PHPUnit_Framework_TestCase
{
    private $postRepository;

    public function setUp()
    {
        $this->postRepository = $this->createPostRepository();
    }

    private function createPostRepository()
    {
        $this->addCustomTypes();

        $em = $this->initEntityManager();

        $this->initSchema($em);

        return new PrecociousDoctrinePostRepository($em);
    }

    private function addCustomTypes()
    {
        if (!Type::hasType('post_id')) {
            Type::addType(
                'post_id',
                'Infrastructure\Persistence\Doctrine\Types\PostIdType'
            );
        }

        if (!Type::hasType('body')) {
            Type::addType(
                'body',
                'Infrastructure\Persistence\Doctrine\Types\BodyType'
            );
        }
    }

    protected function initEntityManager()
    {
        return EntityManager::create(
            ['url' => 'sqlite:///:memory:'],
            Tools\Setup::createXMLMetadataConfiguration(
                ['/Path/To/Infrastructure/Persistence/Doctrine/Mapping'],
                $devMode = true
            )
        );
    }

    private function initSchema(EntityManager $em)
    {
        $tool = new Tools\SchemaTool($em);

        $tool->createSchema([
            $em->getClassMetadata('Domain\Model\Post')
        ]);
    }

    // ...
}
```

```php
class PrecociousDoctrinePostRepository extends DoctrinePostRepository
{
    public function persist(Post $aPost)
    {
        parent::persist($aPost);

        $this->em->flush();
    }

    public function remove(Post $aPost)
    {
        parent::remove($aPost);

        $this->em->flush();
    }
}
```

После того как у нас настроено это окружение, мы можем продолжить тестирование поведения Repository:

```php
class DoctrinePostRepositoryTest extends \PHPUnit_Framework_TestCase
{
    // ...

    /**
     * @test
     */
    public function itShouldRemovePost()
    {
        $post = $this->persistPost('irrelevant body');

        $this->postRepository->remove($post);

        $this->assertPostExist($post->id());
    }

    private function assertPostExist($id)
    {
        $result = $this->postRepository->postOfId($id);

        $this->assertNull($result);
    }

    private function persistPost(
        $body,
        \DateTimeImmutable $createdAt = null
    ) {
        $this->postRepository->add(
            $post = new Post(
                $this->postRepository->nextIdentity(),
                new Body($body),
                $createdAt
            )
        );

        return $post;
    }
}
```

Следуя нашему более раннему утверждению, если мы сохраняем Post, мы ожидаем найти его в точно таком же состоянии.

Теперь мы можем перейти к тестированию получения последних posts, задавая определённую дату:

```php
class DoctrinePostRepositoryTest extends \PHPUnit_Framework_TestCase
{
    // ...

    /**
     * @test
     */
    public function itShouldFetchLatestPosts()
    {
        $this->persistPost(
            'a year ago',
            new \DateTimeImmutable('-1 year')
        );

        $this->persistPost(
            'a month ago',
            new \DateTimeImmutable('-1 month')
        );

        $this->persistPost(
            'few hours ago',
            new \DateTimeImmutable('-3 hours')
        );

        $this->persistPost(
            'few minutes ago',
            new \DateTimeImmutable('-2 minutes')
        );

        $posts = $this->postRepository->latestPosts(
            new \DateTimeImmutable('-24 hours')
        );

        $this->assertCount(2, $posts);

        $this->assertEquals(
            'few hours ago',
            $posts[0]->body()->content()
        );

        $this->assertEquals(
            'few minutes ago',
            $posts[1]->body()->content()
        );
    }
}
```

# Testing Your Services with In-Memory Implementations

Настройка полностью persistent реализации Repository может быть сложной и приводить к медленному выполнению. Вам следует заботиться о том, чтобы ваши тесты оставались быстрыми. Прохождение через полную настройку базы данных, а затем выполнение запросов значительно вас замедлит. Наличие in-memory реализации может помочь отложить решения о persistence до самого конца. Мы можем тестировать тем же способом, что и раньше, но на этот раз будем использовать полнофункциональную, быструю и простую in-memory реализацию:

```php
class MyServiceTest extends \PHPUnit_Framework_TestCase
{
    private $service;

    public function setUp()
    {
        $this->service = new MyService(
            new InMemoryPostRepository()
        );
    }
}
```

# Wrap-Up

Repository — это механизм, который выступает в роли хранилища. Разница между DAO и Repository заключается в том, что DAO следует database-first подходу, уменьшая cohesion большим количеством низкоуровневых методов для запросов к базе данных. В зависимости от лежащего в основе механизма persistence мы рассмотрели различные подходы к Repository:

- Collection-oriented Repositories, как правило, ближе к Domain model, даже если они persist Entities. С точки зрения клиента collection-oriented Repository выглядит как коллекция (Set). Нет необходимости в явных вызовах persistence при обновлении Entity, так как Repository отслеживает изменения объектов. Мы рассмотрели, как использовать Doctrine в качестве лежащего в основе механизма persistence для такого типа Repository.

- Persistence-oriented Repositories требуют явных вызовов persistence, так как они не отслеживают изменения объектов. Мы рассмотрели реализации на Redis и plain SQL.

По пути мы познакомились со Specifications как с паттерном, который помогает выполнять запросы к базе данных без потери гибкости и cohesion. Мы также изучили, как управлять транзакциями и как тестировать наши services с помощью простых и быстрых in-memory реализаций Repository.

