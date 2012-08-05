Symfony2 versus Flat PHP
symfony2与不用任何框架的php相比较
========================

**为什么symfony2比不用任何框架的php代码好？**

If you've never used a PHP framework, aren't familiar with the MVC philosophy,
or just wonder what all the *hype* is around Symfony2, this chapter is for
you. Instead of *telling* you that Symfony2 allows you to develop faster and
better software than with flat PHP, you'll see for yourself.
如果你从未使用过php框架，或不熟悉MVC模式，或仅仅对symfony感到好奇，请阅读本章节。

In this chapter, you'll write a simple application in flat PHP, and then
refactor it to be more organized. You'll travel through time, seeing the
decisions behind why web development has evolved over the past several years
to where it is now.
在本章中，你将使用手写php来创建一个简单的应用，然后重新组织它。你将了解为什么
web开发的理念会演变成它今天的样子。

By the end, you'll see how Symfony2 can rescue you from mundane tasks and
let you take back control of your code.
最后，你将会看到symfony如何将你从繁重的代码中解脱，并使你能够掌控你的代码。

A simple Blog in flat PHP
一个用手写php编写的简单博客
-------------------------

In this chapter, you'll build the token blog application using only flat PHP.
To begin, create a single page that displays blog entries that have been
persisted to the database. Writing in flat PHP is quick and dirty:
在本章中，你将使用手写php来编写一个博客应用。首先，创建一个显示已经被载入数据库的博客文章列表页面：

.. code-block:: html+php

    <?php
    // index.php

    $link = mysql_connect('localhost', 'myuser', 'mypassword');
    mysql_select_db('blog_db', $link);

    $result = mysql_query('SELECT id, title FROM post', $link);
    ?>

    <html>
        <head>
            <title>List of Posts</title>
        </head>
        <body>
            <h1>List of Posts</h1>
            <ul>
                <?php while ($row = mysql_fetch_assoc($result)): ?>
                <li>
                    <a href="/show.php?id=<?php echo $row['id'] ?>">
                        <?php echo $row['title'] ?>
                    </a>
                </li>
                <?php endwhile; ?>
            </ul>
        </body>
    </html>

    <?php
    mysql_close($link);

That's quick to write, fast to execute, and, as your app grows, impossible
to maintain. There are several problems that need to be addressed:
这样写会很快，执行也会很快，但是，当代码更多时，不可能维护。这里有三个问题：

* **没有错误检查**: 如果到数据库的连接失败怎么办？

* **差劲的组织**: 如果应用扩展，这个文件将越来越难以维护。你在哪里放代码来处理表单？你如何
  验证数据？应该在哪里发邮件？

* **代码难以重复使用**: 对于博客网站的另一个页面，这个代码不能被重复使用。

.. note::
    Another problem not mentioned here is the fact that the database is
    tied to MySQL. Though not covered here, Symfony2 fully integrates `Doctrine`_,
    a library dedicated to database abstraction and mapping.
    这里没有提到的一个问题就是数据库是与MYSQL绑定的。symfony2整个都与`Doctrine`_合并了，
    doctrine是一个致力于处理数据库（抽象层和映射）的库。

Let's get to work on solving these problems and more.
下面看我们将如何解决这些问题。

Isolating the Presentation
隔离用于显示页面的代码
~~~~~~~~~~~~~~~~~~~~~~~~~~

The code can immediately gain from separating the application "logic" from
the code that prepares the HTML "presentation":
将应用逻辑从HTML分离出来就会显得清晰许多：

.. code-block:: html+php

    <?php
    // index.php

    $link = mysql_connect('localhost', 'myuser', 'mypassword');
    mysql_select_db('blog_db', $link);

    $result = mysql_query('SELECT id, title FROM post', $link);

    $posts = array();
    while ($row = mysql_fetch_assoc($result)) {
        $posts[] = $row;
    }

    mysql_close($link);

    // include the HTML presentation code
    require 'templates/list.php';

The HTML code is now stored in a separate file (``templates/list.php``), which
is primarily an HTML file that uses a template-like PHP syntax:

.. code-block:: html+php

    <html>
        <head>
            <title>List of Posts</title>
        </head>
        <body>
            <h1>List of Posts</h1>
            <ul>
                <?php foreach ($posts as $post): ?>
                <li>
                    <a href="/read?id=<?php echo $post['id'] ?>">
                        <?php echo $post['title'] ?>
                    </a>
                </li>
                <?php endforeach; ?>
            </ul>
        </body>
    </html>

By convention, the file that contains all of the application logic - ``index.php`` -
is known as a "controller". The term :term:`controller` is a word you'll hear
a lot, regardless of the language or framework you use. It refers simply
to the area of *your* code that processes user input and prepares the response.
一般的，包含所有应用逻辑（``index.php``）的文件被称作控制器（controller）。不你使用何种语言或者
何种框架，你都会听到很多遍“控制器”这个词语。它表示你代码中用于处理用户输入信息和准备响应的代码。

In this case, our controller prepares data from the database and then includes
a template to present that data. With the controller isolated, you could
easily change *just* the template file if you needed to render the blog
entries in some other format (e.g. ``list.json.php`` for JSON format).
在这个例子中，我们的控制器从数据库取出数据，并包含了一个模板来显示那些数据。通过将
控制器隔离，如果你需要用其他格式来显示博客文章的话（比如``list.json.php``），你可以
改变模板文件。

Isolating the Application (Domain) Logic
隔离应用逻辑
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

So far the application contains only one page. But what if a second page
needed to use the same database connection, or even the same array of blog
posts? Refactor the code so that the core behavior and data-access functions
of the application are isolated in a new file called ``model.php``:
现在这个应用包含了仅仅一个页面，那么如果第二个页面需要使用同样的数据库连接，或
同样的存储博客文章的数组呢？重构这个代码，使该应用的获取数据的方法被放置在一个新的名叫
``model.php``的文件中：

.. code-block:: html+php

    <?php
    // model.php

    function open_database_connection()
    {
        $link = mysql_connect('localhost', 'myuser', 'mypassword');
        mysql_select_db('blog_db', $link);

        return $link;
    }

    function close_database_connection($link)
    {
        mysql_close($link);
    }

    function get_all_posts()
    {
        $link = open_database_connection();

        $result = mysql_query('SELECT id, title FROM post', $link);
        $posts = array();
        while ($row = mysql_fetch_assoc($result)) {
            $posts[] = $row;
        }
        close_database_connection($link);

        return $posts;
    }

.. tip::

   The filename ``model.php`` is used because the logic and data access of
   an application is traditionally known as the "model" layer. In a well-organized
   application, the majority of the code representing your "business logic"
   should live in the model (as opposed to living in a controller). And unlike
   in this example, only a portion (or none) of the model is actually concerned
   with accessing a database.这里使用了model.php这个文件名称，因为一个应用的逻辑和数据
   处理层一般被命名为model层。在一个组织良好的应用中，处理逻辑的代码都应该被放置在model层中
   （而不是在控制器中）。

The controller (``index.php``) is now very simple:
控制器(``index.php``)的代码现在就变得非常简单了：

.. code-block:: html+php

    <?php
    require_once 'model.php';

    $posts = get_all_posts();

    require 'templates/list.php';

Now, the sole task of the controller is to get data from the model layer of
the application (the model) and to call a template to render that data.
This is a very simple example of the model-view-controller pattern.
现在，控制器的唯一任务就是从model层中取出数据并命令一个模板来提交这些数据。
这就是一个非常简单的model-view-controller模式（MVC）。

Isolating the Layout
分离显示页面的文件
~~~~~~~~~~~~~~~~~~~~

At this point, the application has been refactored into three distinct pieces
offering various advantages and the opportunity to reuse almost everything
on different pages.
现在，应用已经被重构成为三个不同的部分，你可以在不同的页面利用每个部分的代码。

The only part of the code that *can't* be reused is the page layout. Fix
that by creating a new ``layout.php`` file:
代码中唯一还不能被重复利用的就是显示页面的HTML文件。创建一个新的layout.php文件：

.. code-block:: html+php

    <!-- templates/layout.php -->
    <html>
        <head>
            <title><?php echo $title ?></title>
        </head>
        <body>
            <?php echo $content ?>
        </body>
    </html>

The template (``templates/list.php``) can now be simplified to "extend"
the layout:
模板(``templates/list.php``)现在可以extend这个文件了：

.. code-block:: html+php

    <?php $title = 'List of Posts' ?>

    <?php ob_start() ?>
        <h1>List of Posts</h1>
        <ul>
            <?php foreach ($posts as $post): ?>
            <li>
                <a href="/read?id=<?php echo $post['id'] ?>">
                    <?php echo $post['title'] ?>
                </a>
            </li>
            <?php endforeach; ?>
        </ul>
    <?php $content = ob_get_clean() ?>

    <?php include 'layout.php' ?>

You've now introduced a methodology that allows for the reuse of the
layout. Unfortunately, to accomplish this, you're forced to use a few ugly
PHP functions (``ob_start()``, ``ob_get_clean()``) in the template. Symfony2
uses a ``Templating`` component that allows this to be accomplished cleanly
and easily. You'll see it in action shortly.
你现在已经知道一个允许模板重用的方法。但是，要达到这个目的，你还必须在模板中使用一些php方法（``ob_start()``, ``ob_get_clean()``）。
symfony2提供一个简易的方法来完成这个工作，即使用tamplate component，稍后会详细讲述。

Adding a Blog "show" Page
添加一个博客显示页面
-------------------------

The blog "list" page has now been refactored so that the code is better-organized
and reusable. To prove it, add a blog "show" page, which displays an individual
blog post identified by an ``id`` query parameter.
博客列表页面的代码已经被重构了，这样这个代码就显得更加组织化并可以重用。比如，添加一个
显示页面来通过添加请求参数id显示一个单独的页面。

To begin, create a new function in the ``model.php`` file that retrieves
an individual blog result based on a given id::
首先，在model.php文件中创建一个新的函数，这个函数可以根据一个给定的id来获取一个单个的博客文章::

    // model.php
    function get_post_by_id($id)
    {
        $link = open_database_connection();

        $id = mysql_real_escape_string($id);
        $query = 'SELECT date, title, body FROM post WHERE id = '.$id;
        $result = mysql_query($query);
        $row = mysql_fetch_assoc($result);

        close_database_connection($link);

        return $row;
    }

Next, create a new file called ``show.php`` - the controller for this new
page:
然后，创建一个名叫show.php的文件，作为这个新页面的控制器：

.. code-block:: html+php

    <?php
    require_once 'model.php';

    $post = get_post_by_id($_GET['id']);

    require 'templates/show.php';

Finally, create the new template file - ``templates/show.php`` - to render
the individual blog post:
最后，创建一个新的模板文件``templates/show.php``来提交这个单独的博客文章：

.. code-block:: html+php

    <?php $title = $post['title'] ?>

    <?php ob_start() ?>
        <h1><?php echo $post['title'] ?></h1>

        <div class="date"><?php echo $post['date'] ?></div>
        <div class="body">
            <?php echo $post['body'] ?>
        </div>
    <?php $content = ob_get_clean() ?>

    <?php include 'layout.php' ?>

Creating the second page is now very easy and no code is duplicated. Still,
this page introduces even more lingering problems that a framework can solve
for you. For example, a missing or invalid ``id`` query parameter will cause
the page to crash. It would be better if this caused a 404 page to be rendered,
but this can't really be done easily yet. Worse, had you forgotten to clean
the ``id`` parameter via the ``mysql_real_escape_string()`` function, your
entire database would be at risk for an SQL injection attack.
现在创建这个第二个页面变得非常的容易，并且不用重复写代码。但是这个页面仍有问题。比如，
假如给定的id不存在，就会导致这个页面崩溃。如果能让这段代码提交一个404页面会比较好，
但在这里却不会这么容易。更糟的是，如果你忘记了使用``mysql_real_escape_string()``来过滤id参数
的值，你的整个数据库都会有可能受到sql语句注入攻击。

Another major problem is that each individual controller file must include
the ``model.php`` file. What if each controller file suddenly needed to include
an additional file or perform some other global task (e.g. enforce security)?
As it stands now, that code would need to be added to every controller file.
If you forget to include something in one file, hopefully it doesn't relate
to security...
还有一个主要问题就是每个控制器文件都必须包含model.php文件。但如果一个控制器需要包含一个额外的
文件或者做一些全局性的工作呢（比如安全性问题）？这样的话这些代码就必须被包含到所有的控制器文件中，
如果你忘记了包含这些文件，希望它和安全性无关……

A "Front Controller" to the Rescue
一个前端控制器可以解决这个问题
----------------------------------

The solution is to use a :term:`front controller`: a single PHP file through
which *all* requests are processed. With a front controller, the URIs for the
application change slightly, but start to become more flexible:
解决方案是使用一个前端控制器（:term:`front controller`）：它是一个单独的php文件，
通过它所有的请求被执行。在前端控制器中，应用的URI会稍有变化，但是会更加灵活：

.. code-block:: text

    Without a front controller
    /index.php          => Blog post list page (index.php executed)
    /show.php           => Blog post show page (show.php executed)

    With index.php as the front controller
    /index.php          => Blog post list page (index.php executed)
    /index.php/show     => Blog post show page (index.php executed)

.. tip::
    The ``index.php`` portion of the URI can be removed if using Apache
    rewrite rules (or equivalent). In that case, the resulting URI of the
    blog show page would be simply ``/show``.
    如果使用Apache rewrite规则，则URI的index.php部分可以被移除。这样，这个显示博客文章的
    页面的URI会是简单的/show。

When using a front controller, a single PHP file (``index.php`` in this case)
renders *every* request. For the blog post show page, ``/index.php/show`` will
actually execute the ``index.php`` file, which is now responsible for routing
requests internally based on the full URI. As you'll see, a front controller
is a very powerful tool.
当使用前端控制器的时候，一个单独的php文件（在这里是index.php）会提交所有的请求。对于这里的
博客文章显示页面，/index.php/show 会执行index.php文件，这是内置的根据完整URI的路径请求决定的。
现在你可以看见，前端控制器是一个非常有用的工具。

Creating the Front Controller
创建前端控制器
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

You're about to take a **big** step with the application. With one file handling
all requests, you can centralize things such as security handling, configuration
loading, and routing. In this application, ``index.php`` must now be smart
enough to render the blog post list page *or* the blog post show page based
on the requested URI:
现在你会向前迈出一大步了。通过使用一个文件来处理所有的请求，你可以集中处理所有的工作，如处理
安全问题，配置载入文件，还有路径。在这个应用中，index.php必须根据请求的URI来决定是提交博客列表页面
还是博客文章显示页面。

.. code-block:: html+php

    <?php
    // index.php

    // load and initialize any global libraries
    require_once 'model.php';
    require_once 'controllers.php';

    // route the request internally
    $uri = $_SERVER['REQUEST_URI'];
    if ($uri == '/index.php') {
        list_action();
    } elseif ($uri == '/index.php/show' && isset($_GET['id'])) {
        show_action($_GET['id']);
    } else {
        header('Status: 404 Not Found');
        echo '<html><body><h1>Page Not Found</h1></body></html>';
    }

For organization, both controllers (formerly ``index.php`` and ``show.php``)
are now PHP functions and each has been moved into a separate file, ``controllers.php``:
为了良好的组织起见，两个控制器（``index.php``和``show.php``）现在都是php函数，并被移至一个
单独的文件，controller.php。

.. code-block:: php

    function list_action()
    {
        $posts = get_all_posts();
        require 'templates/list.php';
    }

    function show_action($id)
    {
        $post = get_post_by_id($id);
        require 'templates/show.php';
    }

As a front controller, ``index.php`` has taken on an entirely new role, one
that includes loading the core libraries and routing the application so that
one of the two controllers (the ``list_action()`` and ``show_action()``
functions) is called. In reality, the front controller is beginning to look and
act a lot like Symfony2's mechanism for handling and routing requests.
作为一个前端控制器，index.php具有一个全新的角色，它可以载入核心库，并对各个页面配置路径和对应的
控制器。实际上，前端控制器现在已经开始看起来像symfony2中行了路径请求的机制了。

.. tip::

   Another advantage of a front controller is flexible URLs. Notice that
   the URL to the blog post show page could be changed from ``/show`` to ``/read``
   by changing code in only one location. Before, an entire file needed to
   be renamed. In Symfony2, URLs are even more flexible.
   前端控制器还有一个优势就是灵活的URL。注意链接到博客文章显示页面的URL可以被修改为
   /read或其他，而要达到这个目的，只要修改一个地方的代码就可以了。在以前，要修改URL
   还要重命名整个文件。

By now, the application has evolved from a single PHP file into a structure
that is organized and allows for code reuse. You should be happier, but far
from satisfied. For example, the "routing" system is fickle, and wouldn't
recognize that the list page (``/index.php``) should be accessible also via ``/``
(if Apache rewrite rules were added). Also, instead of developing the blog,
a lot of time is being spent working on the "architecture" of the code (e.g.
routing, calling controllers, templates, etc.). More time will need to be
spent to handle form submissions, input validation, logging and security.
Why should you have to reinvent solutions to all these routine problems?
现在，这个应用已经从一个单独的php文件发展到一个比较好的结构，但是仍然有问题。比如，它
不知道/index.php页面也可以通过/进入。并且，你还要花大量时间来塑造代码结构（如路径，执行控制器，模板，等等），
还有很多其他工作诸如表单提交、输入验证、调试和安全需要花费大量时间来组织。

Add a Touch of Symfony2
加上一些symfony2的代码
~~~~~~~~~~~~~~~~~~~~~~~

Symfony2 to the rescue. Before actually using Symfony2, you need to make
sure PHP knows how to find the Symfony2 classes. This is accomplished via
an autoloader that Symfony provides. An autoloader is a tool that makes it
possible to start using PHP classes without explicitly including the file
containing the class.
可以使用symfony2来参与解决这个问题。在使用symfony2之前，你需要确定php知道如何查找symfony2的类。
这是通过symfony中的autoloader来完成的。使用autoloader可以让你不必显性地包含类文件就能够使用该php类。

First, `download symfony`_ and place it into a ``vendor/symfony/symfony/`` directory.
Next, create an ``app/bootstrap.php`` file. Use it to ``require`` the two
files in the application and to configure the autoloader:
首先，`download symfony`_并将它置于一个``vendor/symfony/symfony/``目录下。然后，创建一个
``app/bootstrap.php``文件，使用它来require应用中的这两个文件并配置这个autoloader：

.. code-block:: html+php

    <?php
    // bootstrap.php
    require_once 'model.php';
    require_once 'controllers.php';
    require_once 'vendor/symfony/symfony/src/Symfony/Component/ClassLoader/UniversalClassLoader.php';

    $loader = new Symfony\Component\ClassLoader\UniversalClassLoader();
    $loader->registerNamespaces(array(
        'Symfony' => __DIR__.'/../vendor/symfony/symfony/src',
    ));

    $loader->register();

This tells the autoloader where the ``Symfony`` classes are. With this, you
can start using Symfony classes without using the ``require`` statement for
the files that contain them.
这告诉了autoloader symfony的类在哪里。通过它你不必使用require语句来包含symfony类文件就可以使用这些类了。

Core to Symfony's philosophy is the idea that an application's main job is
to interpret each request and return a response. To this end, Symfony2 provides
both a :class:`Symfony\\Component\\HttpFoundation\\Request` and a
:class:`Symfony\\Component\\HttpFoundation\\Response` class. These classes are
object-oriented representations of the raw HTTP request being processed and
the HTTP response being returned. Use them to improve the blog:
symfony的核心原理就是一个应用的主要功能就是解析各个请求（request）并返回一个响应（response）。
symfony2提供了:class:`Symfony\\Component\\HttpFoundation\\Request`和:class:`Symfony\\Component\\HttpFoundation\\Response`。
这些类都是以对象的形式来处理原始HTTP请求并返回HTTP响应的。

.. code-block:: html+php

    <?php
    // index.php
    require_once 'app/bootstrap.php';

    use Symfony\Component\HttpFoundation\Request;
    use Symfony\Component\HttpFoundation\Response;

    $request = Request::createFromGlobals();

    $uri = $request->getPathInfo();
    if ($uri == '/') {
        $response = list_action();
    } elseif ($uri == '/show' && $request->query->has('id')) {
        $response = show_action($request->query->get('id'));
    } else {
        $html = '<html><body><h1>Page Not Found</h1></body></html>';
        $response = new Response($html, 404);
    }

    // echo the headers and send the response
    $response->send();

The controllers are now responsible for returning a ``Response`` object.
To make this easier, you can add a new ``render_template()`` function, which,
incidentally, acts quite a bit like the Symfony2 templating engine:
现在这些控制器的工作就是返回一个response类。你可以添加一个新的``render_template()``函数，它就像symfony
中的模板引擎：

.. code-block:: php

    // controllers.php
    use Symfony\Component\HttpFoundation\Response;

    function list_action()
    {
        $posts = get_all_posts();
        $html = render_template('templates/list.php', array('posts' => $posts));

        return new Response($html);
    }

    function show_action($id)
    {
        $post = get_post_by_id($id);
        $html = render_template('templates/show.php', array('post' => $post));

        return new Response($html);
    }

    // helper function to render templates
    function render_template($path, array $args)
    {
        extract($args);
        ob_start();
        require $path;
        $html = ob_get_clean();

        return $html;
    }

By bringing in a small part of Symfony2, the application is more flexible and
reliable. The ``Request`` provides a dependable way to access information
about the HTTP request. Specifically, the ``getPathInfo()`` method returns
a cleaned URI (always returning ``/show`` and never ``/index.php/show``).
So, even if the user goes to ``/index.php/show``, the application is intelligent
enough to route the request through ``show_action()``.
通过使用了一些symfony2代码，这个应用变得更加灵活可靠了。request提供了一个可靠方法来访问
HTTP请求的信息。尤其，``getPathInfo()``方法返回了一个干净的URI（总是返回/show而不是/index.php/show）。
所以，即使用户输入/index.php/show，这个应用也会将请求导向show_action()。

The ``Response`` object gives flexibility when constructing the HTTP response,
allowing HTTP headers and content to be added via an object-oriented interface.
And while the responses in this application are simple, this flexibility
will pay dividends as your application grows.
当创建HTTP响应的时候，response对象提供了很大方便。它允许HTTP头文件和内容通过一个对象接口被
添加。当扩展应用时，它会带来方便。

The Sample Application in Symfony2
使用symfony2来编写一个应用
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

The blog has come a *long* way, but it still contains a lot of code for such
a simple application. Along the way, we've also invented a simple routing
system and a method using ``ob_start()`` and ``ob_get_clean()`` to render
templates. If, for some reason, you needed to continue building this "framework"
from scratch, you could at least use Symfony's standalone `Routing`_ and
`Templating`_ components, which already solve these problems.
现在博客已经变得好多了，但对于如此一个简单的应用，它的代码还是太多了。所以我们还要介绍一个
简单的路由系统和一个使用``ob_start()``和``ob_get_clean()``的方法来提交模板。
如果你需要从头开始来创建这个应用，起码你可以使用symfony中独立的`Routing`_和`Templating`_ component，
它们可以解决以上问题。

Instead of re-solving common problems, you can let Symfony2 take care of
them for you. Here's the same sample application, now built in Symfony2:
不必从头开始解决这些常见问题，symfony2可以为你做这些事情。以下是使用symfony2编写的一个应用：

.. code-block:: html+php

    <?php
    // src/Acme/BlogBundle/Controller/BlogController.php

    namespace Acme\BlogBundle\Controller;
    use Symfony\Bundle\FrameworkBundle\Controller\Controller;

    class BlogController extends Controller
    {
        public function listAction()
        {
            $posts = $this->get('doctrine')->getManager()
                ->createQuery('SELECT p FROM AcmeBlogBundle:Post p')
                ->execute();

            return $this->render('AcmeBlogBundle:Blog:list.html.php', array('posts' => $posts));
        }

        public function showAction($id)
        {
            $post = $this->get('doctrine')
                ->getManager()
                ->getRepository('AcmeBlogBundle:Post')
                ->find($id);

            if (!$post) {
                // cause the 404 page not found to be displayed
                throw $this->createNotFoundException();
            }

            return $this->render('AcmeBlogBundle:Blog:show.html.php', array('post' => $post));
        }
    }

The two controllers are still lightweight. Each uses the Doctrine ORM library
to retrieve objects from the database and the ``Templating`` component to
render a template and return a ``Response`` object. The list template is
now quite a bit simpler:
这两个控制器是轻量级的。每个都使用Doctrine ORM库来从数据库获取对象，
templating component可以提交一个模板并且返回一个response对象。现在编写博客列表的
模板就简单了：

.. code-block:: html+php

    <!-- src/Acme/BlogBundle/Resources/views/Blog/list.html.php -->
    <?php $view->extend('::layout.html.php') ?>

    <?php $view['slots']->set('title', 'List of Posts') ?>

    <h1>List of Posts</h1>
    <ul>
        <?php foreach ($posts as $post): ?>
        <li>
            <a href="<?php echo $view['router']->generate('blog_show', array('id' => $post->getId())) ?>">
                <?php echo $post->getTitle() ?>
            </a>
        </li>
        <?php endforeach; ?>
    </ul>

The layout is nearly identical:
layout也一样：

.. code-block:: html+php

    <!-- app/Resources/views/layout.html.php -->
    <html>
        <head>
            <title><?php echo $view['slots']->output('title', 'Default title') ?></title>
        </head>
        <body>
            <?php echo $view['slots']->output('_content') ?>
        </body>
    </html>

.. note::

    We'll leave the show template as an exercise, as it should be trivial to
    create based on the list template.

When Symfony2's engine (called the ``Kernel``) boots up, it needs a map so
that it knows which controllers to execute based on the request information.
A routing configuration map provides this information in a readable format:
当symfony2引擎（被称作``Kernel``）启动时，它需要一个映射，从而知道根据请求的信息哪个对应的控制器要
被执行。路径配置映射通过可读的格式来提供这个信息：

.. code-block:: yaml

    # app/config/routing.yml
    blog_list:
        pattern:  /blog
        defaults: { _controller: AcmeBlogBundle:Blog:list }

    blog_show:
        pattern:  /blog/show/{id}
        defaults: { _controller: AcmeBlogBundle:Blog:show }

Now that Symfony2 is handling all the mundane tasks, the front controller
is dead simple. And since it does so little, you'll never have to touch
it once it's created (and if you use a Symfony2 distribution, you won't
even need to create it!):
symfony承担了大部分工作，现在控制器的工作非常简单了，由于它只做一点点工作，
当创建它以后你可能根本不必去修改它（如果你使用一套完整的symfony2，你甚至都不必创建它！）：

.. code-block:: html+php

    <?php
    // web/app.php
    require_once __DIR__.'/../app/bootstrap.php';
    require_once __DIR__.'/../app/AppKernel.php';

    use Symfony\Component\HttpFoundation\Request;

    $kernel = new AppKernel('prod', false);
    $kernel->handle(Request::createFromGlobals())->send();

The front controller's only job is to initialize Symfony2's engine (``Kernel``)
and pass it a ``Request`` object to handle. Symfony2's core then uses the
routing map to determine which controller to call. Just like before, the
controller method is responsible for returning the final ``Response`` object.
There's really not much else to it.
前端控制器的唯一工作就是初始化symfony2引擎（kernel）并将一个request对象传递给它处理。symfony2的核心
会使用路径映射来确定哪个控制器要使用。控制器的工作就是返回一个最终的response对象。

For a visual representation of how Symfony2 handles each request, see the
:ref:`request flow diagram<request-flow-figure>`.
要了解symfony2处理每个请求的流程，请参阅图表:ref:`request flow diagram<request-flow-figure>`。

Where Symfony2 Delivers
symfony2带来了哪些好处
~~~~~~~~~~~~~~~~~~~~~~~

In the upcoming chapters, you'll learn more about how each piece of Symfony
works and the recommended organization of a project. For now, let's see how
migrating the blog from flat PHP to Symfony2 has improved life:
在book中，你将学习更多关于symfony的每个部分是如何工作，以及使用它编写的代码是如何组织的。
下面总结一下symfony2与手写php相比的优势所在：

* Your application now has **clear and consistently organized code** (though
  Symfony doesn't force you into this). This promotes **reusability** and
  allows for new developers to be productive in your project more quickly.
  使你的应用有一个组织清晰并可扩展的代码结构。提升了代码的可重用性，并使其他开发者能够
  更迅速地修改或扩展你的代码。

* 100% of the code you write is for *your* application. You **don't need
  to develop or maintain low-level utilities** such as :ref:`autoloading<autoloading-introduction-sidebar>`,
  :doc:`routing</book/routing>`, or rendering :doc:`controllers</book/controller>`.
  你所编写的代码100%都仅仅是你的应用需要的，你不必开发或维护低层次的代码如:ref:`autoloading<autoloading-introduction-sidebar>`，
  :doc:`routing</book/routing>`，或:doc:`controllers</book/controller>`。

* Symfony2 gives you **access to open source tools** such as Doctrine and the
  Templating, Security, Form, Validation and Translation components (to name
  a few).
  symfony2允许你使用开源工具如doctrine，tamplating，security，form，validation和translation
  component。

* The application now enjoys **fully-flexible URLs** thanks to the ``Routing``
  component.
  由于有routing component，应用现在可以支持灵活的URL。

* Symfony2's HTTP-centric architecture gives you access to powerful tools
  such as **HTTP caching** powered by **Symfony2's internal HTTP cache** or
  more powerful tools such as `Varnish`_. This is covered in a later chapter
  all about :doc:`caching</book/http_cache>`.
  symfony2的架构是以HTTP为中心的，你可以使用像HTTP caching或者像`Varnish`_那样的工具。
  这在以后的章节:doc:`caching</book/http_cache>`中会讲到。

And perhaps best of all, by using Symfony2, you now have access to a whole
set of **high-quality open source tools developed by the Symfony2 community**!
A good selection of Symfony2 community tools can be found on `KnpBundles.com`_.
在`KnpBundles.com`_上可以找到symfony2社区的许多工具。

Better templates
更好的模板工具
----------------

If you choose to use it, Symfony2 comes standard with a templating engine
called `Twig`_ that makes templates faster to write and easier to read.
It means that the sample application could contain even less code! Take,
for example, the list template written in Twig:
symfony2内置了一个模板引擎`Twig`_，它可以使你更快更好的编写模板代码。
以下的模板是使用twig编写的：

.. code-block:: html+jinja

    {# src/Acme/BlogBundle/Resources/views/Blog/list.html.twig #}

    {% extends "::layout.html.twig" %}
    {% block title %}List of Posts{% endblock %}

    {% block body %}
        <h1>List of Posts</h1>
        <ul>
            {% for post in posts %}
            <li>
                <a href="{{ path('blog_show', { 'id': post.id }) }}">
                    {{ post.title }}
                </a>
            </li>
            {% endfor %}
        </ul>
    {% endblock %}

The corresponding ``layout.html.twig`` template is also easier to write:
对应的``layout.html.twig``模板则更容易：

.. code-block:: html+jinja

    {# app/Resources/views/layout.html.twig #}

    <html>
        <head>
            <title>{% block title %}Default title{% endblock %}</title>
        </head>
        <body>
            {% block body %}{% endblock %}
        </body>
    </html>

Twig is well-supported in Symfony2. And while PHP templates will always
be supported in Symfony2, we'll continue to discuss the many advantages of
Twig. For more information, see the :doc:`templating chapter</book/templating>`.
twig在symfony2中能够被很好的支持，但symfony2也支持php模板，我们将讨论关于twig的更多优势。
详情请见:doc:`templating chapter</book/templating>`。

Learn more from the Cookbook
----------------------------

* :doc:`/cookbook/templating/PHP`
* :doc:`/cookbook/controller/service`

.. _`Doctrine`: http://www.doctrine-project.org
.. _`download symfony`: http://symfony.com/download
.. _`Routing`: https://github.com/symfony/Routing
.. _`Templating`: https://github.com/symfony/Templating
.. _`KnpBundles.com`: http://knpbundles.com/
.. _`Twig`: http://twig.sensiolabs.org
.. _`Varnish`: http://www.varnish-cache.org
.. _`PHPUnit`: http://www.phpunit.de
