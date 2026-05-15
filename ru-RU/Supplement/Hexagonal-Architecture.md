# Гексагональная Архитектура в PHP

Следующая статья была опубликована в журнале php|architect в июне 2014 года автором Carlos Buenosvinos.

## Введение

С ростом популярности Domain-Driven Design (DDD), архитектуры, продвигающие подходы, ориентированные на домен, становятся всё более популярными. Это относится и к Hexagonal Architecture, также известной как Ports and Adapters, которая, похоже, была заново открыта PHP-разработчиками только сейчас. Изобретённая в 2005 году Alistair Cockburn, одним из авторов Agile Manifesto, Hexagonal Architecture позволяет приложению в равной степени управляться пользователями, программами, автоматическими тестами или batch-скриптами, а также разрабатываться и тестироваться изолированно от его конечных runtime-устройств и баз данных. Это приводит к появлению agnostic infrastructure web applications, которые проще тестировать, писать и поддерживать. Давайте посмотрим, как применять её, используя реальные примеры на PHP.

Ваша компания разрабатывает систему для мозговых штурмов под названием Idy. Пользователи добавляют и оценивают идеи, чтобы самые интересные из них могли быть реализованы в компании. Утро понедельника, начинается очередной спринт, и вы вместе с командой и Product Owner просматриваете пользовательские истории. Как незалогиненный пользователь, я хочу оценивать идею, а автор должен быть уведомлён по email — это действительно важная история, не так ли?

## Первый подход

Как хороший разработчик, вы решаете разделить и завоевать пользовательскую историю, поэтому начнёте с первой части: Я хочу оценить идею. После этого вы займётесь частью автор должен быть уведомлён по email. Звучит как план.

С точки зрения бизнес-правил, оценка идеи так же проста, как найти идею по её идентификатору в репозитории идей, где живут все идеи, добавить оценку, пересчитать среднее значение и сохранить идею обратно. Если идея не существует или репозиторий недоступен, мы должны выбросить исключение, чтобы можно было показать сообщение об ошибке, перенаправить пользователя или сделать всё, что потребуется бизнесу.

Чтобы выполнить этот UseCase, нам нужны только идентификатор идеи и оценка от пользователя. Два целых числа, которые поступят из пользовательского запроса.

Веб-приложение вашей компании работает на legacy-приложении Zend Framework версии 1. Как и в большинстве компаний, вероятно, некоторые части вашего приложения новее, более SOLID, а другие могут быть просто большим клубком грязи. Однако вы знаете, что совершенно неважно, какой framework вы используете — всё дело в написании чистого кода, который делает поддержку дешёвой задачей для вашей компании.

Вы пытаетесь применить некоторые Agile-принципы, которые помните с последней конференции. Как там было… да, помню: “make it work, make it right, make it fast”. После некоторого времени работы у вас получается что-то вроде Listing 1.

```php
class IdeaController extends Zend_Controller_Action
{
    public function rateAction()
    {
        // Getting parameters from the request
        $ideaId = $this->request->getParam('id');
        $rating = $this->request->getParam('rating');

        // Building database connection
        $db = new Zend_Db_Adapter_Pdo_Mysql([
            'host'     => 'localhost',
            'username' => 'idy',
            'password' => '',
            'dbname'   => 'idy'
        ]);

        // Finding the idea in the database
        $sql = 'SELECT * FROM ideas WHERE idea_id = ?';
        $row = $db->fetchRow($sql, $ideaId);

        if (!$row) {
            throw new Exception('Idea does not exist');
        }

        // Building the idea from the database
        $idea = new Idea();
        $idea->setId($row['id']);
        $idea->setTitle($row['title']);
        $idea->setDescription($row['description']);
        $idea->setRating($row['rating']);
        $idea->setVotes($row['votes']);
        $idea->setAuthor($row['email']);

        // Add user rating
        $idea->addRating($rating);

        // Update the idea and save it to the database
        $data = [
            'votes' => $idea->getVotes(),
            'rating' => $idea->getRating()
        ];

        $where['idea_id = ?'] = $ideaId;

        $db->update('ideas', $data, $where);

        // Redirect to view idea page
        $this->redirect('/idea/' . $ideaId);
    }
}
```

Я знаю, о чём думают читатели: *Кто вообще будет обращаться к данным напрямую из controller? Это пример из 90-х!* Ладно, ладно, вы правы. Если вы уже используете framework, вполне вероятно, что вы также используете ORM. Возможно, написанную вами самостоятельно или одну из существующих — Doctrine, Eloquent, Zend и так далее. Если это так, вы уже на шаг впереди тех, у кого есть какой-то Database connection object, но не считайте цыплят раньше времени.

Для новичков код из Listing 1 просто работает. Однако, если вы внимательнее посмотрите на Controller, вы увидите не только бизнес-правила, но также и то, как ваш web framework маршрутизирует запрос к бизнес-правилам, ссылки на базу данных и способ подключения к ней. Совсем рядом — вы видите ссылки на вашу инфраструктуру.

Инфраструктура — это детали, которые заставляют ваши бизнес-правила работать. Очевидно, нам нужен какой-то способ добраться до них (API, web, console apps и так далее), и, конечно, нам нужно физическое место для хранения наших идей (memory, database, NoSQL и так далее). Однако мы должны иметь возможность заменить любую из этих частей на другую, которая ведёт себя аналогично, но имеет другую реализацию. Как насчёт того, чтобы начать с доступа к Database?

Все эти подключения Zend_DB_Adapter (или прямые MySQL-команды, если это ваш случай) буквально просятся быть повышенными до некоторого объекта, который инкапсулирует получение и сохранение объектов Idea. Они умоляют стать Repository.

## Репозитории и граница persistence (Persistence Edge)

Независимо от того, происходит ли изменение в бизнес-правилах или в инфраструктуре, нам приходится редактировать один и тот же участок кода. Поверьте, в Computer Science вы не хотите, чтобы много людей трогали один и тот же участок кода по разным причинам. Старайтесь делать так, чтобы ваши функции выполняли одну и только одну задачу — тогда вероятность того, что люди будут вмешиваться в один и тот же код, станет меньше. Подробнее об этом можно узнать, изучив Single Responsibility Principle (SRP). Дополнительная информация об этом принципе: [http://www.objectmentor.com/resources/articles/sr](http://www.objectmentor.com/resources/articles/sr) p.pdf

Listing 1 — как раз такой случай. Если мы захотим перейти на Redis или добавить функциональность уведомления автора, вам придётся обновлять метод rateAction. Вероятность затронуть аспекты rateAction, не связанные с изменяемой частью, очень высока. Код из Listing 1 — хрупкий. Если в вашей команде часто можно услышать Если работает — не трогай, значит, SRP отсутствует.

Итак, мы должны decouple наш код и инкапсулировать ответственность за получение и сохранение идей в другом объекте. Лучший способ, как уже объяснялось ранее, — использовать Repository. Вызов принят! Давайте посмотрим на результат в Listing 2:

```php
class IdeaController extends Zend_Controller_Action
{
    public function rateAction()
    {
        $ideaId = $this->request->getParam('id');
        $rating = $this->request->getParam('rating');

        $ideaRepository = new IdeaRepository();
        $idea = $ideaRepository->find($ideaId);

        if (!$idea) {
            throw new Exception('Idea does not exist');
        }

        $idea->addRating($rating);

        $ideaRepository->update($idea);

        $this->redirect('/idea/' . $ideaId);
    }
}
```

```php
class IdeaRepository
{
    private $client;

    public function __construct()
    {
        $this->client = new Zend_Db_Adapter_Pdo_Mysql([
            'host' => 'localhost',
            'username' => 'idy',
            'password' => '',
            'dbname' => 'idy'
        ]);
    }

    public function find($id)
    {
        $sql = 'SELECT * FROM ideas WHERE idea_id = ?';
        $row = $this->client->fetchRow($sql, $id);

        if (!$row) {
            return null;
        }

        $idea = new Idea();
        $idea->setId($row['id']);
        $idea->setTitle($row['title']);
        $idea->setDescription($row['description']);
        $idea->setRating($row['rating']);
        $idea->setVotes($row['votes']);
        $idea->setAuthor($row['email']);

        return $idea;
    }

    public function update(Idea $idea)
    {
        $data = [
            'title' => $idea->getTitle(),
            'description' => $idea->getDescription(),
            'rating' => $idea->getRating(),
            'votes' => $idea->getVotes(),
            'email' => $idea->getAuthor(),
        ];

        $where = ['idea_id = ?' => $idea->getId()];

        $this->client->update('ideas', $data, $where);
    }
}
```

Результат выглядит лучше. Метод rateAction в IdeaController стал более понятным. При чтении он говорит о бизнес-правилах. IdeaRepository — это бизнес-концепция. Когда вы разговариваете с бизнес-людьми, они понимают, что такое IdeaRepository: место, куда я помещаю Ideas и откуда их получаю.

“Repository mediates between the domain and data mapping layers using a collection-like interface for accessing domain objects.” — как сказано в каталоге паттернов Martin Fowler.

Если вы уже используете ORM вроде Doctrine, ваши текущие repositories наследуются от EntityRepository. Если вам нужно получить один из этих repositories, вы просите Doctrine EntityManager сделать эту работу. Получившийся код будет почти таким же, с дополнительным обращением к EntityManager в controller action для получения IdeaRepository.

На этом этапе мы можем увидеть на горизонте одну из граней нашего hexagon — persistence edge. Однако эта сторона ещё не до конца очерчена: всё ещё существует связь между тем, чем является IdeaRepository, и тем, как он реализован.

Чтобы добиться эффективного разделения между границей нашего приложения и границей инфраструктуры, нам нужен дополнительный шаг. Нам нужно явно decouple поведение от реализации, используя некоторую форму interface.

## Разделение бизнес-логики и persistence (Decoupling Business and Persistence)

Испытывали ли вы когда-нибудь ситуацию, когда начинали разговаривать с вашим Product Owner, Business Analyst или Project Manager о проблемах с Database? Помните их лица, когда вы объясняли, как сохранять и получать объект? Они не имели ни малейшего представления, о чём вы говорите.

Правда в том, что им всё равно, и это нормально. Если вы решите хранить идеи на MySQL-сервере, в Redis или SQLite — это ваша проблема, а не их. Помните: с точки зрения бизнеса инфраструктура — это деталь. Бизнес-правила не изменятся из-за того, что вы используете Symfony или Zend Framework, MySQL или PostgreSQL, REST или SOAP и так далее.

Именно поэтому важно отделить наш IdeaRepository от его реализации. Самый простой способ — использовать правильный interface. Как мы можем этого добиться? Давайте посмотрим на Listing 3.

```php
class IdeaController extends Zend_Controller_Action
{
    public function rateAction()
    {
        $ideaId = $this->request->getParam('id');
        $rating = $this->request->getParam('rating');

        $ideaRepository = new MySQLIdeaRepository();
        $idea = $ideaRepository->find($ideaId);

        if(!$idea) {
            throw new Exception('Idea does not exist');
        }

        $idea->addRating($rating);

        $ideaRepository->update($idea);

        $this->redirect('/idea/' . $ideaId);
    }
}
```

```php
interface IdeaRepository
{
    /**
     * @param int $id
     * @return null|Idea
     */
    public function find($id);

    /**
     * @param Idea $idea
     */
    public function update(Idea $idea);
}
```

```php
class MySQLIdeaRepository implements IdeaRepository
{
// ... }
```

Просто, не так ли? Мы вынесли поведение IdeaRepository в interface, переименовали IdeaRepository в MySQLIdeaRepository и обновили rateAction так, чтобы использовать наш MySQLIdeaRepository. Но в чём преимущество?

Теперь мы можем заменить repository, используемый в controller, на любой другой, реализующий тот же interface. Итак, давайте попробуем другую реализацию.


## Миграция persistence в Redis

Во время спринта и после разговора с несколькими коллегами вы понимаете, что использование NoSQL-стратегии может улучшить производительность вашей функциональности. Redis — один из ваших лучших друзей. Давайте сделаем это и посмотрим на ваш Listing 4:

```php
class IdeaController extends Zend_Controller_Action
{
    public function rateAction()
    {
        $ideaId = $this->request->getParam('id');
        $rating = $this->request->getParam('rating');

        $ideaRepository = new RedisIdeaRepository();
        $idea = $ideaRepository->find($ideaId);

        if (!$idea) {
            throw new Exception('Idea does not exist');
        }

        $idea->addRating($rating);

        $ideaRepository->update($idea);

        $this->redirect('/idea/' . $ideaId);
    }
}
```

```php
interface IdeaRepository
{
// ... }
```

```php
class RedisIdeaRepository implements IdeaRepository
{
    private $client;

    public function __construct()
    {
        $this->client = new Predis\Client();
    }

    public function find($id)
    {
        $idea = $this->client->get($this->getKey($id));

        if (!$idea) {
            return null;
        }

        return unserialize($idea);
    }

    public function update(Idea $idea)
    {
        $this->client->set(
            $this->getKey($idea->getId()),
            serialize($idea)
        );
    }

    private function getKey($id)
    {
        return 'idea:' . $id;
    }
}
```

Снова просто. Вы создали *RedisIdeaRepository*, который реализует *interface IdeaRepository*, и решили использовать Predis в качестве connection manager. Код выглядит меньше, проще и быстрее. Но что насчёт controller? Он остаётся тем же самым — мы просто изменили используемый repository, и это была всего одна строка кода.

В качестве упражнения для читателя попробуйте создать IdeaRepository для SQLite, файла или in-memory-реализации с использованием массивов. Дополнительные очки, если вы подумаете о том, как ORM Repositories соотносятся с Domain Repositories и как ORM @annotations влияют на эту архитектуру.

## Разделение бизнес-логики и web framework

Мы уже увидели, насколько легко можно перейти от одной persistence-стратегии к другой. Однако persistence — не единственная грань нашего Hexagon. А как насчёт того, как пользователь взаимодействует с приложением?

Ваш CTO добавил в roadmap, что ваша команда переходит на Symfony2, поэтому при разработке новых функциональностей в текущем приложении на ZF1 вы хотели бы упростить предстоящую миграцию. Это уже сложнее — покажите мне ваш Listing 5:

```php
class IdeaController extends Zend_Controller_Action
{
    public function rateAction()
    {
        $ideaId = $this->request->getParam('id');
        $rating = $this->request->getParam('rating');

        $ideaRepository = new RedisIdeaRepository();

        $useCase = new RateIdeaUseCase($ideaRepository);

        $response = $useCase->execute($ideaId, $rating);

        $this->redirect('/idea/' . $ideaId);
    }
}
```

```php
interface IdeaRepository
{
// ... }
```

```php
class RateIdeaUseCase
{
    private $ideaRepository;

    public function __construct(IdeaRepository $ideaRepository)
    {
        $this->ideaRepository = $ideaRepository;
    }

    public function execute($ideaId, $rating)
    {
        try {
            $idea = $this->ideaRepository->find($ideaId);
        } catch(Exception $e) {
            throw new RepositoryNotAvailableException();
        }

        if (!$idea) {
            throw new IdeaDoesNotExistException();
        }

        try {
            $idea->addRating($rating);
            $this->ideaRepository->update($idea);
        } catch(Exception $e) {
            throw new RepositoryNotAvailableException();
        }

        return $idea;
    }
}
```

Давайте рассмотрим изменения. Наш controller больше вообще не содержит бизнес-правил. Мы вынесли всю логику в новый объект под названием RateIdeaUseCase, который её инкапсулирует. Этот объект также известен как Controller, Interactor или Application Service.

Магия происходит в методе execute. Все зависимости, такие как RedisIdeaRepository, передаются через аргумент конструктора. Все ссылки на IdeaRepository внутри нашего UseCase указывают на interface, а не на какую-либо конкретную реализацию.

Это действительно круто. Если вы заглянете внутрь RateIdeaUseCase, там нет ничего, что говорило бы о MySQL или Zend Framework. Никаких ссылок, экземпляров, annotations — ничего. Как будто ваша инфраструктура не имеет значения. Он говорит только о бизнес-логике.

Кроме того, мы также улучшили Exceptions, которые выбрасываем. У бизнес-процессов тоже есть исключения. NotAvailableRepository и IdeaDoesNotExist — два таких примера. В зависимости от того, какое исключение было выброшено, мы можем по-разному реагировать на границе framework.

Иногда количество параметров, которые получает UseCase, может быть слишком большим. Чтобы организовать их, довольно распространено создание UseCase request с использованием Data Transfer Object (DTO), чтобы передавать их вместе. Давайте посмотрим, как это можно решить в Listing 6:

```php
class IdeaController extends Zend_Controller_Action
{
    public function rateAction()
    {
        $ideaId = $this->request->getParam('id');
        $rating = $this->request->getParam('rating');

        $ideaRepository = new RedisIdeaRepository();

        $useCase = new RateIdeaUseCase($ideaRepository);

        $response = $useCase->execute(
            new RateIdeaRequest($ideaId, $rating)
        );

        $this->redirect('/idea/' . $response->idea->getId());
    }
}
```

```php
class RateIdeaRequest
{
    public $ideaId;
    public $rating;

    public function __construct($ideaId, $rating)
    {
        $this->ideaId = $ideaId;
        $this->rating = $rating;
    }
}
```

```php
class RateIdeaResponse
{
    public $idea;

    public function __construct(Idea $idea)
    {
        $this->idea = $idea;
    }
}
```

```php
class RateIdeaUseCase
{
// ...

    public function execute($request)
    {
        $ideaId = $request->ideaId;
        $rating = $request->rating;

// ...

        return new RateIdeaResponse($idea);
    }
}
```

Главные изменения здесь — введение двух новых объектов: Request и Response. Они не обязательны; возможно, у UseCase вообще нет request или response. Ещё одна важная деталь — то, как вы строите этот request. В данном случае мы создаём его, получая параметры из объекта ZF request.

Хорошо, но подождите, в чём реальная польза? Становится проще перейти от одного framework к другому или запускать наш UseCase через другой delivery mechanism. Давайте рассмотрим этот момент.

## Rating an Idea Using the API

В течение дня ваш Product Owner подходит к вам и говорит: кстати, пользователь должен иметь возможность оценивать идею через наше мобильное приложение. Думаю, нам нужно обновить API, сможете сделать это в этом спринте?. Снова этот PO. Без проблем! Бизнес впечатлён вашей вовлечённостью.

**Как говорит Robert C. Martin:**
> «Web — это механизм доставки [...] Архитектура вашей системы должна быть настолько неосведомлённой о способе её доставки, насколько это возможно. Вы должны иметь возможность поставлять её как console app, web app или даже web service app без чрезмерных усложнений и без каких-либо изменений фундаментальной архитектуры.»


Ваш текущий API построен с использованием Silex — PHP micro-framework, основанного на Symfony2 Components. Давайте перейдём к Listing 7:

```php
require_once __DIR__.'/../vendor/autoload.php';

$app = new Silex\Application();

// ... more routes

$app->get(
    '/api/rate/idea/{ideaId}/rating/{rating}',
    function ($ideaId, $rating) use ($app) {

        $ideaRepository = new RedisIdeaRepository();

        $useCase = new RateIdeaUseCase($ideaRepository);

        $response = $useCase->execute(
            new RateIdeaRequest($ideaId, $rating)
        );

        return $app->json($response->idea);
    }
);

$app->run();
```

Есть ли здесь что-то знакомое? Можете определить код, который уже видели раньше? Я дам подсказку:

```php
$ideaRepository = new RedisIdeaRepository();

$useCase = new RateIdeaUseCase($ideaRepository);

$response = $useCase->execute(
    new RateIdeaRequest($ideaId, $rating)
);
```

Чёрт! Я помню эти 3 строки кода. Они выглядят точно так же, как в web application. Именно так, потому что UseCase инкапсулирует бизнес-правила — вам нужно только подготовить request, получить response и соответствующим образом отреагировать.

Мы предоставляем пользователям ещё один способ оценивать идею; ещё один delivery mechanism. Главное различие — это место, где мы создаём RateIdeaRequest. В первом примере он создавался из ZF request, а теперь — из Silex request с использованием параметров, сопоставленных в route.


======



## Оценка через Console App

Иногда UseCase будет выполняться из Cron job или из command line. Например, batch processing или некоторые testing command lines для ускорения разработки. Во время тестирования этой функциональности через web или API вы понимаете, что было бы неплохо иметь command line для этого, чтобы не приходилось каждый раз проходить через browser.

Если вы используете shell script files, я рекомендую вам обратить внимание на компонент Symfony Console. Как будет выглядеть код:

```php
namespace Idy\Console\Command;

use Symfony\Component\Console\Command\Command;
use Symfony\Component\Console\Input\InputArgument;
use Symfony\Component\Console\Input\InputInterface;
use Symfony\Component\Console\Output\OutputInterface;

class VoteIdeaCommand extends Command
{
    protected function configure()
    {
        $this
            ->setName('idea:rate')
            ->setDescription('Rate an idea')
            ->addArgument('id', InputArgument::REQUIRED)
            ->addArgument('rating', InputArgument::REQUIRED);
    }

    protected function execute(
        InputInterface $input,
        OutputInterface $output
    ){
        $ideaId = $input->getArgument('id');
        $rating = $input->getArgument('rating');

        $ideaRepository = new RedisIdeaRepository();

        $useCase = new RateIdeaUseCase($ideaRepository);

        $response = $useCase->execute(
            new RateIdeaRequest($ideaId, $rating)
        );

        $output->writeln('Done!');
    }
}
```

Снова эти 3 строки кода. Как и раньше, UseCase и его бизнес-логика остаются нетронутыми — мы просто предоставляем новый delivery mechanism. Поздравляем, вы обнаружили пользовательскую сторону hexagon edge.

Однако работы ещё много. Как вы, возможно, слышали, настоящий craftsman делает TDD. Мы уже начали нашу историю, так что должны быть согласны хотя бы с тестированием после написания.

## Тестирование UseCase Rating an Idea

Michael Feathers ввёл определение legacy code как code without tests. Вы ведь не хотите, чтобы ваш код стал legacy сразу после рождения?

Чтобы unit test этого объекта UseCase, вы решаете начать с самой простой части: что произойдёт, если repository недоступен? Как можно сгенерировать такое поведение? Останавливаем ли мы Redis server во время выполнения unit tests? Нет. Нам нужен объект, обладающий таким поведением. Давайте используем mock object в Listing 9:

```php
class RateIdeaUseCaseTest extends \PHPUnit_Framework_TestCase
{
    /**
     * @test
     */
    public function whenRepositoryNotAvailableAnExceptionIsThrown()
    {
        $this->setExpectedException(
            'NotAvailableRepositoryException'
        );

        $ideaRepository = new NotAvailableRepository();

        $useCase = new RateIdeaUseCase($ideaRepository);

        $useCase->execute(
            new RateIdeaRequest(1, 5)
        );
    }
}
```

```php
class NotAvailableRepository implements IdeaRepository
{
    public function find($id)
    {
        throw new NotAvailableException();
    }

    public function update(Idea $idea)
    {
        throw new NotAvailableException();
    }
}
```

Отлично. NotAvailableRepository обладает нужным нам поведением, и мы можем использовать его с RateIdeaUseCase, потому что он реализует interface IdeaRepository.

Следующий случай для тестирования — что произойдёт, если idea отсутствует в repository. Listing 10 показывает код:

```php
class RateIdeaUseCaseTest extends \PHPUnit_Framework_TestCase
{
// ...

    /**
     * @test
     */
    public function whenIdeaDoesNotExistAnExceptionShouldBeThrown()
    {
        $this->setExpectedException(
            'IdeaDoesNotExistException'
        );

        $ideaRepository = new EmptyIdeaRepository();

        $useCase = new RateIdeaUseCase($ideaRepository);

        $useCase->execute(
            new RateIdeaRequest(1, 5)
        );
    }
}
```

```php
class EmptyIdeaRepository implements IdeaRepository
{
    public function find($id)
    {
        return null;
    }

    public function update(Idea $idea)
    {
    }
}
```

Здесь мы используем ту же стратегию, но с EmptyIdeaRepository. Он также реализует тот же interface, однако реализация всегда возвращает null независимо от того, какой identifier получает метод find.

Почему мы тестируем эти случаи? Помните слова Kent Beck: “Test everything that could possibly break.”

Давайте продолжим с остальной частью функциональности. Нам нужно проверить особый случай, связанный с наличием repository, доступного для чтения, но недоступного для записи. Решение можно найти в Listing 11:

```php
class RateIdeaUseCaseTest extends \PHPUnit_Framework_TestCase
{
// ...

    /**
     * @test
     */
    public function whenRatingAnIdeaNewRatingShouldBeAdded()
    {
        $ideaRepository = new OneIdeaRepository();

        $useCase = new RateIdeaUseCase($ideaRepository);

        $response = $useCase->execute(
            new RateIdeaRequest(1, 5)
        );

        $this->assertSame(
            5,
            $response->idea->getRating()
        );

        $this->assertTrue(
            $ideaRepository->updateCalled
        );
    }
}
```

```php
class OneIdeaRepository implements IdeaRepository
{
    public $updateCalled = false;

    public function find($id)
    {
        $idea = new Idea();

        $idea->setId(1);
        $idea->setTitle('Subscribe to php[architect]');
        $idea->setDescription('Just buy it!');
        $idea->setRating(5);
        $idea->setVotes(10);
        $idea->setAuthor('john@example.com');

        return $idea;
    }

    public function update(Idea $idea)
    {
        $this->updateCalled = true;
    }
}
```

Хорошо, теперь остаётся ключевая часть функциональности. У нас есть разные способы её тестирования: мы можем написать собственный mock или использовать mocking framework, такой как Mockery или Prophecy. Давайте выберем первый вариант. Ещё одним интересным упражнением было бы переписать этот пример и предыдущие с использованием одного из этих frameworks:

```php
class RateIdeaUseCaseTest extends \PHPUnit_Framework_TestCase
{
// ...

    /**
     * @test
     */
    public function whenRatingAnIdeaNewRatingShouldBeAdded()
    {
        $ideaRepository = new OneIdeaRepository();

        $useCase = new RateIdeaUseCase($ideaRepository);

        $response = $useCase->execute(
            new RateIdeaRequest(1, 5)
        );

        $this->assertSame(
            5,
            $response->idea->getRating()
        );

        $this->assertTrue(
            $ideaRepository->updateCalled
        );
    }
}
```

```php
class OneIdeaRepository implements IdeaRepository
{
    public $updateCalled = false;

    public function find($id)
    {
        $idea = new Idea();

        $idea->setId(1);
        $idea->setTitle('Subscribe to php[architect]');
        $idea->setDescription('Just buy it!');
        $idea->setRating(5);
        $idea->setVotes(10);
        $idea->setAuthor('john@example.com');

        return $idea;
    }

    public function update(Idea $idea)
    {
        $this->updateCalled = true;
    }
}
```

Бам! 100% Coverage для UseCase. Возможно, в следующий раз мы сделаем это с использованием TDD, чтобы тест появился первым. Однако тестирование этой функциональности было действительно простым благодаря тому, как в этой архитектуре продвигается decoupling.

Возможно, вас интересует вот это:

```php
$this->updateCalled = true;
```

Нам нужен способ гарантировать, что метод update был вызван во время выполнения UseCase. Это и решает задачу. Такой test double object называется spy — cousin для mocks.

Когда использовать mocks? В качестве общего правила используйте mocks при пересечении boundaries. В данном случае mocks нужны нам, потому что мы пересекаем границу между domain и persistence boundary.

## Что насчёт тестирования infrastructure?

Если вы хотите достичь 100% coverage для всего приложения, вам также придётся тестировать инфраструктуру. Перед этим нужно понимать, что эти unit tests будут сильнее связаны с вашей реализацией, чем бизнес-тесты. Это означает, что вероятность их поломки при изменениях implementation details выше. Так что это trade-off, который вам придётся учитывать.

Итак, если вы хотите продолжить, нам нужно внести некоторые изменения. Нам нужно decouple ещё сильнее. Давайте посмотрим на код в Listing 13:

```php
class IdeaController extends Zend_Controller_Action
{
    public function rateAction()
    {
        $ideaId = $this->request->getParam('id');
        $rating = $this->request->getParam('rating');

        $useCase = new RateIdeaUseCase(
            new RedisIdeaRepository(
                new Predis\Client()
            )
        );

        $response = $useCase->execute(
            new RateIdeaRequest($ideaId, $rating)
        );

        $this->redirect(
            '/idea/' . $response->idea->getId()
        );
    }
}
```

```php
class RedisIdeaRepository implements IdeaRepository
{
    private $client;

    public function __construct($client)
    {
        $this->client = $client;
    }

// ...

    public function find($id)
    {
        $idea = $this->client->get(
            $this->getKey($id)
        );

        if (!$idea) {
            return null;
        }

        return $idea;
    }
}
```

Если мы хотим на 100% unit test RedisIdeaRepository, нам нужно иметь возможность передавать Predis\Client как параметр в repository без указания TypeHinting, чтобы мы могли передать mock и принудительно создать необходимый code flow для покрытия всех случаев.

Это заставляет нас обновить Controller: теперь он должен создавать Redis connection, передавать его в repository и передавать результат в UseCase.

Теперь всё сводится к созданию mocks, test cases и получению удовольствия от написания asserts.

## Ахх, так много dependencies!

Нормально ли, что мне приходится создавать столько dependencies вручную? Нет. Обычно используют компонент Dependency Injection или Service Container с подобными возможностями. И снова Symfony приходит на помощь, хотя вы также можете обратить внимание на PHP-DI 4.

Давайте посмотрим на итоговый код в Listing 14 после применения компонента Symfony Service Container в нашем приложении:

```php
class IdeaController extends ContainerAwareController
{
    public function rateAction()
    {
        $ideaId = $this->request->getParam('id');
        $rating = $this->request->getParam('rating');

        $useCase = $this->get(
            'rate_idea_use_case'
        );

        $response = $useCase->execute(
            new RateIdeaRequest($ideaId, $rating)
        );

        $this->redirect(
            '/idea/' . $response->idea->getId()
        );
    }
}
```

Controller был изменён так, чтобы иметь доступ к container, именно поэтому он наследуется от нового base controller — ContainerAwareController, который имеет метод get для получения каждого из содержащихся services:

```xml
<?xml version="1.0" ?>

<container
    xmlns="http://symfony.com/schema/dic/services"
    xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="
        http://symfony.com/schema/dic/services
        http://symfony.com/schema/dic/services/services-1.0.xsd">

    <services>

        <service
            id="rate_idea_use_case"
            class="RateIdeaUseCase">

            <argument
                type="service"
                id="idea_repository" />

        </service>

        <service
            id="idea_repository"
            class="RedisIdeaRepository">

            <argument type="service">

                <service class="Predis\Client" />

            </argument>

        </service>

    </services>

</container>
```

В Listing 15 вы также можете найти XML file, используемый для конфигурации Service Container. Его действительно легко понять, но если вам нужна дополнительная информация, посмотрите сайт Symfony Service Container Component.

## Domain Services и notification hexagon edge

Мы ничего не забыли? автор должен быть уведомлён по email — да! Это правда. Давайте посмотрим в Listing 16, как мы обновили UseCase для выполнения этой задачи:

```php
class RateIdeaUseCase
{
    private $ideaRepository;
    private $authorNotifier;

    public function __construct(
        IdeaRepository $ideaRepository,
        AuthorNotifier $authorNotifier
    ){
        $this->ideaRepository = $ideaRepository;
        $this->authorNotifier = $authorNotifier;
    }

    public function execute(RateIdeaRequest $request)
    {
        $ideaId = $request->ideaId;
        $rating = $request->rating;

        try {
            $idea = $this->ideaRepository->find($ideaId);
        } catch(Exception $e) {
            throw new RepositoryNotAvailableException();
        }

        if (!$idea) {
            throw new IdeaDoesNotExistException();
        }

        try {
            $idea->addRating($rating);

            $this->ideaRepository->update($idea);

        } catch(Exception $e) {

            throw new RepositoryNotAvailableException();
        }

        try {

            $this->authorNotifier->notify(
                $idea->getAuthor()
            );

        } catch(Exception $e) {

            throw new NotificationNotSentException();
        }

        return $idea;
    }
}
```

Как вы заметили, мы добавили новый параметр для передачи сервиса AuthorNotifier, который будет отправлять email автору. Это и есть port в терминологии Ports and Adapters. Мы также обновили бизнес-правила внутри метода execute.

Repositories — не единственные объекты, которые могут обращаться к вашей infrastructure и должны быть decoupled с помощью interfaces или abstract classes. Domain Services тоже могут. Когда существует поведение, которое явно не принадлежит одной конкретной Entity в вашем domain, вам следует создать Domain Service. Типичный паттерн — написать abstract Domain Service, у которого есть какая-то concrete implementation и несколько abstract methods, которые adapter будет реализовывать.

В качестве упражнения определите implementation details для abstract service AuthorNotifier. Варианты — SwiftMailer или просто обычные вызовы mail. Решать вам.

## Давайте подведём итог

Чтобы иметь clean architecture, которая помогает создавать простые для написания и тестирования приложения, мы можем использовать Hexagonal Architecture. Для этого мы инкапсулируем бизнес-правила пользовательской истории внутри объекта UseCase или Interactor. Мы создаём UseCase request из framework request, инстанцируем UseCase и все его dependencies, а затем выполняем его. Мы получаем response и действуем соответствующим образом на его основе. Если у вашего framework есть компонент Dependency Injection, вы можете использовать его для упрощения кода.

Те же самые объекты UseCase могут использоваться из разных delivery mechanisms, чтобы позволить пользователям получать доступ к функциональности через разных клиентов (web, API, console и так далее).

Для тестирования используйте mocks, которые ведут себя как все определённые interfaces, чтобы можно было покрыть special cases и error flows. Наслаждайтесь хорошо выполненной работой.

## Hexagonal Architecture

Почти во всех блогах и книгах вы найдёте рисунки с концентрическими кругами, представляющими различные области software. Как объясняет Robert C. Martin в своей публикации Clean Architecture, внешний круг — это место, где находится ваша infrastructure. Внутренний круг — место, где живут ваши Entities. Главное правило, благодаря которому эта архитектура работает, — The Dependency Rule. Это правило говорит, что зависимости исходного кода могут указывать только внутрь. Ничто во внутреннем круге не может знать вообще ничего о чём-либо во внешнем круге.

## Ключевые моменты

Используйте этот подход, если для вашего приложения важно 100% покрытие unit tests. Также — если вы хотите иметь возможность менять стратегию хранения, web framework или любой другой сторонний код. Архитектура особенно полезна для долгоживущих приложений, которым нужно адаптироваться к изменяющимся требованиям.

## Что дальше?

Если вам интересно узнать больше о Hexagonal Architecture и других близких концепциях, вам стоит изучить связанные URL, приведённые в начале статьи, посмотреть на CQRS и Event Sourcing. Также не забудьте подписаться на google groups и RSS по DDD, например [http://dddinphp.org](http://dddinphp.org), и следить в Twitter за такими людьми, как @VaughnVernon и @ericevans0.


