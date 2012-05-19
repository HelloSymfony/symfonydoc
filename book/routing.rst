.. index::
   single: Routing

Routing
路由
=======

Beautiful URLs are an absolute must for any serious web application. This
means leaving behind ugly URLs like ``index.php?article_id=57`` in favor
of something like ``/read/intro-to-symfony``.
对于任何正式的web应用来说，明了的URL是必须的。这意味着你必须避免像``index.php?article_id=57``
这样的URL，而应该使用像``/read/intro-to-symfony``这样的URL。

Having flexibility is even more important. What if you need to change the
URL of a page from ``/blog`` to ``/news``? How many links should you need to
hunt down and update to make the change? If you're using Symfony's router,
the change is simple.
灵活性更重要。比如如果你想把URL从``/blog``转变到``/news``怎么办？你会改变多少链接啊？
但如果你使用symfony提供的路由，这些事情都会变得简单。

The Symfony2 router lets you define creative URLs that you map to different
areas of your application. By the end of this chapter, you'll be able to:
symfony2路由允许你定义灵活的URL，这些URL被映射到不同的应用中。学完本章节，你就可以:

* Create complex routes that map to controllers
* Generate URLs inside templates and controllers
* Load routing resources from bundles (or anywhere else)
* Debug your routes
* 创建映射到控制器的复杂路径
* 在模板和控制器内集成URL
* 从bundle中载入路由
* 调试你的路由

.. index::
   single: Routing; Basics

Routing in Action
控制器的路由
-----------------

A *route* is a map from a URL pattern to a controller. For example, suppose
you want to match any URL like ``/blog/my-post`` or ``/blog/all-about-symfony``
and send it to a controller that can look up and render that blog entry.
The route is simple:
所谓路径，就是从一个控制器到一个URL pattern的映射。比如，假设你想要匹配任何像``/blog/my-post``和``/blog/all-about-symfony``
那样的URL，并将它发送到一个能取出博客内容并输出的控制器，这个路径很简单:

.. configuration-block::

    .. code-block:: yaml

        # app/config/routing.yml
        blog_show:
            pattern:   /blog/{slug}
            defaults:  { _controller: AcmeBlogBundle:Blog:show }

    .. code-block:: xml

        <!-- app/config/routing.xml -->
        <?xml version="1.0" encoding="UTF-8" ?>
        <routes xmlns="http://symfony.com/schema/routing"
            xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
            xsi:schemaLocation="http://symfony.com/schema/routing http://symfony.com/schema/routing/routing-1.0.xsd">

            <route id="blog_show" pattern="/blog/{slug}">
                <default key="_controller">AcmeBlogBundle:Blog:show</default>
            </route>
        </routes>

    .. code-block:: php

        // app/config/routing.php
        use Symfony\Component\Routing\RouteCollection;
        use Symfony\Component\Routing\Route;

        $collection = new RouteCollection();
        $collection->add('blog_show', new Route('/blog/{slug}', array(
            '_controller' => 'AcmeBlogBundle:Blog:show',
        )));

        return $collection;

The pattern defined by the ``blog_show`` route acts like ``/blog/*`` where
the wildcard is given the name ``slug``. For the URL ``/blog/my-blog-post``,
the ``slug`` variable gets a value of ``my-blog-post``, which is available
for you to use in your controller (keep reading).
这个在名为``blog_show``路径中定义的pattern ``/blog/*``是个能够根据给定的``slug``匹配的通配符。
对于URL ``/blog/my-blog-post``，这个slug变量取得了一个``my-blog-post``值，你可以在控制器中用这个值。

The ``_controller`` parameter is a special key that tells Symfony which controller
should be executed when a URL matches this route. The ``_controller`` string
is called the :ref:`logical name<controller-string-syntax>`. It follows a
pattern that points to a specific PHP class and method:
``_controller``参数是一个特别的关键词，它告诉symfony当一个URL匹配这个路径时哪个控制器要用到。``_controller``
被称作:ref:`logical name<controller-string-syntax>`。它能访问一个特定的PHP类和方法:

.. code-block:: php

    // src/Acme/BlogBundle/Controller/BlogController.php

    namespace Acme\BlogBundle\Controller;
    use Symfony\Bundle\FrameworkBundle\Controller\Controller;

    class BlogController extends Controller
    {
        public function showAction($slug)
        {
            $blog = // use the $slug varible to query the database

            return $this->render('AcmeBlogBundle:Blog:show.html.twig', array(
                'blog' => $blog,
            ));
        }
    }

Congratulations! You've just created your first route and connected it to
a controller. Now, when you visit ``/blog/my-post``, the ``showAction`` controller
will be executed and the ``$slug`` variable will be equal to ``my-post``.
祝贺你!你已经创建了你的第一个路径并将它连到了控制器。现在当你访问``/blog/my-post``时，showAction控制器
会被执行，``$slug``变量也就等于``my-post``。

This is the goal of the Symfony2 router: to map the URL of a request to a
controller. Along the way, you'll learn all sorts of tricks that make mapping
even the most complex URLs easy.
这就是symfony路由的目的：将一个URL映射到一个控制器。以下你将学习如何使用更复杂的映射。

.. versionadded:: 2.1

    As of Symfony 2.1, the Routing component also accepts Unicode values
    in routes like: /Жени/
    symfony2.1中，路由可以接受Unicode值。比如：/Жени/

.. index::
   single: Routing; Under the hood

Routing: Under the Hood
路由的内部工作流程
-----------------------

When a request is made to your application, it contains an address to the
exact "resource" that the client is requesting. This address is called the
URL, (or URI), and could be ``/contact``, ``/blog/read-me``, or anything
else. Take the following HTTP request for example:
当在你的应用中创建一个请求时，它包含了一个客户端请求的一个访问特定”源“的地址，这个地址就叫URL（或URI），
它可以是``/contact``, ``/blog/read-me``,或其他任何东西。以下是一个HTTP请求:

.. code-block:: text

    GET /blog/my-blog-post

The goal of the Symfony2 routing system is to parse this URL and determine
which controller should be executed. The whole process looks like this:
symfony2路由系统的目的就是解析这个URL并决定哪个控制器会被执行。整个流程如下：

#. The request is handled by the Symfony2 front controller (e.g. ``app.php``);
#. 这个请求是由symfony2前端控制器（也就是app.php）处理的；

#. The Symfony2 core (i.e. Kernel) asks the router to inspect the request;
#. symony2核心（也就是kernel）要路由监测这个请求；

#. The router matches the incoming URL to a specific route and returns information
   about the route, including the controller that should be executed;
#. 路由将这个URL匹配到某个特定的路径，并返回有关信息，包括哪个控制器需要被执行；

#. The Symfony2 Kernel executes the controller, which ultimately returns
   a ``Response`` object.
#. symfony2 kernel执行这个控制器方法，并最终返回response对象。

.. figure:: /images/request-flow.png
   :align: center
   :alt: Symfony2 request flow

   The routing layer is a tool that translates the incoming URL into a specific
   controller to execute.

.. index::
   single: Routing; Creating routes

Creating Routes
创建路径
---------------

Symfony loads all the routes for your application from a single routing configuration
file. The file is usually ``app/config/routing.yml``, but can be configured
to be anything (including an XML or PHP file) via the application configuration
file:
symfony从一个单独的路径配置文件载入你的应用所需的所有路径。这个文件通常是``app/config/routing.yml``，但是
也可以用别的格式如 XML 或 PHP文件： 

.. configuration-block::

    .. code-block:: yaml

        # app/config/config.yml
        framework:
            # ...
            router:        { resource: "%kernel.root_dir%/config/routing.yml" }

    .. code-block:: xml

        <!-- app/config/config.xml -->
        <framework:config ...>
            <!-- ... -->
            <framework:router resource="%kernel.root_dir%/config/routing.xml" />
        </framework:config>

    .. code-block:: php

        // app/config/config.php
        $container->loadFromExtension('framework', array(
            // ...
            'router'        => array('resource' => '%kernel.root_dir%/config/routing.php'),
        ));

.. tip::

    Even though all routes are loaded from a single file, it's common practice
    to include additional routing resources from inside the file. See the
    :ref:`routing-include-external-resources` section for more information.
    虽然所有的路径都是从一个单独的文件中载入的，但也可以在这个文件中包含外部的路径配置文件。
    请参阅:ref:`routing-include-external-resources`。

Basic Route Configuration
基本路径配置
~~~~~~~~~~~~~~~~~~~~~~~~~

Defining a route is easy, and a typical application will have lots of routes.
A basic route consists of just two parts: the ``pattern`` to match and a
``defaults`` array:
定义一个路径很容易，一个典型的应用会包含很多路径。一个基本路径包含两个部分：要匹配的pattern和一个defaults：

.. configuration-block::

    .. code-block:: yaml

        _welcome:
            pattern:   /
            defaults:  { _controller: AcmeDemoBundle:Main:homepage }

    .. code-block:: xml

        <?xml version="1.0" encoding="UTF-8" ?>

        <routes xmlns="http://symfony.com/schema/routing"
            xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
            xsi:schemaLocation="http://symfony.com/schema/routing http://symfony.com/schema/routing/routing-1.0.xsd">

            <route id="_welcome" pattern="/">
                <default key="_controller">AcmeDemoBundle:Main:homepage</default>
            </route>

        </routes>

    ..  code-block:: php

        use Symfony\Component\Routing\RouteCollection;
        use Symfony\Component\Routing\Route;

        $collection = new RouteCollection();
        $collection->add('_welcome', new Route('/', array(
            '_controller' => 'AcmeDemoBundle:Main:homepage',
        )));

        return $collection;

This route matches the homepage (``/``) and maps it to the ``AcmeDemoBundle:Main:homepage``
controller. The ``_controller`` string is translated by Symfony2 into an
actual PHP function and executed. That process will be explained shortly
in the :ref:`controller-string-syntax` section.
这个路径匹配了主页路径(``/``)并且将它映射到``AcmeDemoBundle:Main:homepage``控制器。``_controller``
被symfony编译成一个特定的PHP方法并执行。这个过程在:ref:`controller-string-syntax`这节会讲述

.. index::
   single: Routing; Placeholders

Routing with Placeholders
有占位符的路径
~~~~~~~~~~~~~~~~~~~~~~~~~

Of course the routing system supports much more interesting routes. Many
routes will contain one or more named "wildcard" placeholders:
当然路由系统包含了更多有意思的路径。许多路径都会包含一个或多个”通配符“占位符：

.. configuration-block::

    .. code-block:: yaml

        blog_show:
            pattern:   /blog/{slug}
            defaults:  { _controller: AcmeBlogBundle:Blog:show }

    .. code-block:: xml

        <?xml version="1.0" encoding="UTF-8" ?>

        <routes xmlns="http://symfony.com/schema/routing"
            xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
            xsi:schemaLocation="http://symfony.com/schema/routing http://symfony.com/schema/routing/routing-1.0.xsd">

            <route id="blog_show" pattern="/blog/{slug}">
                <default key="_controller">AcmeBlogBundle:Blog:show</default>
            </route>
        </routes>

    .. code-block:: php

        use Symfony\Component\Routing\RouteCollection;
        use Symfony\Component\Routing\Route;

        $collection = new RouteCollection();
        $collection->add('blog_show', new Route('/blog/{slug}', array(
            '_controller' => 'AcmeBlogBundle:Blog:show',
        )));

        return $collection;

The pattern will match anything that looks like ``/blog/*``. Even better,
the value matching the ``{slug}`` placeholder will be available inside your
controller. In other words, if the URL is ``/blog/hello-world``, a ``$slug``
variable, with a value of ``hello-world``, will be available in the controller.
This can be used, for example, to load the blog post matching that string.
这个pattern会匹配任何像``/blog/*``的路径。甚至，这个``{slug}``占位符也会在你的控制器作为可用参数。
换句话说，如果URL是``/blog/hello-world``，那么这个$slug变量（值为``hello-world``），就可以在控制器中被使用。
这个可以被用来加载某个匹配该变量的博客文章。

The pattern will *not*, however, match simply ``/blog``. That's because,
by default, all placeholders are required. This can be changed by adding
a placeholder value to the ``defaults`` array.
这个pattern不会匹配``/blog``。这是因为默认情况下，所有的占位符都必须输出。如果你不想输出，
可以把占位符加到defaults中。

Required and Optional Placeholders
必须的和可选的占位符
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

To make things more exciting, add a new route that displays a list of all
the available blog posts for this imaginary blog application:
在这个假设的博客应用中，添加一个新的路径，这个路径可以显示所有的博客文章：

.. configuration-block::

    .. code-block:: yaml

        blog:
            pattern:   /blog
            defaults:  { _controller: AcmeBlogBundle:Blog:index }

    .. code-block:: xml

        <?xml version="1.0" encoding="UTF-8" ?>

        <routes xmlns="http://symfony.com/schema/routing"
            xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
            xsi:schemaLocation="http://symfony.com/schema/routing http://symfony.com/schema/routing/routing-1.0.xsd">

            <route id="blog" pattern="/blog">
                <default key="_controller">AcmeBlogBundle:Blog:index</default>
            </route>
        </routes>

    .. code-block:: php

        use Symfony\Component\Routing\RouteCollection;
        use Symfony\Component\Routing\Route;

        $collection = new RouteCollection();
        $collection->add('blog', new Route('/blog', array(
            '_controller' => 'AcmeBlogBundle:Blog:index',
        )));

        return $collection;

So far, this route is as simple as possible - it contains no placeholders
and will only match the exact URL ``/blog``. But what if you need this route
to support pagination, where ``/blog/2`` displays the second page of blog
entries? Update the route to have a new ``{page}`` placeholder:
目前这个路径尽量简化了——它不包含占位符并只匹配``/blog``这个URL。但是如果你想要这个路径
支持分页，比如``/blog/2``显示这个博客的第二页呢？给这个路径添加一个新的``{page}``占位符吧:

.. configuration-block::

    .. code-block:: yaml

        blog:
            pattern:   /blog/{page}
            defaults:  { _controller: AcmeBlogBundle:Blog:index }

    .. code-block:: xml

        <?xml version="1.0" encoding="UTF-8" ?>

        <routes xmlns="http://symfony.com/schema/routing"
            xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
            xsi:schemaLocation="http://symfony.com/schema/routing http://symfony.com/schema/routing/routing-1.0.xsd">

            <route id="blog" pattern="/blog/{page}">
                <default key="_controller">AcmeBlogBundle:Blog:index</default>
            </route>
        </routes>

    .. code-block:: php

        use Symfony\Component\Routing\RouteCollection;
        use Symfony\Component\Routing\Route;

        $collection = new RouteCollection();
        $collection->add('blog', new Route('/blog/{page}', array(
            '_controller' => 'AcmeBlogBundle:Blog:index',
        )));

        return $collection;

Like the ``{slug}`` placeholder before, the value matching ``{page}`` will
be available inside your controller. Its value can be used to determine which
set of blog posts to display for the given page.
像刚才讲的{slug}占位符，{page}也能被用于你的控制器。它的值可以被用来决定哪些博客文章要在这个
page中被显示。

But hold on! Since placeholders are required by default, this route will
no longer match on simply ``/blog``. Instead, to see page 1 of the blog,
you'd need to use the URL ``/blog/1``! Since that's no way for a rich web
app to behave, modify the route to make the ``{page}`` parameter optional.
This is done by including it in the ``defaults`` collection:
等等！因为默认要一个占位符，所以这个路径不会再匹配/blog,当你想查看第一页时，你就得输入/blog/1!
当然一个好的web应用不应该这样，你可以改变这个路径配置，使得{page}这个参数成为可选的。要达到这个目的，
可以将它包含在defaults中：

.. configuration-block::

    .. code-block:: yaml

        blog:
            pattern:   /blog/{page}
            defaults:  { _controller: AcmeBlogBundle:Blog:index, page: 1 }

    .. code-block:: xml

        <?xml version="1.0" encoding="UTF-8" ?>

        <routes xmlns="http://symfony.com/schema/routing"
            xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
            xsi:schemaLocation="http://symfony.com/schema/routing http://symfony.com/schema/routing/routing-1.0.xsd">

            <route id="blog" pattern="/blog/{page}">
                <default key="_controller">AcmeBlogBundle:Blog:index</default>
                <default key="page">1</default>
            </route>
        </routes>

    .. code-block:: php

        use Symfony\Component\Routing\RouteCollection;
        use Symfony\Component\Routing\Route;

        $collection = new RouteCollection();
        $collection->add('blog', new Route('/blog/{page}', array(
            '_controller' => 'AcmeBlogBundle:Blog:index',
            'page' => 1,
        )));

        return $collection;

By adding ``page`` to the ``defaults`` key, the ``{page}`` placeholder is no
longer required. The URL ``/blog`` will match this route and the value of
the ``page`` parameter will be set to ``1``. The URL ``/blog/2`` will also
match, giving the ``page`` parameter a value of ``2``. Perfect.
当将page添加到defaults中时，这个{page}就可以不必被输出了。URL /blog会匹配这个路径，并且
这个page参数会被设置为”1“。URL /blog/2也能匹配，只要page的参数值为2。

+---------+------------+
| /blog   | {page} = 1 |
+---------+------------+
| /blog/1 | {page} = 1 |
+---------+------------+
| /blog/2 | {page} = 2 |
+---------+------------+

.. index::
   single: Routing; Requirements

Adding Requirements
添加请求
~~~~~~~~~~~~~~~~~~~

Take a quick look at the routes that have been created so far:
让我们看一下我们已经创建的路径：

.. configuration-block::

    .. code-block:: yaml

        blog:
            pattern:   /blog/{page}
            defaults:  { _controller: AcmeBlogBundle:Blog:index, page: 1 }

        blog_show:
            pattern:   /blog/{slug}
            defaults:  { _controller: AcmeBlogBundle:Blog:show }

    .. code-block:: xml

        <?xml version="1.0" encoding="UTF-8" ?>

        <routes xmlns="http://symfony.com/schema/routing"
            xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
            xsi:schemaLocation="http://symfony.com/schema/routing http://symfony.com/schema/routing/routing-1.0.xsd">

            <route id="blog" pattern="/blog/{page}">
                <default key="_controller">AcmeBlogBundle:Blog:index</default>
                <default key="page">1</default>
            </route>

            <route id="blog_show" pattern="/blog/{slug}">
                <default key="_controller">AcmeBlogBundle:Blog:show</default>
            </route>
        </routes>

    .. code-block:: php

        use Symfony\Component\Routing\RouteCollection;
        use Symfony\Component\Routing\Route;

        $collection = new RouteCollection();
        $collection->add('blog', new Route('/blog/{page}', array(
            '_controller' => 'AcmeBlogBundle:Blog:index',
            'page' => 1,
        )));

        $collection->add('blog_show', new Route('/blog/{show}', array(
            '_controller' => 'AcmeBlogBundle:Blog:show',
        )));

        return $collection;

Can you spot the problem? Notice that both routes have patterns that match
URL's that look like ``/blog/*``. The Symfony router will always choose the
**first** matching route it finds. In other words, the ``blog_show`` route
will *never* be matched. Instead, a URL like ``/blog/my-blog-post`` will match
the first route (``blog``) and return a nonsense value of ``my-blog-post``
to the ``{page}`` parameter.
你看到这个问题了吗？注意两个路径都匹配像/blog/*那样的URL，但symfony只会选择第一个匹配的url。
换句话说，路径``blog_show``不会被匹配。相反的，``/blog/my-blog-post``会匹配第一个路径（blog）并将
``my-blog-post``返回到{page}参数，这就导致混乱了。

+--------------------+-------+-----------------------+
| URL                | route | parameters            |
+====================+=======+=======================+
| /blog/2            | blog  | {page} = 2            |
+--------------------+-------+-----------------------+
| /blog/my-blog-post | blog  | {page} = my-blog-post |
+--------------------+-------+-----------------------+

The answer to the problem is to add route *requirements*. The routes in this
example would work perfectly if the ``/blog/{page}`` pattern *only* matched
URLs where the ``{page}`` portion is an integer. Fortunately, regular expression
requirements can easily be added for each parameter. For example:
要解决这个问题，只要添加路径*requirements*就行了。如果在这个例子中，``/blog/{page}`` pattern只匹配
{page}是整数的URL，就能够正常运行了。作为正则表达式的requirements很容易被添加：

.. configuration-block::

    .. code-block:: yaml

        blog:
            pattern:   /blog/{page}
            defaults:  { _controller: AcmeBlogBundle:Blog:index, page: 1 }
            requirements:
                page:  \d+

    .. code-block:: xml

        <?xml version="1.0" encoding="UTF-8" ?>

        <routes xmlns="http://symfony.com/schema/routing"
            xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
            xsi:schemaLocation="http://symfony.com/schema/routing http://symfony.com/schema/routing/routing-1.0.xsd">

            <route id="blog" pattern="/blog/{page}">
                <default key="_controller">AcmeBlogBundle:Blog:index</default>
                <default key="page">1</default>
                <requirement key="page">\d+</requirement>
            </route>
        </routes>

    .. code-block:: php

        use Symfony\Component\Routing\RouteCollection;
        use Symfony\Component\Routing\Route;

        $collection = new RouteCollection();
        $collection->add('blog', new Route('/blog/{page}', array(
            '_controller' => 'AcmeBlogBundle:Blog:index',
            'page' => 1,
        ), array(
            'page' => '\d+',
        )));

        return $collection;

The ``\d+`` requirement is a regular expression that says that the value of
the ``{page}`` parameter must be a digit (i.e. a number). The ``blog`` route
will still match on a URL like ``/blog/2`` (because 2 is a number), but it
will no longer match a URL like ``/blog/my-blog-post`` (because ``my-blog-post``
is *not* a number).
这个``\d+`` requirement是一个正则表达式，它要求{page}参数必须是一个整数。这个blog路径仍然会
匹配像/blog/2这样的路径，但它不会匹配``/blog/my-blog-post``这样的路径（因为my-blog-post不是数字）。

As a result, a URL like ``/blog/my-blog-post`` will now properly match the
``blog_show`` route.
于是，像``/blog/my-blog-post``这样的url是不会匹配blog_show路径的。

+--------------------+-----------+-----------------------+
| URL                | route     | parameters            |
+====================+===========+=======================+
| /blog/2            | blog      | {page} = 2            |
+--------------------+-----------+-----------------------+
| /blog/my-blog-post | blog_show | {slug} = my-blog-post |
+--------------------+-----------+-----------------------+

.. sidebar:: Earlier Routes always Win

    What this all means is that the order of the routes is very important.
    If the ``blog_show`` route were placed above the ``blog`` route, the
    URL ``/blog/2`` would match ``blog_show`` instead of ``blog`` since the
    ``{slug}`` parameter of ``blog_show`` has no requirements. By using proper
    ordering and clever requirements, you can accomplish just about anything.
    以上表明，路径的次序是非常重要的。如果blog_show路径被放置在blog路径之上，那么/blog/2
    就会匹配blog_show而不是blog，因为blog_show的{slug}参数没有requirements。通过良好的排序和
    requirements设置，你可以将路径设置成任何你想要的形式。

Since the parameter requirements are regular expressions, the complexity
and flexibility of each requirement is entirely up to you. Suppose the homepage
of your application is available in two different languages, based on the
URL:
由于参数的requirements是正则表达式，这个requirements的复杂度和灵活度都取决于你。假设你的应用的主页在
两种语言中都可用，在url的基础上：

.. configuration-block::

    .. code-block:: yaml

        homepage:
            pattern:   /{culture}
            defaults:  { _controller: AcmeDemoBundle:Main:homepage, culture: en }
            requirements:
                culture:  en|fr

    .. code-block:: xml

        <?xml version="1.0" encoding="UTF-8" ?>

        <routes xmlns="http://symfony.com/schema/routing"
            xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
            xsi:schemaLocation="http://symfony.com/schema/routing http://symfony.com/schema/routing/routing-1.0.xsd">

            <route id="homepage" pattern="/{culture}">
                <default key="_controller">AcmeDemoBundle:Main:homepage</default>
                <default key="culture">en</default>
                <requirement key="culture">en|fr</requirement>
            </route>
        </routes>

    .. code-block:: php

        use Symfony\Component\Routing\RouteCollection;
        use Symfony\Component\Routing\Route;

        $collection = new RouteCollection();
        $collection->add('homepage', new Route('/{culture}', array(
            '_controller' => 'AcmeDemoBundle:Main:homepage',
            'culture' => 'en',
        ), array(
            'culture' => 'en|fr',
        )));

        return $collection;

For incoming requests, the ``{culture}`` portion of the URL is matched against
the regular expression ``(en|fr)``.
对于发生的请求，url的{culture}选项是与正则表达式(en|fr)匹配的。

+-----+--------------------------+
| /   | {culture} = en           |
+-----+--------------------------+
| /en | {culture} = en           |
+-----+--------------------------+
| /fr | {culture} = fr           |
+-----+--------------------------+
| /es | *won't match this route* |
+-----+--------------------------+

.. index::
   single: Routing; Method requirement

Adding HTTP Method Requirements
添加HTTP方法Requirements
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

In addition to the URL, you can also match on the *method* of the incoming
request (i.e. GET, HEAD, POST, PUT, DELETE). Suppose you have a contact form
with two controllers - one for displaying the form (on a GET request) and one
for processing the form when it's submitted (on a POST request). This can
be accomplished with the following route configuration:
除了直接添加入url的变量，你还可以添加请求的方法（如GET, HEAD, POST, PUT, DELETE）。
假设你有关于两个控制器的表单——一个是用来显示这个表单的（根据get请求），一个是用来提交
表单的（根据post请求）。你可以：

.. configuration-block::

    .. code-block:: yaml

        contact:
            pattern:  /contact
            defaults: { _controller: AcmeDemoBundle:Main:contact }
            requirements:
                _method:  GET

        contact_process:
            pattern:  /contact
            defaults: { _controller: AcmeDemoBundle:Main:contactProcess }
            requirements:
                _method:  POST

    .. code-block:: xml

        <?xml version="1.0" encoding="UTF-8" ?>

        <routes xmlns="http://symfony.com/schema/routing"
            xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
            xsi:schemaLocation="http://symfony.com/schema/routing http://symfony.com/schema/routing/routing-1.0.xsd">

            <route id="contact" pattern="/contact">
                <default key="_controller">AcmeDemoBundle:Main:contact</default>
                <requirement key="_method">GET</requirement>
            </route>

            <route id="contact_process" pattern="/contact">
                <default key="_controller">AcmeDemoBundle:Main:contactProcess</default>
                <requirement key="_method">POST</requirement>
            </route>
        </routes>

    .. code-block:: php

        use Symfony\Component\Routing\RouteCollection;
        use Symfony\Component\Routing\Route;

        $collection = new RouteCollection();
        $collection->add('contact', new Route('/contact', array(
            '_controller' => 'AcmeDemoBundle:Main:contact',
        ), array(
            '_method' => 'GET',
        )));

        $collection->add('contact_process', new Route('/contact', array(
            '_controller' => 'AcmeDemoBundle:Main:contactProcess',
        ), array(
            '_method' => 'POST',
        )));

        return $collection;

Despite the fact that these two routes have identical patterns (``/contact``),
the first route will match only GET requests and the second route will match
only POST requests. This means that you can display the form and submit the
form via the same URL, while using distinct controllers for the two actions.
虽然两个路径有着相同的pattern（/contact），但是第一个路径只会匹配get请求，而第二个路径只会
匹配post请求。这表示你可以用同一个url来显示和提交表单，但用的是两个控制器。

.. note::
    If no ``_method`` requirement is specified, the route will match on
    *all* methods.
    如果_method requirements没有指定，那这个路径会匹配所有的方法。

Like the other requirements, the ``_method`` requirement is parsed as a regular
expression. To match ``GET`` *or* ``POST`` requests, you can use ``GET|POST``.
像其他requirements一样，_method被解析为一个正则表达式。为了匹配get或post请求，你可以使用GET|POST

.. index::
   single: Routing; Advanced example
   single: Routing; _format parameter

.. _advanced-routing-example:

Advanced Routing Example
高级路径范例
~~~~~~~~~~~~~~~~~~~~~~~~

At this point, you have everything you need to create a powerful routing
structure in Symfony. The following is an example of just how flexible the
routing system can be:
现在，你已经可以创建一个强大的路径结构了。以下是一个例子，它表明了这个路径系统的灵活性：

.. configuration-block::

    .. code-block:: yaml

        article_show:
          pattern:  /articles/{culture}/{year}/{title}.{_format}
          defaults: { _controller: AcmeDemoBundle:Article:show, _format: html }
          requirements:
              culture:  en|fr
              _format:  html|rss
              year:     \d+

    .. code-block:: xml

        <?xml version="1.0" encoding="UTF-8" ?>

        <routes xmlns="http://symfony.com/schema/routing"
            xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
            xsi:schemaLocation="http://symfony.com/schema/routing http://symfony.com/schema/routing/routing-1.0.xsd">

            <route id="article_show" pattern="/articles/{culture}/{year}/{title}.{_format}">
                <default key="_controller">AcmeDemoBundle:Article:show</default>
                <default key="_format">html</default>
                <requirement key="culture">en|fr</requirement>
                <requirement key="_format">html|rss</requirement>
                <requirement key="year">\d+</requirement>
            </route>
        </routes>

    .. code-block:: php

        use Symfony\Component\Routing\RouteCollection;
        use Symfony\Component\Routing\Route;

        $collection = new RouteCollection();
        $collection->add('homepage', new Route('/articles/{culture}/{year}/{title}.{_format}', array(
            '_controller' => 'AcmeDemoBundle:Article:show',
            '_format' => 'html',
        ), array(
            'culture' => 'en|fr',
            '_format' => 'html|rss',
            'year' => '\d+',
        )));

        return $collection;

As you've seen, this route will only match if the ``{culture}`` portion of
the URL is either ``en`` or ``fr`` and if the ``{year}`` is a number. This
route also shows how you can use a period between placeholders instead of
a slash. URLs matching this route might look like:
如你所见，这个路径只在当url中的{culture}是en或fr，并且{year}是数字的时候能匹配。这个路径
还展示了你如何使用句号而不是斜杠。匹配这个路径的url可以是这样：

* ``/articles/en/2010/my-post``
* ``/articles/fr/2010/my-post.rss``

.. _book-routing-format-param:

.. sidebar:: The Special ``_format`` Routing Parameter

    This example also highlights the special ``_format`` routing parameter.
    When using this parameter, the matched value becomes the "request format"
    of the ``Request`` object. Ultimately, the request format is used for such
    things such as setting the ``Content-Type`` of the response (e.g. a ``json``
    request format translates into a ``Content-Type`` of ``application/json``).
    It can also be used in the controller to render a different template for
    each value of ``_format``. The ``_format`` parameter is a very powerful way
    to render the same content in different formats.
    这个例子突出显示了这个特殊的_format路径参数。当使用这个参数的时候，被匹配的值成为了request对象的”请求格式（request format）“。
    最终，请求格式被用来创建如响应的”content-type“这样的数据（或者说，一个json请求格式被解析为application/json的content-type）

Special Routing Parameters
特别的路径参数
~~~~~~~~~~~~~~~~~~~~~~~~~~

As you've seen, each routing parameter or default value is eventually available
as an argument in the controller method. Additionally, there are three parameters
that are special: each adds a unique piece of functionality inside your application:
如你所见，每个路径的参数或者默认值都最终被控制器所用。事实上，有三个特别参数：每个参数都
在你的应用中添加了一个唯一功能：

* ``_controller``: As you've seen, this parameter is used to determine which
  controller is executed when the route is matched;

* ``_format``: Used to set the request format (:ref:`read more<book-routing-format-param>`);

* ``_locale``: Used to set the locale on the request (:ref:`read more<book-translation-locale-url>`);

.. tip::

    If you use the ``_locale`` parameter in a route, that value will also
    be stored on the session so that subsequent requests keep this same locale.
    如果你在路径中使用``_locale``参数，它的值也会被存储在session上，这样接下来的请求也会保持相同的locale。

.. index::
   single: Routing; Controllers
   single: Controller; String naming format

.. _controller-string-syntax:

Controller Naming Pattern
控制器命名模式
-------------------------

Every route must have a ``_controller`` parameter, which dictates which
controller should be executed when that route is matched. This parameter
uses a simple string pattern called the *logical controller name*, which
Symfony maps to a specific PHP method and class. The pattern has three parts,
each separated by a colon:
每个路径都必须有个_controller参数，它表示当路径匹配时哪个控制器要用到。这个参数使用一个被称作控制器逻辑名的
简单的pattern，symfony用这个pattern映射到一个特定的PHP方法和类，这个pattern有三个部分，每个部分以冒号隔开：

    **bundle**:**controller**:**action**

For example, a ``_controller`` value of ``AcmeBlogBundle:Blog:show`` means:
比如，一个值为``AcmeBlogBundle:Blog:show``的_controller表示：

+----------------+------------------+-------------+
| Bundle         | Controller Class | Method Name |
+================+==================+=============+
| AcmeBlogBundle | BlogController   | showAction  |
+----------------+------------------+-------------+

The controller might look like this:
而这个控制器是这样的：

.. code-block:: php

    // src/Acme/BlogBundle/Controller/BlogController.php

    namespace Acme\BlogBundle\Controller;
    use Symfony\Bundle\FrameworkBundle\Controller\Controller;

    class BlogController extends Controller
    {
        public function showAction($slug)
        {
            // ...
        }
    }

Notice that Symfony adds the string ``Controller`` to the class name (``Blog``
=> ``BlogController``) and ``Action`` to the method name (``show`` => ``showAction``).
注意symfony将``Controller``加到类名后(``Blog``=> ``BlogController``)，将``Action``加到
方法名后(``show`` => ``showAction``)。

You could also refer to this controller using its fully-qualified class name
and method: ``Acme\BlogBundle\Controller\BlogController::showAction``.
But if you follow some simple conventions, the logical name is more concise
and allows more flexibility.
你也可以通过它的全名来访问这个控制器：``Acme\BlogBundle\Controller\BlogController::showAction``。
但是如果你遵循一些简单的惯例，这个逻辑名显得更精确和灵活。

.. note::

   In addition to using the logical name or the fully-qualified class name,
   Symfony supports a third way of referring to a controller. This method
   uses just one colon separator (e.g. ``service_name:indexAction``) and
   refers to the controller as a service (see :doc:`/cookbook/controller/service`).
   除了使用逻辑名和全名，symfony还支持第三种方式来访问控制器。这种方式只使用一个冒号分隔(e.g. ``service_name:indexAction``)
   并像访问服务一样访问控制器。参阅：:doc:`/cookbook/controller/service`。

Route Parameters and Controller Arguments
路径参数和控制器参数
-----------------------------------------

The route parameters (e.g. ``{slug}``) are especially important because
each is made available as an argument to the controller method:
路径中的参数（如”{slug}“）很重要，因为它可以作为控制器方法的参数：

.. code-block:: php

    public function showAction($slug)
    {
      // ...
    }

In reality, the entire ``defaults`` collection is merged with the parameter
values to form a single array. Each key of that array is available as an
argument on the controller.
事实上，整个defaults的值的集合组成了一个数组，数组的key则作为控制器的参数。

In other words, for each argument of your controller method, Symfony looks
for a route parameter of that name and assigns its value to that argument.
In the advanced example above, any combination (in any order) of the following
variables could be used as arguments to the ``showAction()`` method:
换句话说，对于你的控制器的每个参数，symfony查找相对应的那个路径的参数名，并将它的值传递到控制器中。
在以上的高级范例中，下面变量的任意组合都可以被用来作为showAction（）方法的参数：

* ``$culture``
* ``$year``
* ``$title``
* ``$_format``
* ``$_controller``

Since the placeholders and ``defaults`` collection are merged together, even
the ``$_controller`` variable is available. For a more detailed discussion,
see :ref:`route-parameters-controller-arguments`.
因为占位符和defaults的值集合都被合并到一起，这个``$_controller``变量甚至都是存在的。
详情请见：:ref:`route-parameters-controller-arguments`。

.. tip::

    You can also use a special ``$_route`` variable, which is set to the
    name of the route that was matched.
    你也可以使用变量``$_route``，它表示匹配的路径的名称。

.. index::
   single: Routing; Importing routing resources

.. _routing-include-external-resources:

Including External Routing Resources
包含外部路径源
------------------------------------

All routes are loaded via a single configuration file - usually ``app/config/routing.yml``
(see `Creating Routes`_ above). Commonly, however, you'll want to load routes
from other places, like a routing file that lives inside a bundle. This can
be done by "importing" that file:
所有的路径都是通过一个单独的配置文件载入的——通常是``app/config/routing.yml``（参阅前面的`Creating Routes`_）。
但往往你会希望通过其他地方载入这些路径，比如在bundle里面的文件。你可以”导入”那个文件：

.. configuration-block::

    .. code-block:: yaml

        # app/config/routing.yml
        acme_hello:
            resource: "@AcmeHelloBundle/Resources/config/routing.yml"

    .. code-block:: xml

        <!-- app/config/routing.xml -->
        <?xml version="1.0" encoding="UTF-8" ?>

        <routes xmlns="http://symfony.com/schema/routing"
            xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
            xsi:schemaLocation="http://symfony.com/schema/routing http://symfony.com/schema/routing/routing-1.0.xsd">

            <import resource="@AcmeHelloBundle/Resources/config/routing.xml" />
        </routes>

    .. code-block:: php

        // app/config/routing.php
        use Symfony\Component\Routing\RouteCollection;

        $collection = new RouteCollection();
        $collection->addCollection($loader->import("@AcmeHelloBundle/Resources/config/routing.php"));

        return $collection;

.. note::

   When importing resources from YAML, the key (e.g. ``acme_hello``) is meaningless.
   Just be sure that it's unique so no other lines override it.
   当使用YAML来导入文件时，key(e.g. ``acme_hello``)是不重要的。只要保证它是唯一的就可以了，否则就会被覆盖。

The ``resource`` key loads the given routing resource. In this example the
resource is the full path to a file, where the ``@AcmeHelloBundle`` shortcut
syntax resolves to the path of that bundle. The imported file might look
like this:
key “resource”载入了给定的路径源。在这个例子中，这个源是文件的全貌，其中``@AcmeHelloBundle``
简易语法表示那个bundle的路径。这个被导入的文件可能像这样：

.. configuration-block::

    .. code-block:: yaml

        # src/Acme/HelloBundle/Resources/config/routing.yml
       acme_hello:
            pattern:  /hello/{name}
            defaults: { _controller: AcmeHelloBundle:Hello:index }

    .. code-block:: xml

        <!-- src/Acme/HelloBundle/Resources/config/routing.xml -->
        <?xml version="1.0" encoding="UTF-8" ?>

        <routes xmlns="http://symfony.com/schema/routing"
            xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
            xsi:schemaLocation="http://symfony.com/schema/routing http://symfony.com/schema/routing/routing-1.0.xsd">

            <route id="acme_hello" pattern="/hello/{name}">
                <default key="_controller">AcmeHelloBundle:Hello:index</default>
            </route>
        </routes>

    .. code-block:: php

        // src/Acme/HelloBundle/Resources/config/routing.php
        use Symfony\Component\Routing\RouteCollection;
        use Symfony\Component\Routing\Route;

        $collection = new RouteCollection();
        $collection->add('acme_hello', new Route('/hello/{name}', array(
            '_controller' => 'AcmeHelloBundle:Hello:index',
        )));

        return $collection;

The routes from this file are parsed and loaded in the same way as the main
routing file.
这个文件中的路径就像那个主要路径文件中的数据一样被解析。

Prefixing Imported Routes
导入路径的前缀
~~~~~~~~~~~~~~~~~~~~~~~~~

You can also choose to provide a "prefix" for the imported routes. For example,
suppose you want the ``acme_hello`` route to have a final pattern of ``/admin/hello/{name}``
instead of simply ``/hello/{name}``:
你也可以选择在导入的路径中加上前缀。比如，假设你想要``acme_hello``路径有一个最终pattern ``/admin/hello/{name}``，而
不是简单的``/hello/{name}``：

.. configuration-block::

    .. code-block:: yaml

        # app/config/routing.yml
        acme_hello:
            resource: "@AcmeHelloBundle/Resources/config/routing.yml"
            prefix:   /admin

    .. code-block:: xml

        <!-- app/config/routing.xml -->
        <?xml version="1.0" encoding="UTF-8" ?>

        <routes xmlns="http://symfony.com/schema/routing"
            xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
            xsi:schemaLocation="http://symfony.com/schema/routing http://symfony.com/schema/routing/routing-1.0.xsd">

            <import resource="@AcmeHelloBundle/Resources/config/routing.xml" prefix="/admin" />
        </routes>

    .. code-block:: php

        // app/config/routing.php
        use Symfony\Component\Routing\RouteCollection;

        $collection = new RouteCollection();
        $collection->addCollection($loader->import("@AcmeHelloBundle/Resources/config/routing.php"), '/admin');

        return $collection;

The string ``/admin`` will now be prepended to the pattern of each route
loaded from the new routing resource.
``/admin``将会被作为每个被从这个新的源载入的路径的前缀。

.. index::
   single: Routing; Debugging

Visualizing & Debugging Routes
调试路径
------------------------------

While adding and customizing routes, it's helpful to be able to visualize
and get detailed information about your routes. A great way to see every route
in your application is via the ``router:debug`` console command. Execute
the command by running the following from the root of your project.
在添加和定义路径时，你可能需要获取路径的具体信息。通过``router:debug``控制台命令你就可以看到
你应用中的每个路径。从你的project的根目录运行以下命令：

.. code-block:: bash

    php app/console router:debug

The command will print a helpful list of *all* the configured routes in
your application:
这个命令会输出所有的配置好的路径：

.. code-block:: text

    homepage              ANY       /
    contact               GET       /contact
    contact_process       POST      /contact
    article_show          ANY       /articles/{culture}/{year}/{title}.{_format}
    blog                  ANY       /blog/{page}
    blog_show             ANY       /blog/{slug}

You can also get very specific information on a single route by including
the route name after the command:
你也可以获取某个独立路径的特别信息，只要将这个路径名称加在命令行后面：

.. code-block:: bash

    php app/console router:debug article_show

.. index::
   single: Routing; Generating URLs

Generating URLs
集成URL
---------------

The routing system should also be used to generate URLs. In reality, routing
is a bi-directional system: mapping the URL to a controller+parameters and
a route+parameters back to a URL. The
:method:`Symfony\\Component\\Routing\\Router::match` and
:method:`Symfony\\Component\\Routing\\Router::generate` methods form this bi-directional
system. Take the ``blog_show`` example route from earlier::
路由系统也应该被用来集成url。事实上，路由是一个双向的系统：它同时将url映射到控制器+参数和路径+参数。
:method:`Symfony\\Component\\Routing\\Router::match`和
:method:`Symfony\\Component\\Routing\\Router::generate` 方法组成了这个双向系统。拿blog_show作为例子::

    $params = $router->match('/blog/my-blog-post');
    // array('slug' => 'my-blog-post', '_controller' => 'AcmeBlogBundle:Blog:show')

    $uri = $router->generate('blog_show', array('slug' => 'my-blog-post'));
    // /blog/my-blog-post

To generate a URL, you need to specify the name of the route (e.g. ``blog_show``)
and any wildcards (e.g. ``slug = my-blog-post``) used in the pattern for
that route. With this information, any URL can easily be generated:
要集成一个url，你必须定义路径的名字（比如``blog_show``）以及pattern中的通配符(``slug = my-blog-post``)。
有了这些信息url就能被集成了:

.. code-block:: php

    class MainController extends Controller
    {
        public function showAction($slug)
        {
          // ...

          $url = $this->get('router')->generate('blog_show', array('slug' => 'my-blog-post'));
        }
    }

In an upcoming section, you'll learn how to generate URLs from inside templates.
下一节中，你将学习如何从模板内部集成url。

.. tip::

    If the frontend of your application uses AJAX requests, you might want
    to be able to generate URLs in JavaScript based on your routing configuration.
    By using the `FOSJsRoutingBundle`_, you can do exactly that:
    如果你的应用前端使用AJAX请求，你可能需要在你的路径基础上的Javascript中集成url。只要使用`FOSJsRoutingBundle`_
    就可以了：

    .. code-block:: javascript

        var url = Routing.generate('blog_show', { "slug": 'my-blog-post});

    For more information, see the documentation for that bundle.
    更多信息请参阅那个bundle中的文档。

.. index::
   single: Routing; Absolute URLs

Generating Absolute URLs
集成绝对url
~~~~~~~~~~~~~~~~~~~~~~~~

By default, the router will generate relative URLs (e.g. ``/blog``). To generate
an absolute URL, simply pass ``true`` to the third argument of the ``generate()``
method:
默认情况下，路由会集成相对url（/blog）。要集成绝对url，只要将“true”作为第三个参数传递到``generate()``中就行了：

.. code-block:: php

    $router->generate('blog_show', array('slug' => 'my-blog-post'), true);
    // http://www.example.com/blog/my-blog-post

.. note::

    The host that's used when generating an absolute URL is the host of
    the current ``Request`` object. This is detected automatically based
    on server information supplied by PHP. When generating absolute URLs for
    scripts run from the command line, you'll need to manually set the desired
    host on the ``Request`` object:
    集成绝对url的主机是当前request对象的主机。这个信息是通过能检测服务器信息的PHP方法检测的。
    当集成从命令行运行的脚本时，你必须手动在request对象上设置主机：

    .. code-block:: php

        $request->headers->set('HOST', 'www.example.com');

.. index::
   single: Routing; Generating URLs in a template

Generating URLs with Query Strings
通过请求语句集成url
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

The ``generate`` method takes an array of wildcard values to generate the URI.
But if you pass extra ones, they will be added to the URI as a query string::
generate方法提取一个通配符数组来集成uri。但是如果你想传递额外的数据，它们会被作为一个请求语句添加到uri后面::

    $router->generate('blog', array('page' => 2, 'category' => 'Symfony'));
    // /blog/2?category=Symfony

Generating URLs from a template
从模板中集成url
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

The most common place to generate a URL is from within a template when linking
between pages in your application. This is done just as before, but using
a template helper function:
最常用的集成url的地方是在模板中，当你要在页面之间创建连接的时候。这还是像刚才做的一样，但是要
使用模板的helper方法：

.. configuration-block::

    .. code-block:: html+jinja

        <a href="{{ path('blog_show', { 'slug': 'my-blog-post' }) }}">
          Read this blog post.
        </a>

    .. code-block:: php

        <a href="<?php echo $view['router']->generate('blog_show', array('slug' => 'my-blog-post')) ?>">
            Read this blog post.
        </a>

Absolute URLs can also be generated.
绝对url也可以被集成。

.. configuration-block::

    .. code-block:: html+jinja

        <a href="{{ url('blog_show', { 'slug': 'my-blog-post' }) }}">
          Read this blog post.
        </a>

    .. code-block:: php

        <a href="<?php echo $view['router']->generate('blog_show', array('slug' => 'my-blog-post'), true) ?>">
            Read this blog post.
        </a>

Summary
总结
-------

Routing is a system for mapping the URL of incoming requests to the controller
function that should be called to process the request. It both allows you
to specify beautiful URLs and keeps the functionality of your application
decoupled from those URLs. Routing is a two-way mechanism, meaning that it
should also be used to generate URLs.
路由是一个将请求的url映射到要执行的控制器的系统。它不仅允许你创建良好的url，还可以保证你的应用的功能
与url隔离。路由是一个双向机制，这表示它还应该被用来集成url。

Learn more from the Cookbook
从cookbook学习更多
----------------------------

* :doc:`/cookbook/routing/scheme`

.. _`FOSJsRoutingBundle`: https://github.com/FriendsOfSymfony/FOSJsRoutingBundle
