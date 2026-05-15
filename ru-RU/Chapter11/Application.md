# Глава 11. Application

Слой Application — это область, которая отделяет Domain Model от клиентов, запрашивающих или изменяющих её состояние. Building blocks для такого слоя являются Application Services. Как говорит Vaughn Vernon: «Application Services are the direct clients of the domain model.» Вы можете думать об Application Service как о точке контакта между внешним миром (HTML-формы, API-клиенты, командная строка, фреймворки, UI и так далее) и самой Domain Model. Может помочь представление о высокоуровневых use case’ах, которые ваша система предоставляет миру, например: «как гость, я хочу зарегистрироваться», «как авторизованный пользователь, я хочу купить продукт» и так далее.

В этой главе мы рассмотрим, как реализовывать Application Services, поймём роль паттерна Command и определим обязанности Application Service. Для этого рассмотрим use case *регистрации нового пользователя*.

Концептуально, чтобы зарегистрировать нового пользователя, нам нужно:

* Получить email и пароль от клиента
* Проверить, используется ли уже этот email
* Создать нового пользователя
* Добавить этого нового пользователя в существующий набор пользователей
* Вернуть только что созданного пользователя

Давайте начнём.

## Requests

Нам нужно передать email и пароль в Application Service. Существует множество способов сделать это со стороны клиента (HTML-форма, API-клиент или даже командная строка). Мы могли бы просто передать стандартные параметры (email и password) через сигнатуру метода или построить и отправить структуру данных с этой информацией. Второй подход — отправка DTO — даёт несколько интересных возможностей. Отправляя объект, мы сможем сериализовать его и помещать в очередь через Command Bus. Также станет возможным добавить type safety и помощь IDE.

### Data Transfer Object
DTO — это структура данных, которая переносит информацию между процессами. Не путайте её с полноценным объектом. DTO не имеет никакого поведения, кроме хранения и получения собственных данных (accessors и mutators). DTO — это простые объекты, которые не должны содержать никакой бизнес-логики, требующей тестирования.


Как говорит Vaughn Vernon:
> Сигнатуры методов Application Service используют только primitive types (int, string и так далее), а также, возможно, DTO. Однако в качестве альтернативы этим подходам лучшим подходом может быть проектирование объектов Command. Не существует обязательно правильного или неправильного способа. В основном это зависит от ваших предпочтений и целей.

Реализация DTO, который хранит данные, необходимые для Application Service, может выглядеть примерно так:

```php
namespace Lw\Application\Service\User;

class SignUpUserRequest
{
    private $email;
    private $password;

    public function __construct($email, $password)
    {
        $this->email = $email;
        $this->password = $password;
    }

    public function email()
    {
        return $this->email;
    }
    
    public function password()
    {
        return $this->password;
    }
}
```

Как видите, SignUpUserRequest не имеет поведения, только данные. Эти данные могли прийти из HTML-формы или API endpoint’а, хотя нам не важно, откуда именно.

## Building Application Service Requests

Создание request’а из delivery mechanism, вашего любимого фреймворка, должно быть довольно прямолинейным. В вебе вы можете взять параметры из controller request и передать их в Service внутри DTO. Тот же принцип применяется и к CLI-команде: прочитать входные параметры и снова передать их вниз.

С Symfony мы можем извлечь нужные данные из объекта Request из компонента HttpFoundation:

```php
// ...

class UsersController extends Controller
{
    /**
     * @Route('/signup', name = 'signup')
     * @param Request $request
     * @return Response
     */
    public function signUpAction(Request $request)
    {
        // ...

        $signUpUserRequest = new SignUpUserRequest(
            $request->get('email'),
            $request->get('password')
        );

        // ...
    }

// ...
```

В более сложном приложении на Silex, использующем компонент Form для получения и валидации параметров, это выглядело бы так:

```php
// ...

$app->match('/signup', function (Request $request) use ($app) {
    $form = $app['sign_up_form'];

    $form->handleRequest($request);

    if ($form->isValid()) {
        $data = $form->getData();

        try {
            $app['sign_in_user_application_service']->execute(
                new SignUpUserRequest(
                    $data['email'],
                    $data['password']
                )
            );

            return $app->redirect(
                $app['url_generator']->generate('login')
            );

        } catch (UserAlreadyExistsException $e) {
            $form
                ->get('email')
                ->addError(
                    new FormError(
                        'Email is already registered by another user'
                    )
                );

        } catch (Exception $e) {
            $form
                ->addError(
                    new FormError(
                        'There was an error, please get in touch with us'
                    )
                );
        }
    }

    return $app['twig']->render('signup.html.twig', [
        'form' => $form->createView(),
    ]);
});
```

## Request Design

При проектировании request objects вы всегда должны следовать следующим принципам: использовать примитивы, проектировать с расчётом на сериализацию и не включать внутрь бизнес-логику. Таким образом, вы сможете сэкономить деньги на unit-тестировании.

### Use Primitives

Мы рекомендуем использовать базовые типы для построения request objects — то есть строки, целые числа, boolean и так далее. Мы всего лишь абстрагируем входные параметры. Вы должны иметь возможность использовать Application Services независимо от delivery mechanism. Даже довольно сложные HTML-формы всё время преобразуются в базовые типы на уровне controller’а. Вы не хотите смешивать ваш фреймворк и бизнес-логику.

В некоторых сценариях возникает соблазн использовать Value Objects напрямую. Не делайте этого. Изменения в определении Value Object затронут всех клиентов, и вы свяжете клиентов с вашей Domain-логикой.

### Serializable

Приятный побочный эффект использования базовых типов состоит в том, что любой request object можно легко сериализовать в строку, передать по сети и сохранить в messaging system или базе данных.

### No Business Logic

Избегайте помещения какой-либо бизнес-логики — даже валидации — внутрь request objects. Валидация должна происходить внутри вашего Domain — то есть внутри Entities, Value Objects, Domain Services и так далее. Валидация — это способ обеспечения business invariants и Domain constraints.

### No Tests

Application requests — это структуры данных, а не объекты. Unit-тестирование структур данных похоже на тестирование getters и setters. Здесь нет поведения для тестирования, поэтому в попытках unit-тестировать request objects и DTO мало ценности. Эти структуры будут покрыты как побочный эффект более сложных тестов, например Integration или Acceptance tests.

Commands являются альтернативой request objects. Мы могли бы спроектировать Service с несколькими Application methods, и каждая из них принимала бы параметры, которые вы бы поместили внутрь Request. Это нормально для простых приложений, но к этой теме мы вернёмся позже.

## Anatomy of an Application Service

Как только у нас есть данные, инкапсулированные в request, приходит время бизнес-логики. Как говорит Vaughn Vernon: «Keep Application Services thin, using them only to coordinate tasks on the model.»

Первое, что нужно сделать — извлечь необходимую информацию из request’а. То есть email и password. На высоком уровне нам нужно проверить, существует ли уже пользователь с определённым email. Если нет, то мы создаём пользователя и добавляем его в UserRepository. В особом случае, когда найден пользователь с тем же email, мы выбрасываем exception, чтобы клиент мог обработать её по-своему — показать ошибку, повторить попытку или просто проигнорировать:

```php
namespace Lw\Application\Service\User;

use Ddd\Application\Service\ApplicationService;
use Lw\Domain\Model\User\User;
use Lw\Domain\Model\User\UserAlreadyExistsException;
use Lw\Domain\Model\User\UserRepository;

class SignUpUserService
{
    private $userRepository;

    public function __construct(UserRepository $userRepository)
    {
        $this->userRepository = $userRepository;
    }

    public function execute(SignUpUserRequest $request)
    {
        $email = $request->email();
        $password = $request->password();

        $user = $this->userRepository->ofEmail($email);

        if ($user) {
            throw new UserAlreadyExistsException();
        }

        $this->userRepository->add(
            new User(
                $this->userRepository->nextIdentity(),
                $email ,
                $password
            )
        );
```

Отлично! Если вам интересно, что делает этот UserRepository в конструкторе, мы покажем это дальше.

### Handling Exceptions
Exceptions, выбрасываемые Application Services, являются способом коммуникации необычных случаев и flow с клиентом. Exceptions на этом слое связаны с бизнес-логикой (например, пользователь не найден), а не с деталями реализации (например, PDOException, PredisException или DoctrineException).


## Dependency Inversion

Работа с пользователями — не обязанность Service. Как мы видели в главе 10, Repositories, существует специализированный класс, работающий с коллекциями User: User Repository. Это зависимость от Application Service к Repository. Мы не хотим связывать Application Service с конкретной реализацией Repository, потому что тогда мы свяжем наш Service с деталями Infrastructure. Поэтому мы зависим от контракта (interface), от которого зависят конкретные реализации — UserRepository.

Конкретная реализация UserRepository будет создана и передана во время выполнения — например, DoctrineUserRepository, конкретная реализация, использующая Doctrine. Передача конкретной реализации также будет работать при тестировании. Например, NotAvailableUserRepository может быть конкретной реализацией, выбрасывающей exception каждый раз при выполнении операции. Таким образом, мы можем протестировать все поведения Application Service, включая sad paths — ситуации, когда приложение должно вести себя корректно, даже если что-то пошло не так.

Application Services также могут зависеть от Domain Services, например GetBadgesByUser. Во время выполнения реализация такого Service может быть довольно сложной. Представьте HttpGetBadgesByUser для интеграции Bounded Context через HTTP-протокол.

Завися от абстракций, мы делаем наш Application Service невосприимчивым к низкоуровневым изменениям Infrastructure.


### Instantiating Application Services

Создать сам Application Service просто, но построение дерева зависимостей может быть сложным — в зависимости от того, насколько сложны зависимости. Для этой цели большинство фреймворков поставляются с Dependency Injection Container. Без него вы получите что-то вроде следующего кода где-нибудь внутри controller’а:

```php
$redisClient = new Predis\Client([
    'scheme' => 'tcp',
    'host' => '10.0.0.1',
    'port' => 6379
]);

$userRepository = new RedisUserRepository($redisClient);

$signUp = new SignUpUserService($userRepository);

$signUp->execute(new SignUpUserRequest(
    'user@example.com',
    'password'
));
```

Мы решили использовать Redis-реализацию UserRepository. В предыдущем примере кода мы создали все зависимости, необходимые для построения Repository, который внутри использует Redis. Этими зависимостями являются: Predis client и все параметры подключения к нашему Redis server. Это не только неэффективно, но и распространяет дублирование по controller’ам.

Вы можете вынести логику создания в Factory или использовать Dependency Injection Container — большинство современных фреймворков поставляются с ним.

### Is It Bad to Use a Dependency Injection Container?
Вовсе нет. Dependency Injection Containers — это всего лишь инструмент. Они помогают, абстрагируя сложность построения зависимостей. Они очень полезны для построения Infrastructure artifacts. Symfony предлагает полноценное решение.

Учитывайте тот факт, что передавать весь container целиком в один из Services — плохая практика. Это было бы похоже на связывание всего контекста вашего приложения с Domain. Если Service нужны определённые объекты, создайте их с помощью вашего фреймворка и передайте как зависимости в Service, но не делайте Service aware всего контекста.

Давайте посмотрим, как строятся зависимости в Silex:

```php
$app = new \Silex\Application();

$app['redis_parameters'] = [
     'scheme' => 'tcp',
     'host' => '127.0.0.1',
     'port' => 6379
];

$app['redis'] = $app->share(function ($app) {
    return new Predis\Client($app['redis_parameters']);
});

$app['user_repository'] = $app->share(function($app) {
    return new RedisUserRepository(
        $app['redis']
    );
});

$app['sign_up_user_application_service'] = $app->share(function($app) {
    return new SignUpUserService(
        $app['user_repository']
    );
});

// ...

$app->match('/signup' ,function (Request $request) use ($app) {
    // ...

    $app['sign_up_user_application_service']->execute(
        new SignUpUserRequest(
            $request->get('email'),
            $request->get('password')
        )
    );

    // ...
});
```

Как видите, $app используется как Service Container. Мы регистрируем все необходимые компоненты вместе с их зависимостями. sign_up_user_application_service зависит от определений, созданных выше. Изменить реализацию user_repository так же просто, как вернуть что-то другое (MySQL, MongoDB и так далее), поэтому нам вообще не нужно менять код Service.

Эквивалент для Symfony-приложения выглядит так:

```xml
<?xml version=" 1.0" ?>

<container xmlns="http://symfony.com/schema/dic/services"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="
        http://symfony.com/schema/dic/services
        http://symfony.com/schema/dic/services/services-1.0.xsd">

    <services>

        <service
            id="sign_up_user_application_service"
            class="SignUpUserService">

            <argument type="service" id="user_repository" />
        </service>

        <service
            id="user_repository"
            class="RedisUserRepository">

            <argument type="service">
                <service class="Predis\Client" />
            </argument>

        </service>

    </services>

</container>
```

Теперь, когда у вас есть определение Application Service в Symfony Service Container, получить его позже довольно просто. Все delivery mechanisms — Web Controllers, REST Controllers и даже Console Commands — используют одно и то же определение. Service доступен в любом классе, реализующем интерфейс ContainerAware. Получить Service так же просто, как вызвать `$this->get('sign_up_user_application_service')`.

Подводя итог: не важно, как именно вы строите ваши Services (adhoc, с использованием Service Containers, Factory и так далее). Однако важно держать настройку Application Services вне границы Infrastructure.

## Customize an Application Service

Основной способ кастомизации Application Service — выбор зависимостей, которые вы передаёте внутрь. В зависимости от возможностей вашего Service Container это может быть немного сложно, поэтому вы также можете добавить setter для изменения зависимости на лету. Например, вам может понадобиться изменить output dependency, чтобы сначала установить значение по умолчанию, а затем заменить его позже. Если логика становится слишком сложной, вы можете создать Application Service Factory, которая будет обрабатывать эту ситуацию.

## Execution

Существует два различных подхода к вызову Application Services: выделенный класс на каждый use case с одним execution method и несколько Application Services и use case’ов внутри одного класса.

### One Class Per Application Service

Это наш предпочтительный подход и, вероятно, тот, который подходит для всех сценариев:

```php
class SignUpUserService
{
    // ...

    public function execute(SignUpUserRequest $request)
    {
        // ...
    }
}
```

Использование выделенного класса для каждого Application Service делает код более устойчивым к внешним изменениям (Single Responsibility Principle). У класса меньше причин для изменения, так как Service делает одну и только одну вещь. Такой Application Service будет легче тестировать, поскольку он делает меньше вещей. Также проще реализовать общий контракт Application Service, что облегчает декорирование классов (посмотрите подраздел Transactions главы 10, Repositories). Это также приведёт к большей cohesion, так как все зависимости исключительно посвящены одному use case.

Execution method может иметь более выразительное имя, например signUp. Однако формат execute паттерна Command стандартизирует общий контракт между Application Services, тем самым позволяя легко использовать декорирование, что особенно полезно для транзакций.

### Multiple Application Service Methods per Class

Иногда может быть хорошей идеей группировать cohesive Application Services внутри одного класса:

```php
class UserService
{
    // ...

    public function signUp(SignUpUserRequest $request)
    {
        // ...
    }
```

```php
    public function signIn(SignUpUserRequest $request)
    {
        // ...
    }

    public function logOut(LogOutUserRequest $request)
    {
        // ...
    }
}
```

Мы не рекомендуем такой подход, потому что не все Application Services являются на 100 процентов cohesive. Некоторым Services потребуются разные зависимости, и в итоге вы получите Application Services, зависящие от того, что им не нужно. Другая проблема состоит в том, что такие классы быстро растут. Поскольку это нарушает Single Responsibility Principle, появится множество причин для изменения и, возможно, даже поломки такого класса.

## Returning Values

После регистрации пользователя мы можем подумать о перенаправлении пользователя на страницу профиля. Естественный способ передать необходимую информацию обратно в controller — вернуть User Entity напрямую из Service:

```php
class SignUpUserService
{
    // ...

    public function execute(SignUpUserRequest $request)
    {
        $user = new User(
            $this->userRepository->nextIdentity(),
            $email,
            $password
        );

        $this->userRepository->add($user);

        return $user;
    }
}
```

Затем из controller’а мы бы получили поле id и перенаправили пользователя в другое место. Однако подумайте ещё раз о том, что мы только что сделали. Мы вернули полноценную Entity в controller, что позволит delivery mechanism обойти Application Layer и напрямую взаимодействовать с Domain.

Представьте, что User Entity предоставляет метод updateEmailAddress. Вы можете попытаться предотвратить это, но в какой-то момент в будущем кто-нибудь может решить использовать его:

```php
$app-> match( '/signup' , function (Request $request) use ($app) {
   // ...

   $user = $app['sign_up_user_application_service']->execute(
       new SignUpUserRequest(
           $request->get('email'),
           $request->get('password'))
   );

   $user->updateEmailAddress('shouldnotupdate@email.com');

   // ...
});
```

Более того, данные, которые нужны presentation layer, отличаются от тех, которыми управляет Domain. Мы не хотим развивать и связывать Domain layer вокруг presentation layer. Вместо этого мы хотим, чтобы они развивались свободно.

Чтобы добиться этого, нам нужен гибкий способ развязать оба слоя.


## DTO из экземпляров Aggregate

Мы могли бы возвращать стерильные структуры данных с информацией, необходимой слою представления. Как мы уже видели ранее, DTO хорошо подходят для этого сценария. Нам всего лишь нужно собрать их в Application Service и вернуть клиенту:

```php
class UserDTO
{
    private $email ;

    // ...

    public function __construct(User $user)
    {
        $this->email = $user->email ();

        // ...
    }

    public function email ()
    {
        return $this->email ;
    }
}
```

`UserDTO` будет предоставлять любые данные только для чтения, которые нам нужны из Entity `User` на слое представления, тем самым избегая раскрытия поведения:

```php
class SignUpUserService
{
    public function execute(SignUpUserRequest $request)
    {
        // ...

        $user = // ...

        return new UserDTO($user);
    }
}
```

Миссия выполнена. Теперь мы могли бы передавать параметры в шаблонизатор и преобразовывать их в виджеты, теги или подшаблоны, либо делать с данными всё, что захотим, на стороне представления:

```php
$app->match('/signup' , function (Request $request) use ($app) {
    /**
     * @var UserDTO $user
     */
    $userDto=$app['sign_up_user_application_service']->execute(
        new SignUpUserRequest(
            $request->get('email'),
            $request->get('password')
        )
    );

    // ...
});
```

Однако, если позволить Application Service решать, как строить DTO, это выявляет ещё одно ограничение. Так как построение DTO полностью зависит от Application Service, адаптировать DTO под разных клиентов будет очень сложно. Рассмотрим данные, необходимые для редиректа в Web Controller, и данные, необходимые для REST-ответа для того же самого use case. Это вовсе не одни и те же данные.

Давайте позволим клиенту определять, как строить DTO, передавая специальный DTO Assembler:

```php
class SignUpUserService
{
    private $userDtoAssembler;

    public function __construct(
        UserRepository $userRepository,
        UserDTOAssembler $userDtoAssembler
    ){
        $this->userRepository = $userRepository;
        $this->userDtoAssembler = $userDtoAssembler;
    }

    public function execute(SignUpUserRequest $request)
    {
        $user = // ...

        return $this->userDtoAssembler->assemble($user);
    }
}
```

Теперь клиент может настраивать ответ, передавая конкретный `UserDTOAssembler`.

## Data Transformers

Существуют случаи, когда создание промежуточных DTO для более сложных ответов, таких как JSON, XML, CSV и iCAL Contact, может рассматриваться как ненужные накладные расходы. Мы могли бы выводить представление в буфер и запрашивать его позднее на стороне delivery layer.

Data Transformers помогают сократить эти накладные расходы, преобразуя высокоуровневые Domain concepts в низкоуровневые клиентские детали. Давайте рассмотрим пример:

```php
interface UserDataTransformer
{
    public function write(User $user);

    /**
     * @return mixed
     */
    public function read();
}
```

Рассмотрим случай генерации различных представлений данных для заданного продукта. Обычно информация о продукте предоставляется через web-интерфейс (HTML), но мы можем быть заинтересованы в предоставлении других форматов, таких как XML, JSON или CSV. Это может позволить интеграции с другими Services.

Рассмотрим аналогичный случай для блога. Мы можем раскрывать свой потенциал как писателей миру через HTML, но некоторые люди будут заинтересованы в потреблении наших статей через RSS. Use cases — Application Services — остаются теми же самыми. Представление — нет.

DTO являются чистым и простым решением, которое можно передавать в шаблонизаторы для различных представлений, но это может усложнить логику этого последнего шага трансформации данных, поскольку логика таких шаблонов может стать проблемой с точки зрения поддержки, тестирования и понимания.

Data Transformers данных могут быть лучшим подходом в определённых случаях. Это просто чёрные ящики с Domain concepts (Aggregates, Entities и так далее) на входе и представлениями только для чтения (XML, JSON, CSV и так далее) на выходе. Эти transformers могут быть очень простыми для тестирования:

```php
class JsonUserDataTransformer implements UserDataTransformer
{
    private $data;

    public function write(User $user)
    {
        // Здесь может быть размещена более сложная логика
        // Например использование JMSSerializer, native json и т.д.
        $this->data = json_encode($user);
    }

    /**
     * @return string
     */
    public function read()
    {
        return $this->data;
    }
}
```


Это было просто. Интересно, как выглядела бы XML- или CSV-версия? Давайте посмотрим, как интегрировать Data Transformer с нашим Application Service:

```php
class SignUpUserService
{
    private $userRepository;
    private $userDataTransformer;

    public function __construct(
        UserRepository $userRepository,
        UserDataTransformer $userDataTransformer
    ){
        $this->userRepository = $userRepository;
        $this->userDataTransformer = $userDataTransformer;
    }

    public function execute(SignUpUserRequest $request)
    {
        $user = // ...

        $this->userDataTransformer()->write($user);
    }

    /**
     * @return UserDataTransformer
     */
    public function userDataTransformer()
    {
        return $this->userDataTransformer;
    }
}
```

Это похоже на подход с DTO Assembler, но на этот раз без возврата конкретного значения. Data Transformer используется для хранения данных и взаимодействия с ними.

Основная проблема DTO заключается в накладных расходах на их написание. В большинстве случаев ваши Domain concepts и DTO-представления будут иметь одинаковую структуру. В большинстве случаев вам будет казаться, что подобное отображение не стоит затраченного времени. Тем не менее, отношение между представлениями и Aggregates не является 1:1. Вы можете представлять два Aggregate вместе в одном представлении. Вы также можете представлять один и тот же Aggregate несколькими способами. То, как вы это делаете, всегда зависит от ваших use cases.

Однако, согласно Martin Fowler:

> Один из случаев, когда полезно использовать что-то вроде DTO, — это когда существует значительное несоответствие между моделью в вашем presentation layer и лежащей в основе domain model. В этом случае имеет смысл создать facade/gateway, специфичный для представления, который отображает domain model и предоставляет интерфейс, удобный для presentation layer. Это хорошо сочетается с Presentation Model. Это стоит делать, но только для экранов, где такое несоответствие действительно существует (в этом случае это не дополнительная работа, поскольку вам всё равно пришлось бы делать это на уровне экрана).

Мы считаем, что долгосрочное видение оправдает вложения. В средних и крупных проектах представления интерфейса и Domain concepts изменяются с совершенно разной скоростью. Возможно, вы захотите развязать их друг от друга, чтобы уменьшить трение при обновлениях. Использование DTO или Data Transformers позволяет свободно развивать вашу модель без необходимости постоянно думать о том, что вы сломаете layout.

## Несколько Application Services в составных layout

В большинстве случаев ни один layout не ограничивается одним Application Service. Наши проекты имеют довольно сложные интерфейсы.

Рассмотрим homepage конкретного проекта. Как мы можем отрисовать такое количество частей и use cases? Есть несколько вариантов, так что давайте их рассмотрим.

## Интеграция контента через AJAX

Вы можете позволить браузеру напрямую обращаться к различным endpoints и затем объединять данные в layout через AJAX или Hijax. Это позволит избежать смешивания большого количества Application Services в контроллерах, но может привести к потере производительности в зависимости от количества выполняемых запросов.

## Интеграция контента через ESI

Edge Side Includes (ESI) — это небольшой язык разметки, похожий на предыдущий подход, но работающий на стороне сервера. Он требует дополнительных усилий по настройке промежуточного слоя, такого как NGINX или Varnish, чтобы всё заработало. Includes (ESI) — это небольшой язык разметки, похожий на предыдущий подход, но работающий на стороне сервера. Он требует дополнительных усилий по настройке промежуточного слоя, такого как NGINX или Varnish, чтобы всё заработало.

## Symfony Sub Requests

Если вы используете Symfony, Sub Requests могут быть интересным вариантом. Согласно документации Symfony:

> В дополнение к основному request, который передаётся в `HttpKernel::handle`, вы также можете отправлять так называемые sub request. Sub request выглядит и работает как любой другой request, но обычно служит для рендеринга лишь небольшой части страницы, а не всей страницы целиком. Чаще всего вы будете выполнять sub-request из вашего controller (или, возможно, из шаблона, который рендерится вашим controller). Это создаёт ещё один полный цикл request-response, где новый Request преобразуется в Response. Единственное внутреннее отличие заключается в том, что некоторые listeners (например: security) могут реагировать только на master request. Каждый listener получает некоторый подкласс `KernelEvent`, чей метод `isMasterRequest()` можно использовать для проверки того, является ли текущий request master request или sub request.

Это замечательно, поскольку вы получаете преимущества вызова отдельных Application Services без штрафов AJAX или сложных конфигураций ESI.

## Один Controller, несколько Application Services

Последним вариантом может быть управление несколькими Application Services внутри одного controller, хотя логика controller может стать немного грязной, поскольку ему придётся обрабатывать и объединять ответы перед передачей их во view.




## Тестирование Application Services

Так как вас интересует тестирование поведения самого Application Service, нет необходимости превращать это в integration test со сложной настройкой и работой против реальной базы данных. Вас не интересует тестирование низкоуровневых деталей, поэтому в большинстве случаев unit test будет вполне достаточным:

```php
class SignUpUserServiceTest extends \PHPUnit_Framework_TestCase
{
    /**
     * @var \Lw\Domain\Model\User\UserRepository
     */
    private $userRepository;

    /**
     * @var SignUpUserService
     */
    private $signUpUserService;

    public function setUp()
    {
        $this->userRepository = new InMemoryUserRepository();

        $this->signUpUserService = new SignUpUserService(
            $this->userRepository
        );
    }

    /**
     * @test
     * @expectedException
     *     \Lw\Domain\Model\User\UserAlreadyExistsException
     */
    public function alreadyExistingEmailShouldThrowAnException()
    {
        $this->executeSignUp();
        $this->executeSignUp();
    }

    private function executeSignUp()
    {
        return $this->signUpUserService->execute(
            new SignUpUserRequest(
                'user@example.com',
                'password'
            )
        );
    }

    /**
     * @test
     */
    public function afterUserSignUpItShouldBeInTheRepository()
    {
        $user = $this->executeSignUp();

        $this->assertSame(
            $user,
            $this->userRepository->ofId($user->id())
        );
    }
}
```

Мы использовали in-memory implementation для User Repository. Это то, что называется Fake: полностью функциональная реализация Repository, которая позволит нашему тесту работать как unit test. Нам не нужно обращаться к базе данных, чтобы протестировать поведение этого класса. Это сделало бы наш тест медленным и хрупким.

Проверка отправки Domain Events также может быть интересной. Если создание пользователя инициирует событие user registered, будет хорошей идеей убедиться, что оно действительно было вызвано:

```php
class SignUpUserServiceTest extends \PHPUnit_Framework_TestCase
{
    // ...

    /**
     * @test
     */
    public function itShouldPublishUserRegisteredEvent()
    {
        $subscriber = new SpySubscriber();

        $id = DomainEventPublisher::instance()->subscribe($subscriber);

        $user = $this->executeSignUp();
        $userId = $user->id();

        DomainEventPublisher::instance()->unsubscribe($id);

        $this->assertUserRegisteredEventPublished(
            $subscriber,
            $userId
        );
    }

    private function assertUserRegisteredEventPublished(
        $subscriber,
        $userId
    ){
        $this->assertInstanceOf(
            'UserRegistered',
            $subscriber->domainEvent
        );

        $this->assertTrue(
            $subscriber->domainEvent->userId()->equals($userId)
        );
    }
}

class SpySubscriber implements DomainEventSubscriber
{
    public $domainEvent;

    public function handle($aDomainEvent)
    {
        $this->domainEvent = $aDomainEvent;
    }

    public function isSubscribedTo($aDomainEvent)
    {
        return true;
    }
}
```

## Транзакции

Транзакции — это implementation detail, связанная с механизмом persistence. Domain layer не должен знать об этой низкоуровневой детали реализации. Размышления о начале, коммите или rollback транзакции на этом уровне — это серьёзный smell. Такой уровень детализации принадлежит Infrastructure layer.

Лучший способ работать с транзакциями — вообще ими не заниматься. Мы могли бы обернуть наши Application Services в реализацию Decorator для автоматического управления transactional session.

Мы реализовали решение этой проблемы в одном из наших repositories, и вы можете посмотреть его здесь:

```php
interface TransactionalSession
{
    /**
     * @return mixed
     */
    public function executeAtomically(callable $operation);
}
```

Этот контракт принимает некоторый фрагмент кода и выполняет его атомарно. В зависимости от вашего механизма persistence вы получите разные реализации.

Давайте посмотрим, как это можно сделать с помощью Doctrine ORM:

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

Вот как клиент использовал бы предыдущий код:

```php
/** @var EntityManager $em */

$nonTxApplicationService = new SignUpUserService(
    $em->getRepository('BoundedContext\Domain\Model\User\User')
);

$txApplicationService = new TransactionalApplicationService(
    $nonTxApplicationService,
    new DoctrineSession($em)
);

$response = $txApplicationService->execute(
    new SignUpUserRequest(
        'user@example.com',
        'password'
    )
);
```

Теперь, когда у нас есть Doctrine implementation для transactional sessions, было бы здорово создать Decorator для наших Application Services. С таким подходом мы делаем transactional requests прозрачными для Domain:

```php
class TransactionalApplicationService implements ApplicationService
{
    private $session;
    private $service;

    public function __construct(
        ApplicationService $service,
        TransactionalSession $session
    ){
        $this->session = $session;
        $this->service = $service;
    }

    public function execute(BaseRequest $request)
    {
        $operation = function () use ($request) {
            return $this->service->execute($request);
        };

        return $this->session->executeAtomically($operation);
    }
}
```

Приятным побочным эффектом использования Doctrine Session является то, что она автоматически управляет методом `flush`, поэтому вам не нужно добавлять `flush` внутрь вашего Domain или Infrastructure.

## Безопасность

Если вам интересно, как управлять пользовательскими credentials и безопасностью в целом, то, если это не ответственность вашего Domain, мы рекомендуем позволить framework заниматься этим. Пользовательская session — это concern delivery mechanism. Загрязнение Domain подобными концепциями сделает разработку сложнее.

## Domain Events

Слушатели Domain Event должны быть сконфигурированы до выполнения Application Service, иначе никто не будет уведомлён. Существуют ситуации, когда вам придётся явно сконфигурировать listener перед выполнением Application Service:

```php
// ...

$subscriber = new SpySubscriber();

DomainEventPublisher::instance()->subscribe($subscriber);

$applicationService = // ...

$applicationService->execute(...);
```

В большинстве случаев это будет делаться через конфигурацию Dependency Injection Container.

## Command Handlers

Интересный способ выполнения Application Services — использование библиотеки Command Bus. Хороший вариант — Tactician. С сайта Tactician:

> Что такое Command Bus? Этот термин чаще всего используется, когда мы объединяем Command pattern с service layer. Его задача — принять объект Command (который описывает, что пользователь хочет сделать) и сопоставить его с Handler (который это выполняет). Это может помочь аккуратно структурировать ваш код.

— наши Application Services являются Service Layer, а наши Request objects очень похожи на Commands.

Справедливо — наши Application Services являются Service Layer, а наши Request objects действительно очень похожи на Commands. Было бы здорово, если бы у нас был механизм, связывающий все Application Services, а затем, основываясь на Request, выполняющий нужный? Что ж, именно этим и является Command Bus.

## Tactician Library and Other Options

Tactician — это библиотека Command Bus, которая позволяет использовать Command pattern для ваших Application Services. Она особенно удобна для Application Services, хотя вы можете использовать любой вид входных данных.

Давайте посмотрим пример с сайта Tactician:

```php
// Вы создаёте простой message object вроде этого:

class PurchaseProductCommand
{
    protected $productId;
    protected $userId;

    // ...и constructor для присваивания этих свойств...
}

// И Handler class, который его ожидает:

class PurchaseProductHandler
{
    public function handle(PurchaseProductCommand $command)
    {
        // использовать command для обновления моделей и т.д.
    }
}

// А затем в ваших Controllers вы можете заполнить command,
// используя вашу любимую form или serializer library,
// после чего передать его в CommandBus — и готово!

$command = new PurchaseProductCommand(42, 29);

$commandBus->handle($command);
```

Вот и всё. Tactician — это Service `$commandBus`. Он выполняет всю работу по поиску правильного handler и метода, что позволяет избежать большого количества boilerplate code. Здесь Commands и Handlers — это просто обычные классы, но вы можете сконфигурировать всё так, как лучше подходит вашему приложению.

Подводя итог, можно сделать вывод, что Commands — это просто Request objects, а Command Handlers — это просто Application Services.

Классная особенность Tactician (и Command Bus в целом) заключается в том, что их очень легко расширять. Tactician предоставляет plug-ins для распространённых задач, таких как logging и database transactions. Таким образом, вы можете забыть о настройке wiring для каждого handler.

Ещё один интересный plug-in для Tactician — интеграция с Bernard. Bernard — это asynchronous job queue, которая позволяет откладывать некоторые задачи для последующей обработки. Тяжёлые процессы блокируют response. В большинстве случаев мы можем отделить их и отложить выполнение на потом. Для лучшего пользовательского опыта отвечайте клиенту как можно быстрее и уведомляйте его после завершения отложенных процессов.

Matthias Noback разработал другой похожий проект под названием SimpleBus, который можно использовать как альтернативу Tactician. Основное различие состоит в том, что SimpleBus Command Handlers не имеют возвращаемого значения.

## Итоги

Application Services представляют собой Application layer вашего Bounded Context. Эти высокоуровневые use cases должны быть относительно простыми и тонкими, так как их назначение заключается в координации Domain. Application Services являются точкой входа для взаимодействия с Domain logic. Мы увидели, что Requests и Commands помогают поддерживать порядок; что DTOs и Data Transformers позволяют развязать представление данных и Domain conceptualization; что построение Application Services довольно просто с помощью Dependency Injection Containers; и что у нас есть множество вариантов для комбинирования Application Services в сложных layout.


