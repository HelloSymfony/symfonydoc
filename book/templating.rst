.. index::
   single: Templating

Creating and using Templates
创建并使用模板
============================

As you know, the :doc:`controller </book/controller>` is responsible for
handling each request that comes into a Symfony2 application. In reality,
the controller delegates the most of the heavy work to other places so that
code can be tested and reused. When a controller needs to generate HTML,
CSS or any other content, it hands the work off to the templating engine.
In this chapter, you'll learn how to write powerful templates that can be
used to return content to the user, populate email bodies, and more. You'll
learn shortcuts, clever ways to extend templates and how to reuse template
code.
现在你已经知道，:doc:`controller </book/controller>`可以处理发送到symfony2中的请求。
事实上，控制器处理了大部分的工作，这样代码可以被测试和重复使用。当一个控制器要集成HTML,css或
其他内容时，它会把这个工作转给模板引擎。在这一章，你将学习可以将内容返回给客户端的功能强大的模板，
以及如何创建邮件内容等等。你将学到扩展模板以及重复使用模板代码的便捷方式。

.. index::
   single: Templating; What is a template?

Templates
模板
---------

A template is simply a text file that can generate any text-based format
(HTML, XML, CSV, LaTeX ...). The most familiar type of template is a *PHP*
template - a text file parsed by PHP that contains a mix of text and PHP code:
一个模板仅仅是一个text文件，它可以用来集成任何以text为基础的格式（HTML,XML,CSV,LaTeX...）。
最常用的就是PHP模板——一个通过PHP解析的text文件，中间包含了text和PHP代码的混合体：

.. code-block:: html+php

    <!DOCTYPE html>
    <html>
        <head>
            <title>Welcome to Symfony!</title>
        </head>
        <body>
            <h1><?php echo $page_title ?></h1>

            <ul id="navigation">
                <?php foreach ($navigation as $item): ?>
                    <li>
                        <a href="<?php echo $item->getHref() ?>">
                            <?php echo $item->getCaption() ?>
                        </a>
                    </li>
                <?php endforeach; ?>
            </ul>
        </body>
    </html>

.. index:: Twig; Introduction

But Symfony2 packages an even more powerful templating language called `Twig`_.
Twig allows you to write concise, readable templates that are more friendly
to web designers and, in several ways, more powerful than PHP templates:
但在symfony2中有一个功能更强大的模板语言`Twig`_。twig允许你编写简明、可读性强的代码并且在
某些方面比PHP更强大：

.. code-block:: html+jinja

    <!DOCTYPE html>
    <html>
        <head>
            <title>Welcome to Symfony!</title>
        </head>
        <body>
            <h1>{{ page_title }}</h1>

            <ul id="navigation">
                {% for item in navigation %}
                    <li><a href="{{ item.href }}">{{ item.caption }}</a></li>
                {% endfor %}
            </ul>
        </body>
    </html>

Twig defines two types of special syntax:
twig定义了两种特殊语法：

* ``{{ ... }}``: "Says something": prints a variable or the result of an
  expression to the template;
* ``{{ ... }}``: 表示向模板输出一个变量或一个表达式的值；

* ``{% ... %}``: "Does something": a **tag** that controls the logic of the
  template; it is used to execute statements such as for-loops for example.
* ``{% ... %}``: 这个标签控制这模板的逻辑，它可以执行for循环之类的语句。

.. note::

   There is a third syntax used for creating comments: ``{# this is a comment #}``.
   This syntax can be used across multiple lines like the PHP-equivalent
   ``/* comment */`` syntax.
   还有一种语法，可以用来注释： ``{# this is a comment #}``。
   它可以用于多行，类似于PHP的``/* comment */``语法。

Twig also contains **filters**, which modify content before being rendered.
The following makes the ``title`` variable all uppercase before rendering
it:
twig还包含过滤器（filter），它可以在提交前修改变量值。以下的过滤器使得title变量里的所有字母都大写：

.. code-block:: jinja

    {{ title|upper }}

Twig comes with a long list of `tags`_ and `filters`_ that are available
by default. You can even `add your own extensions`_ to Twig as needed.
twig有许多默认的标签（tag）和过滤器（filter）。你还可以添加扩展（参见`add your own extensions`_）。

.. tip::

    Registering a Twig extension is as easy as creating a new service and tagging
    it with ``twig.extension`` :ref:`tag<reference-dic-tags-twig-extension>`.
    注册一个twig扩展很容易，参见:ref:`tag<reference-dic-tags-twig-extension>`。

As you'll see throughout the documentation, Twig also supports functions
and new functions can be easily added. For example, the following uses a
standard ``for`` tag and the ``cycle`` function to print ten div tags, with
alternating ``odd``, ``even`` classes:
twig可以支持方法，还可以由你自己添加新方法。比如，以下范例使用了一个for标签和cycle方法
来输出十个div标签，并相间地使用odd和even来作为class属性。

.. code-block:: html+jinja

    {% for i in 0..10 %}
        <div class="{{ cycle(['odd', 'even'], i) }}">
          <!-- some HTML here -->
        </div>
    {% endfor %}

Throughout this chapter, template examples will be shown in both Twig and PHP.
在本章中，模板的范例会同时用twig和php来显示。

.. sidebar:: Why Twig?

    Twig templates are meant to be simple and won't process PHP tags. This
    is by design: the Twig template system is meant to express presentation,
    not program logic. The more you use Twig, the more you'll appreciate
    and benefit from this distinction. And of course, you'll be loved by
    web designers everywhere.
    twig被设计得很简单，而且它不会执行php标签。twig模板仅用于显示，而不是执行逻辑。
    你使用twig越多，你就越能感受到它的简洁。而且对于跟你合作的web设计者，也会觉得方便。

    Twig can also do things that PHP can't, such as true template inheritance
    (Twig templates compile down to PHP classes that inherit from each other),
    whitespace control, sandboxing, and the inclusion of custom functions
    and filters that only affect templates. Twig contains little features
    that make writing templates easier and more concise. Take the following
    example, which combines a loop with a logical ``if`` statement:
    twig还可以做php所不能做的事情，比如模板继承（twig模板可以被编译成继承其他类的类），空格删除，
    sandboxing，以及包含仅在模板中使用的过滤器和方法。twig包含的功能很简单，所以编写模板的工作也
    相对变得简单。比如以下例子中，将if语句和循环语句合并在一起：

    .. code-block:: html+jinja

        <ul>
            {% for user in users %}
                <li>{{ user.username }}</li>
            {% else %}
                <li>No users found</li>
            {% endfor %}
        </ul>

.. index::
   pair: Twig; Cache

Twig Template Caching
twig模板缓存
~~~~~~~~~~~~~~~~~~~~~

Twig is fast. Each Twig template is compiled down to a native PHP class
that is rendered at runtime. The compiled classes are located in the
``app/cache/{environment}/twig`` directory (where ``{environment}`` is the
environment, such as ``dev`` or ``prod``) and in some cases can be useful
while debugging. See :ref:`environments-summary` for more information on
environments.
twig很快。每个twig模板都在输出前被编译成了php类。被编译的php类存储在``app/cache/{environment}/twig``
目录下（``{environment}``就是你使用的环境，比如dev或prod），当调试的时候可能会很有用。

When ``debug`` mode is enabled (common in the ``dev`` environment), a Twig
template will be automatically recompiled when changes are made to it. This
means that during development you can happily make changes to a Twig template
and instantly see the changes without needing to worry about clearing any
cache.
当在debug模式下（通常也是dev环境中），当被修改时，一个twig文件会自动被重编译。这表示在开发过程中，
当你修改一个twig模板后，你可以马上看到修改结果，而不必清空缓存。

When ``debug`` mode is disabled (common in the ``prod`` environment), however,
you must clear the Twig cache directory so that the Twig templates will
regenerate. Remember to do this when deploying your application.
当debug模式不被激活（通常在prod环境中），你必须清空twig缓存目录，这样twig模板才会被重集成。当开发你的
应用时记住这一点。

.. index::
   single: Templating; Inheritance

Template Inheritance and Layouts
模板继承和布局
--------------------------------

More often than not, templates in a project share common elements, like the
header, footer, sidebar or more. In Symfony2, we like to think about this
problem differently: a template can be decorated by another one. This works
exactly the same as PHP classes: template inheritance allows you to build
a base "layout" template that contains all the common elements of your site
defined as **blocks** (think "PHP class with base methods"). A child template
can extend the base layout and override any of its blocks (think "PHP subclass
that overrides certain methods of its parent class").
通常，模板总是会分享常用的元素，比如header，footer，sidebar或其他。在symfony2中，我们这样看待
这个问题：一个模板可以被另一个覆盖。这就像php的类一样道理：模板继承机制允许你创建一个基本模板，
这个基本模板可以包含你网站的所有常用元素并定义它们为block。一个子模板可以扩展基本模板并覆盖它的block。

First, build a base layout file:
首先，创建一个基本模板文件：

.. configuration-block::

    .. code-block:: html+jinja

        {# app/Resources/views/base.html.twig #}
        <!DOCTYPE html>
        <html>
            <head>
                <meta http-equiv="Content-Type" content="text/html; charset=utf-8" />
                <title>{% block title %}Test Application{% endblock %}</title>
            </head>
            <body>
                <div id="sidebar">
                    {% block sidebar %}
                    <ul>
                        <li><a href="/">Home</a></li>
                        <li><a href="/blog">Blog</a></li>
                    </ul>
                    {% endblock %}
                </div>

                <div id="content">
                    {% block body %}{% endblock %}
                </div>
            </body>
        </html>

    .. code-block:: html+php

        <!-- app/Resources/views/base.html.php -->
        <!DOCTYPE html>
        <html>
            <head>
                <meta http-equiv="Content-Type" content="text/html; charset=utf-8" />
                <title><?php $view['slots']->output('title', 'Test Application') ?></title>
            </head>
            <body>
                <div id="sidebar">
                    <?php if ($view['slots']->has('sidebar')): ?>
                        <?php $view['slots']->output('sidebar') ?>
                    <?php else: ?>
                        <ul>
                            <li><a href="/">Home</a></li>
                            <li><a href="/blog">Blog</a></li>
                        </ul>
                    <?php endif; ?>
                </div>

                <div id="content">
                    <?php $view['slots']->output('body') ?>
                </div>
            </body>
        </html>

.. note::

    Though the discussion about template inheritance will be in terms of Twig,
    the philosophy is the same between Twig and PHP templates.
    虽然我们用twig来讨论模板继承，但原理和php模板是一样的。

This template defines the base HTML skeleton document of a simple two-column
page. In this example, three ``{% block %}`` areas are defined (``title``,
``sidebar`` and ``body``). Each block may be overridden by a child template
or left with its default implementation. This template could also be rendered
directly. In that case the ``title``, ``sidebar`` and ``body`` blocks would
simply retain the default values used in this template.
这个模板定义了一个简单的两栏式页面的基本HTML框架。在这个例子中，定义了3个``{% block %}``区域（``title``,
``sidebar``和``body``）。每个block都可以被一个子模板覆盖，当然也可以不覆盖，这样输出的仍是它里面的内容。
这个模板还可以被直接输出。这样的话``title``, ``sidebar``和``body`` block还会保持它里面的默认内容。

A child template might look like this:
一个子模板看起来是这样：

.. configuration-block::

    .. code-block:: html+jinja

        {# src/Acme/BlogBundle/Resources/views/Blog/index.html.twig #}
        {% extends '::base.html.twig' %}

        {% block title %}My cool blog posts{% endblock %}

        {% block body %}
            {% for entry in blog_entries %}
                <h2>{{ entry.title }}</h2>
                <p>{{ entry.body }}</p>
            {% endfor %}
        {% endblock %}

    .. code-block:: html+php

        <!-- src/Acme/BlogBundle/Resources/views/Blog/index.html.php -->
        <?php $view->extend('::base.html.php') ?>

        <?php $view['slots']->set('title', 'My cool blog posts') ?>

        <?php $view['slots']->start('body') ?>
            <?php foreach ($blog_entries as $entry): ?>
                <h2><?php echo $entry->getTitle() ?></h2>
                <p><?php echo $entry->getBody() ?></p>
            <?php endforeach; ?>
        <?php $view['slots']->stop() ?>

.. note::

   The parent template is identified by a special string syntax
   (``::base.html.twig``) that indicates that the template lives in the
   ``app/Resources/views`` directory of the project. This naming convention is
   explained fully in :ref:`template-naming-locations`.
   父模板是通过一个特定语句指定的(``::base.html.twig``)，它表示这个模板被置于``app/Resources/views``
   目录下。命名规则详见:ref:`template-naming-locations`。

The key to template inheritance is the ``{% extends %}`` tag. This tells
the templating engine to first evaluate the base template, which sets up
the layout and defines several blocks. The child template is then rendered,
at which point the ``title`` and ``body`` blocks of the parent are replaced
by those from the child. Depending on the value of ``blog_entries``, the
output might look like this:
模板继承的关键是``{% extends %}``标签。这告诉模板引擎首先查看基本模板，这个基本模板
设置了布局并定义了数个block。然后提交子模板，这时父模板中的``title``和``body`` block
都被子模板中的相应部分替换掉了。根据``blog_entries``的值，最后的输出结果会是这样的：

.. code-block:: html

    <!DOCTYPE html>
    <html>
        <head>
            <meta http-equiv="Content-Type" content="text/html; charset=utf-8" />
            <title>My cool blog posts</title>
        </head>
        <body>
            <div id="sidebar">
                <ul>
                    <li><a href="/">Home</a></li>
                    <li><a href="/blog">Blog</a></li>
                </ul>
            </div>

            <div id="content">
                <h2>My first post</h2>
                <p>The body of the first post.</p>

                <h2>Another post</h2>
                <p>The body of the second post.</p>
            </div>
        </body>
    </html>

Notice that since the child template didn't define a ``sidebar`` block, the
value from the parent template is used instead. Content within a ``{% block %}``
tag in a parent template is always used by default.
注意，由于子模板没有定义sidebar block，父模板的值会被使用。父模板中``{% block %}``标签中的值总是默认被使用的。

You can use as many levels of inheritance as you want. In the next section,
a common three-level inheritance model will be explained along with how templates
are organized inside a Symfony2 project.
你可以使用任意多层继承。在下一节中，将解释一个常见的三重继承模式以及模板在symfony2中是如何组织的。

When working with template inheritance, here are some tips to keep in mind:
当使用模板继承的时候，记住以下三点：

* If you use ``{% extends %}`` in a template, it must be the first tag in
  that template.
  如果你在模板中使用``{% extends %}``，必须将它作为该模板的第一个标签。

* The more ``{% block %}`` tags you have in your base templates, the better.
  Remember, child templates don't have to define all parent blocks, so create
  as many blocks in your base templates as you want and give each a sensible
  default. The more blocks your base templates have, the more flexible your
  layout will be.
  你的基本模板中的``{% block %}``标签越多越好。记住，子模板不用定义父模板中的所有block，所以为了
  灵活起见，你可以尽可能多地在父模板中定义block。

* If you find yourself duplicating content in a number of templates, it probably
  means you should move that content to a ``{% block %}`` in a parent template.
  In some cases, a better solution may be to move the content to a new template
  and ``include`` it (see :ref:`including-templates`).
  如果你发现自己正在好几个模板中复制内容，这可能意味着你应该在父模板中定义一个block并将内容移至那里。
  在某些情况下，还可以将内容移至一个新的模板文件并include它（参见:ref:`including-templates`）。

* If you need to get the content of a block from the parent template, you
  can use the ``{{ parent() }}`` function. This is useful if you want to add
  to the contents of a parent block instead of completely overriding it:
  如果你需要从父模板的block中取出内容，你可以使用``{{ parent() }}``方法。如果你想要将
  父模板block中的内容添加到子模板中，而不是完全覆盖它，就可以使用这个方法：

    .. code-block:: html+jinja

        {% block sidebar %}
            <h3>Table of Contents</h3>
            ...
            {{ parent() }}
        {% endblock %}

.. index::
   single: Templating; Naming Conventions
   single: Templating; File Locations

.. _template-naming-locations:

Template Naming and Locations
模板命名和位置
-----------------------------

By default, templates can live in two different locations:
默认情况下，模板可以存放在两个不同位置：

* ``app/Resources/views/``: The applications ``views`` directory can contain
  application-wide base templates (i.e. your application's layouts) as well as
  templates that override bundle templates (see
  :ref:`overriding-bundle-templates`);
* ``app/Resources/views/``: application views目录可以包含可以在整个应用中使用的基本模板，
  以及可以覆盖bundle模板的模板；

* ``path/to/bundle/Resources/views/``: Each bundle houses its templates in its
  ``Resources/views`` directory (and subdirectories). The majority of templates
  will live inside a bundle.
* ``path/to/bundle/Resources/views/``: 每个bundle都将它的模板存放在它的``Resources/views``
  目录下。大部分模板都会被放置在bundle中。

Symfony2 uses a **bundle**:**controller**:**template** string syntax for
templates. This allows for several different types of templates, each which
lives in a specific location:
symfony2使用**bundle**:**controller**:**template**语法来指示模板。这可以指示数个不同类型的模板，
或被放置在不同位置的模板：

* ``AcmeBlogBundle:Blog:index.html.twig``: This syntax is used to specify a
  template for a specific page. The three parts of the string, each separated
  by a colon (``:``), mean the following:
* ``AcmeBlogBundle:Blog:index.html.twig``: 这个语法是用来指定某个页面的模板的。这个语句
  有三个部分，每个部分都用冒号（:）隔开，它们表示：

    * ``AcmeBlogBundle``: (*bundle*) the template lives inside the
      ``AcmeBlogBundle`` (e.g. ``src/Acme/BlogBundle``);
    * ``AcmeBlogBundle``: (*bundle*)这个模板被置于``AcmeBlogBundle``中（也就是``src/Acme/BlogBundle``）；

    * ``Blog``: (*controller*) indicates that the template lives inside the
      ``Blog`` subdirectory of ``Resources/views``;
    * ``Blog``: (*controller*) 这个模板被置于``Resources/views``下的``Blog``子目录中；

    * ``index.html.twig``: (*template*) the actual name of the file is
      ``index.html.twig``.
    * ``index.html.twig``: (*template*) 这个文件的名称是``index.html.twig``。

  Assuming that the ``AcmeBlogBundle`` lives at ``src/Acme/BlogBundle``, the
  final path to the layout would be ``src/Acme/BlogBundle/Resources/views/Blog/index.html.twig``.
  假设``AcmeBlogBundle``是在``src/Acme/BlogBundle``下的，这个模板的最终路径就是``src/Acme/BlogBundle/Resources/views/Blog/index.html.twig``。

* ``AcmeBlogBundle::layout.html.twig``: This syntax refers to a base template
  that's specific to the ``AcmeBlogBundle``. Since the middle, "controller",
  portion is missing (e.g. ``Blog``), the template lives at
  ``Resources/views/layout.html.twig`` inside ``AcmeBlogBundle``.
* ``AcmeBlogBundle::layout.html.twig``: 这个语句表示``AcmeBlogBundle``指定的基本模板。由于中间
  控制器的部分缺失（blog），这个模板被置于``AcmeBlogBundle``中的``Resources/views/layout.html.twig``。

* ``::base.html.twig``: This syntax refers to an application-wide base template
  or layout. Notice that the string begins with two colons (``::``), meaning
  that both the *bundle* and *controller* portions are missing. This means
  that the template is not located in any bundle, but instead in the root
  ``app/Resources/views/`` directory.
* ``::base.html.twig``: 这个语句表示整个应用都可以使用的基本模板。注意这个语句的开头有两个冒号（::），这表示
   *bundle*和*controller* 部分都缺失。这个模板不在任何bundle中，而在根目录的``app/Resources/views/``中。

In the :ref:`overriding-bundle-templates` section, you'll find out how each
template living inside the ``AcmeBlogBundle``, for example, can be overridden
by placing a template of the same name in the ``app/Resources/AcmeBlogBundle/views/``
directory. This gives the power to override templates from any vendor bundle.
在:ref:`overriding-bundle-templates`这一节中，你将学习各个``AcmeBlogBundle``中的文件可以通过将一个相同
名称的文件放置在``app/Resources/AcmeBlogBundle/views/``中，从而达到覆盖的目的。这样做可以覆盖任意bundle中的任意文件。

.. tip::

    Hopefully the template naming syntax looks familiar - it's the same naming
    convention used to refer to :ref:`controller-string-syntax`.
    模板命名是不是看起来很熟悉：它和:ref:`controller-string-syntax`差不多。

Template Suffix
模板后缀
~~~~~~~~~~~~~~~

The **bundle**:**controller**:**template** format of each template specifies
*where* the template file is located. Every template name also has two extensions
that specify the *format* and *engine* for that template.
**bundle**:**controller**:**template**格式可以确定模板文件在哪里放置。每个模板名称也有两个扩展，
这两个扩展指定了这个模板的格式和引擎。

* **AcmeBlogBundle:Blog:index.html.twig** - HTML format, Twig engine

* **AcmeBlogBundle:Blog:index.html.php** - HTML format, PHP engine

* **AcmeBlogBundle:Blog:index.css.twig** - CSS format, Twig engine

By default, any Symfony2 template can be written in either Twig or PHP, and
the last part of the extension (e.g. ``.twig`` or ``.php``) specifies which
of these two *engines* should be used. The first part of the extension,
(e.g. ``.html``, ``.css``, etc) is the final format that the template will
generate. Unlike the engine, which determines how Symfony2 parses the template,
this is simply an organizational tactic used in case the same resource needs
to be rendered as HTML (``index.html.twig``), XML (``index.xml.twig``),
or any other format. For more information, read the :ref:`template-formats`
section.
默认情况下，任何symfony2模板可以用twig或php两种方式编写，扩展的后缀(e.g. ``.twig`` or ``.php``)指定了
哪种引擎应该被使用。扩展的第一个部分(e.g. ``.html``, ``.css``, etc)指定了模板将要集成的最终格式。
引擎决定了symfony2将如何解析模板，但这个部分只是决定同一源文件要以哪种方式提交，如HTML(``index.html.twig``),
XML (``index.xml.twig``)，等等。参阅:ref:`template-formats`一节。

.. note::

   The available "engines" can be configured and even new engines added.
   See :ref:`Templating Configuration<template-configuration>` for more details.
   引擎（engine）可以被配置。参阅:ref:`Templating Configuration<template-configuration>`。

.. index::
   single: Templating; Tags and Helpers
   single: Templating; Helpers

Tags and Helpers
标签和helper方法
----------------

You already understand the basics of templates, how they're named and how
to use template inheritance. The hardest parts are already behind you. In
this section, you'll learn about a large group of tools available to help
perform the most common template tasks such as including other templates,
linking to pages and including images.
你现在已经学习了模板的基础，它们如何命名以及如何继承。在这节，你将学习如何使用已有的
一些工具来做常用的工作，比如包含其他模板，链接到页面以及包含图像。

Symfony2 comes bundled with several specialized Twig tags and functions that
ease the work of the template designer. In PHP, the templating system provides
an extensible *helper* system that provides useful features in a template
context.
symfony2内置了一些twig标签和方法，它们可以使模板设计变得简单。在php中，模板系统提供扩展的helper系统，
这个系统在在模板环境中有许多可用功能。

We've already seen a few built-in Twig tags (``{% block %}`` & ``{% extends %}``)
as well as an example of a PHP helper (``$view['slots']``). Let's learn a
few more.
我们已经知道了一些内置的twig标签（``{% block %}`` & ``{% extends %}``），以及php helper（``$view['slots']``）。
以下我们将学习更多。

.. index::
   single: Templating; Including other templates

.. _including-templates:

Including other Templates
包含其他模板
~~~~~~~~~~~~~~~~~~~~~~~~~

You'll often want to include the same template or code fragment on several
different pages. For example, in an application with "news articles", the
template code displaying an article might be used on the article detail page,
on a page displaying the most popular articles, or in a list of the latest
articles.
往往你需要将同样的模板或代码片段应用到多个不同页面。比如，在一个新闻文章显示应用中，
这个显示文章的模板代码可能要在文章详细展示页面使用，或者在显示最热门文章页面使用，或者在
显示最新文章页面中使用。

When you need to reuse a chunk of PHP code, you typically move the code to
a new PHP class or function. The same is true for templates. By moving the
reused template code into its own template, it can be included from any other
template. First, create the template that you'll need to reuse.
当你需要重复使用一段php代码时，你一般会将这段代码移至一个新的php类或函数。对模板也是一样。通过将要重复使用的
模板代码移至一个独立的文件，它就可以被其他任何模板包含了。首先，创建一个你想重复使用的模板。

.. configuration-block::

    .. code-block:: html+jinja

        {# src/Acme/ArticleBundle/Resources/views/Article/articleDetails.html.twig #}
        <h2>{{ article.title }}</h2>
        <h3 class="byline">by {{ article.authorName }}</h3>

        <p>
            {{ article.body }}
        </p>

    .. code-block:: html+php

        <!-- src/Acme/ArticleBundle/Resources/views/Article/articleDetails.html.php -->
        <h2><?php echo $article->getTitle() ?></h2>
        <h3 class="byline">by <?php echo $article->getAuthorName() ?></h3>

        <p>
            <?php echo $article->getBody() ?>
        </p>

Including this template from any other template is simple:
从其他模板中包含这个模板很容易：

.. configuration-block::

    .. code-block:: html+jinja

        {# src/Acme/ArticleBundle/Resources/Article/list.html.twig #}
        {% extends 'AcmeArticleBundle::layout.html.twig' %}

        {% block body %}
            <h1>Recent Articles<h1>

            {% for article in articles %}
                {% include 'AcmeArticleBundle:Article:articleDetails.html.twig' with {'article': article} %}
            {% endfor %}
        {% endblock %}

    .. code-block:: html+php

        <!-- src/Acme/ArticleBundle/Resources/Article/list.html.php -->
        <?php $view->extend('AcmeArticleBundle::layout.html.php') ?>

        <?php $view['slots']->start('body') ?>
            <h1>Recent Articles</h1>

            <?php foreach ($articles as $article): ?>
                <?php echo $view->render('AcmeArticleBundle:Article:articleDetails.html.php', array('article' => $article)) ?>
            <?php endforeach; ?>
        <?php $view['slots']->stop() ?>

The template is included using the ``{% include %}`` tag. Notice that the
template name follows the same typical convention. The ``articleDetails.html.twig``
template uses an ``article`` variable. This is passed in by the ``list.html.twig``
template using the ``with`` command.
通过使用``{% include %}``标签，这个模板被包含了。注意这个模板名称遵循同样的命名规则。
``articleDetails.html.twig``使用了一个article变量，这个变量是从``list.html.twig``模板中通过with命令传递的。

.. tip::

    The ``{'article': article}`` syntax is the standard Twig syntax for hash
    maps (i.e. an array with named keys). If we needed to pass in multiple
    elements, it would look like this: ``{'foo': foo, 'bar': bar}``.
    ``{'article': article}``语句是一个twig语法，它其实是一个array，key则是冒号前面的那个article。如果
    要传递多个元素，会像这样：``{'foo': foo, 'bar': bar}``。

.. index::
   single: Templating; Embedding action

.. _templating-embedding-controller:

Embedding Controllers
嵌入控制器
~~~~~~~~~~~~~~~~~~~~~

In some cases, you need to do more than include a simple template. Suppose
you have a sidebar in your layout that contains the three most recent articles.
Retrieving the three articles may include querying the database or performing
other heavy logic that can't be done from within a template.
某些情况下，你需要做比仅仅包含一个简单模板更多的工作。比如在你布局的一个sidebar中要包括三个
最新的文章。要获取这些文章，你可能需要请求数据库或做一些逻辑性强的工作，而这些工作在一个模板中
是不能被完成的。

The solution is to simply embed the result of an entire controller from your
template. First, create a controller that renders a certain number of recent
articles:
解决方法是将一个控制器的结果嵌入你的模板中。首先，创建一个控制器，它可以提交一定数量的最新文章：

.. code-block:: php

    // src/Acme/ArticleBundle/Controller/ArticleController.php

    class ArticleController extends Controller
    {
        public function recentArticlesAction($max = 3)
        {
            // make a database call or other logic to get the "$max" most recent articles
            $articles = ...;

            return $this->render('AcmeArticleBundle:Article:recentList.html.twig', array('articles' => $articles));
        }
    }

The ``recentList`` template is perfectly straightforward:
``recentList``相当简明：

.. configuration-block::

    .. code-block:: html+jinja

        {# src/Acme/ArticleBundle/Resources/views/Article/recentList.html.twig #}
        {% for article in articles %}
            <a href="/article/{{ article.slug }}">
                {{ article.title }}
            </a>
        {% endfor %}

    .. code-block:: html+php

        <!-- src/Acme/ArticleBundle/Resources/views/Article/recentList.html.php -->
        <?php foreach ($articles as $article): ?>
            <a href="/article/<?php echo $article->getSlug() ?>">
                <?php echo $article->getTitle() ?>
            </a>
        <?php endforeach; ?>

.. note::

    Notice that we've cheated and hardcoded the article URL in this example
    (e.g. ``/article/*slug*``). This is a bad practice. In the next section,
    you'll learn how to do this correctly.
    注意你对这个文章的URL实行了硬编码（e.g. ``/article/*slug*``），这个方法很不好。下一节你将学习
    正确的做法。

To include the controller, you'll need to refer to it using the standard string
syntax for controllers (i.e. **bundle**:**controller**:**action**):
要包含这个控制器，你需要使用标准语法来访问它（i.e. **bundle**:**controller**:**action**）：

.. configuration-block::

    .. code-block:: html+jinja

        {# app/Resources/views/base.html.twig #}
        ...

        <div id="sidebar">
            {% render "AcmeArticleBundle:Article:recentArticles" with {'max': 3} %}
        </div>

    .. code-block:: html+php

        <!-- app/Resources/views/base.html.php -->
        ...

        <div id="sidebar">
            <?php echo $view['actions']->render('AcmeArticleBundle:Article:recentArticles', array('max' => 3)) ?>
        </div>

Whenever you find that you need a variable or a piece of information that
you don't have access to in a template, consider rendering a controller.
Controllers are fast to execute and promote good code organization and reuse.
当你发现你需要一个变量或信息，但在这个模板中你却不能访问这个变量，这个时候就要用到控制器了。
使用控制器能够迅速的执行并使得代码组织良好。

Asyncronous Content with hinclude.js
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. versionadded:: 2.1
    hinclude.js support was added in Symfony 2.1

Controllers can be embedded asyncronous using the hinclude.js_ javascript library.
As the embedded content comes from another page (or controller for that matter),
Symfony2 uses the standard ``render`` helper to configure ``hinclude`` tags:

.. configuration-block::

    .. code-block:: jinja

        {% render '...:news' with {}, {'standalone': 'js'} %}

    .. code-block:: php

        <?php echo $view['actions']->render('...:news', array(), array('standalone' => 'js')) ?>

.. note::

   hinclude.js_ needs to be included in your page to work.

Default content (while loading or if javascript is disabled) can be set globally
in your application configuration:

.. configuration-block::

    .. code-block:: yaml

        # app/config/config.yml
        framework:
            # ...
            templating:
                hinclude_default_template: AcmeDemoBundle::hinclude.html.twig

    .. code-block:: xml

        <!-- app/config/config.xml -->
        <framework:config>
            <framework:templating hinclude-default-template="AcmeDemoBundle::hinclude.html.twig" />
        </framework:config>

    .. code-block:: php

        // app/config/config.php
        $container->loadFromExtension('framework', array(
            // ...
            'templating'      => array(
                'hinclude_default_template' => array('AcmeDemoBundle::hinclude.html.twig'),
            ),
        ));

.. index::
   single: Templating; Linking to pages

Linking to Pages
链接到页面
~~~~~~~~~~~~~~~~

Creating links to other pages in your application is one of the most common
jobs for a template. Instead of hardcoding URLs in templates, use the ``path``
Twig function (or the ``router`` helper in PHP) to generate URLs based on
the routing configuration. Later, if you want to modify the URL of a particular
page, all you'll need to do is change the routing configuration; the templates
will automatically generate the new URL.
在你的应用中创建其他页面的链接是模板的一个最常用的功能。你可以使用path这个twig函数（或者php中的router方法）来集成
基于路由配置的URL。以后，如果你要修改这个页面的URL，你只需要修改路由配置就可以了；这个模板会
自动集成这个新的URL。

First, link to the "_welcome" page, which is accessible via the following routing
configuration:

.. configuration-block::

    .. code-block:: yaml

        _welcome:
            pattern:  /
            defaults: { _controller: AcmeDemoBundle:Welcome:index }

    .. code-block:: xml

        <route id="_welcome" pattern="/">
            <default key="_controller">AcmeDemoBundle:Welcome:index</default>
        </route>

    .. code-block:: php

        $collection = new RouteCollection();
        $collection->add('_welcome', new Route('/', array(
            '_controller' => 'AcmeDemoBundle:Welcome:index',
        )));

        return $collection;

To link to the page, just use the ``path`` Twig function and refer to the route:
要链接到这个页面，只要使用path这个twig函数就可以了：

.. configuration-block::

    .. code-block:: html+jinja

        <a href="{{ path('_welcome') }}">Home</a>

    .. code-block:: html+php

        <a href="<?php echo $view['router']->generate('_welcome') ?>">Home</a>

As expected, this will generate the URL ``/``. Let's see how this works with
a more complicated route:
于是，这样就会集成URL“/”。让我们看看更复杂的路径：

.. configuration-block::

    .. code-block:: yaml

        article_show:
            pattern:  /article/{slug}
            defaults: { _controller: AcmeArticleBundle:Article:show }

    .. code-block:: xml

        <route id="article_show" pattern="/article/{slug}">
            <default key="_controller">AcmeArticleBundle:Article:show</default>
        </route>

    .. code-block:: php

        $collection = new RouteCollection();
        $collection->add('article_show', new Route('/article/{slug}', array(
            '_controller' => 'AcmeArticleBundle:Article:show',
        )));

        return $collection;

In this case, you need to specify both the route name (``article_show``) and
a value for the ``{slug}`` parameter. Using this route, let's revisit the
``recentList`` template from the previous section and link to the articles
correctly:
在这个例子中，你需要指定路径名称（``article_show``）和``{slug}``参数的值。
我们可以使用这个路径来访问上一节的``recentList``模板，并正确的链接到文章：

.. configuration-block::

    .. code-block:: html+jinja

        {# src/Acme/ArticleBundle/Resources/views/Article/recentList.html.twig #}
        {% for article in articles %}
            <a href="{{ path('article_show', { 'slug': article.slug }) }}">
                {{ article.title }}
            </a>
        {% endfor %}

    .. code-block:: html+php

        <!-- src/Acme/ArticleBundle/Resources/views/Article/recentList.html.php -->
        <?php foreach ($articles in $article): ?>
            <a href="<?php echo $view['router']->generate('article_show', array('slug' => $article->getSlug()) ?>">
                <?php echo $article->getTitle() ?>
            </a>
        <?php endforeach; ?>

.. tip::

    You can also generate an absolute URL by using the ``url`` Twig function:
    你还可以通过url这个twig函数来集成绝对URL：

    .. code-block:: html+jinja

        <a href="{{ url('_welcome') }}">Home</a>

    The same can be done in PHP templates by passing a third argument to
    the ``generate()`` method:

    .. code-block:: html+php

        <a href="<?php echo $view['router']->generate('_welcome', array(), true) ?>">Home</a>

.. index::
   single: Templating; Linking to assets

Linking to Assets
链接到asset
~~~~~~~~~~~~~~~~~

Templates also commonly refer to images, Javascript, stylesheets and other
assets. Of course you could hard-code the path to these assets (e.g. ``/images/logo.png``),
but Symfony2 provides a more dynamic option via the ``asset`` Twig function:
模板还可以访问图像，javascript，样式表以及其他asset。当然，你可以将访问这些asset的路径硬编码（e.g. ``/images/logo.png``），
但是symfony2通过asset这个twig函数提供了一个更动态的路径：

.. configuration-block::

    .. code-block:: html+jinja

        <img src="{{ asset('images/logo.png') }}" alt="Symfony!" />

        <link href="{{ asset('css/blog.css') }}" rel="stylesheet" type="text/css" />

    .. code-block:: html+php

        <img src="<?php echo $view['assets']->getUrl('images/logo.png') ?>" alt="Symfony!" />

        <link href="<?php echo $view['assets']->getUrl('css/blog.css') ?>" rel="stylesheet" type="text/css" />

The ``asset`` function's main purpose is to make your application more portable.
If your application lives at the root of your host (e.g. http://example.com),
then the rendered paths should be ``/images/logo.png``. But if your application
lives in a subdirectory (e.g. http://example.com/my_app), each asset path
should render with the subdirectory (e.g. ``/my_app/images/logo.png``). The
``asset`` function takes care of this by determining how your application is
being used and generating the correct paths accordingly.
asset函数的主要目的是使你的代码编写更方便。如果你的应用在主机的根目录下（e.g. http://example.com），
那么这个输出的路径就是``/images/logo.png``。但是如果你的应用在一个子目录下（e.g. http://example.com/my_app），那么
每个asset的路径就是在子目录下的(e.g. ``/my_app/images/logo.png``)。asset函数可以自动判断你的应用
是如何放置的，并集成正确的路径。

Additionally, if you use the ``asset`` function, Symfony can automatically
append a query string to your asset, in order to guarantee that updated static
assets won't be cached when deployed. For example, ``/images/logo.png`` might
look like ``/images/logo.png?v2``. For more information, see the :ref:`ref-framework-assets-version`
configuration option.
并且，如果你使用asset函数，symfony可以自动在你的asset后添加一个请求语句，这是为了被更新的静态
asset不会被缓存。比如，``/images/logo.png``可能会是``/images/logo.png?v2``。详情请参见:ref:`ref-framework-assets-version`。

.. index::
   single: Templating; Including stylesheets and Javascripts
   single: Stylesheets; Including stylesheets
   single: Javascripts; Including Javascripts

Including Stylesheets and Javascripts in Twig
在twig中包含样式表和javascript
---------------------------------------------

No site would be complete without including Javascript files and stylesheets.
In Symfony, the inclusion of these assets is handled elegantly by taking
advantage of Symfony's template inheritance.
没有一个网站没有javascript和样式表的。在symfony中，可以通过symfony的模板继承来处理
对asset的包含。

.. tip::

    This section will teach you the philosophy behind including stylesheet
    and Javascript assets in Symfony. Symfony also packages another library,
    called Assetic, which follows this philosophy but allows you to do much
    more interesting things with those assets. For more information on
    using Assetic see :doc:`/cookbook/assetic/asset_management`.
    这一节会讲解如何在symfony中包含javascript和样式表。symfony还有一个内置的库名叫assetic，
    使用它可以更方便地处理asset。参见:doc:`/cookbook/assetic/asset_management`。


Start by adding two blocks to your base template that will hold your assets:
one called ``stylesheets`` inside the ``head`` tag and another called ``javascripts``
just above the closing ``body`` tag. These blocks will contain all of the
stylesheets and Javascripts that you'll need throughout your site:
首先在你的基本模板中添加两个block来存储你的asset：一个叫做stylesheets，在head标签中；另一个叫做javascripts，
在body的关闭标签前。这些block会存储所有在网站中你要使用的stylesheets和javascripts：

.. code-block:: html+jinja

    {# 'app/Resources/views/base.html.twig' #}
    <html>
        <head>
            {# ... #}

            {% block stylesheets %}
                <link href="{{ asset('/css/main.css') }}" type="text/css" rel="stylesheet" />
            {% endblock %}
        </head>
        <body>
            {# ... #}

            {% block javascripts %}
                <script src="{{ asset('/js/main.js') }}" type="text/javascript"></script>
            {% endblock %}
        </body>
    </html>

That's easy enough! But what if you need to include an extra stylesheet or
Javascript from a child template? For example, suppose you have a contact
page and you need to include a ``contact.css`` stylesheet *just* on that
page. From inside that contact page's template, do the following:
如果你需要在子模板中包含一个额外的样式表和javascript，比如假设你有一个contact页面，而且你需要仅仅在那一个页面中包含一个
``contact.css``样式表。在那个contact页面的模板中，添加以下代码：

.. code-block:: html+jinja

    {# src/Acme/DemoBundle/Resources/views/Contact/contact.html.twig #}
    {% extends '::base.html.twig' %}

    {% block stylesheets %}
        {{ parent() }}

        <link href="{{ asset('/css/contact.css') }}" type="text/css" rel="stylesheet" />
    {% endblock %}

    {# ... #}

In the child template, you simply override the ``stylesheets`` block and
put your new stylesheet tag inside of that block. Of course, since you want
to add to the parent block's content (and not actually *replace* it), you
should use the ``parent()`` Twig function to include everything from the ``stylesheets``
block of the base template.
在子模板中，你只需要覆盖stylesheets这个block并将你的新stylesheet标签置于能够block中。当然，
由于你需要的是将内容添加入父模板的block（而不是替代它），你应该使用``parent()`` twig函数来
包含基本模板的stylesheets block。

You can also include assets located in your bundles' ``Resources/public`` folder.
You will need to run the ``php app/console assets:install target [--symlink]``
command, which moves (or symlinks) files into the correct location. (target
is by default "web").
你还可以包含放置在你的bundle的``Resources/public``文件中的asset。你需要运行``php app/console assets:install target [--symlink]``
命令行来将目标文件移至正确位置（target默认是web）。

.. code-block:: html+jinja

   <link href="{{ asset('bundles/acmedemo/css/contact.css') }}" type="text/css" rel="stylesheet" />

The end result is a page that includes both the ``main.css`` and ``contact.css``
stylesheets.
最终结果是一个包含``main.css``和``contact.css``的页面。

Global Template Variables
全局模板变量
-------------------------

During each request, Symfony2 will set a global template variable ``app``
in both Twig and PHP template engines by default.  The ``app`` variable
is a :class:`Symfony\\Bundle\\FrameworkBundle\\Templating\\GlobalVariables`
instance which will give you access to some application specific variables
automatically:
对于每个请求，symfony2会自动在twig和php模板中设置一个全局模板变量app。这个app变量是一个
:class:`Symfony\\Bundle\\FrameworkBundle\\Templating\\GlobalVariables`实例，你可以通过它访问
某些应用的特定变量：

* ``app.security`` - The security context.
* ``app.user`` - The current user object.
* ``app.request`` - The request object.
* ``app.session`` - The session object.
* ``app.environment`` - The current environment (dev, prod, etc).
* ``app.debug`` - True if in debug mode. False otherwise.

.. configuration-block::

    .. code-block:: html+jinja

        <p>Username: {{ app.user.username }}</p>
        {% if app.debug %}
            <p>Request method: {{ app.request.method }}</p>
            <p>Application Environment: {{ app.environment }}</p>
        {% endif %}

    .. code-block:: html+php

        <p>Username: <?php echo $app->getUser()->getUsername() ?></p>
        <?php if ($app->getDebug()): ?>
            <p>Request method: <?php echo $app->getRequest()->getMethod() ?></p>
            <p>Application Environment: <?php echo $app->getEnvironment() ?></p>
        <?php endif; ?>

.. tip::

    You can add your own global template variables. See the cookbook example
    on :doc:`Global Variables</cookbook/templating/global_variables>`.
    你还可以添加你自己的全局变量。参见:doc:`Global Variables</cookbook/templating/global_variables>`。

.. index::
   single: Templating; The templating service

Configuring and using the ``templating`` Service
配置并使用templating服务
------------------------------------------------

The heart of the template system in Symfony2 is the templating ``Engine``.
This special object is responsible for rendering templates and returning
their content. When you render a template in a controller, for example,
you're actually using the templating engine service. For example:
symfony模板系统的核心是模板"引擎"。这个类可以用来提交模板并返回它的内容。当你在控制器中使用
模板时，你实际上是在使用模板引擎服务。比如：

.. code-block:: php

    return $this->render('AcmeArticleBundle:Article:index.html.twig');

is equivalent to
等同于

.. code-block:: php

    $engine = $this->container->get('templating');
    $content = $engine->render('AcmeArticleBundle:Article:index.html.twig');

    return $response = new Response($content);

.. _template-configuration:

The templating engine (or "service") is preconfigured to work automatically
inside Symfony2. It can, of course, be configured further in the application
configuration file:
模板引擎（或称service）会在symfony中自动工作。当然，你还可以在应用配置文件中更多地配置它：

.. configuration-block::

    .. code-block:: yaml

        # app/config/config.yml
        framework:
            # ...
            templating: { engines: ['twig'] }

    .. code-block:: xml

        <!-- app/config/config.xml -->
        <framework:templating>
            <framework:engine id="twig" />
        </framework:templating>

    .. code-block:: php

        // app/config/config.php
        $container->loadFromExtension('framework', array(
            // ...
            'templating'      => array(
                'engines' => array('twig'),
            ),
        ));

Several configuration options are available and are covered in the
:doc:`Configuration Appendix</reference/configuration/framework>`.
更多配置参数请参见:doc:`Configuration Appendix</reference/configuration/framework>`。

.. note::

   The ``twig`` engine is mandatory to use the webprofiler (as well as many
   third-party bundles).
   要使用webprofiler（包括许多其他的第三方bundle），必须使用twig引擎。

.. index::
    single; Template; Overriding templates

.. _overriding-bundle-templates:

Overriding Bundle Templates
覆盖bundle模板
---------------------------

The Symfony2 community prides itself on creating and maintaining high quality
bundles (see `KnpBundles.com`_) for a large number of different features.
Once you use a third-party bundle, you'll likely need to override and customize
one or more of its templates.
symfony2的特色就是允许你创建不同功能的bundle。假如你要使用某个第三方bundle的话，你可能
需要覆盖或自定义它的模板。

Suppose you've included the imaginary open-source ``AcmeBlogBundle`` in your
project (e.g. in the ``src/Acme/BlogBundle`` directory). And while you're
really happy with everything, you want to override the blog "list" page to
customize the markup specifically for your application. By digging into the
``Blog`` controller of the ``AcmeBlogBundle``, you find the following::
假设你已经包含了一个开源的``AcmeBlogBundle``（也就是在``src/Acme/BlogBundle``目录下），
你需要修改它的博客列表页面的样式。在``AcmeBlogBundle``的blog控制器中，你发现了以下代码::

    public function indexAction()
    {
        $blogs = // some logic to retrieve the blogs

        $this->render('AcmeBlogBundle:Blog:index.html.twig', array('blogs' => $blogs));
    }

When the ``AcmeBlogBundle:Blog:index.html.twig`` is rendered, Symfony2 actually
looks in two different locations for the template:
当``AcmeBlogBundle:Blog:index.html.twig``被提交时，symfony2实际上从这两个地方查找模板：

#. ``app/Resources/AcmeBlogBundle/views/Blog/index.html.twig``
#. ``src/Acme/BlogBundle/Resources/views/Blog/index.html.twig``

To override the bundle template, just copy the ``index.html.twig`` template
from the bundle to ``app/Resources/AcmeBlogBundle/views/Blog/index.html.twig``
(the ``app/Resources/AcmeBlogBundle`` directory won't exist, so you'll need
to create it). You're now free to customize the template.
要想覆盖这个bundle的模板，只要从这个bundle中将``index.html.twig``复制到
``app/Resources/AcmeBlogBundle/views/Blog/index.html.twig``（``app/Resources/AcmeBlogBundle``本来是不存在的，
你必须自己创建它）。这样你就可以改变样式了。

This logic also applies to base bundle templates. Suppose also that each
template in ``AcmeBlogBundle`` inherits from a base template called
``AcmeBlogBundle::layout.html.twig``. Just as before, Symfony2 will look in
the following two places for the template:
这个道理也可以被应用于基本bundle模板。假设每个``AcmeBlogBundle``中的模板都继承一个名叫
``AcmeBlogBundle::layout.html.twig``的基本模板。像刚才一样，symfony2会查看以下两个地方：

#. ``app/Resources/AcmeBlogBundle/views/layout.html.twig``
#. ``src/Acme/BlogBundle/Resources/views/layout.html.twig``

Once again, to override the template, just copy it from the bundle to
``app/Resources/AcmeBlogBundle/views/layout.html.twig``. You're now free to
customize this copy as you see fit.
要覆盖这个模板，只要将它从bundle中复制到``app/Resources/AcmeBlogBundle/views/layout.html.twig``
就可以了。你现在可以自己定制它。

If you take a step back, you'll see that Symfony2 always starts by looking in
the ``app/Resources/{BUNDLE_NAME}/views/`` directory for a template. If the
template doesn't exist there, it continues by checking inside the
``Resources/views`` directory of the bundle itself. This means that all bundle
templates can be overridden by placing them in the correct ``app/Resources``
subdirectory.
symfony2首先会查看``app/Resources/{BUNDLE_NAME}/views/``目录下是否有模板。如果没有，
它会继续查看bundle中的``Resources/views``目录。这表示只要将模板放置在正确的``app/Resources``
子目录下，就可以覆盖任何bundle模板。

.. note::

    You can also override templates from within a bundle by using bundle
    inheritance. For more information, see :doc:`/cookbook/bundles/inheritance`.
    你还可以通过bundle继承来在bundle中覆盖模板，参见:doc:`/cookbook/bundles/inheritance`。

.. _templating-overriding-core-templates:

.. index::
    single; Template; Overriding exception templates

Overriding Core Templates
覆盖核心模板
~~~~~~~~~~~~~~~~~~~~~~~~~

Since the Symfony2 framework itself is just a bundle, core templates can be
overridden in the same way. For example, the core ``TwigBundle`` contains
a number of different "exception" and "error" templates that can be overridden
by copying each from the ``Resources/views/Exception`` directory of the
``TwigBundle`` to, you guessed it, the
``app/Resources/TwigBundle/views/Exception`` directory.
由于symfony2框架本身就是一个bundle，它的核心模板也可以这样被覆盖。比如，核心``TwigBundle``
包含了一系列不同"exception"和"error"模板，你可以通过从``TwigBundle``的``Resources/views/Exception``目录复制它们，
然后将它们置于``app/Resources/TwigBundle/views/Exception``目录下。

.. index::
   single: Templating; Three-level inheritance pattern

Three-level Inheritance
三级继承
-----------------------

One common way to use inheritance is to use a three-level approach. This
method works perfectly with the three different types of templates we've just
covered:
一个使用继承的常用方法就是使用一个三级继承。这个方法可以对以上我们所讲的三种不同类型的模板
进行很好的运用：

* Create a ``app/Resources/views/base.html.twig`` file that contains the main
  layout for your application (like in the previous example). Internally, this
  template is called ``::base.html.twig``;
  创建一个包含了你应用主要布局的``app/Resources/views/base.html.twig``。这个模板被称作``::base.html.twig``；

* Create a template for each "section" of your site. For example, an ``AcmeBlogBundle``,
  would have a template called ``AcmeBlogBundle::layout.html.twig`` that contains
  only blog section-specific elements;
  为你的网站的每个部分创建一个模板。比如，一个``AcmeBlogBundle``有一个名叫``AcmeBlogBundle::layout.html.twig``
  的模板，这个模板包含了博客部分的元素；

    .. code-block:: html+jinja

        {# src/Acme/BlogBundle/Resources/views/layout.html.twig #}
        {% extends '::base.html.twig' %}

        {% block body %}
            <h1>Blog Application</h1>

            {% block content %}{% endblock %}
        {% endblock %}

* Create individual templates for each page and make each extend the appropriate
  section template. For example, the "index" page would be called something
  close to ``AcmeBlogBundle:Blog:index.html.twig`` and list the actual blog posts.
  为每个页面创建模板，并使每一个模板扩展合适的模板。比如，index页面会被称作
  ``AcmeBlogBundle:Blog:index.html.twig``并列出博客文章。

    .. code-block:: html+jinja

        {# src/Acme/BlogBundle/Resources/views/Blog/index.html.twig #}
        {% extends 'AcmeBlogBundle::layout.html.twig' %}

        {% block content %}
            {% for entry in blog_entries %}
                <h2>{{ entry.title }}</h2>
                <p>{{ entry.body }}</p>
            {% endfor %}
        {% endblock %}

Notice that this template extends the section template -(``AcmeBlogBundle::layout.html.twig``)
which in-turn extends the base application layout (``::base.html.twig``).
This is the common three-level inheritance model.
注意这个模板扩展``AcmeBlogBundle::layout.html.twig``模板，而``AcmeBlogBundle::layout.html.twig``
模板又扩展基本模板``::base.html.twig``。这就是常见的三级继承模式。

When building your application, you may choose to follow this method or simply
make each page template extend the base application template directly
(e.g. ``{% extends '::base.html.twig' %}``). The three-template model is
a best-practice method used by vendor bundles so that the base template for
a bundle can be easily overridden to properly extend your application's base
layout.
当创建网站时，你可能选择遵循这个三级继承模式或直接让每个页面模板继承基本模板（e.g. ``{% extends '::base.html.twig' %}``）。
不过三级继承是一个最优方法，这样基本模板会很容易地被覆盖。

.. index::
   single: Templating; Output escaping

Output Escaping
输出转义
---------------

When generating HTML from a template, there is always a risk that a template
variable may output unintended HTML or dangerous client-side code. The result
is that dynamic content could break the HTML of the resulting page or allow
a malicious user to perform a `Cross Site Scripting`_ (XSS) attack. Consider
this classic example:
当从一个模板中集成HTML时，模板变量可能会输出HTML或者不期望的客户端代码。结果是
动态内容可能会破坏页面的HTML或允许一个恶意用户进行`Cross Site Scripting`_ (XSS)攻击。
举个经典例子：

.. configuration-block::

    .. code-block:: jinja

        Hello {{ name }}

    .. code-block:: html+php

        Hello <?php echo $name ?>

Imagine that the user enters the following code as his/her name::
假如这个用户将以下代码作为他的名字输出::

    <script>alert('hello!')</script>

Without any output escaping, the resulting template will cause a JavaScript
alert box to pop up::
如果没有输出转义，结果这个模板就会导致一个javascript警告框弹出::

    Hello <script>alert('hello!')</script>

And while this seems harmless, if a user can get this far, that same user
should also be able to write JavaScript that performs malicious actions
inside the secure area of an unknowing, legitimate user.
这可能没什么，但这个用户也可以在一个合法用户的需要认证的区域内输入javascript代码，并进行破坏。

The answer to the problem is output escaping. With output escaping on, the
same template will render harmlessly, and literally print the ``script``
tag to the screen::
解决这个问题需要输出转义。输出转义如果被激活，那么这个模板就没有问题，并且
将script标签输出到屏幕::

    Hello &lt;script&gt;alert(&#39;helloe&#39;)&lt;/script&gt;

The Twig and PHP templating systems approach the problem in different ways.
If you're using Twig, output escaping is on by default and you're protected.
In PHP, output escaping is not automatic, meaning you'll need to manually
escape where necessary.
twig和php模板系统不同，如果你使用twig，输出转义默认是激活的；但在php中，输出escape不是自动的，
这表示你必须手动转义。

Output Escaping in Twig
在twig中的输出转义
~~~~~~~~~~~~~~~~~~~~~~~

If you're using Twig templates, then output escaping is on by default. This
means that you're protected out-of-the-box from the unintentional consequences
of user-submitted code. By default, the output escaping assumes that content
is being escaped for HTML output.
如果你使用twig模板，那么输出转义是默认激活的。这表示用户提交的代码不会危害你的页面。
默认情况下，输出转义假设内容转义是为HTML输出的。

In some cases, you'll need to disable output escaping when you're rendering
a variable that is trusted and contains markup that should not be escaped.
Suppose that administrative users are able to write articles that contain
HTML code. By default, Twig will escape the article body. To render it normally,
add the ``raw`` filter: ``{{ article.body|raw }}``.
某些情况下，当你提交一个可被信任的变量或包括不应该被转义的代码时，你需要禁止转义。
比如，管理员用户要提交包含HTML代码的文章。默认情况下，twig会将文章转义。要正常
输出它，需要添加raw过滤器：``{{ article.body|raw }}``。

You can also disable output escaping inside a ``{% block %}`` area or
for an entire template. For more information, see `Output Escaping`_ in
the Twig documentation.
你还可以在block或整个模板中禁止转义。参见`Output Escaping`_。

Output Escaping in PHP
在php中的输出转义
~~~~~~~~~~~~~~~~~~~~~~

Output escaping is not automatic when using PHP templates. This means that
unless you explicitly choose to escape a variable, you're not protected. To
use output escaping, use the special ``escape()`` view method::
当你使用php模板时，输出转义不是自动进行的。这表示你必须显性的将一个变量转义。
要想转义输出，就要使用``escape()`` view方法:: 

    Hello <?php echo $view->escape($name) ?>

By default, the ``escape()`` method assumes that the variable is being rendered
within an HTML context (and thus the variable is escaped to be safe for HTML).
The second argument lets you change the context. For example, to output something
in a JavaScript string, use the ``js`` context:
默认情况下，``escape()``方法假设变量是在HTML环境下被输出的（所以变量只针对HTML转义）。
第二个参数允许你改变环境。比如，要在javascript字符串中输出，要使用js环境：

.. code-block:: js

    var myMsg = 'Hello <?php echo $view->escape($name, 'js') ?>';

.. index::
   single: Templating; Formats

.. _template-formats:

Debugging
调试
---------

.. versionadded:: 2.0.9
    This feature is available as of Twig ``1.5.x``, which was first shipped
    with Symfony 2.0.9.

When using PHP, you can use ``var_dump()`` if you need to quickly find the
value of a variable passed. This is useful, for example, inside your controller.
The same can be achieved when using Twig by using the debug extension. This
needs to be enabled in the config:
当使用php时，如果你需要快速查看一个变量的值，可以使用``var_dump()``，这在你的控制器中很有用。
通过调试配置，在使用twig也可以这样。这需要在config中激活：

.. configuration-block::

    .. code-block:: yaml

        # app/config/config.yml
        services:
            acme_hello.twig.extension.debug:
                class:        Twig_Extension_Debug
                tags:
                     - { name: 'twig.extension' }

    .. code-block:: xml

        <!-- app/config/config.xml -->
        <services>
            <service id="acme_hello.twig.extension.debug" class="Twig_Extension_Debug">
                <tag name="twig.extension" />
            </service>
        </services>

    .. code-block:: php

        // app/config/config.php
        use Symfony\Component\DependencyInjection\Definition;

        $definition = new Definition('Twig_Extension_Debug');
        $definition->addTag('twig.extension');
        $container->setDefinition('acme_hello.twig.extension.debug', $definition);

Template parameters can then be dumped using the ``dump`` function:
这样就可以dump模板参数了：

.. code-block:: html+jinja

    {# src/Acme/ArticleBundle/Resources/views/Article/recentList.html.twig #}

    {{ dump(articles) }}

    {% for article in articles %}
        <a href="/article/{{ article.slug }}">
            {{ article.title }}
        </a>
    {% endfor %}


The variables will only be dumped if Twig's ``debug`` setting (in ``config.yml``)
is ``true``. By default this means that the variables will be dumped in the
``dev`` environment but not the ``prod`` environment.
只有当twig的debug（``config.yml``中）被设置为true时，变量才会被dump。默认情况下，
变量会在dev环境下被dump，而在prod环境下则不会。

Template Formats
模板格式
----------------

Templates are a generic way to render content in *any* format. And while in
most cases you'll use templates to render HTML content, a template can just
as easily generate JavaScript, CSS, XML or any other format you can dream of.
模板是一个输出任何格式内容的大体方法。虽然大部分情况下你会使用模板来输出HTML内容，
但模板也可以同样容易地集成javascript、css、XML或其他任何格式。

For example, the same "resource" is often rendered in several different formats.
To render an article index page in XML, simply include the format in the
template name:
比如，同样一个文件可能用数个不同的格式来输出。要用XML来输出一个文章的index页面，只要在模板名称中
包含这个格式就可以了：

* *XML template name*: ``AcmeArticleBundle:Article:index.xml.twig``
* *XML template filename*: ``index.xml.twig``

In reality, this is nothing more than a naming convention and the template
isn't actually rendered differently based on its format.
实际上，这只不过是个命名方法问题，并且在提交模板的时候也并不是根据它的格式来提交的。

In many cases, you may want to allow a single controller to render multiple
different formats based on the "request format". For that reason, a common
pattern is to do the following:
很多情况下，你可能需要允许一个控制器根据"request format"来提交多个不同格式的模板。一般是
这样做的：

.. code-block:: php

    public function indexAction()
    {
        $format = $this->getRequest()->getRequestFormat();

        return $this->render('AcmeBlogBundle:Blog:index.'.$format.'.twig');
    }

The ``getRequestFormat`` on the ``Request`` object defaults to ``html``,
but can return any other format based on the format requested by the user.
The request format is most often managed by the routing, where a route can
be configured so that ``/contact`` sets the request format to ``html`` while
``/contact.xml`` sets the format to ``xml``. For more information, see the
:ref:`Advanced Example in the Routing chapter <advanced-routing-example>`.
``Request``对象的``getRequestFormat``方法默认返回的值是``html``，但是根据用户请求的
格式，也可以是其他格式。请求格式通常是由路径配置决定的，一个路径可以被配置，从而/contact
对应结果的格式是html，而/contact.xml对应的结果的格式是xml。更多信息请参阅:ref:`Advanced Example in the Routing chapter <advanced-routing-example>`。

To create links that include the format parameter, include a ``_format``
key in the parameter hash:
要创建包含格式参数的链接，只要包含一个``_format``参数：

.. configuration-block::

    .. code-block:: html+jinja

        <a href="{{ path('article_show', {'id': 123, '_format': 'pdf'}) }}">
            PDF Version
        </a>

    .. code-block:: html+php

        <a href="<?php echo $view['router']->generate('article_show', array('id' => 123, '_format' => 'pdf')) ?>">
            PDF Version
        </a>

Final Thoughts
总结
--------------

The templating engine in Symfony is a powerful tool that can be used each time
you need to generate presentational content in HTML, XML or any other format.
And though templates are a common way to generate content in a controller,
their use is not mandatory. The ``Response`` object returned by a controller
can be created with our without the use of a template:
symfony的模板引擎是一个强大的工具，它可以用于集成HTML,XML或其他格式的内容。
虽然模板是一个集成内容的常用方法，但并不是要强制使用它们。控制器返回的response对象
可以用模板，也可以不用模板。

.. code-block:: php

    // creates a Response object whose content is the rendered template
    $response = $this->render('AcmeArticleBundle:Article:index.html.twig');

    // creates a Response object whose content is simple text
    $response = new Response('response content');

Symfony's templating engine is very flexible and two different template
renderers are available by default: the traditional *PHP* templates and the
sleek and powerful *Twig* templates. Both support a template hierarchy and
come packaged with a rich set of helper functions capable of performing
the most common tasks.
symfony有两种模板提交方法：传统的php方法和twig方法。两种方法都支持继承，并都
内置了一套丰富的helper方法。

Overall, the topic of templating should be thought of as a powerful tool
that's at your disposal. In some cases, you may not need to render a template,
and in Symfony2, that's absolutely fine.

Learn more from the Cookbook
----------------------------

* :doc:`/cookbook/templating/PHP`
* :doc:`/cookbook/controller/error_pages`
* :doc:`/cookbook/templating/twig_extension`

.. _`Twig`: http://twig.sensiolabs.org
.. _`KnpBundles.com`: http://knpbundles.com
.. _`Cross Site Scripting`: http://en.wikipedia.org/wiki/Cross-site_scripting
.. _`Output Escaping`: http://twig.sensiolabs.org/doc/api.html#escaper-extension
.. _`tags`: http://twig.sensiolabs.org/doc/tags/index.html
.. _`filters`: http://twig.sensiolabs.org/doc/filters/index.html
.. _`add your own extensions`: http://twig.sensiolabs.org/doc/advanced.html#creating-an-extension
.. _`hinclude.js`: http://mnot.github.com/hinclude/
