# Регистрация плагина

- [Ознакомление](#introduction)
- [Регистрация файла](#registration-file)
- [Роутинг и инициализация](#routing-initialization)
- [Регистрация компонента](#component-registration)
- [Extending Twig](#extending-twig)
- [Регистрация виджета](#widget-registration)
- [Навигация и ограниченияNavigation and permissions](#navigation-permissions)
- [Настройки Бекэнда](#backend-settings)
- [Миграции и история версий](#migrations-version-history)

Плагины - это основа для расширения функционала CMS. Процесс регистрации плагина позволяет определить функции плагина, такие как [components](components) или менюшки и страницы бек энда. Некоторые примеры того, что можно сделать с помощью плагинов:

- Определить [components](components).
- Определить ограничения пользователя.
- Добавить в бек энд страницы, менюхи и формы.
- Создать структуру базы данных и внести в нее данные.
- Изменить функциональность ядра или других плагинов.
- Описать классы, контроллеры бек энда, представления, виды, и другие файлы.

<a name="introduction" class="anchor" href="#introduction"></a>
## Ознакомление

Все плагины находятся в подпапке **/plugins**. Структура директории плагина выглядит следующим образом:

    plugins/
      acme/              <=== Имя автора
        blog/            <=== Имя плагина
          classes/
          components/
          controllers/
          models/
          updates/
          ...
          Plugin.php     <=== Регистрационный файл плагина

Но не для всех плагинов требуется такая структура. Только те плагины, в которых используется **Plugin.php** нуждаются в такой структуре. Если же Ваш плагин предусматривается только еденичный [component](components), то тогда, структура для такого плагина должна быть гораздо проще, например:

    plugins/
      acme/              <=== Имя автора
        blog/            <=== Имя плагина
          components/
          Plugin.php     <=== Регистрационный файл плагина

> **Помните**: если вы являетесь разработчиком плагина для [Marketplace](../help/marketplace), наличие файла [updates/version.yaml](#migrations-version-history) обязательно.

<a name="namespaces" class="anchor" href="#namespaces"></a>
### Символы в имени плагина

Символы в именах плагинов очень важны, особенно если планируется, что Ваш плагин будет опубликова на [October Marketplace](http://octobercms.com/plugins). Когда Вы регистрируетесь как автор на Marketplace Вам будет предложено ввести авторский код, который как раз таки будет использован в качестве корневого имени директории для всех Ваших плагинов. Вы можете ввести авторский код только один раз, когда проходите регистрацию. По умолчанию Вам будет предложен авторский код от Marketplace, состоящий из Вашего имени и фамилии: VasyaPupkin. Повторимся, что данный код невозможно будет поменять после регистрации. Все Ваши плагины должны будут быть определены в папке `\VasyaPupkin\Blog`.

<a name="registration-file" class="anchor" href="#registration-file"></a>
## Регистрация файла

Файл **Plugin.php**, названный как *Регистрационный файл плагина*, является скриптом инициализации, который объявляет основные функции и содержит в себе информацию о плагине. Регистрационный файлы могут содержать следующее:

- Информацию о плагине, его имя и автора
- Регистрировать методы, для улучшения CMS

Регистрационные скрипты должны использовать имена плагинов. Скрипт регистрации должен определить класс с именем `Plugin` который расширяет `\System\Classes\PluginBase` класс. Единственным обязательным методом класса регистрации плагина является  `pluginDetails()`. Пример файла регистрации плагина:

    namespace Acme\Blog;

    class Plugin extends \System\Classes\PluginBase
    {
        public function pluginDetails()
        {
            return [
                'name' => 'Blog Plugin',
                'description' => 'Provides some really cool blog features.',
                'author' => 'ACME Corporation',
                'icon' => 'icon-leaf'
            ];
        }

        public function registerComponents()
        {
            return [
                'Acme\Blog\Components\Post' => 'blogPost'
            ];
        }
    }

<a name="registration-methods" class="anchor" href="#registration-methods"></a>
### Поддерживаемые методы

Следующие методы поддерживаются в регистрационном файле плагина:

- **pluginDetails()** - возвращает информацию плагина.
- **register()** - зарегистрированный метод, вызывается, когда плагин впервые регистрируется.
- **boot()** - метод загрузки, вызывается непосредственно перед маршрутизацией запроса.
- **registerComponents()** - регистрирует какой либо фронт эндовский компонент, который будет пользоваться этим плагином.
- **registerMarkupTags()** - регистрирует дополнительные теги разметки, который могут быть использованы в CMS.
- **registerNavigation()** - регистрирует бэк эндовские элементы навигации для этого плагина.
- **registerPermissions()** - регистрирует любые бек-эндовские ограничения, который будут использованы этим плагином.
- **registerSettings()** - регистрирует любые ссылки конфигурации бэк энда, используемые этим плагином.
- **registerFormWidgets()** - регистрирует любые виджеты бэк энда, используемые этим плагином.
- **registerReportWidgets()** - регистрирует любые виджеты отчетов, в том числе виджеты дашборда. 

<a name="basic-plugin-information" class="anchor" href="#basic-plugin-information"></a>
### Основная информация плагина

Метод `pluginDetails()` является обязательным методом класса регистрации плагина. Он должен возвращать массив, состоящий из 4 ключей:

- **name** - имя плагина.
- **description** - описание плагина.
- **author** - имя автора плагина.
- **icon** - имя иконки плагина. OctoberCMS пользует [Font Autumn icons](http://daftspunk.github.io/Font-Autumn/). Любое название иконки, предусмотренное этой коллекцией действительно, например **icon-glass**, **icon-music**.

<a name="routing-initialization" class="anchor" href="#routing-initialization"></a>
## Маршрутизация и инициализация

Файлы регистрации плагина могут содержать для метода `boot()` и `register()`. С этими методами Вы можете делать всё, что только пожелаете, допустим зарегистрировать маршруты или пришпилить обработчики к событиям.

Метод `register()` вызывается непосредственно в тот момент, когда плагин регистрируется. Метод `boot()` вызывается прямо перед мршрутизацией запроса. Таким образом если ваши действия зависят от другого плагина, вы должны пользовать метод загрузки. Например, внутри метода `boot()` Вы можете расширить модели:

    public function boot()
    {
        User::extend(function($model) {
            $model->hasOne['author'] = ['Acme\Blog\Models\Author'];
        });
    }

В плагины может быть также вставлен файл с именем **routes.php** в котором будет содержаться пользовательская логика маршрутизации, как это определено в [документация Laravel Маршрутизация](http://laravel.ru/docs/v4/routing). Например:

    Route::group(['prefix' => 'api_acme_blog'], function() {

        Route::get('cleanup_posts', function(){ return Posts::cleanUp(); });

    });

<a name="component-registration" class="anchor" href="#component-registration"></a>
## Регистрация компонента

[Components](components) должен быть зарегистрирован в [Файле регистрации плагина](#registration-file). Это укажет CMS о компоненте и предоставит **short name** для использования. Пример регистрации компонента:

    public function registerComponents()
    {
        return [
            'October\Demo\Components\Todo' => 'demoTodo'
        ];
    }

Этот код регистрирует класс компонента ТуДу с именем алиаса **demoTodo** по умолчанию. Больше информации по созданию компонентов Вы можете найти на страницах [Создание компонентов](components).

<a name="extending-twig" class="anchor" href="#extending-twig"></a>
## Расширения Twig

Пользовательские Twif фильтры и функции могут быть зарегистрированы в CMS с методом класса регистрации плагина `registerMarkupTags()`. Следующий пример регистрирует два Twig фильтра и две функции.

    public function registerMarkupTags()
    {
        return [
            'filters' => [
                // A global function, i.e str_plural()
                'plural' => 'str_plural',

                // A local method, i.e $this->makeTextAllCaps()
                'uppercase' => [$this, 'makeTextAllCaps']
            ],
            'functions' => [
                // A static method call, i.e Form::open()
                'form_open' => ['October\Rain\Html\Form', 'open'],

                // Using an inline closure
                'helloWorld' => function() { return 'Hello World!'; }
            ]
        ];
    }

    public function makeTextAllCaps($text)
    {
        return strtoupper($text);
    }

<a name="widget-registration" class="anchor" href="#widget-registration"></a>
## Регистрация виджета

Плагины могут регистрировать [формы виджетов](../backend/widgets#form-widgets) путем переопределения метода `registerFormWidgets()` в классе регистрации плагина. Этот метод должен возвращать массив, содержащий классы виджетов в ключах, имена виджетов и контекст в значениях. Например:

    public function registerFormWidgets()
    {
        return [
            'Backend\FormWidgets\CodeEditor' => [
                'label' => 'Code editor',
                'alias' => 'codeeditor'
            ]
        ];
    }

Плагины могут регистрировать [отчет виджетов](../backend/widgets#report-widgets) путем переопределения метода `registerReportWidgets()` в классе регистрации плагина. Этот метод должен возвращать массив, содержащий классы виджетов в ключах, а также имя виджета и контекст в значениях. Пример:

    public function registerReportWidgets()
    {
        return [
            'RainLab\GoogleAnalytics\ReportWidgets\TrafficOverview'=>[
                'label'   => 'Google Analytics traffic overview',
                'context' => 'dashboard'
            ],
            'RainLab\GoogleAnalytics\ReportWidgets\TrafficSources'=>[
                'label'   => 'Google Analytics traffic sources',
                'context' => 'dashboard'
            ]
        ];
    }

Тут, элемент **name** определяет имя виджета для кнопки Добавить Виджет во всплывающем окошке. Элемент **context** определяет контекст, где виджет может быть запользован. Система отчетов виджетов OctoberCMS позволяет разместить контейнер с отчетом на любой странице, и имя контейнера контекста должно быть уникальным. Контейнер виджета на дашбордовской странице пользует контекст **dashboard**.

<a name="navigation-permissions" class="anchor" href="#navigation-permissions"></a>
## Навигация и ограничения

Плагины могут расширить в бек-энде навигационные менюшки и ограничения путем переопределения методов [Класс регистрации плагина](#registration-file). В этом разделе показано, как добавить пункты меню и разрешения на области навигации в бек энде. Пример регистрации верхнего уровня навигации менюхи, с двумя пунктами подменю:

    public function registerNavigation()
    {
        return [
            'blog' => [
                'label'       => 'Blog',
                'url'         => Backend::url('acme/blog/posts'),
                'icon'        => 'icon-pencil',
                'permissions' => ['acme.blog.*'],
                'order'       => 500,

                'sideMenu' => [
                    'posts' => [
                        'label'       => 'Posts',
                        'icon'        => 'icon-copy',
                        'url'         => Backend::url('acme/blog/posts'),
                        'permissions' => ['acme.blog.access_posts'],
                    ],
                    'categories' => [
                        'label'       => 'Categories',
                        'icon'        => 'icon-copy',
                        'url'         => Backend::url('acme/blog/categories'),
                        'permissions' => ['acme.blog.access_categories']
                    ],
                ]

            ]
        ];
    }

При регистрации навигации в бек энде, можно пользовать строки локализации для значения `ярлыков`. Локализация описана в статье [плагин локализации](localization).

В следующем примере показано, как зарегистрировать элементы доступа в бек энде. Разрешения определяются с ключом разрешения и описанием. В бек энде для пользователя, который управляет доступом это показано в виде чекбоксов. Контроллеры бек энда могут использовать ограничения, определенные плагином, для ограничения [доступа пользователей](../backend/users) к страницам или функциям.

    public function registerPermissions()
    {
        return [
            'acme.blog.access_posts'       => ['label' => 'Manage the blog posts'],
            'acme.blog.access_categories'  => ['label' => 'Manage the blog categories']
        ];
    }

<a name="backend-settings" class="anchor" href="#backend-settings"></a>
## Настройки бек энда

Страница Система / Настройки содержит список всех параметров для конфигурирования страниц. Список этих параметров может быть увеличен настройками плагинов, путем переопределения метода `registerSettings()` в [Классе регистрации плагина](#registration-file). Когда Вы создаете конфигурационный линк, перед Вами стоит два варианта - создать линк на специфичную страницу бек энда, или же создать линк с настройками модели. Следующий пример показывает, как создать линк на страницу бек энда.

    public function registerSettings()
    {
        return [
            'location' => [
                'label'       => 'Locations',
                'description' => 'Manage available user countries and states.',
                'category'    => 'Users',
                'icon'        => 'icon-globe',
                'url'         => Backend::url('october/user/locations'),
                'order'       => 100
            ]
        ];
    }

А этот пример покажет Вам как создать линк на настройки модели. Настройки модели это часть настроек API, которая описывается в статье [Настройки и Конфигурация](settings).

    public function registerSettings()
    {
        return [
            'settings' => [
                'label'       => 'User Settings',
                'description' => 'Manage user based settings.',
                'category'    => 'Users',
                'icon'        => 'icon-cog',
                'class'       => 'October\User\Models\Settings',
                'order'       => 100
            ]
        ];
    }

<a name="migrations-version-history" class="anchor" href="#migrations-version-history"></a>
## Миграции и история версий

Плагины хранят историю изменений в папке **/updates**. Также в этой директории хранится информация о текущей версии и информацию о структуре базы данных. Например, структура директории обновления может выглядить так:

    plugins/
      author/
        myplugin/
          updates/                    <=== Папка обновлений
            version.yaml              <=== Файл версии плагина
            create_tables.php         <=== Скрипты базы данных
            seed_the_database.php     <=== Миграционный файл
            create_another_table.php  <=== Миграционный файл

Файл **version.yaml** названный как *Файл версии плагина*, содержит комментарии к версии и четкий порядок вызова скриптов базы данных. Пожалуйста, прочтите [структура Базы Данных](../database/structure) для ознкомления с информацией по миграционным файлам. Также этот файл необходим если Вы собрались отправить плагин в [Marketplace](../help/marketplace). Пример файла версии плагина:

    1.0.1:
        - Первая версия
        - create_tables.php
        - seed_the_database.php
    1.0.2: Небольшие правки, неиспользующие скрипты
    1.0.3: Другие незначительные исправления
    1.0.4:
        - Создание другой таблицы для новой фишки
        - create_another_table.php

> **Запомните:** В процессе разработки, чтобы применить обновления плагина необходимо всего лишь выйти из бек энда и авторизоваться вновь. Вся история версии плагина применяется, когда администратор авторизуется в бек энде. Также обновления плагина применяются автоматически, для плагинов, установленных на маркетплайсе, когда Вы обновляете систему. 
