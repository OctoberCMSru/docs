# CMS Layouts

- [Введение](#introduction)
- [Заполнители](#placeholders)
- [Динамические макеты](#dynamic-layouts)

Layouts define the page scaffold, that is everything that repeats on a page, such as a header and footer. Layouts often contain the HTML tag as well as the HEAD, TITLE and BODY tags.

<a name="introduction" class="anchor" id="introduction"></a>
## Введение

Шаблоны компоновки находятся в подкаталоге **/layouts** каталога  **theme**. Файлы шаблонов макетов должны иметь расширение **htm**. Внутри файла макета вы должны использовать тег  `{% page %}` для вывода содержимого страницы. Простейший пример макета:

    <html>
        <body>
            {% page %}
        </body>
    </html>

Чтобы использовать макет [страницы](./cms-pages.md), страница должна указывать имя файла макета (без расширения) в разделе  [Конфигурация](./cms-themes.md#configuration-section). Помните, что если вы ссылаетесь на макет из [подкаталога](./cms-themes.md#subdirectories), вы должны указать имя подкаталога. Пример шаблона страницы с использованием макета default.md:

    url = "/"
    layout = "default"
    ==
    <p>Hello, world!</p>

Когда эта страница запрашивается, ее содержимое объединяется с макетом, или, точнее, - тег `{% page %}` макета заменяется содержимым страницы. Предыдущие примеры генерировали бы следующую разметку:

    <html>
        <body>
            <p>Hello, world!</p>
        </body>
    </html>

Обратите внимание, что вы можете визуализировать  [чанки](./cms-partials.md) в макетах. Это позволяет вам совместно использовать общие элементы разметки между различными макетами. Например, у вас может быть часть, которая выводит ссылки CSS и JavaScript на сайте. Этот подход упрощает управление ресурсами - если вы хотите добавить ссылку на JavaScript, вы должны изменить один частный, а не редактировать все макеты.

Раздел [Конфигурация](./cms-themes.md#configuration-section) является необязательным для макетов. Поддерживаемые параметры конфигурации - это имя (**name**) и описание (**description**). Параметры являются необязательными и используются в пользовательском интерфейсе. Пример шаблона макета с описанием:

    description = "Basic layout example"
    ==
    <html>
        <body>
            {% page %}
        </body>
    </html>

<a name="placeholders" class="anchor" id="placeholders"></a>
## Заполнители

Заполнители позволяет страницам вводить контент в макет. Заполнители определяются в шаблонах макета с тегом `{% placeholder name %}`. В следующем примере показан шаблон макета с заполнителем **head**, определяющий  HTML раздел HEAD.

    <html>
        <head>
            {% placeholder head %}
        </head>
        ...

Страницы могут добавлять контент к заполнителям с тегами `{% put %}` и `{% endput %}`.  В следующем примере демонстрируется простой шаблон страницы, который вставляет ссылку CSS на место заполнителя **head**, определенного в предыдущем примере


    url = "/my-page"
    layout = "default"
    ==
    {% put head %}
        <link href="/themes/demo/assets/css/page.css" rel="stylesheet">
    {% endput %}

    <p>The page content goes here.</p>

У заполнителей может быть содержание по умолчанию (default), которое может быть заменено или дополнено страницей. Пример определения заполнителя в шаблоне макета:

    {% placeholder sidebar default %}
        <p><a href="/contacts">Contact us</a></p>
    {% endplaceholder %}

На странице может быть добавлено больше содержимого для заполнителя. Тег `{% default %}` указывает место, где должно отображаться содержимое заполнителя по умолчанию. Если тег не используется, содержимое заполнителя полностью заменяется.

    {% put sidebar %}
        <p><a href="/services">Services</a></p>
        {% default %}
    {% endput %}

<a name="checking-placeholder-exits" class="anchor" href="#checking-placeholder-exits"></a>
### Проверка наличия заполнителя

В шаблоне макета вы можете проверить наличие содержимого заполнителя с помощью функции `placeholder()`. Это позволяет генерировать разную разметку в зависимости от того, получает ли страница контент на место заполнителя. Пример:

    {% if placeholder('sidemenu') %}
        <!-- Markup for a page with a sidebar -->
        <div class="row">
            <div class="col-md-3">
                {% placeholder sidemenu %}
            </div>
            <div class="col-md-9">
                {% page %}
            </div>
        </div>
    {% else %}
        <!-- Markup for a page without a sidebar -->
        {% page %}
    {% endif %}

<a name="dynamic-layouts" class="anchor" id="dynamic-layouts"></a>
## Динамические макеты

Макеты, например страницы, могут использовать любые функции Twig. Подробную информацию см. В документации по [динамическим страницам](./cms-pages.md#dynamic-pages). В [разделе PHP](./cms-themes.md#php-section) макета вы можете определить следующие функции для обработки жизненного цикла выполнения страницы: `onStart()`, `onBeforePageStart()` и `onEnd()`. Функция  `onStart()` выполняется в начале обработки страницы. Функция `onBeforePageStart()` выполняется после запуска [компонентов](./cms-components.md) макета, но до выполнения функции `onStart()` страницы. Функция `onEnd()` выполняется после отображения страницы. Последовательность, выполняемая обработчиками, следующая:

* Функция `onStart()`шаблона.
* Функция `onRun()` компонентов шаблона.
* Функция`onBeforePageStart()`шаблона.
* Функция `onStart()` страницы.
* Функция `onRun()` компонентов страницы.
* Функция `onEnd()` страницы.
* Функция `onEnd()` шаблона.

