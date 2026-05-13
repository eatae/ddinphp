Перевод книги DDD in PHP
=
[Книга](https://leanpub.com/ddd-in-php)

[Репозиторий с примерами](https://github.com/dddshelf/ddd-in-php-book-examples)

## Содержание
## Предисловие
## Предисловие от Авторов
## [DDD и PHP Сообщество](https://github.com/eatae/dddinphp/blob/main/ru-RU/Preface/DDD-and-PHP-Community.md)
<br>

## [Краткое содержание глав](https://github.com/eatae/dddinphp/blob/main/ru-RU/Preface/Summary-of-Chapters.md)
- [Глава 1. Начало работы с DDD](https://github.com/eatae/dddinphp/blob/main/ru-RU/Chapter1/Getting-Started-with-Domain-Driven-Design.md)
- [Глава 2. Архитектурные стили](https://github.com/eatae/dddinphp/blob/main/ru-RU/Chapter2/Architectural-Styles.md)
- [Глава 3. Объекты-Значения](https://github.com/eatae/dddinphp/blob/main/ru-RU/Chapter3/Value-Objects.md)
- [Глава 4. Сущности (Entities)](https://github.com/eatae/dddinphp/blob/main/ru-RU/Chapter4/Entities.md)
- [Глава 5. Сервисы (Services)](https://github.com/eatae/dddinphp/blob/main/ru-RU/Chapter5/Services.md)
- [Глава 6. Доменные события](https://github.com/eatae/dddinphp/blob/main/ru-RU/Chapter6/Domain-Events.md)
- [Глава 7. Модули (Modules)](https://github.com/eatae/dddinphp/blob/main/ru-RU/Chapter7/Modules.md#%D0%B3%D0%BB%D0%B0%D0%B2%D0%B0-7-%D0%BC%D0%BE%D0%B4%D1%83%D0%BB%D0%B8-modules)
- [Глава 8. Агрегаты (Aggregates)]
- [Глава 9. Фабрики (Factories)]
- [Глава 10: Репозитории (Repositories)]
- [Глава 11: Приложение (Application)]
- [Глава 12: Интеграция Ограниченных Контекстов (Bounded Contexts)]
- [Приложение: Гексагональная Архитектура (Hexagonal Architecture) в PHP]
- [Код и Примеры]
<br>

## [Глава 1. Начало работы с DDD](https://github.com/eatae/dddinphp/blob/main/ru-RU/Chapter1/Getting-Started-with-Domain-Driven-Design.md#%D0%B3%D0%BB%D0%B0%D0%B2%D0%B0-1-%D0%BD%D0%B0%D1%87%D0%B0%D0%BB%D0%BE-%D1%80%D0%B0%D0%B1%D0%BE%D1%82%D1%8B-%D1%81-ddd)
- [Почему DDD так важен](https://github.com/eatae/dddinphp/blob/main/ru-RU/Chapter1/Getting-Started-with-Domain-Driven-Design.md#%D0%BF%D0%BE%D1%87%D0%B5%D0%BC%D1%83-ddd-%D1%82%D0%B0%D0%BA-%D0%B2%D0%B0%D0%B6%D0%B5%D0%BD)
- [Три столпа DDD](https://github.com/eatae/dddinphp/blob/main/ru-RU/Chapter1/Getting-Started-with-Domain-Driven-Design.md#%D0%BF%D0%BE%D1%87%D0%B5%D0%BC%D1%83-ddd-%D1%82%D0%B0%D0%BA-%D0%B2%D0%B0%D0%B6%D0%B5%D0%BD)
  - [Единый язык (Ubiquitous Language)](https://github.com/eatae/dddinphp/blob/main/ru-RU/Chapter1/Getting-Started-with-Domain-Driven-Design.md#%D0%B5%D0%B4%D0%B8%D0%BD%D1%8B%D0%B9-%D1%8F%D0%B7%D1%8B%D0%BA-ubiquitous-language)
    - [Событийный Штурм (Event Storming)](https://github.com/eatae/dddinphp/blob/main/ru-RU/Chapter1/Getting-Started-with-Domain-Driven-Design.md#%D0%B5%D0%B4%D0%B8%D0%BD%D1%8B%D0%B9-%D1%8F%D0%B7%D1%8B%D0%BA-ubiquitous-language)
- [Определение DDD](https://github.com/eatae/dddinphp/blob/main/ru-RU/Chapter1/Getting-Started-with-Domain-Driven-Design.md#%D0%BE%D0%BF%D1%80%D0%B5%D0%B4%D0%B5%D0%BB%D0%B5%D0%BD%D0%B8%D0%B5-ddd)
- [Некоторые нюансы](https://github.com/eatae/dddinphp/blob/main/ru-RU/Chapter1/Getting-Started-with-Domain-Driven-Design.md#%D0%BD%D0%B5%D0%BA%D0%BE%D1%82%D0%BE%D1%80%D1%8B%D0%B5-%D0%BD%D1%8E%D0%B0%D0%BD%D1%81%D1%8B)
- [Стратегический обзор](https://github.com/eatae/dddinphp/blob/main/ru-RU/Chapter1/Getting-Started-with-Domain-Driven-Design.md#%D0%BD%D0%B5%D0%BA%D0%BE%D1%82%D0%BE%D1%80%D1%8B%D0%B5-%D0%BD%D1%8E%D0%B0%D0%BD%D1%81%D1%8B)
- [Связанные механизмы: Микросервисы и автономные системы](https://github.com/eatae/dddinphp/blob/main/ru-RU/Chapter1/Getting-Started-with-Domain-Driven-Design.md#%D0%BD%D0%B5%D0%BA%D0%BE%D1%82%D0%BE%D1%80%D1%8B%D0%B5-%D0%BD%D1%8E%D0%B0%D0%BD%D1%81%D1%8B)
- [Резюмируем](https://github.com/eatae/dddinphp/blob/main/ru-RU/Chapter1/Getting-Started-with-Domain-Driven-Design.md#%D1%80%D0%B5%D0%B7%D1%8E%D0%BC%D0%B8%D1%80%D1%83%D0%B5%D0%BC)
<br>

## [Глава 2. Архитектурные стили](https://github.com/eatae/dddinphp/blob/main/ru-RU/Chapter2/Architectural-Styles.md#%D0%B3%D0%BB%D0%B0%D0%B2%D0%B0-2-%D0%B0%D1%80%D1%85%D0%B8%D1%82%D0%B5%D0%BA%D1%82%D1%83%D1%80%D0%BD%D1%8B%D0%B5-%D1%81%D1%82%D0%B8%D0%BB%D0%B8)
- [Старые, добрые времена](https://github.com/eatae/dddinphp/blob/main/ru-RU/Chapter2/Architectural-Styles.md#%D1%81%D1%82%D0%B0%D1%80%D1%8B%D0%B5-%D0%B4%D0%BE%D0%B1%D1%80%D1%8B%D0%B5-%D0%B2%D1%80%D0%B5%D0%BC%D0%B5%D0%BD%D0%B0)
- [Многоуровневая архитектура (Layered Architecture)](https://github.com/eatae/dddinphp/blob/main/ru-RU/Chapter2/Architectural-Styles.md#%D0%BC%D0%BD%D0%BE%D0%B3%D0%BE%D1%83%D1%80%D0%BE%D0%B2%D0%BD%D0%B5%D0%B2%D0%B0%D1%8F-%D0%B0%D1%80%D1%85%D0%B8%D1%82%D0%B5%D0%BA%D1%82%D1%83%D1%80%D0%B0-layered-architecture)
  - [Пример Многоуровневой Архитектуры](https://github.com/eatae/dddinphp/blob/main/ru-RU/Chapter2/Architectural-Styles.md#%D0%BC%D0%BD%D0%BE%D0%B3%D0%BE%D1%83%D1%80%D0%BE%D0%B2%D0%BD%D0%B5%D0%B2%D0%B0%D1%8F-%D0%B0%D1%80%D1%85%D0%B8%D1%82%D0%B5%D0%BA%D1%82%D1%83%D1%80%D0%B0-layered-architecture)
  - [Инверсия зависимостей. Гексагональная архитектура.](https://github.com/eatae/dddinphp/blob/main/ru-RU/Chapter2/Architectural-Styles.md#%D0%BC%D0%BD%D0%BE%D0%B3%D0%BE%D1%83%D1%80%D0%BE%D0%B2%D0%BD%D0%B5%D0%B2%D0%B0%D1%8F-%D0%B0%D1%80%D1%85%D0%B8%D1%82%D0%B5%D0%BA%D1%82%D1%83%D1%80%D0%B0-layered-architecture)
  - [Command Query Responsibility Segregation (CQRS)](https://github.com/eatae/dddinphp/blob/main/ru-RU/Chapter2/Architectural-Styles.md#command-query-responsibility-segregation-cqrs)
  - [Event Sourcing](https://github.com/eatae/dddinphp/blob/main/ru-RU/Chapter2/Architectural-Styles.md#event-sourcing)
  <br>

## [Глава 3. Объекты-Значения](https://github.com/eatae/dddinphp/blob/main/ru-RU/Chapter3/Value-Objects.md#%D0%B3%D0%BB%D0%B0%D0%B2%D0%B0-3-%D0%BE%D0%B1%D1%8A%D0%B5%D0%BA%D1%82%D1%8B-%D0%B7%D0%BD%D0%B0%D1%87%D0%B5%D0%BD%D0%B8%D1%8F-the-value-objects)
- [Определение](https://github.com/eatae/dddinphp/blob/main/ru-RU/Chapter3/Value-Objects.md#%D0%BE%D0%BF%D1%80%D0%B5%D0%B4%D0%B5%D0%BB%D0%B5%D0%BD%D0%B8%D0%B5)
- [Объекты значения vs Сущности](https://github.com/eatae/dddinphp/blob/main/ru-RU/Chapter3/Value-Objects.md#%D0%BE%D0%B1%D1%8A%D0%B5%D0%BA%D1%82%D1%8B-%D0%B7%D0%BD%D0%B0%D1%87%D0%B5%D0%BD%D0%B8%D1%8F-vs-%D1%81%D1%83%D1%89%D0%BD%D0%BE%D1%81%D1%82%D0%B8)
- [Пример "Валюта" и "Стоимость"](https://github.com/eatae/dddinphp/blob/main/ru-RU/Chapter3/Value-Objects.md#%D0%BF%D1%80%D0%B8%D0%BC%D0%B5%D1%80-%D0%B2%D0%B0%D0%BB%D1%8E%D1%82%D0%B0-%D0%B8-%D1%81%D1%82%D0%BE%D0%B8%D0%BC%D0%BE%D1%81%D1%82%D1%8C)
- [Характеристики](https://github.com/eatae/dddinphp/blob/main/ru-RU/Chapter3/Value-Objects.md#%D1%85%D0%B0%D1%80%D0%B0%D0%BA%D1%82%D0%B5%D1%80%D0%B8%D1%81%D1%82%D0%B8%D0%BA%D0%B8)
- [Базовые типы](https://github.com/eatae/dddinphp/blob/main/ru-RU/Chapter3/Value-Objects.md#%D0%B1%D0%B0%D0%B7%D0%BE%D0%B2%D1%8B%D0%B5-%D1%82%D0%B8%D0%BF%D1%8B)
- [Тестирование Объектов Значения](https://github.com/eatae/dddinphp/blob/main/ru-RU/Chapter3/Value-Objects.md#%D1%82%D0%B5%D1%81%D1%82%D0%B8%D1%80%D0%BE%D0%B2%D0%B0%D0%BD%D0%B8%D0%B5-%D0%BE%D0%B1%D1%8A%D0%B5%D0%BA%D1%82%D0%BE%D0%B2-%D0%B7%D0%BD%D0%B0%D1%87%D0%B5%D0%BD%D0%B8%D1%8F)
- [Сохранение Объекта Значения](https://github.com/eatae/dddinphp/blob/main/ru-RU/Chapter3/Value-Objects.md#%D1%81%D0%BE%D1%85%D1%80%D0%B0%D0%BD%D0%B5%D0%BD%D0%B8%D0%B5-%D0%BE%D0%B1%D1%8A%D0%B5%D0%BA%D1%82%D0%B0-%D0%B7%D0%BD%D0%B0%D1%87%D0%B5%D0%BD%D0%B8%D1%8F)
  <br>

## [Глава 4. Сущности](https://github.com/eatae/dddinphp/blob/main/ru-RU/Chapter4/Entities.md#%D0%B3%D0%BB%D0%B0%D0%B2%D0%B0-4-%D1%81%D1%83%D1%89%D0%BD%D0%BE%D1%81%D1%82%D0%B8-entities)
- [Введение](https://github.com/eatae/dddinphp/blob/main/ru-RU/Chapter4/Entities.md#%D0%B2%D0%B2%D0%B5%D0%B4%D0%B5%D0%BD%D0%B8%D0%B5)
- [Объекты Vs Примитивные типы](https://github.com/eatae/dddinphp/blob/main/ru-RU/Chapter4/Entities.md#%D0%BE%D0%B1%D1%8A%D0%B5%D0%BA%D1%82%D1%8B-vs-%D0%BF%D1%80%D0%B8%D0%BC%D0%B8%D1%82%D0%B8%D0%B2%D0%BD%D1%8B%D0%B5-%D1%82%D0%B8%D0%BF%D1%8B)
- [Операция идентификации](https://github.com/eatae/dddinphp/blob/main/ru-RU/Chapter4/Entities.md#%D0%BE%D0%BF%D0%B5%D1%80%D0%B0%D1%86%D0%B8%D1%8F-%D0%B8%D0%B4%D0%B5%D0%BD%D1%82%D0%B8%D1%84%D0%B8%D0%BA%D0%B0%D1%86%D0%B8%D0%B8)
  - [Механизмы хранения - БД генерирует Идентификатор](https://github.com/eatae/dddinphp/blob/main/ru-RU/Chapter4/Entities.md#%D0%BC%D0%B5%D1%85%D0%B0%D0%BD%D0%B8%D0%B7%D0%BC-%D1%85%D1%80%D0%B0%D0%BD%D0%B5%D0%BD%D0%B8%D1%8F---%D0%B1%D0%B4-%D0%B3%D0%B5%D0%BD%D0%B5%D1%80%D0%B8%D1%80%D1%83%D0%B5%D1%82-%D0%B8%D0%B4%D0%B5%D0%BD%D1%82%D0%B8%D1%84%D0%B8%D0%BA%D0%B0%D1%82%D0%BE%D1%80)
    - [Суррогатная идентичность](https://github.com/eatae/dddinphp/blob/main/ru-RU/Chapter4/Entities.md#%D1%81%D1%83%D1%80%D1%80%D0%BE%D0%B3%D0%B0%D1%82%D0%BD%D0%B0%D1%8F-%D0%B8%D0%B4%D0%B5%D0%BD%D1%82%D0%B8%D1%87%D0%BD%D0%BE%D1%81%D1%82%D1%8C)
    - [Active Record Vs. Data Mapper для Богатых Доменных Моделей](https://github.com/eatae/dddinphp/blob/main/ru-RU/Chapter4/Entities.md#active-record-vs-data-mapper-%D0%B4%D0%BB%D1%8F-%D0%B1%D0%BE%D0%B3%D0%B0%D1%82%D1%8B%D1%85-%D0%B4%D0%BE%D0%BC%D0%B5%D0%BD%D0%BD%D1%8B%D1%85-%D0%BC%D0%BE%D0%B4%D0%B5%D0%BB%D0%B5%D0%B9)
  - [Клиент предоставляет Идентификатор](https://github.com/eatae/dddinphp/blob/main/ru-RU/Chapter4/Entities.md#%D0%BA%D0%BB%D0%B8%D0%B5%D0%BD%D1%82-%D0%BF%D1%80%D0%B5%D0%B4%D0%BE%D1%81%D1%82%D0%B0%D0%B2%D0%BB%D1%8F%D0%B5%D1%82-%D0%B8%D0%B4%D0%B5%D0%BD%D1%82%D0%B8%D1%84%D0%B8%D0%BA%D0%B0%D1%82%D0%BE%D1%80)
  - [Приложение создаёт Идентификатор](https://github.com/eatae/dddinphp/blob/main/ru-RU/Chapter4/Entities.md#%D0%BF%D1%80%D0%B8%D0%BB%D0%BE%D0%B6%D0%B5%D0%BD%D0%B8%D0%B5-%D1%81%D0%BE%D0%B7%D0%B4%D0%B0%D1%91%D1%82-%D0%B8%D0%B4%D0%B5%D0%BD%D1%82%D0%B8%D1%84%D0%B8%D0%BA%D0%B0%D1%82%D0%BE%D1%80)
  - [Другой ограниченный контекст генерирует идентификатор](https://github.com/eatae/dddinphp/blob/main/ru-RU/Chapter4/Entities.md#%D0%B4%D1%80%D1%83%D0%B3%D0%BE%D0%B9-%D0%BE%D0%B3%D1%80%D0%B0%D0%BD%D0%B8%D1%87%D0%B5%D0%BD%D0%BD%D1%8B%D0%B9-%D0%BA%D0%BE%D0%BD%D1%82%D0%B5%D0%BA%D1%81%D1%82-%D0%B3%D0%B5%D0%BD%D0%B5%D1%80%D0%B8%D1%80%D1%83%D0%B5%D1%82-%D0%B8%D0%B4%D0%B5%D0%BD%D1%82%D0%B8%D1%84%D0%B8%D0%BA%D0%B0%D1%82%D0%BE%D1%80)
- [Сохраняющиеся сущности](https://github.com/eatae/dddinphp/blob/main/ru-RU/Chapter4/Entities.md#%D1%81%D0%BE%D1%85%D1%80%D0%B0%D0%BD%D1%8F%D1%8E%D1%89%D0%B8%D0%B5%D1%81%D1%8F-%D1%81%D1%83%D1%89%D0%BD%D0%BE%D1%81%D1%82%D0%B8)
  - [Настройка доктрины](https://github.com/eatae/dddinphp/blob/main/ru-RU/Chapter4/Entities.md#%D0%BD%D0%B0%D1%81%D1%82%D1%80%D0%BE%D0%B9%D0%BA%D0%B0-%D0%B4%D0%BE%D0%BA%D1%82%D1%80%D0%B8%D0%BD%D1%8B)
  - [Маппинг Entities](https://github.com/eatae/dddinphp/blob/main/ru-RU/Chapter4/Entities.md#%D0%BC%D0%B0%D0%BF%D0%BF%D0%B8%D0%BD%D0%B3-entities)
    - [Маппинг Entities с помощью аннотированного кода](https://github.com/eatae/dddinphp/blob/main/ru-RU/Chapter4/Entities.md#%D0%BC%D0%B0%D0%BF%D0%BF%D0%B8%D0%BD%D0%B3-entities-%D1%81-%D0%BF%D0%BE%D0%BC%D0%BE%D1%89%D1%8C%D1%8E-%D0%B0%D0%BD%D0%BD%D0%BE%D1%82%D0%B8%D1%80%D0%BE%D0%B2%D0%B0%D0%BD%D0%BD%D0%BE%D0%B3%D0%BE-%D0%BA%D0%BE%D0%B4%D0%B0)
    - [Маппинг Entities с помощью XML](https://github.com/eatae/dddinphp/blob/main/ru-RU/Chapter4/Entities.md#%D0%BC%D0%B0%D0%BF%D0%BF%D0%B8%D0%BD%D0%B3-entities-%D1%81-%D0%BF%D0%BE%D0%BC%D0%BE%D1%89%D1%8C%D1%8E-xml)
    - [Сопоставление Идентификатора Сущности](https://github.com/eatae/dddinphp/blob/main/ru-RU/Chapter4/Entities.md#%D1%81%D0%BE%D0%BF%D0%BE%D1%81%D1%82%D0%B0%D0%B2%D0%BB%D0%B5%D0%BD%D0%B8%D0%B5-%D0%B8%D0%B4%D0%B5%D0%BD%D1%82%D0%B8%D1%84%D0%B8%D0%BA%D0%B0%D1%82%D0%BE%D1%80%D0%B0-%D1%81%D1%83%D1%89%D0%BD%D0%BE%D1%81%D1%82%D0%B8)
    - [Окончательный файл сопоставления](https://github.com/eatae/dddinphp/blob/main/ru-RU/Chapter4/Entities.md#%D0%BE%D0%BA%D0%BE%D0%BD%D1%87%D0%B0%D1%82%D0%B5%D0%BB%D1%8C%D0%BD%D1%8B%D0%B9-%D1%84%D0%B0%D0%B9%D0%BB-%D1%81%D0%BE%D0%BF%D0%BE%D1%81%D1%82%D0%B0%D0%B2%D0%BB%D0%B5%D0%BD%D0%B8%D1%8F)
- [Тестирование Entities](https://github.com/eatae/dddinphp/blob/main/ru-RU/Chapter4/Entities.md#%D1%82%D0%B5%D1%81%D1%82%D0%B8%D1%80%D0%BE%D0%B2%D0%B0%D0%BD%D0%B8%D0%B5-entities)
  - [Дата Время (DateTimes)](https://github.com/eatae/dddinphp/blob/main/ru-RU/Chapter4/Entities.md#%D0%B4%D0%B0%D1%82%D0%B0-%D0%B2%D1%80%D0%B5%D0%BC%D1%8F-datetimes)
    - [Передача всех дат в виде параметров](https://github.com/eatae/dddinphp/blob/main/ru-RU/Chapter4/Entities.md#%D0%BF%D0%B5%D1%80%D0%B5%D0%B4%D0%B0%D1%87%D0%B0-%D0%B2%D1%81%D0%B5%D1%85-%D0%B4%D0%B0%D1%82-%D0%B2-%D0%B2%D0%B8%D0%B4%D0%B5-%D0%BF%D0%B0%D1%80%D0%B0%D0%BC%D0%B5%D1%82%D1%80%D0%BE%D0%B2)
    - [Класс тестирования](https://github.com/eatae/dddinphp/blob/main/ru-RU/Chapter4/Entities.md#%D0%BA%D0%BB%D0%B0%D1%81%D1%81-%D1%82%D0%B5%D1%81%D1%82%D0%B8%D1%80%D0%BE%D0%B2%D0%B0%D0%BD%D0%B8%D1%8F)
    - [External Fake](https://github.com/eatae/dddinphp/blob/main/ru-RU/Chapter4/Entities.md#external-fake)
    - [Рефлексия](https://github.com/eatae/dddinphp/blob/main/ru-RU/Chapter4/Entities.md#%D1%80%D0%B5%D1%84%D0%BB%D0%B5%D0%BA%D1%81%D0%B8%D1%8F)
- [Валидация](https://github.com/eatae/dddinphp/blob/main/ru-RU/Chapter4/Entities.md#%D0%B2%D0%B0%D0%BB%D0%B8%D0%B4%D0%B0%D1%86%D0%B8%D1%8F)
  - [Валидация атрибутов](https://github.com/eatae/dddinphp/blob/main/ru-RU/Chapter4/Entities.md#%D0%B2%D0%B0%D0%BB%D0%B8%D0%B4%D0%B0%D1%86%D0%B8%D1%8F-%D0%B0%D1%82%D1%80%D0%B8%D0%B1%D1%83%D1%82%D0%BE%D0%B2)
  - [Проверка всего объекта](https://github.com/eatae/dddinphp/blob/main/ru-RU/Chapter4/Entities.md#%D0%BF%D1%80%D0%BE%D0%B2%D0%B5%D1%80%D0%BA%D0%B0-%D0%B2%D1%81%D0%B5%D0%B3%D0%BE-%D0%BE%D0%B1%D1%8A%D0%B5%D0%BA%D1%82%D0%B0)
    - [Decoupling сообщений валидации](https://github.com/eatae/dddinphp/blob/main/ru-RU/Chapter4/Entities.md#decoupling-%D1%81%D0%BE%D0%BE%D0%B1%D1%89%D0%B5%D0%BD%D0%B8%D0%B9-%D0%B2%D0%B0%D0%BB%D0%B8%D0%B4%D0%B0%D1%86%D0%B8%D0%B8)
  - [Валидация композиции объектов](https://github.com/eatae/dddinphp/blob/main/ru-RU/Chapter4/Entities.md#%D0%B2%D0%B0%D0%BB%D0%B8%D0%B4%D0%B0%D1%86%D0%B8%D1%8F-%D0%BA%D0%BE%D0%BC%D0%BF%D0%BE%D0%B7%D0%B8%D1%86%D0%B8%D0%B8-%D0%BE%D0%B1%D1%8A%D0%B5%D0%BA%D1%82%D0%BE%D0%B2)
  - [Entities и Events домена](https://github.com/eatae/dddinphp/blob/main/ru-RU/Chapter4/Entities.md#entities-%D0%B8-events-%D0%B4%D0%BE%D0%BC%D0%B5%D0%BD%D0%B0)
- [В заключении](https://github.com/eatae/dddinphp/blob/main/ru-RU/Chapter4/Entities.md#%D0%B2-%D0%B7%D0%B0%D0%BA%D0%BB%D1%8E%D1%87%D0%B5%D0%BD%D0%B8%D0%B8)
  <br>

## [Глава 5. Сервисы (Services)](https://github.com/eatae/dddinphp/blob/main/ru-RU/Chapter5/Services.md#%D1%81%D0%B5%D1%80%D0%B2%D0%B8%D1%81%D1%8B-services)
- [Application Services](https://github.com/eatae/dddinphp/blob/main/ru-RU/Chapter5/Services.md#application-services)
- [Domain Service](https://github.com/eatae/dddinphp/blob/main/ru-RU/Chapter5/Services.md#domain-services)
- [Domain Services и Infrastructure Services](https://github.com/eatae/dddinphp/blob/main/ru-RU/Chapter5/Services.md#domain-services-%D0%B8-infrastructure-services)
- [Проблема переиспользования кода](https://github.com/eatae/dddinphp/blob/main/ru-RU/Chapter5/Services.md#%D0%BF%D1%80%D0%BE%D0%B1%D0%BB%D0%B5%D0%BC%D0%B0-%D0%BF%D0%B5%D1%80%D0%B5%D0%B8%D1%81%D0%BF%D0%BE%D0%BB%D1%8C%D0%B7%D0%BE%D0%B2%D0%B0%D0%BD%D0%B8%D1%8F-%D0%BA%D0%BE%D0%B4%D0%B0)
- [Тестирование Domain Services](https://github.com/eatae/dddinphp/blob/main/ru-RU/Chapter5/Services.md#%D1%82%D0%B5%D1%81%D1%82%D0%B8%D1%80%D0%BE%D0%B2%D0%B0%D0%BD%D0%B8%D0%B5-domain-services)
- [Anemic Domain Models vs Rich Domain Models](https://github.com/eatae/dddinphp/blob/main/ru-RU/Chapter5/Services.md#anemic-domain-models-vs-rich-domain-models)
- [Anemic Domain Model](https://github.com/eatae/dddinphp/blob/main/ru-RU/Chapter5/Services.md#anemic-domain-model)
- [Как избежать Anemic Domain Model](https://github.com/eatae/dddinphp/blob/main/ru-RU/Chapter5/Services.md#%D0%BA%D0%B0%D0%BA-%D0%B8%D0%B7%D0%B1%D0%B5%D0%B6%D0%B0%D1%82%D1%8C-anemic-domain-model)
- [Итоги](https://github.com/eatae/dddinphp/blob/main/ru-RU/Chapter5/Services.md#%D0%B8%D1%82%D0%BE%D0%B3%D0%B8)
<br>

## [Глава 6. Доменные События (Domain Events)](https://github.com/eatae/dddinphp/blob/main/ru-RU/Chapter6/Domain-Events.md#%D0%B3%D0%BB%D0%B0%D0%B2%D0%B0-6-%D0%B4%D0%BE%D0%BC%D0%B5%D0%BD%D0%BD%D1%8B%D0%B5-%D1%81%D0%BE%D0%B1%D1%8B%D1%82%D0%B8%D1%8F-domain-events)
- [Введение](https://github.com/eatae/dddinphp/blob/main/ru-RU/Chapter6/Domain-Events.md#%D0%B2%D0%B2%D0%B5%D0%B4%D0%B5%D0%BD%D0%B8%D0%B5)
- [Определение](https://github.com/eatae/dddinphp/blob/main/ru-RU/Chapter6/Domain-Events.md#%D0%BE%D0%BF%D1%80%D0%B5%D0%B4%D0%B5%D0%BB%D0%B5%D0%BD%D0%B8%D0%B5)
- [Короткая история](https://github.com/eatae/dddinphp/blob/main/ru-RU/Chapter6/Domain-Events.md#%D0%BA%D0%BE%D1%80%D0%BE%D1%82%D0%BA%D0%B0%D1%8F-%D0%B8%D1%81%D1%82%D0%BE%D1%80%D0%B8%D1%8F)
- [Метафора](https://github.com/eatae/dddinphp/blob/main/ru-RU/Chapter6/Domain-Events.md#%D0%BC%D0%B5%D1%82%D0%B0%D1%84%D0%BE%D1%80%D0%B0)
- [Пример из реального мира](https://github.com/eatae/dddinphp/blob/main/ru-RU/Chapter6/Domain-Events.md#%D0%BF%D1%80%D0%B8%D0%BC%D0%B5%D1%80-%D0%B8%D0%B7-%D1%80%D0%B5%D0%B0%D0%BB%D1%8C%D0%BD%D0%BE%D0%B3%D0%BE-%D0%BC%D0%B8%D1%80%D0%B0)
- [Характеристики](https://github.com/eatae/dddinphp/blob/main/ru-RU/Chapter6/Domain-Events.md#%D1%85%D0%B0%D1%80%D0%B0%D0%BA%D1%82%D0%B5%D1%80%D0%B8%D1%81%D1%82%D0%B8%D0%BA%D0%B8)
- [Соглашения по именованию](https://github.com/eatae/dddinphp/blob/main/ru-RU/Chapter6/Domain-Events.md#%D1%81%D0%BE%D0%B3%D0%BB%D0%B0%D1%88%D0%B5%D0%BD%D0%B8%D1%8F-%D0%BF%D0%BE-%D0%B8%D0%BC%D0%B5%D0%BD%D0%BE%D0%B2%D0%B0%D0%BD%D0%B8%D1%8E)
- [Доменные События и Ubiquitous Language](https://github.com/eatae/dddinphp/blob/main/ru-RU/Chapter6/Domain-Events.md#%D0%B4%D0%BE%D0%BC%D0%B5%D0%BD%D0%BD%D1%8B%D0%B5-%D1%81%D0%BE%D0%B1%D1%8B%D1%82%D0%B8%D1%8F-%D0%B8-ubiquitous-language)
- [Неизменяемость (Immutability)](https://github.com/eatae/dddinphp/blob/main/ru-RU/Chapter6/Domain-Events.md#%D0%BD%D0%B5%D0%B8%D0%B7%D0%BC%D0%B5%D0%BD%D1%8F%D0%B5%D0%BC%D0%BE%D1%81%D1%82%D1%8C-immutability)
  - [Symfony Event Dispatcher](https://github.com/eatae/dddinphp/blob/main/ru-RU/Chapter6/Domain-Events.md#symfony-event-dispatcher)
- [Моделирование Событий](https://github.com/eatae/dddinphp/blob/main/ru-RU/Chapter6/Domain-Events.md#%D0%BC%D0%BE%D0%B4%D0%B5%D0%BB%D0%B8%D1%80%D0%BE%D0%B2%D0%B0%D0%BD%D0%B8%D0%B5-%D1%81%D0%BE%D0%B1%D1%8B%D1%82%D0%B8%D0%B9)
  - [Как практическое правило](https://github.com/eatae/dddinphp/blob/main/ru-RU/Chapter6/Domain-Events.md#%D0%BA%D0%B0%D0%BA-%D0%BF%D1%80%D0%B0%D0%BA%D1%82%D0%B8%D1%87%D0%B5%D1%81%D0%BA%D0%BE%D0%B5-%D0%BF%D1%80%D0%B0%D0%B2%D0%B8%D0%BB%D0%BE)
  - [Почему не вся сущность User целиком?](https://github.com/eatae/dddinphp/blob/main/ru-RU/Chapter6/Domain-Events.md#%D0%BF%D0%BE%D1%87%D0%B5%D0%BC%D1%83-%D0%BD%D0%B5-%D0%B2%D1%81%D1%8F-%D1%81%D1%83%D1%89%D0%BD%D0%BE%D1%81%D1%82%D1%8C-user-%D1%86%D0%B5%D0%BB%D0%B8%D0%BA%D0%BE%D0%BC)
- [Doctrine Events](https://github.com/eatae/dddinphp/blob/main/ru-RU/Chapter6/Domain-Events.md#doctrine-events)
- [Сохранение Доменных Событий](https://github.com/eatae/dddinphp/blob/main/ru-RU/Chapter6/Domain-Events.md#%D1%81%D0%BE%D1%85%D1%80%D0%B0%D0%BD%D0%B5%D0%BD%D0%B8%D0%B5-%D0%B4%D0%BE%D0%BC%D0%B5%D0%BD%D0%BD%D1%8B%D1%85-%D1%81%D0%BE%D0%B1%D1%8B%D1%82%D0%B8%D0%B9)
- [Event Store](https://github.com/eatae/dddinphp/blob/main/ru-RU/Chapter6/Domain-Events.md#event-store)
- [Публикация Событий из Domain Model](https://github.com/eatae/dddinphp/blob/main/ru-RU/Chapter6/Domain-Events.md#%D0%BF%D1%83%D0%B1%D0%BB%D0%B8%D0%BA%D0%B0%D1%86%D0%B8%D1%8F-%D1%81%D0%BE%D0%B1%D1%8B%D1%82%D0%B8%D0%B9-%D0%B8%D0%B7-domain-model)
- [Публикация Доменного События из Entity](https://github.com/eatae/dddinphp/blob/main/ru-RU/Chapter6/Domain-Events.md#%D0%BF%D1%83%D0%B1%D0%BB%D0%B8%D0%BA%D0%B0%D1%86%D0%B8%D1%8F-%D0%B4%D0%BE%D0%BC%D0%B5%D0%BD%D0%BD%D0%BE%D0%B3%D0%BE-%D1%81%D0%BE%D0%B1%D1%8B%D1%82%D0%B8%D1%8F-%D0%B8%D0%B7-entity)
- [Публикация ваших Доменных Событий из Domain или Application Services](https://github.com/eatae/dddinphp/blob/main/ru-RU/Chapter6/Domain-Events.md#%D0%BF%D1%83%D0%B1%D0%BB%D0%B8%D0%BA%D0%B0%D1%86%D0%B8%D1%8F-%D0%B2%D0%B0%D1%88%D0%B8%D1%85-%D0%B4%D0%BE%D0%BC%D0%B5%D0%BD%D0%BD%D1%8B%D1%85-%D1%81%D0%BE%D0%B1%D1%8B%D1%82%D0%B8%D0%B9-%D0%B8%D0%B7-domain-%D0%B8%D0%BB%D0%B8-application-services)
- [Как работает Domain Event Publisher](https://github.com/eatae/dddinphp/blob/main/ru-RU/Chapter6/Domain-Events.md#%D0%BA%D0%B0%D0%BA-%D1%80%D0%B0%D0%B1%D0%BE%D1%82%D0%B0%D0%B5%D1%82-domain-event-publisher)
- [Настройка DomainEventListeners](https://github.com/eatae/dddinphp/blob/main/ru-RU/Chapter6/Domain-Events.md#%D0%BD%D0%B0%D1%81%D1%82%D1%80%D0%BE%D0%B9%D0%BA%D0%B0-domaineventlisteners)
- [Тестирование Доменных Событий](https://github.com/eatae/dddinphp/blob/main/ru-RU/Chapter6/Domain-Events.md#%D1%82%D0%B5%D1%81%D1%82%D0%B8%D1%80%D0%BE%D0%B2%D0%B0%D0%BD%D0%B8%D0%B5-%D0%B4%D0%BE%D0%BC%D0%B5%D0%BD%D0%BD%D1%8B%D1%85-%D1%81%D0%BE%D0%B1%D1%8B%D1%82%D0%B8%D0%B9)
- [Распространение новостей в удалённые Ограниченные Контексты](https://github.com/eatae/dddinphp/blob/main/ru-RU/Chapter6/Domain-Events.md#%D1%80%D0%B0%D1%81%D0%BF%D1%80%D0%BE%D1%81%D1%82%D1%80%D0%B0%D0%BD%D0%B5%D0%BD%D0%B8%D0%B5-%D0%BD%D0%BE%D0%B2%D0%BE%D1%81%D1%82%D0%B5%D0%B9-%D0%B2-%D1%83%D0%B4%D0%B0%D0%BB%D1%91%D0%BD%D0%BD%D1%8B%D0%B5-%D0%BE%D0%B3%D1%80%D0%B0%D0%BD%D0%B8%D1%87%D0%B5%D0%BD%D0%BD%D1%8B%D0%B5-%D0%BA%D0%BE%D0%BD%D1%82%D0%B5%D0%BA%D1%81%D1%82%D1%8B)
  - [Messaging](https://github.com/eatae/dddinphp/blob/main/ru-RU/Chapter6/Domain-Events.md#messaging)
    - [Зачем нужен Exchange Name?](https://github.com/eatae/dddinphp/blob/main/ru-RU/Chapter6/Domain-Events.md#%D0%B7%D0%B0%D1%87%D0%B5%D0%BC-%D0%BD%D1%83%D0%B6%D0%B5%D0%BD-exchange-name)
- [Синхронизация Domain Services с REST](https://github.com/eatae/dddinphp/blob/main/ru-RU/Chapter6/Domain-Events.md#%D1%81%D0%B8%D0%BD%D1%85%D1%80%D0%BE%D0%BD%D0%B8%D0%B7%D0%B0%D1%86%D0%B8%D1%8F-domain-services-%D1%81-rest)
- [Итоги](https://github.com/eatae/dddinphp/blob/main/ru-RU/Chapter6/Domain-Events.md#%D0%B8%D1%82%D0%BE%D0%B3%D0%B8)
<br>

## [Глава 7. Модули (Modules)](https://github.com/eatae/dddinphp/blob/main/ru-RU/Chapter7/Modules.md#%D0%B3%D0%BB%D0%B0%D0%B2%D0%B0-7-%D0%BC%D0%BE%D0%B4%D1%83%D0%BB%D0%B8-modules)
- [Общий обзор](https://github.com/eatae/dddinphp/blob/main/ru-RU/Chapter7/Modules.md#%D0%BE%D0%B1%D1%89%D0%B8%D0%B9-%D0%BE%D0%B1%D0%B7%D0%BE%D1%80)
- [Использование Modules в PHP](https://github.com/eatae/dddinphp/blob/main/ru-RU/Chapter7/Modules.md#%D0%B8%D1%81%D0%BF%D0%BE%D0%BB%D1%8C%D0%B7%D0%BE%D0%B2%D0%B0%D0%BD%D0%B8%D0%B5-modules-%D0%B2-php)
- [Пространства имён первого уровня](https://github.com/eatae/dddinphp/blob/main/ru-RU/Chapter7/Modules.md#%D0%BF%D1%80%D0%BE%D1%81%D1%82%D1%80%D0%B0%D0%BD%D1%81%D1%82%D0%B2%D0%B0-%D0%B8%D0%BC%D1%91%D0%BD-%D0%BF%D0%B5%D1%80%D0%B2%D0%BE%D0%B3%D0%BE-%D1%83%D1%80%D0%BE%D0%B2%D0%BD%D1%8F)
- [Namespacing в стиле PEAR](https://github.com/eatae/dddinphp/blob/main/ru-RU/Chapter7/Modules.md#namespacing-%D0%B2-%D1%81%D1%82%D0%B8%D0%BB%D0%B5-pear)
- [Namespacing PSR-0 и PSR-4](https://github.com/eatae/dddinphp/blob/main/ru-RU/Chapter7/Modules.md#namespacing-psr-0-%D0%B8-psr-4)
- [Могут ли два Bounded Contexts находиться в одном Application? И что насчёт обратного варианта?](https://github.com/eatae/dddinphp/blob/main/ru-RU/Chapter7/Modules.md#%D0%BC%D0%BE%D0%B3%D1%83%D1%82-%D0%BB%D0%B8-%D0%B4%D0%B2%D0%B0-bounded-contexts-%D0%BD%D0%B0%D1%85%D0%BE%D0%B4%D0%B8%D1%82%D1%8C%D1%81%D1%8F-%D0%B2-%D0%BE%D0%B4%D0%BD%D0%BE%D0%BC-application-%D0%B8-%D1%87%D1%82%D0%BE-%D0%BD%D0%B0%D1%81%D1%87%D1%91%D1%82-%D0%BE%D0%B1%D1%80%D0%B0%D1%82%D0%BD%D0%BE%D0%B3%D0%BE-%D0%B2%D0%B0%D1%80%D0%B8%D0%B0%D0%BD%D1%82%D0%B0)
- [Структурирование кода в Modules](https://github.com/eatae/dddinphp/blob/main/ru-RU/Chapter7/Modules.md#%D1%81%D1%82%D1%80%D1%83%D0%BA%D1%82%D1%83%D1%80%D0%B8%D1%80%D0%BE%D0%B2%D0%B0%D0%BD%D0%B8%D0%B5-%D0%BA%D0%BE%D0%B4%D0%B0-%D0%B2-modules)
- [Рекомендации по дизайну](https://github.com/eatae/dddinphp/blob/main/ru-RU/Chapter7/Modules.md#%D1%80%D0%B5%D0%BA%D0%BE%D0%BC%D0%B5%D0%BD%D0%B4%D0%B0%D1%86%D0%B8%D0%B8-%D0%BF%D0%BE-%D0%B4%D0%B8%D0%B7%D0%B0%D0%B9%D0%BD%D1%83)
- [Следует ли помещать Repositories, Factories, Domain Events и Services в отдельные подпапки?](https://github.com/eatae/dddinphp/blob/main/ru-RU/Chapter7/Modules.md#%D1%81%D0%BB%D0%B5%D0%B4%D1%83%D0%B5%D1%82-%D0%BB%D0%B8-%D0%BF%D0%BE%D0%BC%D0%B5%D1%89%D0%B0%D1%82%D1%8C-repositories-factories-domain-events-%D0%B8-services-%D0%B2-%D0%BE%D1%82%D0%B4%D0%B5%D0%BB%D1%8C%D0%BD%D1%8B%D0%B5-%D0%BF%D0%BE%D0%B4%D0%BF%D0%B0%D0%BF%D0%BA%D0%B8)
- [Modules в Infrastructure Layer](https://github.com/eatae/dddinphp/blob/main/ru-RU/Chapter7/Modules.md#modules-%D0%B2-infrastructure-layer)
- [Смешивание различных технологий](https://github.com/eatae/dddinphp/blob/main/ru-RU/Chapter7/Modules.md#%D1%81%D0%BC%D0%B5%D1%88%D0%B8%D0%B2%D0%B0%D0%BD%D0%B8%D0%B5-%D1%80%D0%B0%D0%B7%D0%BB%D0%B8%D1%87%D0%BD%D1%8B%D1%85-%D1%82%D0%B5%D1%85%D0%BD%D0%BE%D0%BB%D0%BE%D0%B3%D0%B8%D0%B9)
- [Modules в Application Layer](https://github.com/eatae/dddinphp/blob/main/ru-RU/Chapter7/Modules.md#modules-%D0%B2-application-layer)
- [Итоги](https://github.com/eatae/dddinphp/blob/main/ru-RU/Chapter7/Modules.md#%D0%B8%D1%82%D0%BE%D0%B3%D0%B8)
<br>

- [Оглавление]()
- [Оглавление]()
- [Оглавление]()