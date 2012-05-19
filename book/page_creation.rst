.. index::
   single: Page creation

Creating Pages in Symfony2
用symfony2创建页面
==========================

Creating a new page in Symfony2 is a simple two-step process:
用symfony2创建页面很简单，只需两步：

* *Create a route*: A route defines the URL (e.g. ``/about``) to your page
  and specifies a controller (which is a PHP function) that Symfony2 should
  execute when the URL of an incoming request matches the route pattern;
  *创建路径*：一个路径定义了一个你的页面的URL（如/about）并指定了一个在请求的URL匹配路径pattern的时候，symfony2要
  执行的控制器（也就是一个php方法）；

* *Create a controller*: A controller is a PHP function that takes the incoming
  request and transforms it into the Symfony2 ``Response`` object that's
  returned to the user.
  *创建一个控制器*：一个控制器就是一个php方法，它获取请求（request）并处理它，转化为一个symfony2 response
  对象并返回到客户端。

This simple approach is beautiful because it matches the way that the Web works.
Every interaction on the Web is initiated by an HTTP request. The job of
your application is simply to interpret the request and return the appropriate
HTTP response.
这个解决方案是很好的，因为它与web的工作原理相匹配。每个web的交互都是从HTTP请求开始的。
你的应用（application）只是解析这个请求并返回相应的HTTP响应。

Symfony2 follows this philosophy and provides you with tools and conventions
to keep your application organized as it grows in users and complexity.
symfony2遵循这个原则并提供工具和协定来保证你的应用在用户和复杂性增加的时候依然能够保持其组织性。

Sounds simple enough? Let's dive in!
是不是很简单？接着往下看！

.. index::
   single: Page creation; Example

The "Hello Symfony!" Page
"Hello Symfony!"页面
-------------------------

Let's start with a spin off of the classic "Hello World!" application. When
you're finished, the user will be able to get a personal greeting (e.g. "Hello Symfony")
by going to the following URL:
让我们从经典的"Hello World!"应用开始。当你结束时，用户会通过访问以下的URL来得到一个欢迎页面(e.g. "Hello Symfony")：

.. code-block:: text

    http://localhost/app_dev.php/hello/Symfony

Actually, you'll be able to replace ``Symfony`` with any other name to be
greeted. To create the page, follow the simple two-step process.
事实上，你可以将以上Symfony换成任何其他单词。要创建这个页面，遵循以下过程。

.. note::

    The tutorial assumes that you've already downloaded Symfony2 and configured
    your webserver. The above URL assumes that ``localhost`` points to the
    ``web`` directory of your new Symfony2 project. For detailed information
    on this process, see the documentation on the web server you are using.
    Here's the relevant documentation page for some web server you might be using:
    这个教程假设你已经下载symfony并配置了你的服务器。以上的URL假设localhost指向你的代码
    的web根目录。要了解详细信息，请参阅你所使用的服务器的文档。以下是你可能在使用的一些服务器的文档：
    
    * For Apache HTTP Server, refer to `Apache's DirectoryIndex documentation`_.
    * 对于Apache HTTP服务器，参阅`Apache's DirectoryIndex documentation`_。
    * For Nginx, refer to `Nginx HttpCoreModule location documentation`_.
    * 对于Nginx,参阅`Nginx HttpCoreModule location documentation`_。

Before you begin: Create the Bundle
在开始之前：创建bundle
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Before you begin, you'll need to create a *bundle*. In Symfony2, a :term:`bundle`
is like a plugin, except that all of the code in your application will live
inside a bundle.
在你开始之前，你需要创建一个bundle。在symfony2中，一个:term:`bundle`就像一个插件，只不过你的应用
的所有代码都在一个bundle中。

A bundle is nothing more than a directory that houses everything related
to a specific feature, including PHP classes, configuration, and even stylesheets
and Javascript files (see :ref:`page-creation-bundles`).
bundle就是一个目录，它存储了所有关于一个特定功能的代码，如php类，配置，以及css和javascript文件。（参阅 :ref:`page-creation-bundles`）

To create a bundle called ``AcmeHelloBundle`` (a play bundle that you'll
build in this chapter), run the following command and follow the on-screen
instructions (use all of the default options):
要创建一个名叫``AcmeHelloBundle``的bundle（本章你要创建的一个例子），运行以下命令行并
遵循屏幕上的指令（使用默认选项）：

.. code-block:: bash

    php app/console generate:bundle --namespace=Acme/HelloBundle --format=yml

Behind the scenes, a directory is created for the bundle at ``src/Acme/HelloBundle``.
A line is also automatically added to the ``app/AppKernel.php`` file so that
the bundle is registered with the kernel::
这样，在``src/Acme/HelloBundle``中，这个bundle的目录被创建。另外一行代码也被自动添加到``app/AppKernel.php``
文件中，这样bundle就能在核心（kernel）中注册了::

    // app/AppKernel.php
    public function registerBundles()
    {
        $bundles = array(
            // ...
            new Acme\HelloBundle\AcmeHelloBundle(),
        );
        // ...

        return $bundles;
    }

Now that you have a bundle setup, you can begin building your application
inside the bundle.
现在你已经设置了一个bundle，你可以开始在bundle中创建你的应用了。

Step 1: Create the Route
第一步：创建路径
~~~~~~~~~~~~~~~~~~~~~~~~

By default, the routing configuration file in a Symfony2 application is
located at ``app/config/routing.yml``. Like all configuration in Symfony2,
you can also choose to use XML or PHP out of the box to configure routes.
默认情况下，在symfony2应用的路径配置文件是``app/config/routing.yml``。像symfony2中的所有
配置一样，你可以选择使用XML或者PHP来配置路径。

If you look at the main routing file, you'll see that Symfony already added
an entry when you generated the ``AcmeHelloBundle``:
如果你查看主要的路径文件，你会看见当你集成``AcmeHelloBundle``的时候，symfony已经添加了
一个入口：

.. configuration-block::

    .. code-block:: yaml

        # app/config/routing.yml
        AcmeHelloBundle:
            resource: "@AcmeHelloBundle/Resources/config/routing.yml"
            prefix:   /

    .. code-block:: xml

        <!-- app/config/routing.xml -->
        <?xml version="1.0" encoding="UTF-8" ?>

        <routes xmlns="http://symfony.com/schema/routing"
            xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
            xsi:schemaLocation="http://symfony.com/schema/routing http://symfony.com/schema/routing/routing-1.0.xsd">

            <import resource="@AcmeHelloBundle/Resources/config/routing.xml" prefix="/" />
        </routes>

    .. code-block:: php

        // app/config/routing.php
        use Symfony\Component\Routing\RouteCollection;
        use Symfony\Component\Routing\Route;

        $collection = new RouteCollection();
        $collection->addCollection(
            $loader->import('@AcmeHelloBundle/Resources/config/routing.php'),
            '/',
        );

        return $collection;

This entry is pretty basic: it tells Symfony to load routing configuration
from the ``Resources/config/routing.yml`` file that lives inside the ``AcmeHelloBundle``.
This means that you place routing configuration directly in ``app/config/routing.yml``
or organize your routes throughout your application, and import them from here.
这个入口是起码的：它告诉symfony从``AcmeHelloBundle``中的``Resources/config/routing.yml``中载入路径配置。
这表示你直接将路径配置放在``app/config/routing.yml``中或在你的应用中组织你的路径，并将它们导入到这儿。

Now that the ``routing.yml`` file from the bundle is being imported, add
the new route that defines the URL of the page that you're about to create:
现在bundle中的``routing.yml``文件被导入了，添加你要创建页面的URL的路径：

.. configuration-block::

    .. code-block:: yaml

        # src/Acme/HelloBundle/Resources/config/routing.yml
        hello:
            pattern:  /hello/{name}
            defaults: { _controller: AcmeHelloBundle:Hello:index }

    .. code-block:: xml

        <!-- src/Acme/HelloBundle/Resources/config/routing.xml -->
        <?xml version="1.0" encoding="UTF-8" ?>

        <routes xmlns="http://symfony.com/schema/routing"
            xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
            xsi:schemaLocation="http://symfony.com/schema/routing http://symfony.com/schema/routing/routing-1.0.xsd">

            <route id="hello" pattern="/hello/{name}">
                <default key="_controller">AcmeHelloBundle:Hello:index</default>
            </route>
        </routes>

    .. code-block:: php

        // src/Acme/HelloBundle/Resources/config/routing.php
        use Symfony\Component\Routing\RouteCollection;
        use Symfony\Component\Routing\Route;

        $collection = new RouteCollection();
        $collection->add('hello', new Route('/hello/{name}', array(
            '_controller' => 'AcmeHelloBundle:Hello:index',
        )));

        return $collection;

The routing consists of two basic pieces: the ``pattern``, which is the URL
that this route will match, and a ``defaults`` array, which specifies the
controller that should be executed. The placeholder syntax in the pattern
(``{name}``) is a wildcard. It means that ``/hello/Ryan``, ``/hello/Fabien``
or any other similar URL will match this route. The ``{name}`` placeholder
parameter will also be passed to the controller so that you can use its value
to personally greet the user.
路径包含了两个主要部分：pattern，是这个路径将要匹配的URL；defaults数组，则确定了要被
执行的控制器。在pattern中的placeholder（``{name}``）是个通配符。它表示``/hello/Ryan``, ``/hello/Fabien``
或者其他相似的会匹配这个路径的URL。{name}这个placeholder参数也会被传递给控制器，这样你就可以自定义它的值了。

.. note::

  The routing system has many more great features for creating flexible
  and powerful URL structures in your application. For more details, see
  the chapter all about :doc:`Routing </book/routing>`.
  路径系统有更多有用的功能来创建灵活强大的URL结构。详情请见:doc:`Routing </book/routing>`。

Step 2: Create the Controller
第二步：创建控制器
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

When a URL such as ``/hello/Ryan`` is handled by the application, the ``hello``
route is matched and the ``AcmeHelloBundle:Hello:index`` controller is executed
by the framework. The second step of the page-creation process is to create
that controller.
当一个像``/hello/Ryan``这样的URL被应用处理时，hello路径被匹配，``AcmeHelloBundle:Hello:index``
控制器被执行。页面创建的第二步就是要创建那个控制器。

The controller - ``AcmeHelloBundle:Hello:index`` is the *logical* name of
the controller, and it maps to the ``indexAction`` method of a PHP class
called ``Acme\HelloBundle\Controller\Hello``. Start by creating this file
inside your ``AcmeHelloBundle``::
``AcmeHelloBundle:Hello:index``控制器是这个控制器的逻辑名，并且它映射到一个名叫
``Acme\HelloBundle\Controller\Hello``的php类中的``indexAction``方法。首先在你的
``AcmeHelloBundle``中创建这个文件::

    // src/Acme/HelloBundle/Controller/HelloController.php
    namespace Acme\HelloBundle\Controller;

    use Symfony\Component\HttpFoundation\Response;

    class HelloController
    {
    }

In reality, the controller is nothing more than a PHP method that you create
and Symfony executes. This is where your code uses information from the request
to build and prepare the resource being requested. Except in some advanced
cases, the end product of a controller is always the same: a Symfony2 ``Response``
object.
实际上，控制器仅仅是一个你创建的php方法。你的代码就在控制器中接收请求的信息并将响应的数据
准备好。除了一些特别的情况，控制器的最终输出基本都是一个symfony2 response对象。

Create the ``indexAction`` method that Symfony will execute when the ``hello``
route is matched::
当hello路径被匹配时，创建``indexAction``方法::

    // src/Acme/HelloBundle/Controller/HelloController.php

    // ...
    class HelloController
    {
        public function indexAction($name)
        {
            return new Response('<html><body>Hello '.$name.'!</body></html>');
        }
    }

The controller is simple: it creates a new ``Response`` object, whose first
argument is the content that should be used in the response (a small HTML
page in this example).
控制器很简单：它创建一个新的response对象，它的第一个参数是要被响应使用的内容（在这个例子中是一个HTML文件）。

Congratulations! After creating only a route and a controller, you already
have a fully-functional page! If you've setup everything correctly, your
application should greet you:
祝贺你！在创建一个路径和一个控制器之后，你已经有了一个具有完整功能的页面了!如果你
将所有东西都设置正确的话，你的应用会是：

.. code-block:: text

    http://localhost/app_dev.php/hello/Ryan

.. tip::

    You can also view your app in the "prod" :ref:`environment<environments-summary>`
    by visiting:
    你也可以在prod环境（:ref:`environment<environments-summary>`）中查看你的应用，通过：

    .. code-block:: text

        http://localhost/app.php/hello/Ryan

    If you get an error, it's likely because you need to clear your cache
    by running:
    如果你得到一个错误，可能你还要清空缓存：

    .. code-block:: bash

        php app/console cache:clear --env=prod --no-debug

An optional, but common, third step in the process is to create a template.
还有一个可选的，但也很常用的第三步。

.. note::

   Controllers are the main entry point for your code and a key ingredient
   when creating pages. Much more information can be found in the
   :doc:`Controller Chapter </book/controller>`.

Optional Step 3: Create the Template
第三步：创建模板
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Templates allows you to move all of the presentation (e.g. HTML code) into
a separate file and reuse different portions of the page layout. Instead
of writing the HTML inside the controller, render a template instead:
模板允许你将所有要输出的结果放在一个独立的文件中，并重复使用页面的不同部分。
提交一个模板，而不是直接在控制器中写HTML代码：

.. code-block:: php
    :linenos:

    // src/Acme/HelloBundle/Controller/HelloController.php
    namespace Acme\HelloBundle\Controller;

    use Symfony\Bundle\FrameworkBundle\Controller\Controller;

    class HelloController extends Controller
    {
        public function indexAction($name)
        {
            return $this->render('AcmeHelloBundle:Hello:index.html.twig', array('name' => $name));

            // render a PHP template instead
            // return $this->render('AcmeHelloBundle:Hello:index.html.php', array('name' => $name));
        }
    }

.. note::

   In order to use the ``render()`` method, your controller must extend the
   ``Symfony\Bundle\FrameworkBundle\Controller\Controller`` class (API
   docs: :class:`Symfony\\Bundle\\FrameworkBundle\\Controller\\Controller`),
   which adds shortcuts for tasks that are common inside controllers. This
   is done in the above example by adding the ``use`` statement on line 4
   and then extending ``Controller`` on line 6.
   要使用render()方法，你的控制器必须扩展``Symfony\Bundle\FrameworkBundle\Controller\Controller``
   类（参见docs: :class:`Symfony\\Bundle\\FrameworkBundle\\Controller\\Controller`），
   其中有许多常用任务的简便方法。在以上的例子中，这是通过在第4行添加use语句并在第6行扩展Controller
   实现的。

The ``render()`` method creates a ``Response`` object filled with the content
of the given, rendered template. Like any other controller, you will ultimately
return that ``Response`` object.
render()方法创建了一个response对象并提交了模板。像其他控制器一样，你最终会返回response对象。

Notice that there are two different examples for rendering the template.
By default, Symfony2 supports two different templating languages: classic
PHP templates and the succinct but powerful `Twig`_ templates. Don't be
alarmed - you're free to choose either or even both in the same project.
注意有两种不同的方法提交模板。默认的，symfony2支持两种不同的模板语言：经典php模板以及
简介但强大的`Twig`_模板。你可以自由选择其中之一或者两者都用。

The controller renders the ``AcmeHelloBundle:Hello:index.html.twig`` template,
which uses the following naming convention:
控制器提交``AcmeHelloBundle:Hello:index.html.twig``模板，这个模板使用如下的命名模式：

    **BundleName**:**ControllerName**:**TemplateName**

This is the *logical* name of the template, which is mapped to a physical
location using the following convention.
这是这个模板的逻辑名，它会映射到如下路径。

    **/path/to/BundleName**/Resources/views/**ControllerName**/**TemplateName**

In this case, ``AcmeHelloBundle`` is the bundle name, ``Hello`` is the
controller, and ``index.html.twig`` the template:
在这个例子中，``AcmeHelloBundle``是bundle名，Hello是控制器，``index.html.twig``是模板：

.. configuration-block::

    .. code-block:: jinja
       :linenos:

        {# src/Acme/HelloBundle/Resources/views/Hello/index.html.twig #}
        {% extends '::base.html.twig' %}

        {% block body %}
            Hello {{ name }}!
        {% endblock %}

    .. code-block:: php

        <!-- src/Acme/HelloBundle/Resources/views/Hello/index.html.php -->
        <?php $view->extend('::base.html.php') ?>

        Hello <?php echo $view->escape($name) ?>!

Let's step through the Twig template line-by-line:
让我们学习下以上代码：

* *line 2*: The ``extends`` token defines a parent template. The template
  explicitly defines a layout file inside of which it will be placed.
  *第二行*：extends定义了一个父模板。这个模板显性地定义了一个外部文件，它将被放置在这个外部文件中。

* *line 4*: The ``block`` token says that everything inside should be placed
  inside a block called ``body``. As you'll see, it's the responsibility
  of the parent template (``base.html.twig``) to ultimately render the
  block called ``body``.
  *第四行*：block表示在它里面的所有东西都必须被放置在一个名叫body的block中。
  如你所见，提交这个名叫body的block是父模板（``base.html.twig``）的任务。

The parent template, ``::base.html.twig``, is missing both the **BundleName**
and **ControllerName** portions of its name (hence the double colon (``::``)
at the beginning). This means that the template lives outside of the bundles
and in the ``app`` directory:
父模板（``base.html.twig``）中，**BundleName**和**ControllerName**部分都没有写出（于是就有了两个引号(``::``)）。
这表示这个模板在bundle外部并在app目录中：

.. configuration-block::

    .. code-block:: html+jinja

        {# app/Resources/views/base.html.twig #}
        <!DOCTYPE html>
        <html>
            <head>
                <meta http-equiv="Content-Type" content="text/html; charset=utf-8" />
                <title>{% block title %}Welcome!{% endblock %}</title>
                {% block stylesheets %}{% endblock %}
                <link rel="shortcut icon" href="{{ asset('favicon.ico') }}" />
            </head>
            <body>
                {% block body %}{% endblock %}
                {% block javascripts %}{% endblock %}
            </body>
        </html>

    .. code-block:: php

        <!-- app/Resources/views/base.html.php -->
        <!DOCTYPE html>
        <html>
            <head>
                <meta http-equiv="Content-Type" content="text/html; charset=utf-8" />
                <title><?php $view['slots']->output('title', 'Welcome!') ?></title>
                <?php $view['slots']->output('stylesheets') ?>
                <link rel="shortcut icon" href="<?php echo $view['assets']->getUrl('favicon.ico') ?>" />
            </head>
            <body>
                <?php $view['slots']->output('_content') ?>
                <?php $view['slots']->output('stylesheets') ?>
            </body>
        </html>

The base template file defines the HTML layout and renders the ``body`` block
that you defined in the ``index.html.twig`` template. It also renders a ``title``
block, which you could choose to define in the ``index.html.twig`` template.
Since you did not define the ``title`` block in the child template, it defaults
to "Welcome!".
父模板定义了HTML并输出你在``index.html.twig``模板中定义的body这个block。它还输出了一个title
block，你也可以选择在``index.html.twig``模板中定义它。但因为你没有在子模板里定义这个title block，
它默认就是"Welcome!"。

Templates are a powerful way to render and organize the content for your
page. A template can render anything, from HTML markup, to CSS code, or anything
else that the controller may need to return.
模板是一个输出和组织你的页面的强大方式。一个模板可以输出任何东西，从HTML到CSS，或任何控制器要返回的东西。

In the lifecycle of handling a request, the templating engine is simply
an optional tool. Recall that the goal of each controller is to return a
``Response`` object. Templates are a powerful, but optional, tool for creating
the content for that ``Response`` object.
在处理一个请求的过程中，模板引擎仅仅是一个可选工具。回忆起每个控制器的目标是返回一个response对象，模板
很强大，但是是可选的，仅是一个为那个response对象创建内容的工具。

.. index::
   single: Directory Structure

The Directory Structure
目录结构
-----------------------

After just a few short sections, you already understand the philosophy behind
creating and rendering pages in Symfony2. You've also already begun to see
how Symfony2 projects are structured and organized. By the end of this section,
you'll know where to find and put different types of files and why.
现在你已经了解了symfony2中创建并输出页面的方法。你已经开始看到symfony2的代码是如何组织的。
在本节末尾，你将知道在哪以及如何放置不同类型的文件，以及为什么。

Though entirely flexible, by default, each Symfony :term:`application` has
the same basic and recommended directory structure:
虽然十分灵活，默认情况下，每个symfony :term:`application`都有它基本的目录架构：

* ``app/``: This directory contains the application configuration;
* ``app/``: 这个目录包含了应用的配置；

* ``src/``: All the project PHP code is stored under this directory;
* ``src/``: 所有应用的php代码都存储在这个目录中；

* ``vendor/``: Any vendor libraries are placed here by convention;
* ``vendor/``: 按惯例所有的vendor库都被放置在这里；

* ``web/``: This is the web root directory and contains any publicly accessible files;
* ``web/``: 这是web根目录并且包含了任何公共的可以进入的文件。

The Web Directory
web目录
~~~~~~~~~~~~~~~~~

The web root directory is the home of all public and static files including
images, stylesheets, and JavaScript files. It is also where each
:term:`front controller` lives::
web根目录是所有公共和静态的文件的所在地，包括图像，样式表，以及javascript文件。它也是:term:`front controller`
的所在地::

    // web/app.php
    require_once __DIR__.'/../app/bootstrap.php.cache';
    require_once __DIR__.'/../app/AppKernel.php';

    use Symfony\Component\HttpFoundation\Request;

    $kernel = new AppKernel('prod', false);
    $kernel->loadClassCache();
    $kernel->handle(Request::createFromGlobals())->send();

The front controller file (``app.php`` in this example) is the actual PHP
file that's executed when using a Symfony2 application and its job is to
use a Kernel class, ``AppKernel``, to bootstrap the application.
前端控制器（front controller）文件（在这个例子中是app.php）是一个在使用symfony2应用的
时候会被执行的php文件，它的任务就是使用一个核心类（``AppKernel``）来引导这个应用。

.. tip::

    Having a front controller means different and more flexible URLs than
    are used in a typical flat PHP application. When using a front controller,
    URLs are formatted in the following way:
    有一个前端控制器意味着你可以使用比传统php应用更加灵活的URL。当使用前端控制器的时候，
    URL都按下面的形式：

    .. code-block:: text

        http://localhost/app.php/hello/Ryan

    The front controller, ``app.php``, is executed and the "internal:" URL
    ``/hello/Ryan`` is routed internally using the routing configuration.
    By using Apache ``mod_rewrite`` rules, you can force the ``app.php`` file
    to be executed without needing to specify it in the URL:
    这个前端控制器（``app.php``）被执行，并且这个"internal:" URL ``/hello/Ryan``
    被通过路径配置生效了。通过使用Apache ``mod_rewrite``规则，你不需要在URL中指定它，就可以强制app.php文件执行了。

    .. code-block:: text

        http://localhost/hello/Ryan

Though front controllers are essential in handling every request, you'll
rarely need to modify or even think about them. We'll mention them again
briefly in the `Environments`_ section.
虽然前端控制器很重要，但你几乎不需要考虑，也不需要修改它。我们在`Environments`_一节中会提到它。

The Application (``app``) Directory
应用(``app``)目录
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

As you saw in the front controller, the ``AppKernel`` class is the main entry
point of the application and is responsible for all configuration. As such,
it is stored in the ``app/`` directory.
像你在前端控制器中看到的，AppKernel类是应用的主要入口，并管理所有配置。所以，它被存储在app/目录下。

This class must implement two methods that define everything that Symfony
needs to know about your application. You don't even need to worry about
these methods when starting - Symfony fills them in for you with sensible
defaults.
这个类必须植入（implement）两个方法，这两个方法定义所有symfony需要知道的关于你的应用的配置。
你不必担心这些方法——symfony为你设置了默认的信息。

* ``registerBundles()``: Returns an array of all bundles needed to run the
  application (see :ref:`page-creation-bundles`);
* ``registerBundles()``: 以数组的形式返回应用所需的所有bundle（参见:ref:`page-creation-bundles`）；

* ``registerContainerConfiguration()``: Loads the main application configuration
  resource file (see the `Application Configuration`_ section).
* ``registerContainerConfiguration()``: 载入主要的应用配置文件（参见`Application Configuration`_）；
  
In day-to-day development, you'll mostly use the ``app/`` directory to modify
configuration and routing files in the ``app/config/`` directory (see
`Application Configuration`_). It also contains the application cache
directory (``app/cache``), a log directory (``app/logs``) and a directory
for application-level resource files, such as templates (``app/Resources``).
You'll learn more about each of these directories in later chapters.
在开发过程中，你基本会使用app/目录来修改配置并在``app/config/``目录下配置路径（参见`Application Configuration`_）。
它还包含应用缓存目录（``app/cache``），一个log目录（``app/logs``）和一个应用层的源文件的目录，比如
模板(``app/Resources``)。在后面的章节中，你还会学习更多关于这些目录的知识。

.. _autoloading-introduction-sidebar:

.. sidebar:: Autoloading

    When Symfony is loading, a special file - ``app/autoload.php`` - is included.
    This file is responsible for configuring the autoloader, which will autoload
    your application files from the ``src/`` directory and third-party libraries
    from the ``vendor/`` directory.
    当symfony载入时，一个``app/autoload.php``被包含。这个文件是用于配置autoloader，
    它可以从src/目录下和vendor/目录的第三方库中自动载入你的应用文件。

    Because of the autoloader, you never need to worry about using ``include``
    or ``require`` statements. Instead, Symfony2 uses the namespace of a class
    to determine its location and automatically includes the file on your
    behalf the instant you need a class.
    因为有了autoloader，你不必担心include或require语句。相反的，symfony2使用一个类的命名空间（namespace）
    来决定它的位置并自动将这个文件包含。

    The autoloader is already configured to look in the ``src/`` directory
    for any of your PHP classes. For autoloading to work, the class name and
    path to the file have to follow the same pattern:
    autoloader已经被配置好，并可以查看src/目录并查找任何你的php类。若要自动载入能够工作，你要
    使类名和访问文件的路径遵循以下的模式：

    .. code-block:: text

        Class Name:
            Acme\HelloBundle\Controller\HelloController
        Path:
            src/Acme/HelloBundle/Controller/HelloController.php

    Typically, the only time you'll need to worry about the ``app/autoload.php``
    file is when you're including a new third-party library in the ``vendor/``
    directory. For more information on autoloading, see
    :doc:`How to autoload Classes</components/class_loader>`.
    通常，你唯一需要关注``app/autoload.php``的时候是你需要在vendor/中包含一个第三方库的时候。
    自动加载的更多信息请参见:doc:`How to autoload Classes</components/class_loader>`。

The Source (``src``) Directory
源文件（``src``）目录
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Put simply, the ``src/`` directory contains all of the actual code (PHP code,
templates, configuration files, stylesheets, etc) that drives *your* application.
When developing, the vast majority of your work will be done inside one or
more bundles that you create in this directory.
简单地说，``src/``目录包含所有你编写的实现应用的代码（php代码，模板，配置文件，css等等）。
当开发是，你的主要工作会在这个目录下的你所创建的一个或多个bundle中完成。

But what exactly is a :term:`bundle`?
什么是:term:`bundle`？

.. _page-creation-bundles:

The Bundle System
bundle系统
-----------------

A bundle is similar to a plugin in other software, but even better. The key
difference is that *everything* is a bundle in Symfony2, including both the
core framework functionality and the code written for your application.
Bundles are first-class citizens in Symfony2. This gives you the flexibility
to use pre-built features packaged in `third-party bundles`_ or to distribute
your own bundles. It makes it easy to pick and choose which features to enable
in your application and to optimize them the way you want.
bundle就好比其他软件中的插件，主要不同就是symfony2中的每样东西都是bundle，包括它的核心框架
功能以及你为你的应用所写的代码。
bundle是symfony2中的一级市民，这使你能够使用在`third-party bundles`_中事先打包的功能
或发布你自己的bundle。你能够很容易地选择要使用哪个bundle并用你自己的方法来优化它们。

.. note::

   While you'll learn the basics here, an entire cookbook entry is devoted
   to the organization and best practices of :doc:`bundles</cookbook/bundles/best_practices>`.
   在这里你学习的只是基础，要深入学习请参阅:doc:`bundles</cookbook/bundles/best_practices>`。

A bundle is simply a structured set of files within a directory that implement
a single feature. You might create a ``BlogBundle``, a ``ForumBundle`` or
a bundle for user management (many of these exist already as open source
bundles). Each directory contains everything related to that feature, including
PHP files, templates, stylesheets, JavaScripts, tests and anything else.
Every aspect of a feature exists in a bundle and every feature lives in a
bundle.
一个bundle只是在一个植入了某功能目录中的一系列架构好的文件。你可能会创建一个``BlogBundle``, 一个``ForumBundle`` 
或一个用于用户管理的bundle（有许多已经作为开源文件存在）。每个目录都包含与那个功能相关的东西，包括php文件，模板，
样式表，javascript，test以及其他。每个功能的每个部分都在bundle中。

An application is made up of bundles as defined in the ``registerBundles()``
method of the ``AppKernel`` class::
一个应用是一些bundle构成的，这在AppKernel类中的``registerBundles()``方法中定义了::

    // app/AppKernel.php
    public function registerBundles()
    {
        $bundles = array(
            new Symfony\Bundle\FrameworkBundle\FrameworkBundle(),
            new Symfony\Bundle\SecurityBundle\SecurityBundle(),
            new Symfony\Bundle\TwigBundle\TwigBundle(),
            new Symfony\Bundle\MonologBundle\MonologBundle(),
            new Symfony\Bundle\SwiftmailerBundle\SwiftmailerBundle(),
            new Symfony\Bundle\DoctrineBundle\DoctrineBundle(),
            new Symfony\Bundle\AsseticBundle\AsseticBundle(),
            new Sensio\Bundle\FrameworkExtraBundle\SensioFrameworkExtraBundle(),
            new JMS\SecurityExtraBundle\JMSSecurityExtraBundle(),
        );

        if (in_array($this->getEnvironment(), array('dev', 'test'))) {
            $bundles[] = new Acme\DemoBundle\AcmeDemoBundle();
            $bundles[] = new Symfony\Bundle\WebProfilerBundle\WebProfilerBundle();
            $bundles[] = new Sensio\Bundle\DistributionBundle\SensioDistributionBundle();
            $bundles[] = new Sensio\Bundle\GeneratorBundle\SensioGeneratorBundle();
        }

        return $bundles;
    }

With the ``registerBundles()`` method, you have total control over which bundles
are used by your application (including the core Symfony bundles).
通过``registerBundles()``方法，你对你的应用要用到哪些bundle有着完全的控制。

.. tip::

   A bundle can live *anywhere* as long as it can be autoloaded (via the
   autoloader configured at ``app/autoload.php``).
   一个bundle可以被放置在任何地方，因为它可以被自动加载（通过在``app/autoload.php``中的autoloader）。

Creating a Bundle
创建一个bundle
~~~~~~~~~~~~~~~~~

The Symfony Standard Edition comes with a handy task that creates a fully-functional
bundle for you. Of course, creating a bundle by hand is pretty easy as well.
symfony标准版能方便地创建一个全功能的bundle。当然，手动创建一个bundle也很容易。

To show you how simple the bundle system is, create a new bundle called
``AcmeTestBundle`` and enable it.
现在要展示bundle系统有多么容易，创建一个名叫``AcmeTestBundle``的bundle并且
激活它。

.. tip::

    The ``Acme`` portion is just a dummy name that should be replaced by
    some "vendor" name that represents you or your organization (e.g. ``ABCTestBundle``
    for some company named ``ABC``).
    Acme部分只是一个假名而已，你应该把它替换成能代表你或你的组织的vendor名称（如给一个名为ABC的公司
    将bundle命名为``ABCTestBundle``）。

Start by creating a ``src/Acme/TestBundle/`` directory and adding a new file
called ``AcmeTestBundle.php``::
创建一个``src/Acme/TestBundle/``目录，并添加一个名叫``AcmeTestBundle.php``的新文件::

    // src/Acme/TestBundle/AcmeTestBundle.php
    namespace Acme\TestBundle;

    use Symfony\Component\HttpKernel\Bundle\Bundle;

    class AcmeTestBundle extends Bundle
    {
    }

.. tip::

   The name ``AcmeTestBundle`` follows the standard :ref:`Bundle naming conventions<bundles-naming-conventions>`.
   You could also choose to shorten the name of the bundle to simply ``TestBundle``
   by naming this class ``TestBundle`` (and naming the file ``TestBundle.php``).
   ``AcmeTestBundle``这个名称遵循标准:ref:`Bundle naming conventions<bundles-naming-conventions>`。你也可以
   选择将这个bundle名称简化为``TestBundle``，只要将类名命名为``TestBundle``（并将文件命名为``TestBundle.php``）。

This empty class is the only piece you need to create the new bundle. Though
commonly empty, this class is powerful and can be used to customize the behavior
of the bundle.
这个空类是唯一你要创建一个新bundle所需要的。虽然通常是空的，这个类也很有用，它可以被用来定制bundle的行为。

Now that you've created the bundle, enable it via the ``AppKernel`` class::
现在你已经创建了bundle，通过``AppKernel``类来激活它::

    // app/AppKernel.php
    public function registerBundles()
    {
        $bundles = array(
            // ...

            // register your bundles
            new Acme\TestBundle\AcmeTestBundle(),
        );
        // ...

        return $bundles;
    }

And while it doesn't do anything yet, ``AcmeTestBundle`` is now ready to
be used.
虽然现在它还没有做任何事情，``AcmeTestBundle``现在已经可以被使用了。

And as easy as this is, Symfony also provides a command-line interface for
generating a basic bundle skeleton:
像以上方法一样容易的是，symfony提供命令行方法来集成一个基本bundle框架：

.. code-block:: bash

    php app/console generate:bundle --namespace=Acme/TestBundle

The bundle skeleton generates with a basic controller, template and routing
resource that can be customized. You'll learn more about Symfony2's command-line
tools later.
这个bundle框架集成了基本的控制器，模板和路径源文件，这些都可以被定制。你将学习更多有关
symfony2的命令行知识。

.. tip::

   Whenever creating a new bundle or using a third-party bundle, always make
   sure the bundle has been enabled in ``registerBundles()``. When using
   the ``generate:bundle`` command, this is done for you.
   当你创建一个新bundle或使用一个第三方bundle的时候，要确保这个bundle已经在``registerBundles()``
   中被激活。在命令行中，你只要运行``generate:bundle``，就可以激活了。

Bundle Directory Structure
bundle的目录组成
~~~~~~~~~~~~~~~~~~~~~~~~~~

The directory structure of a bundle is simple and flexible. By default, the
bundle system follows a set of conventions that help to keep code consistent
between all Symfony2 bundles. Take a look at ``AcmeHelloBundle``, as it contains
some of the most common elements of a bundle:
bundle的目录架构十分灵活而且简单。默认情况下，bundle系统遵循一系列惯例来保证各个bundle间
代码的联系性。看一下``AcmeHelloBundle``，它包含了一个bundle中的最常用的元素：

* ``Controller/`` contains the controllers of the bundle (e.g. ``HelloController.php``);
* ``Controller/`` 包含了bundle中的控制器（e.g. ``HelloController.php``）；

* ``Resources/config/`` houses configuration, including routing configuration
  (e.g. ``routing.yml``);
* ``Resources/config/`` 存储配置，包括路径配置（e.g. ``routing.yml``）；

* ``Resources/views/`` holds templates organized by controller name (e.g.
  ``Hello/index.html.twig``);
* ``Resources/views/``存储控制器集成的模板（``Hello/index.html.twig``）；

* ``Resources/public/`` contains web assets (images, stylesheets, etc) and is
  copied or symbolically linked into the project ``web/`` directory via
  the ``assets:install`` console command;
* ``Resources/public/`` 包含web设置（图像，样式表，等等）并且通过``assets:install``命令行
  被复制到或者象征性地链接到web/目录；

* ``Tests/`` holds all tests for the bundle.
* ``Tests/`` 存储这个bundle的测试。

A bundle can be as small or large as the feature it implements. It contains
only the files you need and nothing else.
一个bundle可以很小也可以很大，取决于它所植入的功能。它只包含你所需要的文件，没有别的。

As you move through the book, you'll learn how to persist objects to a database,
create and validate forms, create translations for your application, write
tests and much more. Each of these has their own place and role within the
bundle.
你还会学到如何将对象插入数据库，创建并验证表单，为你的应用创建翻译（translation），编写测试或别的更多的工作。
这些在bundle中都有它们自己的位置和作用。

Application Configuration
应用配置
-------------------------

An application consists of a collection of bundles representing all of the
features and capabilities of your application. Each bundle can be customized
via configuration files written in YAML, XML or PHP. By default, the main
configuration file lives in the ``app/config/`` directory and is called
either ``config.yml``, ``config.xml`` or ``config.php`` depending on which
format you prefer:
一个应用包含一系列bundle的组合，这些bundle包含你的应用的所有功能。每个bundle都可以被通过``app/config/``目录中
YAML,XML或者PHP编写的文件进行配置，根据你要使用的格式，这个文件可以是``config.yml``, ``config.xml``或``config.php``：

.. configuration-block::

    .. code-block:: yaml

        # app/config/config.yml
        imports:
            - { resource: parameters.yml }
            - { resource: security.yml }

        framework:
            secret:          "%secret%"
            charset:         UTF-8
            router:          { resource: "%kernel.root_dir%/config/routing.yml" }
            # ...

        # Twig Configuration
        twig:
            debug:            "%kernel.debug%"
            strict_variables: "%kernel.debug%"

        # ...

    .. code-block:: xml

        <!-- app/config/config.xml -->
        <imports>
            <import resource="parameters.yml" />
            <import resource="security.yml" />
        </imports>

        <framework:config charset="UTF-8" secret="%secret%">
            <framework:router resource="%kernel.root_dir%/config/routing.xml" />
            <!-- ... -->
        </framework:config>

        <!-- Twig Configuration -->
        <twig:config debug="%kernel.debug%" strict-variables="%kernel.debug%" />

        <!-- ... -->

    .. code-block:: php

        $this->import('parameters.yml');
        $this->import('security.yml');

        $container->loadFromExtension('framework', array(
            'secret'          => '%secret%',
            'charset'         => 'UTF-8',
            'router'          => array('resource' => '%kernel.root_dir%/config/routing.php'),
            // ...
            ),
        ));

        // Twig Configuration
        $container->loadFromExtension('twig', array(
            'debug'            => '%kernel.debug%',
            'strict_variables' => '%kernel.debug%',
        ));

        // ...

.. note::

   You'll learn exactly how to load each file/format in the next section
   `Environments`_.
   在`Environments`_这一节，你将学习如何载入每个文件/格式。

Each top-level entry like ``framework`` or ``twig`` defines the configuration
for a particular bundle. For example, the ``framework`` key defines the configuration
for the core Symfony ``FrameworkBundle`` and includes configuration for the
routing, templating, and other core systems.
每个如``framework``或``twig``一样的顶层入口都定义了某个bundle的配置。比如，``framework``这个键（key）
定义了symfony核心``FrameworkBundle``的配置并包含了路径、模板和其他核心系统的配置。

For now, don't worry about the specific configuration options in each section.
The configuration file ships with sensible defaults. As you read more and
explore each part of Symfony2, you'll learn about the specific configuration
options of each feature.
不要担心这些特定配置。它们都被设定好了默认值。随着你对symfony2学习的深入，你将会学到每个功能的特定配置。

.. sidebar:: Configuration Formats

    Throughout the chapters, all configuration examples will be shown in all
    three formats (YAML, XML and PHP). Each has its own advantages and
    disadvantages. The choice of which to use is up to you:
    在每个章节中，每个配置范例都会以三种格式出现(YAML, XML and PHP)。每个都有它的优点和缺点。
    要用哪一个取决于你自己：

    * *YAML*: Simple, clean and readable;
    * *YAML*: 简单，明了，可读性强；

    * *XML*: More powerful than YAML at times and supports IDE autocompletion;
    * *XML*: 有时比YAML强大，并支持IDE自动完成；

    * *PHP*: Very powerful but less readable than standard configuration formats.
    * *PHP*: 很强大，但比标准配置格式的可读性差。

Default Configuration Dump
输出全部默认配置
~~~~~~~~~~~~~~~~~~~~~~~~~~

.. versionadded:: 2.1
    The ``config:dump-reference`` command was added in Symfony 2.1
    ``config:dump-reference``命令行在symfony2.1中有添加

You can dump the default configuration for a bundle in yaml to the console using
the ``config:dump-reference`` command.  Here is an example of dumping the default
FrameworkBundle configuration:
你可以将一个bundle的默认配置以yaml的形式全部输出，只要使用``config:dump-reference``命令行。以下是
一个输出FrameworkBundle配置的范例：

.. code-block:: text

    app/console config:dump-reference FrameworkBundle

The extension alias (configuration key) can also be used:
扩展别名（配置的键）也可以被使用：

.. code-block:: text

    app/console config:dump-reference framework

.. note::

    See the cookbook article: :doc:`How to expose a Semantic Configuration for
    a Bundle</cookbook/bundles/extension>` for information on adding
    configuration for your own bundle.
    要为你的bundle添加配置请参见:doc:`How to expose a Semantic Configuration for
    a Bundle</cookbook/bundles/extension>`。

.. index::
   single: Environments; Introduction

.. _environments-summary:

Environments
环境
------------

An application can run in various environments. The different environments
share the same PHP code (apart from the front controller), but use different
configuration. For instance, a ``dev`` environment will log warnings and
errors, while a ``prod`` environment will only log errors. Some files are
rebuilt on each request in the ``dev`` environment (for the developer's convenience),
but cached in the ``prod`` environment. All environments live together on
the same machine and execute the same application.
一个应用可以在不同的环境中运行。不同的环境可以使用同样的php代码，但是却使用不同的配置。
比如，一个dev环境可以记录警告和错误，但一个prod环境只会记录错误。有些文件只在dev环境中会在发送
请求时被重置（为了开发的便利），但在prod环境中却被缓存。所有环境都在同一个机器上并执行同一应用。

A Symfony2 project generally begins with three environments (``dev``, ``test``
and ``prod``), though creating new environments is easy. You can view your
application in different environments simply by changing the front controller
in your browser. To see the application in the ``dev`` environment, access
the application via the development front controller:
symfony2大致有三种环境（``dev``, ``test``和``prod``），但创建新环境很容易。你可以通过修改你的浏览器中的
前端控制器来在不同的环境里查看你的应用。如果要在dev环境中查看应用，通过：

.. code-block:: text

    http://localhost/app_dev.php/hello/Ryan

If you'd like to see how your application will behave in the production environment,
call the ``prod`` front controller instead:
如果你要看你的应用在生成（prod）环境中如何运行，使用prod前端控制器：

.. code-block:: text

    http://localhost/app.php/hello/Ryan

Since the ``prod`` environment is optimized for speed; the configuration,
routing and Twig templates are compiled into flat PHP classes and cached.
When viewing changes in the ``prod`` environment, you'll need to clear these
cached files and allow them to rebuild::
prod环境对于速度更优先；配置、路由和twig模板文件都被编译成普通php类并缓存。
当要查看prod环境中的修改时，你需要清空缓存并允许它们重建：

    php app/console cache:clear --env=prod --no-debug

.. note::

   If you open the ``web/app.php`` file, you'll find that it's configured explicitly
   to use the ``prod`` environment::
   如果你打开``web/app.php``文件，你会发现它显性地配置了要使用peod环境::

       $kernel = new AppKernel('prod', false);

   You can create a new front controller for a new environment by copying
   this file and changing ``prod`` to some other value.
   要使用一个新环境，你可以创建一个新的前端控制器，复制这个文件并将prod修改为其他值。

.. note::

    The ``test`` environment is used when running automated tests and cannot
    be accessed directly through the browser. See the :doc:`testing chapter</book/testing>`
    for more details.
    当运行自动测试的时候，测试（test）环境会自动被使用，你不能直接通过浏览器访问它。参见:doc:`testing chapter</book/testing>`。

.. index::
   single: Environments; Configuration

Environment Configuration
环境配置
~~~~~~~~~~~~~~~~~~~~~~~~~

The ``AppKernel`` class is responsible for actually loading the configuration
file of your choice::
``AppKernel``类可以加载你的配置文件::

    // app/AppKernel.php
    public function registerContainerConfiguration(LoaderInterface $loader)
    {
        $loader->load(__DIR__.'/config/config_'.$this->getEnvironment().'.yml');
    }

You already know that the ``.yml`` extension can be changed to ``.xml`` or
``.php`` if you prefer to use either XML or PHP to write your configuration.
Notice also that each environment loads its own configuration file. Consider
the configuration file for the ``dev`` environment.
你现在已经知道，``.yml``扩展可以被修改为``.xml``或``.php``扩展，如果你想使用这两种格式来配置的话。
注意每个环境都加载它自己的配置文件。假如是dev环境的配置文件：

.. configuration-block::

    .. code-block:: yaml

        # app/config/config_dev.yml
        imports:
            - { resource: config.yml }

        framework:
            router:   { resource: "%kernel.root_dir%/config/routing_dev.yml" }
            profiler: { only_exceptions: false }

        # ...

    .. code-block:: xml

        <!-- app/config/config_dev.xml -->
        <imports>
            <import resource="config.xml" />
        </imports>

        <framework:config>
            <framework:router resource="%kernel.root_dir%/config/routing_dev.xml" />
            <framework:profiler only-exceptions="false" />
        </framework:config>

        <!-- ... -->

    .. code-block:: php

        // app/config/config_dev.php
        $loader->import('config.php');

        $container->loadFromExtension('framework', array(
            'router'   => array('resource' => '%kernel.root_dir%/config/routing_dev.php'),
            'profiler' => array('only-exceptions' => false),
        ));

        // ...

The ``imports`` key is similar to a PHP ``include`` statement and guarantees
that the main configuration file (``config.yml``) is loaded first. The rest
of the file tweaks the default configuration for increased logging and other
settings conducive to a development environment.
``imports``键相当于php的include语句，并保证了主要配置文件(``config.yml``)会被首先加载。
这个文件的其他部分则修改了默认配置，使得记录和其他设置更有利于开发环境。

Both the ``prod`` and ``test`` environments follow the same model: each environment
imports the base configuration file and then modifies its configuration values
to fit the needs of the specific environment. This is just a convention,
but one that allows you to reuse most of your configuration and customize
just pieces of it between environments.
prod和test环境都遵循同样的模式：每个环境都导入基本配置文件并根据自己的需要来修改它的配置值。
这只是一个惯例，但它允许你重复使用你的配置的大部分并在不同环境之间只需修改极小部分。

Summary
总结
-------

Congratulations! You've now seen every fundamental aspect of Symfony2 and have
hopefully discovered how easy and flexible it can be. And while there are
*a lot* of features still to come, be sure to keep the following basic points
in mind:
祝贺你！现在你已经大致了解了symfony2中的基本部分，并了解了它是多么易用好学。由于还有
许多功能要掌握，请记住以下几点：

* creating a page is a three-step process involving a **route**, a **controller**
  and (optionally) a **template**.
  创建一个页面是一个分三步的过程，包括一个路径，一个控制器，和一个模板。

* each project contains just a few main directories: ``web/`` (web assets and
  the front controllers), ``app/`` (configuration), ``src/`` (your bundles),
  and ``vendor/`` (third-party code) (there's also a ``bin/`` directory that's
  used to help updated vendor libraries);
  整个网站代码只有一些主要目录：``web/`` (css，javascript和前端控制器), ``app/`` (配置), ``src/`` (你创建的bundle),
  和``vendor/``（第三方代码）（还有一个bin/目录，用于支持vendor更新）；

* each feature in Symfony2 (including the Symfony2 framework core) is organized
  into a *bundle*, which is a structured set of files for that feature;
  symfony2中的每个功能（包括symfony2框架的核心）都被组织在一个bundle中，它包含一系列为这些功能架构好的目录；

* the **configuration** for each bundle lives in the ``app/config`` directory
  and can be specified in YAML, XML or PHP;
  每个bundle的配置文件都在``app/config``目录中，并可以使用YAML,XML和PHP三种格式；

* each **environment** is accessible via a different front controller (e.g.
  ``app.php`` and ``app_dev.php``) and loads a different configuration file.
  每个环境都可以通过不同的前端控制器进入（``app.php``和``app_dev.php``）并加载不同的配置文件。

From here, each chapter will introduce you to more and more powerful tools
and advanced concepts. The more you know about Symfony2, the more you'll
appreciate the flexibility of its architecture and the power it gives you
to rapidly develop applications.
以后每个章节都会介绍给你更强大的工具和更先进的概念。你对symfony2了解得越多，你就会
越欣赏它架构的灵活，以及迅速创建应用的便捷。

.. _`Twig`: http://twig.sensiolabs.org
.. _`third-party bundles`: http://symfony2bundles.org/
.. _`Symfony Standard Edition`: http://symfony.com/download
.. _`Apache's DirectoryIndex documentation`: http://httpd.apache.org/docs/2.0/mod/mod_dir.html
.. _`Nginx HttpCoreModule location documentation`: http://wiki.nginx.org/HttpCoreModule#location
