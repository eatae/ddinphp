# Сервисы (Services)

Вы уже познакомились с тем, что такое **Сущности (Entities)** и **Объекты-значения (Value Objects)**. Как базовые строительные блоки системы, именно они должны содержать большую часть бизнес-логики любого приложения. Однако существуют сценарии, в которых Сущности и Объекты-значения оказываются не самым подходящим решением. Давайте посмотрим, что Эрик Эванс пишет об этом в своей книге *Domain-Driven Design: Tackling Complexity in the Heart of Software*:

> Когда значимый процесс или преобразование в предметной области не является естественной обязанностью Сущности или Объекта-значения, добавьте операцию в модель как отдельный интерфейс, объявленный в виде Сервиса (Service). Определите интерфейс на языке модели и убедитесь, что имя операции является частью Единого языка (Ubiquitous Language). Сделайте Сервис не имеющим состояния (stateless).

Иными словами, когда в системе существуют операции, которые необходимо выразить в модели, но при этом Сущности и Объекты-значения не являются подходящим местом для их размещения, стоит рассмотреть моделирование этих операций в виде Сервисов.

В Domain-Driven Design обычно выделяют три различных типа сервисов:

* **Application Services (Прикладные сервисы)**
  Работают со скалярными типами и преобразуют их в доменные типы. Под скалярными типами понимаются любые типы, неизвестные доменной модели. Сюда относятся как примитивные типы, так и типы, не принадлежащие домену. В этой главе будет дан обзор темы, а более подробно она рассматривается в главе 11 — *Application*.

* **Domain Services (Доменные сервисы)**
  Работают исключительно с типами, принадлежащими предметной области. Они содержат значимые концепции, присутствующие в Едином языке. В них размещаются операции, которые плохо вписываются в Объекты-значения или Сущности.

* **Infrastructure Services (Инфраструктурные сервисы)**
  Представляют операции, связанные с инфраструктурными задачами: отправка email, логирование важной информации и т.д. С точки зрения Hexagonal Architecture, они находятся за пределами доменной границы.

---

## Application Services

Application Services — это промежуточный слой между внешним миром и доменной логикой. Назначение такого механизма — преобразовывать команды, приходящие извне, в осмысленные инструкции для домена.

Рассмотрим use-case: пользователь регистрируется на платформе.

Начнём с подхода outside-in: начиная с механизма доставки (delivery mechanism), нам необходимо подготовить входной запрос для доменной операции. Если в качестве delivery mechanism используется Symfony, код может выглядеть примерно так:

```php
class SignUpController extends Controller
{
    public function signUpAction(Request $request)
    {
        $signUpService = new SignUpUserService(
            $this->get('user_repository')
        );

        try {
            $response = $signUpService->execute(
                new SignUpUserRequest(
                    $request->request->get('email'),
                    $request->request->get('password')
                )
            );
        } catch (UserAlreadyExistsException $e) {
            return $this->render('error.html.twig', $response);
        }

        return $this->render('success.html.twig', $response);
    }
}
```

Как можно заметить, мы создаём экземпляр Application Service и передаём ему все необходимые зависимости — в данном случае `UserRepository`.

`UserRepository` — это интерфейс, который может быть реализован с использованием любой конкретной технологии: MySQL, Redis, Elasticsearch и т.д.

Затем создаётся объект запроса (`SignUpUserRequest`) для Application Service. Это делается для того, чтобы абстрагировать delivery mechanism (в данном случае HTTP-запрос) от бизнес-логики.

Наконец:

1. выполняется Application Service;
2. получается ответ;
3. ответ используется для рендеринга результата.

Со стороны домена возможная реализация Application Service, координирующего логику use-case «регистрация пользователя», может выглядеть следующим образом:

```php
class SignUpUserService
{
    private $userRepository;

    public function __construct(UserRepository $userRepository)
    {
        $this->userRepository = $userRepository;
    }

    public function execute(SignUpUserRequest $request)
    {
        $user = $this->userRepository->userOfEmail($request->email);

        if ($user) {
            throw new UserAlreadyExistsException();
        }

        $user = new User(
            $this->userRepository->nextIdentity(),
            $request->email,
            $request->password
        );

        $this->userRepository->add($user);

        return new SignUpUserResponse($user);
    }
}
```

Весь код здесь посвящён решению доменной задачи, а не специфике используемой технологии. Благодаря такому подходу мы можем отделить высокоуровневые правила (high-level policies) от низкоуровневых деталей реализации.

Коммуникация между delivery mechanism и доменом осуществляется с помощью структур данных, называемых DTO (Data Transfer Objects), которые были представлены ранее в главе 2 — *Architectural Styles*:

```php
class SignUpUserRequest
{
    public $email;
    public $password;

    public function __construct($email, $password)
    {
        $this->email = $email;
        $this->password = $password;
    }
}
```

Существуют разные стратегии возврата данных, однако пока важно понимать следующее: не стоит возвращать наружу сами Entity, чтобы они не могли быть изменены за пределами Application Services.

Именно поэтому распространённой практикой является возврат другого DTO, содержащего только необходимую информацию, а не всей Entity целиком:

```php
class SignUpUserResponse
{
    public $id;
    public $email;

    public function __construct(User $user)
    {
        $this->id = $user->id();
        $this->email = $user->email();
    }
}
```

При создании response-объектов можно использовать как getter-методы, так и публичные свойства.

Application Services также должны учитывать:

* границы транзакций;
* безопасность;
* другие инфраструктурные аспекты.

Более подробно эти темы рассматриваются в главе 11 — *Application*.

---

## Domain Services

Во время общения с экспертами предметной области (Domain Experts) вы будете сталкиваться с концепциями Единого языка, которые невозможно аккуратно представить ни как Сущность, ни как Объект-значение.

Например:

* возможность пользователя самостоятельно войти в систему;
* превращение корзины в заказ.

Это реальные доменные концепции, но ни одна из них естественным образом не принадлежит Entity или Value Object.

Попробуем смоделировать это поведение следующим образом:

```php
class User
{
    public function signUp($username, $password)
    {
        // ...
    }
}
```

```php
class Cart
{
    public function createOrder()
    {
        // ...
    }
}
```

В случае первой реализации невозможно понять, относятся ли переданные username и password к экземпляру пользователя, на котором вызван метод. Очевидно, что такая операция плохо подходит данной Entity. Вместо этого её следует вынести в отдельный класс, явно выражающий намерение.

С учётом этого можно создать Domain Service, единственной ответственностью которого будет аутентификация пользователей:

```php
class SignUp
{
    public function execute($username, $password)
    {
        // ...
    }
}
```

Аналогично во втором примере можно создать Domain Service, специализирующийся на создании заказа из корзины:

```php
class CreateOrderFromCart
{
    public function execute(Cart $cart)
    {
        // ...
    }
}
```

Domain Service можно определить как операцию, выполняющую доменную задачу и естественным образом не вписывающуюся ни в Entity, ни в Value Object.

Поскольку Domain Services представляют операции предметной области:

* они должны использоваться клиентами независимо от истории выполнения;
* они не должны хранить собственное состояние;
* они являются stateless-операциями.

---

## Domain Services и Infrastructure Services

При моделировании Domain Service часто возникают инфраструктурные зависимости. Например, в случае механизма аутентификации может понадобиться логика хеширования паролей.

В такой ситуации можно использовать паттерн **Separated Interface**, позволяющий определить несколько механизмов хеширования.

Это обеспечивает чёткое разделение ответственности между доменом и инфраструктурой.

Интерфейс в домене:

```php
namespace Ddd\Auth\Domain\Model;

interface SignUp
{
    public function execute($username, $password);
}
```

Реализация в инфраструктурном слое:

```php
namespace Ddd\Auth\Infrastructure\Authentication;

class DefaultHashingSignUp implements Ddd\Auth\Domain\Model\SignUp
{
    private $userRepository;

    public function __construct(UserRepository $userRepository)
    {
        $this->userRepository = $userRepository;
    }

    public function execute($username, $password)
    {
        if (!$this->userRepository->has($username)) {
            throw UserDoesNotExistException::fromUsername($username);
        }

        $user = $this->userRepository->byUsername($username);

        if (!$this->isPasswordValidForUser($user, $password)) {
            throw new BadCredentialsException($user, $password);
        }

        return $user;
    }

    private function isPasswordValidForUser(
        User $user,
        $plainPassword
    ) {
        return password_verify($plainPassword, $user->hash());
    }
}
```

Другой вариант — реализация на основе MD5:

```php
class Md5HashingSignUp implements SignUp
{
    const SALT = 'S0m3S4lT';

    // ...
}
```

Выбор такого подхода позволяет иметь несколько реализаций интерфейса Domain Service на инфраструктурном уровне.

Иными словами:

* появляется несколько Infrastructure Domain Services;
* каждая реализация отвечает за собственный механизм хеширования;
* конкретная реализация может легко переключаться через Dependency Injection Container.

Например, в Symfony DI Container это может выглядеть так:

```xml
<service id="sign_in" alias="sign_in.default" />

<service id="sign_in.default"
    class="Ddd\Auth\Infrastructure\Authentication\DefaultHashingSignUp">
    <argument type="service" id="user_repository"/>
</service>

<service id="sign_in.md5"
    class="Ddd\Auth\Infrastructure\Authentication\Md5HashingSignUp">
    <argument type="service" id="user_repository"/>
</service>
```

Если в будущем понадобится новый алгоритм хеширования:

1. достаточно реализовать интерфейс Domain Service;
2. зарегистрировать новый сервис в DI-контейнере;
3. заменить alias.

---

## Проблема переиспользования кода

Хотя описанная реализация хорошо разделяет ответственности, возникает проблема: алгоритм проверки пароля приходится дублировать в каждой реализации.

Альтернативное решение — вынести логику хеширования в специализированный класс, используя паттерн Strategy:

```php
interface PasswordHashing
{
    public function verify($plainPassword, $hash);
}
```

Теперь разные стратегии хеширования реализуются отдельно:

```php
class BasicPasswordHashing implements PasswordHashing
{
}
```

```php
class Md5PasswordHashing implements PasswordHashing
{
}
```

Такой подход:

* улучшает переиспользование кода;
* делает систему открытой для расширения;
* закрывает её для модификации (Open/Closed Principle).

---

## Тестирование Domain Services

На примере аутентификации особенно полезно иметь возможность легко тестировать Domain Service.

Для тестов можно использовать простую реализацию:

```php
class PlainPasswordHashing implements PasswordHashing
{
    public function verify($plainPassword, $hash)
    {
        return $plainPassword === $hash;
    }
}
```

Теперь можно протестировать все сценарии работы сервиса:

* пользователь не существует;
* пароль неверный;
* пользователь успешно аутентифицирован.

---

## Anemic Domain Models vs Rich Domain Models

Необходимо быть осторожным и не злоупотреблять Domain Services.

Чрезмерное количество Domain Service abstractions может привести к тому, что:

* Entity и Value Objects лишаются поведения;
* они превращаются в простые контейнеры данных.

Это противоречит цели объектно-ориентированного программирования, которое заключается в объединении данных и поведения в семантические единицы — объекты.

Подобная проблема считается anti-pattern и называется:

## Anemic Domain Model

Очень часто при создании новой системы разработчики начинают моделирование с данных:

* сначала проектируются таблицы БД;
* затем создаются ORM-модели;
* после этого добавляется логика.

Например:

```sql
CREATE TABLE orders (
    ID INTEGER NOT NULL AUTO_INCREMENT,
    CUSTOMER_ID INTEGER NOT NULL,
    AMOUNT DECIMAL(17, 2) NOT NULL DEFAULT '0.00',
    STATUS TINYINT NOT NULL DEFAULT 0,
    CREATED_AT DATETIME NOT NULL,
    UPDATED_AT DATETIME NOT NULL
);
```

Из этого легко получается класс:

```php
class Order
{
    // getters/setters
}
```

И дальнейшая работа выглядит так:

```php
$order = $orderRepository->find(1);

$order->setStatus(Order::STATUS_ACCEPTED);
$order->setUpdatedAt(new DateTimeImmutable());

$orderRepository->save($order);
```

Чтобы уменьшить дублирование, разработчики начинают создавать сервисы:

```php
class ChangeOrderStatusService
{
}
```

или:

```php
class UpdateOrderAmountService
{
}
```

На первый взгляд это выглядит как хорошее переиспользование кода.

Но проблема в том, что:

* нарушается инкапсуляция;
* клиенты знают внутреннее устройство Entity;
* бизнес-инварианты не защищены.

Клиент может обойти сервис и напрямую изменить Entity.

Вместо этого логика должна находиться внутри самой Entity:

```php
class Order
{
    public function changeAmount($amount)
    {
        $this->amount = $amount;
        $this->setUpdatedAt(new DateTimeImmutable());
    }
}
```

Теперь:

* инварианты защищены;
* поведение инкапсулировано;
* повторное использование достигается через поведение объекта.

Это называется:

## Rich Domain Model

---

## Как избежать Anemic Domain Model

Чтобы избежать Anemic Domain Model:

* при проектировании сначала думайте о поведении;
* не начинайте с таблиц БД;
* воспринимайте ORM и БД как детали реализации;
* старайтесь откладывать инфраструктурные решения как можно позже.

Точно так же, как и Entity, Domain Services могут генерировать Domain Events. Однако если события в основном генерируются Domain Services, а не Entity, это снова может быть признаком Anemic Domain Model.

---

## Итоги

Как мы увидели, сервисы представляют операции внутри системы, и их можно разделить на три типа:

### Application Services

* координируют запросы извне;
* преобразуют внешние команды в доменные инструкции;
* не должны содержать доменную логику;
* обычно управляют транзакциями.

### Domain Services

* работают только с доменными концепциями;
* выражают операции Единого языка;
* используются, когда поведение не подходит Entity или Value Object.

### Infrastructure Services

* работают с инфраструктурой;
* отправляют email;
* логируют;
* взаимодействуют с внешними системами.

Главная рекомендация авторов:
прежде чем создавать Domain Service, тщательно рассмотрите все варианты.

Сначала попробуйте:

1. перенести бизнес-логику в Entity;
2. затем — в Value Object;
3. обсудить решение с коллегами;
4. пересмотреть модель ещё раз.

И только если после нескольких подходов лучшим вариантом остаётся Domain Service — используйте его.
