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
- **registerMarkupTags()** - registers additional markup tags that can be used in the CMS.
- **registerNavigation()** - registers back-end navigation items for this plugin.
- **registerPermissions()** - registers any back-end permissions used by this plugin.
- **registerSettings()** - registers any back-end configuration links used by this plugin.
- **registerFormWidgets()** - registers any back-end widgets used by this plugin.
- **registerReportWidgets()** - registers any report widgets, including the dashboard widgets.

<a name="basic-plugin-information" class="anchor" href="#basic-plugin-information"></a>
### Basic plugin information

The `pluginDetails()` is a required method of the plugin registration class. It should return an array containing 4 keys: 

- **name** - the plugin name.
- **description** - the plugin description.
- **author** - the plugin author name.
- **icon** - a name of the plugin icon. October uses [Font Autumn icons](http://daftspunk.github.io/Font-Autumn/). Any icon names provided by this font are valid, for example **icon-glass**, **icon-music**.

<a name="routing-initialization" class="anchor" href="#routing-initialization"></a>
## Routing and initialization

Plugin registration files can contain two methods `boot()` and `register()`. With these methods you can do anything you like, like register routes or attach handlers to events.

The `register()` method is called immediately when the plugin is registered. The `boot()` method is called right before a request is routed. So if your actions rely on another plugin, you should use the boot method. For example, inside the `boot()` method you can extend models:

    public function boot()
    {
        User::extend(function($model) {
            $model->hasOne['author'] = ['Acme\Blog\Models\Author'];
        });
    }

Plugins can also supply a file named **routes.php** that contain custom routing logic, as defined in the [Laravel Routing documentation](http://laravel.com/docs/routing). For example:

    Route::group(['prefix' => 'api_acme_blog'], function() {

        Route::get('cleanup_posts', function(){ return Posts::cleanUp(); });

    });

<a name="component-registration" class="anchor" href="#component-registration"></a>
## Component registration

[Components](components) must be registered in the [Plugin registration file](#registration-file). This tells the CMS about the Component and provides a **short name** for using it. An example of registering a component:

    public function registerComponents()
    {
        return [
            'October\Demo\Components\Todo' => 'demoTodo'
        ];
    }

This will register the Todo component class with the default alias name **demoTodo**. More information on building components can be found at the [Building Components](components) article.

<a name="extending-twig" class="anchor" href="#extending-twig"></a>
## Extending Twig

Custom Twig filters and functions can be registered in the CMS with the `registerMarkupTags()` method of the plugin registration class. The next example registers two Twig filters and two functions.

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
## Widget registration

Plugins can register [form widgets](../backend/widgets#form-widgets) by overriding the `registerFormWidgets()` method in the plugin registration class. The method should return an array containing the widget classes in the keys and widget name and context in the values. Example:

    public function registerFormWidgets()
    {
        return [
            'Backend\FormWidgets\CodeEditor' => [
                'label' => 'Code editor',
                'alias' => 'codeeditor'
            ]
        ];
    }

Plugins can register [report widgets](../backend/widgets#report-widgets) by overriding the `registerReportWidgets()` method in the plugin registration class. The method should return an array containing the widget classes in the keys and widget name and context in the values. Example:

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

The **name** element defines the widget name for the Add Widget popup window. The **context** element defines the context where the widget could be used. October's report widget system allows to host the report container on any page, and the container context name is unique. The widget container on the Dashboard page uses the **dashboard** context.

<a name="navigation-permissions" class="anchor" href="#navigation-permissions"></a>
## Navigation and permissions

Plugins can extend the back-end navigation menus and permissions by overriding methods of the [Plugin registration class](#registration-file). This section shows you how to add menu items and permissions to the back-end navigation area. An example of registering a top-level navigation menu item with two sub-menu items:

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

When you register the back-end navigation you can use localization strings for the `label` values. The localization is described in the [plugin localization](localization) article.

The next example shows how to register back-end permission items. Permissions are defined with a permission key and description. In the back-end permission management user interface permissions are displayed as a checkbox list. Back-end controllers can use permissions defined by plugins for restricting the [user access](../backend/users) to pages or features.

    public function registerPermissions()
    {
        return [
            'acme.blog.access_posts'       => ['label' => 'Manage the blog posts'],
            'acme.blog.access_categories'  => ['label' => 'Manage the blog categories']
        ];
    }

<a name="backend-settings" class="anchor" href="#backend-settings"></a>
## Backend settings

The System / Settings page contains a list of links to the configuration pages. The list of links can be extended by plugins by overriding the `registerSettings()` method of the [Plugin registration class](#registration-file). When you create a configuration link you have two options - create a link to a specific back-end page, or create a link to a settings model. The next example shows how to create a link to a back-end page.

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

The following example creates a link to a settings model. Settings models is a part of the settings API which is described in the [Settings & Config](settings) article.

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
## Migrations and version history

Plugins keep a change log inside the **/updates** directory to maintain version information and database structure. An example of an updates directory structure:

    plugins/
      author/
        myplugin/
          updates/                    <=== Updates directory
            version.yaml              <=== Plugin version file
            create_tables.php         <=== Database scripts
            seed_the_database.php     <=== Migration file
            create_another_table.php  <=== Migration file

The **version.yaml** file, called the *Plugin version file*, contains the version comments and refers to database scripts in the correct order. Please read the [Database structure](../database/structure) article for information about the migration files. This file is required if you're going to submit the plugin to the [Marketplace](../help/marketplace). An example Plugin version file:

    1.0.1:
        - First version
        - create_tables.php
        - seed_the_database.php
    1.0.2: Small fix that uses no scripts
    1.0.3: Another minor fix
    1.0.4:
        - Creates another table for this new feature
        - create_another_table.php

> **Note:** During the development, to apply plugin updates, log out of the back-end and sign in again. The plugin version history is applied when an administrator signs in to the back-end. Plugin updates are applied automatically for plugins installed from the Marketplace when you update the system.
