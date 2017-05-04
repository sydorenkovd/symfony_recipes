# symfony_recipes
### Я буду писать здесь рецепты для Symfony на русском

* [Принцип работы symfony](#Принцип-работы-symfony)
* [Многие ко многим](#many-to-many)
* [Работа с базой через миграции](#Работа-с-базой-через-миграции)
* [Fixtures and Repository](#fixtures-and-repository)
* [Плагины для PhpStorm](#Плагины-для-phpstorm)
* [ArrayCollection](#arraycollection)
* [Сервисы](#Сервисы)
* [Сервисы twig](#Сервисы-twig)
* [macro в Symfony](#macro-в-symfony)
* [Генерация сущностей из таблиц](#Генерация-сущностей-из-таблиц)
* [MongoDb](#mongodb)
* [Быстрое создание таблиц HTML](#Быстрое-создание-таблиц )
* [Сохранение нескольких изображений в symfony](#Сохранение-нескольких-изображений-в-symfony)
* [Составные шаблоны](#Составные-шаблоны)
* [Быстрое сохранение данных в Symfony](#Быстрое-сохранение-данных-в-symfony)
* [Мультиязычность](#Мультиязычность)

Принцип работы symfony
------------------------

Symfony практически не использует функций ядра, для всего есть свой сервис. Вы можете убедится в этом используя 
```php
php bin/console debug/container log 
```

или просто посмотреть, как работает тот же метод render. 
Который на самом деле вызывает сервис **tempating** со всеми последующими методами этого сервиса, и по сути возвращает тот же html документ.

Работа с базой через миграции
------------------------------
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

Fixtures and Repository
-------------------------
Для создании ложных данных в symfony используются **fixtures** и лучше использовать >**nelmio/alice**, написанную видимо **saldaek**, и использующая faker. Очень удобно и подробная документация. 
Также в symfony очень просто можно создать свои запросы, для этого нужно создать репозиторий под нужную сущность и добавить туда кастомный сложный запрос, хорошо его описав, используя query builder.

-----

Many to many
--------------

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

Плагины для PhpStorm
----------------------

Для работы с Symfony идеально использовать **phpstorm**. К нему есть несколько отличных плагинов 

### (symfony plugin и php annotations) 

которые откроют все возможности для быстрого доступа к методам и свойствам. А названия настолько понятны, что в документацию лезть не приходится.

ArrayCollection
----------------

При связях, Symfony возвращает объект **ArrayCollection** который имеет в себе множество методов. Например метод filter.
```php
$recentNotes = $genus->getNotes()->filter(function (GenusNote $note){
return $note->getCreatedAt() > new \DateTime('-3 month');
});
```
На этом примере мы фильтруем записи по определенному отрезку времени (последние три месяца). На этом магия ArrayCollection не заканчивается.

Сервисы
---------

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

Сервисы twig
-------------

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


macro в Symfony
-----------------


Macro это нечто вроде include, только используешь twig и можно передавать параметры. Массивы, объекты, что угодно. В macro принимаем параметры, прописываем логику шаблона.
Вот пример сложного **macro**:

> Мы отрисовываем удобный редактор для текста в нескольких языковых вариациях, используя расширение [text-angular](https://github.com/textAngular/textAngular) лучшее намой взгляд, обычный WYSIWYG редактор, который гибко работает с Angular.

![пример](https://github.com/sydorenkovd/symfony_recipes/blob/master/assets/1text.png "The translation fields")

Этот macro отрисовывает поля для редактирования текстов в нескольких языковых фарматах.

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
                    {% for lang in langs %}
                            <li class="checked {% if lang == 'ru' %}active{% endif %}" id="LanguageTab_{{ lang }}">
                                <a class="margin-custom" href="#Language_{{ lang }}"
                                data-toggle="tab" data-lang="{{ lang }}">
                                    <label class="icheckbox_minimal-blue">
                                            <input id="langs[{{ lang }}]" type="checkbox" 
                                            class="minimal languageCheckBox" autocomplete="off" 
                                            data-lang="{{ lang }}" checked="" name="langs[{{ lang }}]">
                                    </label><i class="famfamfam-flag-{{ lang }}"></i> Русский </a>
                            </li>
                        {% endfor %}
                </ul>
                <div class="tab-content">
                    {% for lang in langs %}
                        <input type="hidden" ng-model="$formGet.langs.{{ lang }}" 
                        ng-init-value="{% if lang == 'ru' %}Русский{% else %}Украинский{% endif %}">
                        <div class="tab-pane {% if lang == 'ru' %}active{% endif %}" id="Language_{{ lang }}">
                            {% set i = 0 %}
                            {% for field,type in fields %}
                                <div class="form-group">
                                    <label for="{{ field }}">{{ labels[i] }}</label>
                                    {% if type == 'input' %}
                                        <input class="form-control admin_input_string" type="text" 
                                        name="{{ field }}" ng-model="$formGet.{{ field }}.{{ lang }}"
                                        aria-invalid="false" ng-init-value="{{ value[i]|getValueByKey(lang)|raw }}">
                                    {% endif %}
                                    {% if type == 'text' %}
                                        <div ng-model="$formGet.{{ field }}.{{ lang }}" text-angular="text-angular" 
                                        name="{{ field }}[{{ lang }}]" 
                                        ng-init="$formGet.{{ field }}.{{ lang }} = 
                                        '{{ value[i]|getValueByKey(lang)|replace({"'":"&acute;"}) }}'">
                                        </div>
                                    {% endif %}
                                </div>
                                {% set i = i + 1 %}
                            {% endfor %}
                        </div>
                    {% endfor %}
                </div>
            </div>
        </div>
{% endmacro %}
```
Используем глабольные переменные twig для того, чтобы обьявить используемые языки.

```yml
parameters:
    locale: ru
    app.locales: uk
    locale_supported:
      ru_UA: ru
      uk_UA: uk

twig:
    globals:
        langs: '%locale_supported%'
```

Также мы используем сервис **twig**, чтобы получать значения по ключу языка из коллекций, которые мы передаем в macro. Средствами самого twig у меня этого добиться не получилось.

```php
public function getValueByKey($value, $key)
    {
        return $value[$key];
    }
```
Больше о мультиязычности вы можете почитать здесь:
* [Мультиязычность](#Мультиязычность)

К сожалению twig не поддерживает парсинг значений в шаблоне, как это умеет php, но зато мы более наглядно описываем, что нам нужно и также гибко можем это изменять.

Благодаря разного радо **macro** можно шаблонизировать приложение, и это увеличивает скорость разработки кардинально. Лично у меня разного рода input, select, textarea, button, image, images controller и другие загнаны под шаблоны и я их могу быстро и легко настраивать под свои нужды, не дублируя код.

Генерация сущностей из таблиц
------------------------------

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

MongoDb
----------

MongoDb отличная система для хранения данных, и быстрой выборки. Давайте заиспользуем ее используя doctrine.
Для начала нужно установить зависимости DoctrineMongoDBBundle. Есть дока расписывать не буду.
Скажу только что загрузчик composer должен быть перед
```bash
use Doctrine\ODM\MongoDB\Mapping\Driver\AnnotationDriver;
AnnotationDriver::registerAnnotationClasses();
```
иначе он просто не увидит его при автозагрузке.
Отличие при работе с mongo в doctrine в том, что вы работаете с ней как системой хранилища документов и помущаете соотвественно под таким вот namespace
```
namespace Acme\StoreBundle\Document;
```

Также вместо доктрины вы используете сервис **doctrine_mongodb** 

Вы также как и обфычными сущностями можете создавать репозитории для сложных запросов.

Также вы можете связать сущности mongo с mysql сущностями при помощи хитрых манипуляций. Некоторые не рекомендуют этого делать, однако это открывает огромное удобство и позволяет писать меньше кода. Мы же лентяи.

Быстрое создание таблиц 
--------------------------

Symfony великолепен. А мы лентяи. Я покажу как можно создать таблицу с данными при помощи всего "двух" строчек кода. Но это не точно.

В итоге у нас выйдет:

![пример](https://github.com/sydorenkovd/symfony_recipes/blob/master/assets/table.png "The result of our work")

### Начнем
Индексный http метод
```php
    return $this->render('admin/subscriber/index.html.twig');
```
Это все что мы пишем в этом методе, мы просто возващаем view файл.
Теперь давайте загляним в сам view файл.
```twig
{% extends 'admin/admin_base.html.twig' %}
{% block body %}
    {{ forms.table('hottours', ['Тип назначения (страна, курорт)', 'Назначение (страна, курорт)', 'Индекс сортировки', 'Активен', 'Действие']) }}
{% endblock %}
```
Имеет значение здесь только одна строка. Остальное это наследование базового шаблона.
Все! Это и есть наше view и controller. А как мы знаем Symfony *МС фреймворк, так как не имеет слоя моделей как таковых.

Но давайте все таки раскроем то чт опроисходит за сценой.
Во первых macro для такого типа таблиц
```twig
{% macro table(page, fields) %}
<div class="box">
    <div class="box-header">
        <h3 class="box-title">Список</h3>
        <div class="box-tools">
            <a href="/admin/{{ page }}/update" class="btn btn-sm btn-flat btn-success">
                <i class="fa fa-plus"></i> Добавить
            </a>
        </div>
    </div>
    <div class="box-body">
        <table table-admin ajax-url="/admin/{{ page }}/lists" class="table table-striped table-bordered">
            <thead>
            <tr>
                {% for field in fields %}
                <th>{{ field }}</th>
                {% endfor %}
            </tr>
            </thead>
            <tfoot>
            <tr>
                {% for field in fields %}
                <th>{{ field }}</th>
                {% endfor %}
            </tr>
            </tfoot>
            <tbody>
            </tbody>
        </table>
    </div>
    <div class="box-footer"></div>
</div>
{% endmacro %}

```
Мы передаем название страницы для динамических ссылкок. Я не очень люблю path или другие способы создания ссылок.
И также передаем список полей в нашей таблице, точнее это заголовки. Один шаблон для 95% таблиц. Мы его не копипастим из view во view и не нарушаем принцип DRY. Но откуда же данные? Обратите внимание на странный аттрибут **table-admin** именно он тут и заправляеят всем.

Это директива **angularjs** 

```javascript
Admin.directive('tableAdmin', ["$", "$compile", function($, $compile) {
    return {
        scope: {
            'ajaxUrl': '@ajaxUrl'
        },
        link: function($scope, table) {
            var height = $('html').height();
            var tableJ = $(table).DataTable({
                scrollY:        '100%',
                "scrollCollapse": true,
                "pagingType": "full_numbers",
                "stateSave": true,
                "ajax": {
                    'url': $scope.ajaxUrl,
                    'type': 'POST',
                    "dataSrc": "answer"
                },
                "language": {
                    'loadingRecords': 'Загружаем список элементов',
                    'search': 'Поиск в таблице',
                    "lengthMenu": "Показать _MENU_ элементов на страницу",
                    "zeroRecords": "Нечего не найдено",
                    "info": "Показана _PAGE_ страница из _PAGES_, всего _TOTAL_ записей",
                    "infoEmpty": "Нет записей",
                    "infoFiltered": "(фильтруют от _MAX_ всех записей)",
                    "paginate": {
                        "first":      "В начало",
                        "last":       "В конец",
                        "next":       "Следующая",
                        "previous":   "Предыдущая"
                    }
                }
            });
            setTimeout(function() {
                $compile(angular.element(table).find('tr'));
            }, 4000);
        }
    }
}]);
```

Здесь самым важным есть обработка таблиц с помощью [DataTable (плагин)](https://datatables.net "DataTable (плагин)"). Мы считываем со скоупа
```javascript
 scope: {
            'ajaxUrl': '@ajaxUrl'
        },
```
  URL и отправляем AJAX запрос на сервер. Остальные настройки и больше вы можете найти в документации DataTable.
  
  И вот мы подошли к завершающей стадии, обработка запроса.
```php
 $listsHotTours = $this->getDoctrine()->getRepository('Model:BlockHottour')->findAll();
        $listsItems = array_map(function ($item) {
            /** @var BlockHottour $item */
            return [
                $item->getDestinationType() == 'country' ? 'Страна' : 'Курорт',
                $item->getDestinationType() == 'country' ? $item->getCountry()->getName() : $item->getCity()->getName(),
                $item->getSortOrder(),
                $item->getActive() ? '<span class="text-success">Активен' : '<span class="text-warning">Неактивен',
                HTML::tag('div', implode('', [
                    HTML::tag('a', 'Редактировать', ['class' => 'btn btn-flat btn-primary', 'href' => $this->generateUrl('Admin_UpdateHottour', ['id' => $item->getId()])]),
                ]), ['class' => 'btn-group btn-group-sm'])
            ];
        }, $listsHotTours);
        return new JsonResponse(['answer' => $listsItems, 'status_ok' => true]);
```

  Здесь мы при помощи **array_map** строим нужный нам массив, который отдаем на обработку в DataTable и он строит таблицу за нас, а мы можем пользоваться его возможностями, как сортировки колонок,полнотекстовый поиск и многое другое.

Если у вас возникают сложности с **array_map** ознакомьтесь с [документацией] (http://php.net/manual/ru/function.array-map.php "документацией")

Возвращаем json такого вида:

![пример](https://github.com/sydorenkovd/symfony_recipes/blob/master/assets/table_json.png "json")


Согласен, что это не "две" строчки, и то что находится за сценой выглядит ненужной сложностью. Но это ускорит вашу работу по созданию таблиц несколько раз. Сделает быстрой открисовку, так как данные приходят ajax. А сторонние библиотеки сделают за вас монотонную работу не избавляя от гибкости их использования и кастомизации.

Сохранение нескольких изображений в symfony
--------------------------------------------
Например удобно так работать с галереями.
На самом деле этот процес занимает довольно таки много времени и кода. Руководствуясь моим любимым принципом DRY я попробую минифицировать количество строк которое нам необходимо при сохранении нескольких изображении.

Передавать изображения с клиента на сервер мы будем по средством base64 кода. Это удобно и просто.
Будем передавать такого вида json:
![пример](https://github.com/sydorenkovd/symfony_recipes/blob/master/assets/64base.png "base64")

```php
$data = $this->getDataArrayFromRequest($request);
        if (isset($data['image'])) {
            $this->saveImages('Model\Entity\OfficeImage', $data['image'], 'Office', $data['id']);
        }
```

Метод который нас интересует **saveImages**, но вот вы заметили метод **getDataArrayFromRequest** он также шаблонизирован под нужды и превращает из json в удобный массив php контент который мы получаем по средством ajax запроса.

```php
 protected function getDataArrayFromRequest(Request $request)
    {
        return json_decode($request->getContent(), true);
    }
```

Но идея топика в другом. Вернемся к сохранению изображений.

```php

 public function saveImages($modelName, $images, $mainModelName, $mainModelId)
    {
            foreach ($images as $image) {
                $imageId = $this->getDoctrine()->getRepository('Model:Image')->getLastId();
                $saved = $this->get('app.image')->addNewImageBase64($image, $imageId);
                if($saved) { $this->save(new Image()); }
                $model = new $modelName();
                $imageModel = $this->getDoctrine()->getRepository('Model:Image')->find($imageId);
                $mainModel = $this->getDoctrine()->getRepository('Model:'.$mainModelName)->find($mainModelId);
                $model->setImage($imageModel);
                $key = 'set' . $mainModelName;
                $model->$key($mainModel);
                $this->save($model);
            }
    }
    
```

Этот метод у меня определен в базовом контроллере от которого наследуются все контроллеры, которые хотят получить методы-шаблоны. Что здесь происходит?
Для начала мы передаем полное имя модели, сам массив изображений, имя основной модели и идентификатор основной модели.


 * We get lastId, and save image with name of last id
 * if everything doing well we create new Entity Image
 * And after all we create Extra Entities those connect Image and MainTable, MainTableImage for instance
 * and we are setting id of image and main id
-----
 Мы имеем таблицу всех изображений, где храним идентификаторы. Основную таблицу к которой относится "галерея", и промежуточную таблицу. Где связываем идентификаторы изображений с основной таблицей. Стандартные многие ко многим. Но мне нужно немного больше гибкости, по этому я не использвал такой тип связи.
 
 Основная идея здесь, то php может парсить переменные как методы. Мы можем очень динамично работать с переменными и значит использовать один код много раз, меняя только регуляторы, которые в него передаем.
 
 Составные шаблоны
 ------------------
 
 Используем macro внутри macro.
```html

{% macro header(model, backUrl) %}
    {% import _self as forms %}
    <div class="box-header" ng-cloak="">
        <div class="box-title">
            {{ forms.errors }}
        </div>
        <div class="box-tools">
            {{ forms.button('save', '') }}{{ forms.button('delete', model.id) }}{{ forms.button('back', backUrl) }}
        </div>
    </div>
    
{% endmacro %}
```

Таким образом можно формировать постоянные элементы быстро и гибко.

Быстрое сохранение данных в Symfony
-------------------------------------

Вас как и меня наверное раздражает писать много кода в Symfony. Он многих отталкивает своей так называемо монструозностью. Но это не так. Тот же самый CRUD на Symfony пишется очень легко, просто и быстро. Правда, как и все в Symfony требует настройки.

Про index мы уже говорили выше. Create и Update мы соеденяем простой фабрикой.

```php
 $model = $this->factory($id, 'Morfer');
```

Метод factory выглядит так:
```php
 protected function factory($id, $repo, $namespace = null)
    {
        /** @var BaseEntity $model */
        if (!empty($id)) {
            $model = $this->getDoctrine()->getRepository('Model:' . $repo)->find($id);
        } else {
            if (isset($namespace)) {
                $repo = $namespace . $repo;
                $model = new $repo();
            } else {
                $repo = 'Model\Entity\\' . $repo;
                $model = new $repo();
            }
        }
        return $model;
    }
    
```
Но метод сохранения остается всегда самым большим.

```php

public function saveAction(Request $request) {
        $data = $this->getDataArrayFromRequest($request);
        $model = $this->factory($data['id'], 'Morfer');
        $this->save($model->trustValues($data));
        return new JsonResponse(['data' => $data]);
    }
    
```
Три строчки. Не оень то и много. На самом деле можно сократить до одной.

```php

public function trustValues($values) {
        foreach ($values as $field => $val) {
                $method = 'set' . ucfirst($field);
                if(method_exists($this, $method)) {
                    $this->$method($values[$field]);
                }
        }
       return $this;
    }
    
```

Здесь мы создаем сеттеры для свойств которые к нам приходят. Да они должны строго именоватся, чтобы этот метод работал, но если это быстрое сохранение, и эту систему пишете вы, то это не составляет труда. Я ведь пишу, только идеи и примеры реализации, а не конечный вариант.

В этой реализации можно также схитрить, или кастомизировать.
И метод **values** будет такого вида:
```php

 public function values($values, $fields, $relationBindingModels = [])
    {
        foreach ($fields as $field) {
            if (isset($values[$field])) {
                $method = 'set' . ucfirst($field);
                $this->$method($values[$field]);
            }
        }
        if (count($relationBindingModels) > 0) {
            foreach ($relationBindingModels as $name => $model) {
                $method = 'set' . ucfirst($name);
                $this->$method($model);
            }
        }
        return $this;
    }
    
```

В итоге мы получаем гибкую настройку

```php

$model->values($data, 
['active', 'h1', 'metaDescription', 'metaKeywords', 'metaTitle', 'text', 'title'], 
['country' => $country]);

```

Сами задаем значения. Значения которые зранят идентификаторы связи между таблицами в doctrine так просто присвоить нельзя, по-этому мы передаем объект и как бы сохраняем связанную сущность. Это нужно дял целостности данных, чтобы вы не присвоили идентификатор не существующей сущности и не поламали связь.



Мультиязычность
-------------------

![пример](https://github.com/sydorenkovd/symfony_recipes/blob/master/assets/trans.jpg "trans")

Для того, чтобы связывать сущности добавим в SlideTranslation:

```php
    /**
     * @ORM\ManyToOne(targetEntity="Model\Entity\Slide", inversedBy="tranlations")
     * @ORM\JoinColumn(name="translatable_id", referencedColumnName="id")
     */
    private $transtable;

```
А в Slide:
```php

 /**
     * @ORM\OneToMany(targetEntity="Model\Entity\SlideTranslation", mappedBy="transtable")
     * @ORM\JoinColumn(name="id", referencedColumnName="translatable_id")
     */
    private $tranlations;

    public function __construct() {
        $this->tranlations = new ArrayCollection();
    }
    
```

Наши сущности связаны, как только можно. Теперь можем приступить к построению структуры сохранения записей и получения как строк по локации, так и массивом для редактирования в админке.

Первое, что мы делаем, это инициализируем базовую сущность Slide. Я использую фабрику из базового контроллера для того, чтобы не парится с новыми и уже существующими сущностями. Не люблю засорять контроллеры.

```php

 protected function factory($id, $repo, $namespace = null)
    {
        /** @var BaseEntity $model */
        if (!empty($id)) {
            $model = $this->getDoctrine()->getRepository('Model:' . $repo)->find($id);
        } else {
            if (isset($namespace)) {
                $repo = $namespace . $repo;
                $model = new $repo();
                $model->setTranslate($repo . 'Translation');
            } else {
                $repo = 'Model\Entity\\' . $repo;
                $model = new $repo();
                $model->setTranslate($repo . 'Translation');
                if (class_exists($model->getTranslate())) {
                    $langs = Config::load('lang.short');
                    $count = count($langs);
                    for ($i = 0; $i < $count; $i++) {
                        $translationModelName = $model->getTranslate();
                        /** @var BaseEntity $translationModel */
                        $translationModel = new $translationModelName();
                        $translationModel->setLocale($langs[$i]);
                        $model->addTranslationModel($translationModel);
                    }
                }
            }
        }
        return $model;
    }
    
```
Если приходит id то мы возвращаем сущность. Если нет, то смотрим стандартен ли наш namespace. При сохранении новой сущности нам нужно определить является ли она Translatable, то есть имеет ли она поля ,которые нужны в нескольких языковых вариантах. Мы проверяем и записываем новые сущности с таблицы переводов в список. Они будут с нами до конца. Это симулирует связи.

Когда мы маппаем обьект, наполняе его свойствами, то наши сеттеры не такие простые, а специальные. И работает общий метод.

```php
$this->setTranslation(__FUNCTION__, $value);
```

Который определен в базовой сущности, как:

```php
 public function setTranslation($functionName, $data)
    {
        try {
            $translations = lcfirst(str_replace('set', '', $functionName)) . 'Translation';
           if(isset($this->$translations)) {
                   foreach ($this->$translations as $translation) {
                       foreach ($data as $lang => $text) {
                           if ($translation->getLocale() == $lang) {
                               $translation->$functionName($text);
                           }
                       }
                   }
           } else {
               foreach ($this->getTranslationModels() as $translation) {
                   foreach ($data as $lang => $text) {
                       if ($translation->getLocale() == $lang) {
                           $translation->$functionName($text);
                       }
                   }
               }

           }

        } catch (\Exception $e) {
            throw new Exception($e->getMessage());
        }
    }

```

Мы идем по связям, строим имя сеттера, и благодаря специальном соглашении имен и возможностью php парсить переменные как угодно, мы получаем метод, который отрабатывает вне зависимости от сущности и метода. А если эта сущность новая, то мы обращаемся к нашей коллекции обьектов, которые мы описали ,при создании обьекта.

И после того как у нас есть все нужные нам данные в обьектах мы выполняем.

```php
 $transTable = $entity->getTranslationModels();
            $em->persist($entity);
            $em->flush();
            /** @var BaseEntity $model */
            foreach ($transTable as $model) {
               $em->persist($model);
                $em->flush();
                $model->setTranstable($entity);
                $em->persist($model);
                $em->flush();
            }

```
Которое мы вызываем в случае новой сущности. Я пытался обойти двойной **flush** есть даже тема на stacloverflow, но эллегантного решения я пока не нашел. 
