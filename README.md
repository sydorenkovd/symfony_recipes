# symfony_recipes
### Я буду писать здесь рецепты для Symfony на русском
Symfony практически не использует функций ядра, для всего есть свой сервис. Вы можете убедится в этом используя 
```php
php bin/console debug/container log 
```

или просто посмотреть, как работает тот же метод render. 
Который на самом деле вызывает сервис **tempating** со всеми последующими методами этого сервиса, и по сути возвращает тот же html документ.

-------
```php
doctrine:schema:update —force
```
поможет создать таблицу на основе объекта. Также редактируя объект можно добавлять поля в таблицу. Но лучшим способом являются миграции, которые работаю в symfony просто восхитительно. 
Редактируешь объект, после чего для генерации миграции запускаешь 
```php
doctrine:migrations:diff
```
и генерируется sql запрос, который редактирует таблицу путем запуска с консоли 
```php
doctrine:migrations:migrate.
```
-----
Для создании ложных данных в symfony используются **fixtures** и лучше использовать >**nelmio/alice**, написанную видимо **saldaek**, и использующая faker. Очень удобно и подробная документация. 
Также в symfony очень просто можно создать свои запросы, для этого нужно создать репозиторий под нужную сущность и добавить туда кастомный сложный запрос, хорошо его описав, используя query builder.

-----

Многие ко многим в symfony работаю чудеснейшим образом. 

Пример кода:
```php
class User
{
/**
* Many Users have Many Groups.
* @ORM\ManyToMany(targetEntity="Group", inversedBy="users")
* @ORM\JoinTable(name="roles_users")
*/
private $roles;

public function __construct() {
$this->roles = new ArrayCollection();
}
============================
class Role
{
/**
* Many Roles have Many Users.
* @ORM\ManyToMany(targetEntity="User", mappedBy="roles")
*/
private $users;

public function __construct() {
$this->users = new ArrayCollection();
}
```
Использование:
```php
<li><a>Количество ролей <span class="pull-right badge bg-blue">{{ user.roles.count }}</span></a></li>
```
Считаем количество ролей конкретного пользователя. Roles мы получаем **Persistentcollection** которая имеет методы стандартного итератора и методы для работы с коллекциями, в общем все что душе угодно, любые манипуляции.


------

Для работы с Symfony идеально использовать **phpstorm**. К нему есть несколько отличных плагинов 

#####(symfony plugin и php annotations) 

которые откроют все возможности для быстрого доступа к методам и свойствам. А названия настолько понятны, что в документацию лезть не приходится.

------

При связях, Symfony возвращает объект **ArrayCollection** который имеет в себе множество методов. Например метод filter.
```php
$recentNotes = $genus->getNotes()->filter(function (GenusNote $note){
return $note->getCreatedAt() > new \DateTime('-3 month');
});
```
На этом примере мы фильтруем записи по определенному отрезку времени (последние три месяца). На этом магия ArrayCollection не заканчивается.

-------

Создавать сервисы в symfony очень просто. Создаем свой сервис в папке service. При необходимости мы можем добавлять уже существующие сервисы в помощь нашему. Используем **Dependency Injection** и передаем сервисы в конструктор, нужные для сервисов классы можно найти с помощью консольной команды php bin/console debug:container "имя сервиса". После чего регистрируем наш сервис в symfony в конфиге **services.yml**.
Пример:
```yml
services:
app.markdown_transformer:
class: AppBundle\Service\MarkdownTransformer
arguments: ['@markdown.parser', '@doctrine_cache.providers.my_markdown_cache']
```
И теперь используем его:
```php
$transformer = $this->get('app.markdown_transformer');

```
Наш сервис готов.


------

Также есть возможность написать сервис для шаблонизатора twig. Для этого создадим папку twig в AppBundle и создадим наше расширение, унаследуем его от **Twig_Extension**, и реализуем методы **getFilters()**
```php
return [
new \Twig_SimpleFilter('markdownify', [$this, 'parseMarkdown'])
];
```

где первый аргумент название нашего фильтра, а второй метод по которому он будет работать, фильтровать.
Также регистрируем его как twig сервис 
```php
app.markdown_extension:
class: AppBundle\Twig\MarkdownExtension
tags:
- { name: twig.extension}
arguments: ['@app.markdown_transformer']

```
Теперь наш сервис можно использовать **{{string|markdownify}}**

Можно использовать сервис из прошлого примера и реализовать метод parseMarkdown так:
```php
return $this->markdownTransformer->parse($str);
```
Использовав **dependencyInjection**. Кстати symfony творит чудеса, нам не нужно нигде создавать объект, мы все уже прописали в .yml массив arguments это и есть наш сервис который будет передан в качестве зависимости.

------
macro в Symfony.

Macro это нечто вроде include, только используешь twig и можно передавать параметры. Массивы, объекты, что угодно. В macro принимаем параметры, прописываем логику шаблона.
Вот пример сложного **macro**:
```php
{{ forms.multilang(
['Заголовок страницы', 'Мета-теги', 'Содержание страницы'],
{'title': 'input', 'meta': 'input', 'text': 'text'},
[page.title, page.meta, page.text]
) }}

```

```twig
{% macro multilang(label, field, value) %}
    <div class="box-body">
        <div class="nav-tabs-custom">
            <ul class="nav nav-tabs">
                <li class="checked active" id="LanguageTab_ru">
                    <a class="margin-custom" href="#Language_ru" data-toggle="tab" data-lang="ru">
                        <label class="icheckbox_minimal-blue">
                            <div class="icheckbox_minimal-blue checked" aria-checked="false" aria-disabled="false" style="position: relative;">
                                <input id="langs[ru]" type="checkbox" class="minimal languageCheckBox" autocomplete="off" data-lang="ru" checked="" name="langs[ru]"></div>
                        </label>
                        <i class="famfamfam-flag-ru"></i> Русский                </a>
                </li>
                <li class="checked" id="LanguageTab_uk">
                    <a class="margin-custom" href="#Language_uk" data-toggle="tab" data-lang="uk">
                        <label class="icheckbox_minimal-blue">
                            <div class="icheckbox_minimal-blue checked" aria-checked="false" aria-disabled="false" style="position: relative;"><input id="langs[uk]" type="checkbox" class="minimal languageCheckBox" autocomplete="off" data-lang="uk" checked="" name="langs[uk]"></div>
                        </label>
                        <i class="famfamfam-flag-ua"></i> Українська                </a>
                </li>
            </ul>
            <div class="tab-content">
                <div class="tab-pane active" id="Language_ru">
                    <input type="hidden" ng-model="$formGet.langs.ru" ng-init-value="Русский">
                    <div class="form-group">
                        <label for="html">{{ label }}</label>
                        <div ng-model="$formGet.{{ field }}.ru" text-angular="" ng-init="$formGet.{{ field }}.ru = '{{ value.ru|escape|raw }}'"></div>
                    </div>
                </div>
                <div class="tab-pane " id="Language_uk">
                    <input type="hidden" ng-model="$formGet.langs.uk" ng-init-value="Українська">
                    <div class="form-group">
                        <label for="html">{{ label }}</label>
                        <div ng-model="$formGet.{{ field }}.uk" text-angular="" ng-init="$formGet.{{ field }}.uk = '{{ value.uk|escape|raw }}'"></div>
                    </div>
                </div>
            </div>
        </div>
    </div>
{% endmacro %}
```
Этот macro отрисовывает поля для редактирования текстов в двух языковых фарматах.
К сожалению twig не поддерживает парсинг значений в шаблоне, как это умеет php, но зато мы более наглядно описываем, что нам нужно и также гибко можем это изменять.



-----------
Жутко раздражает создавать постоянно сущности в доктрине. Я ленивый хочу найти выход. И он есть. Можно их генерировать с базы данных.
```bash
php bin/console doctrine:mapping:import --force AppBundle xml
```
Эта команда сгенерит xml который будет содержать данные о всех таблицах. Также с помощью дополнительных параметров можно сделать это только для определенных таблиц
```bash
 php bin/console doctrine:mapping:import --force AppBundle xml --filter="Table"

```

После чего необходимо выполнить комманды:
```bash
 php bin/console doctrine:mapping:convert annotation ./src
 php bin/console doctrine:generate:entities AppBundle
```

Первая сгенерирует аннотации, вторая сами сущности. Если не хотите использовать аннотации ... используйте аннотации.
