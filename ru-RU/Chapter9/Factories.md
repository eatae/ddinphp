# Глава 9. Фабрики (Factories)

## Фабрики

Фабрики — это мощная абстракция. Они помогают отделить клиента от деталей взаимодействия с Domain. Клиенту не нужно знать, как строить сложные объекты и Aggregate, поэтому вы можете использовать Factory для создания целых Aggregate, тем самым обеспечивая соблюдение их инвариантов.

## Factory Method в корне Aggregate

Паттерн `Factory Method`, как он определён в классической книге Design Patterns: Elements of Reusable Object-Oriented Software, является порождающим паттерном, который:

> Определяет интерфейс для создания объекта, но оставляет выбор его типа подклассам, откладывая создание до времени выполнения.

Добавление `Factory Method` в корень Aggregate скрывает внутренние детали реализации создания Aggregate от любого внешнего клиента. Это также возвращает ответственность за целостность Aggregate обратно к корню.

В Domain Model, где у нас есть Entity `User` и Entity `Wish`, `User` выступает корнем Aggregate. Не существует `Wish` без `User`. Entity `User` должна управлять своими Aggregate.

Способ вернуть контроль над `Wish` обратно Entity `User` — разместить Factory method в корне Aggregate:

```php
class User
{
    // ...

    public function makeWish(WishId $wishId, $email, $content)
    {
        $wish = new WishEmail(
            $wishId,
            $this->id(),
            $email,
            $content
        );

        DomainEventPublisher::instance()->publish(
            new WishMade($wishId)
        );

        return $wish;
    }
}
```

Клиенту не нужно знать внутренние детали того, как корень Aggregate обрабатывает логику создания:

```php
$wish = $aUser->makeWish(
    $wishRepository->nextIdentity(),
    'user@example.com',
    'I want to be free!'
);
```

## Принудительное соблюдение инвариантов

Factory Methods в корне Aggregate также являются хорошим местом для инвариантов.

В Domain Model с Entity `Forum` и `Post`, где `Post` является агрегированной частью корня Aggregate `Forum`, публикация `Post` могла бы выглядеть примерно так:

```php
class Forum
{
    // ...

    public function publishPost(PostId $postId, $content)
    {
        $post = new Post($this->id, $postId, $content);

        DomainEventPublisher::instance()->publish(
            new PostPublished($postId)
        );

        return $post;
    }
}
```

После разговора с Domain Expert мы пришли к выводу, что `Post` не должен публиковаться, когда `Forum` закрыт. Это инвариант, и мы можем обеспечить его прямо при создании `Post`, тем самым предотвращая несогласованное состояние Domain:

```php
class Forum
{
    // ...

    public function publishPost(PostId $postId, $content)
    {
        if ($this->isClosed()) {
            throw new ForumClosedException();
        }

        $post = new Post($this->id, $postId, $content);

        DomainEventPublisher::instance()->publish(
            new PostPublished($postId)
        );

        return $post;
    }
}
```

## Factory в Service

Разделение логики создания также оказывается полезным в наших Service.

### Построение Specification

Использование Specification в наших Service, возможно, является лучшим примером того, как использовать Factory внутри Service.

Рассмотрим следующий пример Service. Получив запрос из внешнего мира, мы хотим построить feed на основе последних `Post`, добавленных в систему:

```php
namespace Application\Service;

use Domain\Model\Post;
use Domain\Model\PostRepository;

class LatestPostsFeedService
{
    private $postRepository;

    public function __construct(PostRepository $postRepository)
    {
        $this->postRepository = $postRepository;
    }

    /**
     * @param LatestPostsFeedRequest $request
     */
    public function execute($request)
    {
        $posts = $this->postRepository->latestPosts($request->since);

        return array_map(function(Post $post) {
            return [
                'id' => $post->id()->id(),
                'content' => $post->body()->content(),
                'created_at' => $post-> createdAt()
            ];
        }, $posts);
    }
}
```

Методы поиска в Repository, такие как `latestPosts`, имеют некоторые ограничения, поскольку они бесконечно добавляют сложность в наши Repository. Как мы обсудим в главе 10, *Repositories*, более хорошим подходом являются `Specification`.

К счастью для нас, у нас есть хороший query method в `PostRepository`, который работает со `Specification`:

```php
class LatestPostsFeedService
{
    // ...

    public function execute($request)
    {
        $posts = $this->postRepository->query($specification);
    }
}
```

Использование конкретной реализации `Specification` — плохая идея:

```php
class LatestPostsFeedService
{
    public function execute($request)
    {
        $posts = $this->postRepository->query(
            new SqlLatestPostSpecification($request->since)
        );
    }
}
```

Связывание нашего высокоуровневого `Application Service` с низкоуровневой реализацией `Specification` смешивает слои и нарушает `Separation of Concerns`. Кроме того, это довольно плохой способ связать наш Service с конкретной реализацией Infrastructure. Нет никакой возможности использовать этот Service вне SQL persistence solution. Что, если мы захотим протестировать наш Service с in-memory implementation?

Решение этой проблемы — отделить создание `Specification` от самого Service с помощью паттерна `Abstract Factory`. Согласно OODesign.com:

> Abstract Factory предоставляет интерфейс для создания семейства связанных объектов без явного указания их классов.

Поскольку у нас может быть несколько реализаций `Specification`, сначала нам нужно создать интерфейс для Factory:

```php
namespace Domain\Model;

interface PostSpecificationFactory
{
    public function createLatestPosts(DateTimeImmutable $since);
}
```

Затем нам нужно создать Factory для каждой реализации `PostRepository`. Например, Factory для in-memory реализации `PostRepository` может выглядеть следующим образом:

```php
namespace Infrastructure\Persistence\InMemory;

use Domain\Model\PostSpecificationFactory;

class InMemoryPostSpecificationFactory
    implements PostSpecificationFactory
{
    public function createLatestPosts(DateTimeImmutable $since)
    {
        return new InMemoryLatestPostSpecification($since);
    }
}
```

Как только у нас появляется централизованное место для логики создания, становится легко отделить её от Service:

```php
class LatestPostsFeedService
{
    private $postRepository;
    private $postSpecificationFactory;

    public function __construct(
        PostRepository $postRepository,
        PostSpecificationFactory $postSpecificationFactory
    ) {
        $this->postRepository = $postRepository;
        $this->postSpecificationFactory = $postSpecificationFactory;
    }

    public function execute($request)
    {
        $posts = $this->postRepository->query(
            $this->postSpecificationFactory->createLatestPosts(
                $request->since
            )
        );
    }
}
```

Теперь unit-тестирование нашего Service через in-memory реализацию `PostRepository` становится довольно простым:

```php
namespace Application\Service;

use Domain\Model\Body;
use Domain\Model\Post;
use Domain\Model\PostId;
use Infrastructure\Persistence\InMemory\InMemoryPostRepositor;

class LatestPostsFeedServiceTest extends PHPUnit_Framework_TestCase
{
    /**
     * @var \Infrastructure\Persistence\InMemory\InMemoryPostRepository
     */
    private $postRepository;

    /**
     * @var LatestPostsFeedService
     */
    private $latestPostsFeedService;

    public function setUp()
    {
        $this->latestPostsFeedService = new LatestPostsFeedService(
            $this->postRepository = new InMemoryPostRepository()
        );
    }

    /**
     * @test
     */
    public function shouldBuildAFeedFromLatestPosts()
    {
        $this->addPost(1, 'first', '-2 hours');
        $this->addPost(2, 'second', '-3 hours');
        $this->addPost(3, 'third', '-5 hours');

        $feed = $this->latestPostsFeedService->execute(
            new LatestPostsFeedRequest(
                new \DateTimeImmutable('-4 hours')
            )
        );

        $this->assertFeedContains([
            ['id' => 1, 'content' => 'first'],
            ['id' => 2, 'content' => 'second']
        ], $feed);
    }
```

```php
    private function addPost($id, $content, $createdAt)
    {
        $this->postRepository->add(new Post(
            new PostId($id),
            new Body($content),
            new \DateTimeImmutable($createdAt)
        ));
    }

    private function assertFeedContains($expected, $feed)
    {
        foreach ($expected as $index => $contents) {
            $this->assertArraySubset($contents, $feed[$index]);
            $this->assertNotNull($feed[$index]['created_at']);
        }
    }
}
```

### Построение Aggregate

Entity не зависят от механизма persistence. Вы не хотите связывать и загрязнять ваши Entity деталями persistence. Взгляните на следующий `Application Service`:

```php
class SignUpUserService
{
    private $userRepository;

    public function __construct(UserRepository $userRepository)
    {
        $this->userRepository = $userRepository;
    }

    /**
     * @param SignUpUserRequest $request
     */
    public function execute($request)
    {
        $email = $request->email();
        $password = $request->password();

        $user = $this->userRepository->userOfEmail($email);

        if (null !== $user) {
            throw new UserAlreadyExistsException();
        }

        $this->userRepository->persist(new User(
            $this->userRepository->nextIdentity(),
            $email,
            $password
        ));

        return $user;
    }
}
```

Представьте Entity `User` следующего вида:

```php
class User
{
    private $userId;
    private $email;
    private $password;

    public function __construct(UserId $userId, $email, $password)
    {
        // ...
    }

    // ...
}
```

Представьте, что мы хотим использовать Doctrine ORM как наш Infrastructure persistence mechanism. Doctrine требует наличия `id` как обычной строковой instance variable, чтобы работать корректно. В нашей Entity `$userId` — это Value Object `UserId`. Добавление дополнительного `id` в нашу Entity `User` только из-за Doctrine связало бы наш механизм persistence с нашей Domain Model. В главе 4, *Entities*, мы видели, что можем решить эту проблему с помощью `Surrogate ID`, создав wrapper вокруг нашей Entity `User` в слое Infrastructure:

```php
class DoctrineUser extends User
{
    private $surrogateUserId;

    public function __construct(UserId $userId, $email, $password)
    {
        parent::__construct($userId, $email, $password);

        $this->surrogateUserId = $userId->id();
    }
}
```

Поскольку создание `DoctrineUser` в нашем `Application Service` снова связало бы слой persistence с нашим Domain, нам нужно отделить логику создания от Service с помощью `Abstract Factory`.

Мы можем сделать это, создав интерфейс в нашем Domain:

```php
interface UserFactory
{
    public function build(UserId $userId, $email, $password);
}
```

Затем мы размещаем его реализацию внутри слоя Infrastructure:

```php
class DoctrineUserFactory implements UserFactory
{
    public function build(UserId $userId, $email, $password)
    {
        return new DoctrineUser($userId, $email, $password);
    }
}
```

После разделения нам остаётся только внедрить Factory в наш `Application Service`:

```php
class SignUpUserService
{
    private $userRepository;
    private $userFactory;

    public function __construct(
        UserRepository $userRepository,
        UserFactory $userFactory
    ) {
        $this->userRepository = $userRepository;
        $this->userFactory = $userFactory;
    }

    /**
     * @param SignUpUserRequest $request
     */
    public function execute($request)
    {
    }
}
```



=====


## Тестирование Factory

Во время написания тестов вы заметите общий паттерн. Это происходит потому, что построение Entity и сложных Aggregate может быть очень утомительным и повторяющимся процессом. Неизбежно сложность и дублирование начнут проникать в ваш набор тестов. Рассмотрим следующую Entity:

```php
class Author
{
    private $username;
    private $email;
    private $fullName;

    public function __construct(
        Username $aUsername,
        FullName $aFullName,
        Email $anEmail
    ) {
        $this->username = $aUsername;
        $this->email = $anEmail;
        $this->fullName = $aFullName;
    }

    // ...
}
```

Где-то в вашей системе у вас появится тест, похожий на этот:

```php
class MyTest extends PHPUnit_Framework_TestCase
{
    /**
     * @test
     */
    public function itDoesSomething()
    {
        $author = new Author(
            new Username('johndoe'),
            new FullName('John', 'Doe'),
            new Email('john@doe.com')
        );

        // do something with author
    }
}
```

Service внутри границ контекста разделяют такие концепции, как Entity, Aggregate и Value Object. Представьте себе беспорядок от повторения одной и той же логики построения снова и снова во всех тестах. Как мы увидим, вынесение логики построения из тестов оказывается полезным и предотвращает дублирование.

### Object Mother

`Object Mother` — это броское название для Factory, которая создаёт фиксированные fixtures для ваших тестов. Аналогично предыдущему примеру, мы могли бы вынести дублирующуюся логику в `Object Mother`, чтобы её можно было переиспользовать в тестах:

```php
class AuthorObjectMother
{
    public static function createOne()
    {
        return new Author(
            new Username('johndoe'),
            new FullName('John', 'Doe'),
            new Email('john@doe.com')
        );
    }
}

class MyTest extends PHPUnit_Framework_TestCase
{
    /**
     * @test
     */
    public function itDoesSomething()
    {
        $author = AuthorObjectMother::createOne();
    }
}
```

Вы заметите, что чем больше тестов и ситуаций у вас появляется, тем больше методов будет у Factory.

Поскольку `Object Mother` не очень гибки, они имеют тенденцию быстро расти в сложности. К счастью, существует более гибкая альтернатива для ваших тестов.

### Test Data Builder

`Test Data Builder` — это просто обычные `Builder` со значениями по умолчанию, используемые исключительно в ваших тестовых наборах, чтобы вам не приходилось указывать несущественные параметры в конкретных тестовых случаях:

```php
class AuthorBuilder
{
    private $username;
    private $email;
    private $fullName;

    private function __construct()
    {
        $this->username = new Username('johndoe');
        $this->email = new Email('john@doe.com');
        $this->fullName = new FullName('John', 'Doe');
    }

    public static function anAuthor()
    {
        return new self();
    }

    public function withFullName(FullName $aFullName)
    {
        $this->fullName = $aFullName;

        return $this;
    }

    public function withUsername(Username $aUsername)
    {
        $this->username = $aUsername;

        return $this;
    }

    public function withEmail(Email $anEmail)
    {
        $this->email = $anEmail;

        return $this;
    }

    public function build()
    {
        return new Author(
            $this->username,
            $this->fullName,
            $this->email
        );
    }
}
```

```php
class MyTest extends PHPUnit_Framework_TestCase
{
    /**
     * @test
     */
    public function itDoesSomething()
    {
        $author = AuthorBuilder::anAuthor()
            ->withEmail(new Email('other@email.com'))
            ->build();
    }
}
```

Мы даже могли бы комбинировать `Test Data Builder` для построения более сложных Aggregate, например `Post`:

```php
class Post
{
    private $id;
    private $author;
    private $body;
    private $createdAt;

    public function __construct(
        PostId $anId,
        Author $anAuthor,
        Body $aBody
    ) {
        $this->id = $anId;
        $this->author = $anAuthor;
        $this->body = $aBody;
        $this->createdAt = new DateTimeImmutable();
    }
}
```

Давайте посмотрим на соответствующий `Test Data Builder` для нашего `Post`. Мы можем переиспользовать `AuthorBuilder` для построения Author по умолчанию:

```php
class PostBuilder
{
    private $postId;
    private $author;
    private $body;

    private function __construct()
    {
        $this->postId = new PostId();
        $this->author = AuthorBuilder::anAuthor()->build();
        $this->body = new Body('Post body');
    }

    public static function aPost()
    {
        return new self();
    }

    public function withAuthor(Author $anAuthor)
    {
        $this->author = $anAuthor;

        return $this;
    }

    public function withPostId(PostId $aPostId)
    {
        $this->postId = $aPostId;

        return $this;
    }

    public function withBody(Body $body)
    {
        $this->body = $body;

        return $this;
    }

    public function build()
    {
        return new Post(
            $this->postId,
            $this->author,
            $this->body
        );
    }
}
```

Это решение теперь достаточно гибкое, чтобы покрыть любой тестовый случай, включая возможность построения внутренних Entity:

```php
class MyTest extends PHPUnit_Framework_TestCase
{
    /**
     * @test
     */
    public function itDoesSomething()
    {
        $post = PostBuilder::aPost()
            ->withAuthor(
                AuthorBuilder::anAuthor()
                    ->withUsername(new Username('other'))
                    ->build()
            )
            ->withBody(new Body('Another body'))
            ->build();

        // do something with the post
    }
}
```

## Итоги

Factory — это мощный инструмент для отделения логики построения от нашей бизнес-логики. Паттерн `Factory Method` не только помогает перенести ответственность за создание в корень Aggregate, но и может обеспечивать соблюдение инвариантов Domain.

Использование паттерна `Abstract Factory` в наших Service позволяет отделить логику Domain от деталей создания Infrastructure. Распространённый сценарий использования — `Specification` и соответствующие им реализации persistence.

Мы увидели, что Factory также очень полезны в наших тестовых наборах. Хотя мы можем выносить логику построения в `Object Mother Factory`, `Test Data Builder` предоставляют большую гибкость для наших тестов.











