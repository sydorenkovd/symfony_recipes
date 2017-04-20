# symfony_recipes
### Я буду писать здесь рецепты для Symfony на русском

 * [Many-to-Many](#many-to-many)


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


------

Для работы с Symfony идеально использовать **phpstorm**. К нему есть несколько отличных плагинов 

### (symfony plugin и php annotations) 

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


------
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

-------
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


------
Сохранение нескольких изображений в symfony. Например удобно так работать с галереями.
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
 
 
 --------
 Используем macro внутри macro. Составные шаблоны

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

------------

Быстрое сохранение данных в Symfony

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
