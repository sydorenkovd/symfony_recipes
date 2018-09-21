# symfony_recipes
### Я буду писать здесь рецепты для Symfony на русском

* [Принцип работы symfony](#Принцип-работы-symfony)
* [Ошибки и хаки](#Ошибки-и-хаки)
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
* [Аутентификация](#Аутентификация)
* [Тестирование](#Тестирование)
* [REST](#rest)

P.S.
---------------------
* [Рецепты для PHP/Javascript](#Рецепты-для-phpjavascript)

Предисловие
============

* Это набор практик и проблем с которыми я сталкивался и решал при работе с **symfony**
* Это не guide по **symfony**, и уж точно не лучшие практики, это мои наработки и идеи
* Это не варианты для production, а лишь идеи реализаций, той или иной проблемы

#### Надесь вы найдете для себя что почерпнуть и покритиковать. И самое главное, надеюсь, что это будет полезно.

Принцип работы symfony
------------------------

Symfony практически не использует функций ядра, для всего есть свой сервис. Вы можете убедится в этом используя 
```php
php bin/console debug/container log 
```

или просто посмотреть, как работает тот же метод render. 
Который на самом деле вызывает сервис **tempating** со всеми последующими методами этого сервиса, и по сути возвращает тот же html документ.


Ошибки и хаки
---------------
Обновлять схему таблиц через миграции можно так

```php
$this->addSql('ALTER TABLE pages CHANGE title_txtid title_txtid INT DEFAULT NULL');
```

Другой синтаксис с **ALTER COLUMN** и т.д не поддерживает.


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

```php
public function deleteAllTransportsByCountry($countryId) {
        return $this->createQueryBuilder('countryTransport')
            ->delete('Model:CountryTransport', 's')
            ->where('s.countryId = :countryId')
            ->setParameter('countryId', $countryId)
            ->getQuery()
            ->execute();
    }
```

И после вызываем как 

```php
$em->getRepository('Model:CountryTransport')->deleteAllTransportsByCountry($country->getId());
```
Также мы можем обрабатывать данные из базы в репозиториях и делать разного рода выборки, но не рекомендуется слишком засорять репозитории без надобности. И поверьте, это сделать очень легко.

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

Обновление:

```php
$isNotChanged = ($roles == $userRoles);
       if(!$isNotChanged ) {
           if ( !empty($requestResultArr['id'])) {
               $this->getDoctrine()->getRepository('Model:RoleUser')->clearRoles($user->getId());
           }
           foreach ($roles as $role) {
               $roleModel = $em->getRepository('Model:Role')->findOneBy(['name' => $role]);
               $user->addRole($roleModel);
               $em->persist($roleModel);
           }
       }
        $em->persist($user);
        $em->flush();
```
Сначало смотрим не изменились ли значения, зачем лишний раз дергать базу. Потом сносим все записи. И накатываем новые. Неакуратно, можно сравнивать с уже существующими и добавлять недостающие, но мне лень. 

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
Все! Это и есть наше view и controller. А как мы знаем Symfony *VС фреймворк, так как не имеет слоя моделей как таковых.

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
            $this->saveImages(OfficeImage::class, $data['image'], Office::class, $data['id']);
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
                $mainModel = $this->getDoctrine()->getRepository($mainModelName)->find($mainModelId);
                $model->setImage($imageModel);
                $key = 'set' . substr(strrchr($mainModelName, '/'), 1);
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
 $model = $this->factory($id, Morfer::class);
```

Метод factory выглядит так:
```php
 protected function factory($id, $repo)
    {
        /** @var BaseEntity $model */
        if (!empty($id)) {
            $model = $this->getDoctrine()->getRepository($repo)->find($id);
        } else {
            $model = new $repo();
        }
      //  Also can be used in more elegant way
      //  $model = empty($id) ? new $repo() : $this->getDoctrine()->getRepository($repo)->find($id);
        return $model;
    }
    
```
Но метод сохранения остается всегда самым большим.

```php

public function saveAction(Request $request) {
        $data = $this->getDataArrayFromRequest($request);
        $model = $this->factory($data['id'], Morfer::class);
        $this->save($model->trustValues($data));
        return new JsonResponse(['data' => $data]);
    }
    
```
Три строчки. Не очень то и много. На самом деле можно сократить до одной.

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

Сами задаем значения. Значения которые зранят идентификаторы связи между таблицами в doctrine так просто присвоить нельзя, по-этому мы передаем объект и как бы сохраняем связанную сущность. Это нужно для целостности данных, чтобы вы не присвоили идентификатор не существующей сущности и не поламали связь.



Мультиязычность
-------------------

![пример](https://github.com/sydorenkovd/symfony_recipes/blob/master/assets/trans.jpg "trans")

Создаем таблицу переводов с помощью миграции.

```php
$migration->createTable($schema, 'landing_countries_translation', ['meta_title', 'meta_description', 'meta_keywords', 'html_text']);
```
И наш метод для создания подобных таблиц:

```php
    /**
     * @param Schema $schema
     * @param string $table Name of the new table
     * @param array $translatableFields list of fields
     */
    public function createTable(Schema &$schema, $table, $translatableFields) {
        if(!$schema->hasTable($table)) {
            $mainTable = explode('_', $table)[0];
            $tableNewSchema = $schema->createTable($table);
            $tableNewSchema->addColumn('id', Type::INTEGER, [
                'autoincrement' => true,
            ]);
            $tableNewSchema->setPrimaryKey(['id']);
            $tableNewSchema->addColumn('translatable_id', Type::INTEGER, [
                'notnull' => false
            ]);
            $tableNewSchema->addIndex(['translatable_id'], 'idx_'.$table);
            foreach ($translatableFields as $translatableField) {
                if (strpos($translatableField, '_text') !== false) {
                    $translatableField = str_replace('_text', '', $translatableField);
                    $tableNewSchema->addColumn($translatableField, Type::TEXT, [
                        'notnull' => false
                    ]);
                } else {
                    $tableNewSchema->addColumn($translatableField, Type::STRING, [
                        'notnull' => false
                    ]);
                }
            }
            $tableNewSchema->addColumn('locale', Type::STRING, [
                'length' => 2,
                'notnull' => true
            ]);
        }
    }
```
Можно написать еще методы по работе с миграциями в сервисе **Migration**, и если вам понадобятся в нем методы AbstractMigration класса, то можно передать таким образом. 

```php
        $dd = $this;
        $func = function($query) use (&$dd) {
            $dd->addSql($query);
        };
        $migration->writeInDataIntoTranslatable($schema, 'landing_countries_translation', ['meta_title', 'meta_description', 'meta_keywords', 'html'], $func);
```
И внутри метода **writeInDataIntoTranslatable** вызываем как

```php
$obj("INSERT INTO {$schema->getName()}.{$table} ( ...
```

Можно и через DI из конфигов сервисов, но так мне тоже нравится.


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
                $model->setTranstable($entity);
                $em->persist($model);
                $em->flush();
            }

```
Которое мы вызываем в случае новой сущности. Я пытался обойти двойной **flush** есть даже тема на stacloverflow, но эллегантного решения я пока не нашел. 

Аутентификация
---------------------

Ход работы:
Реализуем полную авторизацию и регистрацию с нуля с помощью Doctrine.

В [репозитории](https://github.com/sydorenkovd/symfony_app) 28/29 июня 2017 и немного раньше, есть реализация по коммитам.

Для начала нужно создать сущность User и реализовать UserInterface
Реализовываем методы интерфейса.

Создаем SecurityController и http метод login

```php
/**
     * @Route("/login", name="login")
     */
    public function loginAction() {
        $authenticationUtils = $this->get('security.authentication_utils');

        // get the login error if there is one
        $error = $authenticationUtils->getLastAuthenticationError();

        // last username entered by the user
        $lastUsername = $authenticationUtils->getLastUsername();

        $form = $this->createForm(LoginForm::class, [
            '_username' => $lastUsername
        ]);
        return $this->render('security/login.html.twig', array(
            'form' => $form->createView(),
            'error'         => $error,
        ));
    }
```

К нему view
```html
 <div class="container">
        <div class="row">
            <div class="col-xs-12">
                {% if error %}
                    <div class="alert alert-danger">{{ error.messageKey|trans(error.messageData, 'security') }}</div>
                {% endif %}
                {{ form_start(form) }}
                {{ form_row(form._username) }}
                {{ form_row(form._password) }}
                <button type="submit" class="btn btn-success">Login <span class="fa fa-lock"></span></button>
                <a href="{{ path('user_register') }}">Register</a>
                {{ form_end(form) }}
            </div>
        </div>
    </div>
```
Чтобы удобно отслеживать ошибки и события установите 
```yml
intercept_redirects: true
```
в config_dev.yml

---------------------------

Теперь создаем сервис login_form_authenticator в котором и происходит все взаимодействие с doctrine

```php
namespace AppBundle\Security;


use AppBundle\FormType\LoginForm;
use Doctrine\ORM\EntityManager;
use Symfony\Component\Form\FormFactoryInterface;
use Symfony\Component\HttpFoundation\Request;
use Symfony\Component\Routing\RouterInterface;
use Symfony\Component\Security\Core\Exception\AuthenticationException;
use Symfony\Component\Security\Core\User\UserInterface;
use Symfony\Component\Security\Core\User\UserProviderInterface;
use Symfony\Component\Security\Guard\Authenticator\AbstractFormLoginAuthenticator;

class LoginFormAuthenticator extends AbstractFormLoginAuthenticator
{

    /**
     * @var FormFactoryInterface
     */
    private $formFactory;
    /**
     * @var EntityManager
     */
    private $em;
    /**
     * @var RouterInterface
     */
    private $router;

    public function __construct(FormFactoryInterface $formFactory, EntityManager $entityManager, RouterInterface $router)
     {

         $this->formFactory = $formFactory;
         $this->em = $entityManager;
         $this->router = $router;
     }

    public function getCredentials(Request $request)
     {
            $isLoginSubmit = $request->getPathInfo() == '/login' && $request->isMethod('POST');
            if (!$isLoginSubmit) {
                    return null;
         }
         $form = $this->formFactory->create(LoginForm::class);
         $form->handleRequest($request);
         $data = $form->getData();
         return $data;

     }

     public function getUser($credentials, UserProviderInterface $userProvider)
     {
            $username = $credentials['_username'];
            return $this->em->getRepository('AppBundle:User')->findOneBy(['email' => $username]);
     }

     public function checkCredentials($credentials, UserInterface $user)
     {
           $password = $credentials['_password'];

           if($password == 'ilike') {
                   return true;
        }
        return false;
     }

     protected function getLoginUrl()
     {
            return $this->router->generate('login');
     }
     protected function getDefaultSuccessRedirectUrl() {
            $this->router->generate('genus_notes');
        }


 }
```
На данном этапе все просто, и мы используем очень простые провеки, а вместо пароля строку. Но в дальнейшем мы реализуем механизм шифрования.

Также нужно зарегистрировать наш сервис
```
app.security.login_form_auth:
       class: AppBundle\Security\LoginFormAuthenticator
       autowire: true
```
И прописать его в security.yml

```
 main:
            anonymous: ~
            guard:
                authenticators:
                    - app.security.login_form_authenticator
                   
```
также добавляем провайдер
```
providers:
        our_users:
            entity: { class: AppBundle\Entity\User, property: email }
          
```

Чтобы в случае неверного пароля сохранялся наш username добавим запись в сессию в методе getCredentials

```php
$request->getSession()->set(
            Security::LAST_USERNAME, $data['_username']
        );
```
И так у нас есть Login, нужен и logout. В Doctrine это очень просто.

В основном фаерволе прописываем security.yml

```
 logout:
        path: /logout
```
И в SecurityController делаем метод logoutAction, в нем лучше бросить какой-нибудь exception

```php
throw new \Exception('this should not be reached');
```
потому что этот метод не должен отрабатывать.

-------------------------

Дальше нам нужно порабоать над паролем.
Добавим новое поле в User plainPassword, он будет служить как помощник, хранилище, и в базу само собой писать его не нужно. Или где-либо сохранять.


```php
public function setPlainPassword($plainPassword)
     {
         $this->plainPassword = $plainPassword;
         $this->password = null;
     }
     
     public function eraseCredentials()
      {
        $this->plainPassword = null;
      }
     
```

При создании новых пользователей работает hash_password_listener этот сервис кодирует пароль и сохраняет его.

```php
namespace AppBundle\Doctrine;


use AppBundle\Entity\User;
use Doctrine\Common\EventSubscriber;
use Doctrine\ORM\Event\LifecycleEventArgs;
use Symfony\Component\Security\Core\Encoder\UserPasswordEncoder;

class HashPasswordListener implements EventSubscriber
{

    /**
     * @var UserPasswordEncoder
     */
    private $passwordEncoder;

    public function __construct(UserPasswordEncoder $passwordEncoder)
    {
        $this->passwordEncoder = $passwordEncoder;
    }

    public function prePersist(LifecycleEventArgs $args) {
        $entity = $args->getEntity();
        if(!$entity instanceof User) {
            return;
        }
      $this->encodePassword($entity);
    }
    public function preUpdate(LifecycleEventArgs $args) {
        $entity = $args->getEntity();
        if(!$entity instanceof User) {
            return;
        }
        $this->encodePassword($entity);
        $em = $args->getEntityManager();
        $meta = $em->getClassMetadata(get_class($entity));
        $em->getUnitOfWork()->recomputeSingleEntityChangeSet($meta, $entity);
    }

    public function getSubscribedEvents()
    {
        return ['prePersist', 'preUpdate'];
    }

    /**
     * @param User $entity
     */
    private function encodePassword(User $entity) {
        $encoded = $this->passwordEncoder->encodePassword($entity, $entity->getPlainPassword());
        $entity->setPassword($encoded);
    }
}
```
Регистрируем сервис в services.yml
```
 app.doctrine.hash_password_listener:
      class: AppBundle\Doctrine\HashPasswordListener
      autowire: true
      tags:
          - { name: doctrine.event_subscriber } 
```
Добавляем способ шифрования

```
encoders:
        AppBundle\Entity\User: bcrypt
```
И меняем проверку пароля в checkCredentials
добавляем с помощью DI сервис кодирования паролей

```php
use Symfony\Component\Security\Core\Encoder\UserPasswordEncoder

if($this->passwordEncoder->isPasswordValid($user, $password)) {
           return true;
       }
```

------------------------------

***Роли

Роли сохраняем как 
```php
  /**
    * @ORM\Column(type="json_array")
    */
    private $roles = [];
    .......
    
$roles = $this->roles;
        if(!in_array('ROLE_USER', $roles)) {
             $roles[] = 'ROLE_USER';
        }
        return $roles;
```

И проверяем в контроллере
```
* @Security("is_granted('ROLE_USERS')")
```

Сделать блокировку на отдельную часть:

```yml
access_control:
- { path: ^/admin, roles: IS_AUTHENTICATED_FULLY }
```
У вас при успешном входе в систему будет доступен пользователь

```
{{ app.user.username }}

$this->getUser()->getUsername()
```

Можно также строить дерево роле, иерархию.

```
role_hierarchy:
        ROLE_ADMIN: [ROLE_MANAGE_ITEMS, ROLE_ALLOWED_TO_SWITCH]
```
Таким образом админу доступно управлять записями и также смотреть страницы от другого пользователя
об этом вы можете почитать в [документации](https://symfony.com/doc/current/security/impersonating_user.html)


Также полезная вещь, что вы можете создать форму регистрации и после успеха сразу логинится под только что зарегистрированным пользователем

```php
 return $this->get('security.authentication.guard_handler')->authenticateUserAndHandleSuccess(
              $user,
              $request,
              $this->get('app.security.login_form_authenticator'),
              'main'
            );
```

В процессе логирования происходят ридеректы, и чтобы они не выглядели страшно, работает сервис redirect_listener
он перехватывает 301-й редирект и выводит кастомный шаблон.

```php
namespace Admin\Doctrine;

use Symfony\Component\Templating\EngineInterface;
use Symfony\Component\HttpKernel\Event\FilterResponseEvent;
use Symfony\Component\HttpFoundation\RedirectResponse;

class RedirectListener
{
    protected $templating;

    public function __construct(EngineInterface $templating)
    {
        $this->templating = $templating;
    }

    public function onKernelResponse(FilterResponseEvent $event)
    {
        $response = $event->getResponse();

        if (!($response instanceof RedirectResponse)) {
            return;
        }

        $uri  = $response->getTargetUrl();
        $html = $this->templating->render(
            '301.html.twig',
            array('uri' => $uri)
        );

        $response->setContent($html);
    }
}

```

Регистрируем
```
app.redirect_listener:
        class: Admin\Doctrine\RedirectListener
        arguments: [ '@templating' ]
        tags:
             - { name: kernel.event_listener, event: kernel.response, method: onKernelResponse }

```

При запросах работает сервис request_listener который проверяет имеет ли пользователь со своими правами доступ к запрашиваемой странице.

```php
namespace Admin\Doctrine;


use Doctrine\ORM\EntityManager;
use Symfony\Component\HttpKernel\Event\GetResponseEvent;
use Symfony\Component\HttpKernel\HttpKernel;
use Symfony\Component\HttpKernel\HttpKernelInterface;
use Symfony\Component\Security\Core\Authentication\Token\Storage\TokenStorage;

class RequestListener
{
    private $em;
    private $token;
    public function __construct(EntityManager $em, TokenStorage $token)
    {
        $this->em = $em;
        $this->token = $token;
    }

    public function onKernelRequest(GetResponseEvent $event)
    {
        if (strpos($event->getRequest()->getUri(), '/admin') !== false) {
// some actions
        } else {
            return;
        }
    }
}

```

Регистрируем как 

```
app.request_listener:
        class: Admin\Doctrine\RequestListener
        arguments: ['@doctrine.orm.entity_manager', '@security.token_storage']
        tags:
            - { name: kernel.event_listener, event: kernel.request, method: onKernelRequest }

```

Тестирование
--------------
В Symfony 4.1 появилась нативаня возможность тестировать приватные сервисы, с поощью специального контейнера 
```php
        $client = static::createClient();
        $container = $client->getContainer();
       
       // или альтернативный способ
        $container = self::$container;
```

Тестирование API:
Наследуем WebTestCase, который нам позволит иметь нативный клиент для запросов в тестовом окружении. Но мне больше нравиться guzzle по этому я в основном ставлю его.

```php
public function setUp()
    {
        $this->guzzle = new Client(
            [
                'base_uri' => self::$container->getParameter('uri'),
                'verify'   => false
            ]
        );
    }
```
Также для тестирования приватных методов вам наверняка понадобиться такая конструкция.
```php
 protected function invokeMethod(string $methodName, array $data)
    {
        $reflection = new \ReflectionClass(get_class($this->util));
        $method = $reflection->getMethod($methodName);
        $method->setAccessible(true);

        return $method->invokeArgs($this->util, $data);
    }
```

С помощью рефлексии можно сделать метод доступным и вызвать его.

```php
public function tearDown()
    {
        gc_collect_cycles();
    }
```

После выполнения тестов запустить сборщик мусора, чтобы устранить циклические ссылки.  [Подробнее](http://php.net/manual/en/features.gc.collecting-cycles.php)

Также для начала тестирования вам необходимы знания phpUnit в частности очень полезные вещи как:

```php
    /**
     * @covers ManagerVoipController::login()
     */
```
Что позволит соеденить тест и end-point который он тестирует. И 

```php
    /**
     * @depends testActiveDepends
     */
```
позволит сделать зависимости между тестами. Например создаете ложные записи, после возвращаете Id и проверяете все необходиомое, после прохождения удаляете. Помогает более явно тестировать систему, при этом сохраняя правильность данных даже девовской среды.

Все конфиги окружения для тестирования храняться в **phpunit.xml**

```xml
  <php>
        <ini name="error_reporting" value="-1" />
        <env name="KERNEL_CLASS" value="App\Kernel" />
        <env name="APP_ENV" value="test" />
        <env name="APP_DEBUG" value="1" />
        <env name="APP_SECRET" value="secret" />
        <env name="SHELL_VERBOSITY" value="-1" />
        <env name="APP_URI" value="http://0.0.0.0:8000/" />
        ...
    </php>
```
Также есть еще другие рекомендации и синтаксис, но с этим проще уже ознакамливатся по задачам с помощью официальных доков.

REST
-----------------

Ведется разработка. Скоро будет материал.

Рецепты для PHP/Javascript
-----------------

### Расчет длины маршрута перелетов с учетом часовых поясов.

У нас есть дата отправления и дата прибытия, например (2017-08-11 12:00 с Киева) и (2017-08-11 15:00 в Вашингтон) по виду три часа лету, на самом деле 9 часов.

Для того, чтобы по датам определять длительность перелета, нам нужно две даты и временную зону аэрапорта вылета. Вот и все данные. Временная зона в формате "Europe/Kiev". В простой реализации всего этого нам поможет [moment.js](https://momentjs.com/)

```javascript

  var fromTime = moment.tz(checkTimestampOnValid(fromTimestamp), timezone).format('HH:mm');
  var toTime = moment.tz(checkTimestampOnValid(toTimestamp), timezone).format('HH:mm');
  var duration = moment.utc(moment(toTime, "HH:mm").diff(moment(fromTime, "HH:mm"))).format("HH:mm");

```
Чтобы работать с timezone нужно подключить [данные для moment.js](https://momentjs.com/downloads/moment-timezone-with-data.js). 
Получаем время вылета, после чего время прилета рассчитываем относительно временной зоны вылета. После чего просим **moment.js** расчитать разницу для точности.

```javascript
    var durationArr = duration.split(':');
    if (durationArr[0].charAt(0) == '0') {
        durationArr[0] = durationArr[0].substr(1);
    }
    var textReturn = '';
    if (durationArr[0] != '0') {
        textReturn += durationArr[0] + " " + ___("ч") + ". ";
    }
    return textReturn + durationArr[1] + " " + ___("мин") + '.';
```
После можем превратить строку ("9:20") в удобочитаемую **9 ч. 20 мин.** и подогнать под несколько локаций.

Сделать текст многоязычным просто. 

```javascript
if(typeof thisLang == 'undefined') {
    thisLang = 'ru';
}
function ___(text, replace) {
    if(thisLang == 'ru') {
        return text;
    } else if(labelsTwo.hasOwnProperty(text)) {
        if(typeof replace !== 'undefined') {
            var returnText = labelsTwo[text].clone();
            for (var keyFind in replace) {
                returnText = returnText.replace(keyFind, replace[keyFind]);
            }
            return returnText;
        } else {
            return labelsTwo[text];
        }
    } else {
        return text;
    }
}
```
Язык инициализируется

```javascript
thisLang = document.getElementsByTagName('html')[0].getAttribute('lang');
```

А ***labelsTwo** это объект с вариациями.
```javascript
labelsTwo = {
    'Вылет': 'Виліт',
    ... ...
    }
    
```
Также можно сделать несколько языков при необходимости

```javascript
var labels = {
    'ru': {
        'from': 'Вылет',
        ... ...
        }
     'uk': {...}
```
Немного изменив метод **___()** также можно получать данные. Все просто.

-------------------

### Обновление данных в Myql

Иногда так бывает, что нужно обновить данные в одной таблице исользуя данные из другой таблице,плюс к ним применить какие-то фильтры и зависимости.

```sql
SET SQL_SAFE_UPDATES = 0;

update viktor.persons per, 
(   SELECT id, p_id 
    FROM viktor.persons_box 
) box
 set per.connection_type_id = 2 where per.id = box.p_id

SET SQL_SAFE_UPDATES = 1;

```
Лучше такие манипуляции проделывать черех миграции, и вытаскивать нужные id, с проверками на то существуют ли они, а можно и вот так захаркодить.

--------------------

### Тернарный оператор

Можно избежать больших if-statements если заиспользовать тернарный оператор. Но не рекомендуется выходить за 140 символов.

```php

$input = ['name' => 'value'];
$output = isset($input['key']) ?: isset($input['name']) ? 'yes' : 'no';

var_dump($output); // yes

```
