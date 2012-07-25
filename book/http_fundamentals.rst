.. index::
   single: Symfony2 Fundamentals

Symfony2 and HTTP Fundamentals
symfony2与HTTP基础
==============================

Congratulations! By learning about Symfony2, you're well on your way towards
being a more *productive*, *well-rounded* and *popular* web developer (actually,
you're on your own for the last part). Symfony2 is built to get back to
basics: to develop tools that let you develop faster and build more robust
applications, while staying out of your way. Symfony is built on the best
ideas from many technologies: the tools and concepts you're about to learn
represent the efforts of thousands of people, over many years. In other words,
you're not just learning "Symfony", you're learning the fundamentals of the
web, development best practices, and how to use many amazing new PHP libraries,
inside or independent of Symfony2. So, get ready.
symfony2的目标很基础也很明确：帮助你开发更快的应用以及编写更健壮的代码，而且不干扰你。
symfony2建立于最佳技术概念的基础上，你将要学习的工具和概念代表了几千人以及多年的努力，
换句话说，你不仅仅在学习symfony，你学习的是web开发的最佳实践以及如何使用多个新的php库，
这些库被集成于symfony内部或者独立于symfony之外。

True to the Symfony2 philosophy, this chapter begins by explaining the fundamental
concept common to web development: HTTP. Regardless of your background or
preferred programming language, this chapter is a **must-read** for everyone.
本章介绍web开发的基础概念:HTTP。不管你的背景如何，不管你使用什么编程语言，请务必阅读本章。

HTTP is Simple
HTTP很简单
--------------

HTTP (Hypertext Transfer Protocol to the geeks) is a text language that allows
two machines to communicate with each other. That's it! For example, when
checking for the latest `xkcd`_ comic, the following (approximate) conversation
takes place:
HTTP (Hypertext Transfer Protocol to the geeks)就是一种允两个机器通信的文本语言，就是这么简单！
比如，如果你访问`xkcd`_这个 网站，以下的会话将发生:

.. image:: /images/http-xkcd.png
   :align: center

And while the actual language used is a bit more formal, it's still dead-simple.
HTTP is the term used to describe this simple text-based language. And no
matter how you develop on the web, the goal of your server is *always* to
understand simple text requests, and return simple text responses.
虽然实际的HTTP语言更正式，但是它还是很简单。HTTP这个术语就是用来描述这个简单的文本语言的。
不管你如何在web上开发，你的服务器的目的就是理解简单的文本请求（request），并返回简单的文本响应（response）。

Symfony2 is built from the ground-up around that reality. Whether you realize
it or not, HTTP is something you use everyday. With Symfony2, you'll learn
how to master it.
symfony2就是根据这个理念来创建的。通过symfony2，你将学习如何掌握它。

.. index::
   single: HTTP; Request-response paradigm

Step1: The Client sends a Request
第一步：客户端发送一个请求
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Every conversation on the web starts with a *request*. The request is a text
message created by a client (e.g. a browser, an iPhone app, etc) in a
special format known as HTTP. The client sends that request to a server,
and then waits for the response.
每个web上的会话都开始于一个请求（request），这个请求是客户端（比如一个浏览器，或一个iPhone app，等等）使用HTTP格式创建的一个文本信息。
客户端会将请求发送到服务器，并等待响应（response）。

Take a look at the first part of the interaction (the request) between a
browser and the xkcd web server:
请参见这个浏览器与xkcd网站的web服务器交互的第一个部分:

.. image:: /images/http-xkcd-request.png
   :align: center

In HTTP-speak, this HTTP request would actually look something like this:
如果使用HTTP语言来描述，这个HTTP请求会看起来像这样:

.. code-block:: text

    GET / HTTP/1.1
    Host: xkcd.com
    Accept: text/html
    User-Agent: Mozilla/5.0 (Macintosh)

This simple message communicates *everything* necessary about exactly which
resource the client is requesting. The first line of an HTTP request is the
most important and contains two things: the URI and the HTTP method.
这个简单的信息描述了客户端要请求哪个源（resource）。HTTP请求的第一行是最重要的，
它包含了两个信息：URI和HTTP 方法（method）。

The URI (e.g. ``/``, ``/contact``, etc) is the unique address or location
that identifies the resource the client wants. The HTTP method (e.g. ``GET``)
defines what you want to *do* with the resource. The HTTP methods are the
*verbs* of the request and define the few common ways that you can act upon
the resource:
URI（比如``/``, ``/contact``, 等等）就是指向客户端需要的源的唯一地址。HTTP方法（比如GET）
定义了你想对这个源做什么，它就是请求中的动作:

+----------+---------------------------------------+
| *GET*    | Retrieve the resource from the server |
+----------+---------------------------------------+
| *POST*   | Create a resource on the server       |
+----------+---------------------------------------+
| *PUT*    | Update the resource on the server     |
+----------+---------------------------------------+
| *DELETE* | Delete the resource from the server   |
+----------+---------------------------------------+

With this in mind, you can imagine what an HTTP request might look like to
delete a specific blog entry, for example:
比如假定你想要删除一个博客，这个HTTP请求就会像这样:

.. code-block:: text

    DELETE /blog/15 HTTP/1.1

.. note::

    There are actually nine HTTP methods defined by the HTTP specification,
    but many of them are not widely used or supported. In reality, many modern
    browsers don't support the ``PUT`` and ``DELETE`` methods.
    实际上HTTP规则定义了9个HTTP方法，但是它们大部分不被广泛使用或支持。实际上，现在的浏览器
    不支持PUT和DELETE方法。

In addition to the first line, an HTTP request invariably contains other
lines of information called request headers. The headers can supply a wide
range of information such as the requested ``Host``, the response formats
the client accepts (``Accept``) and the application the client is using to
make the request (``User-Agent``). Many other headers exist and can be found
on Wikipedia's `List of HTTP header fields`_ article.
除了第一行，一个HTTP请求还包含其他信息行，这些行被称作请求头文件（header）。这些头文件能
支持许多信息，比如被请求的主机（Host）、客户端支持的响应格式（Accept）、客户端使用的用来
创建请求的应用（``User-Agent``），等等。还有很多其他头文件可以在`List of HTTP header fields`_中找到。

Step 2: The Server returns a Response
第二步：服务器返回一个响应
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Once a server has received the request, it knows exactly which resource the
client needs (via the URI) and what the client wants to do with that resource
(via the method). For example, in the case of a GET request, the server
prepares the resource and returns it in an HTTP response. Consider the response
from the xkcd web server:
一旦服务器接收到请求，它会通过URI知道客户端需要哪个源，以及通过HTTP方法知道客户端要如何处理那个源。
比如，在一个GET请求中，服务器将源在HTTP响应中返回。比如来自xkcd网站的web服务器的响应:

.. image:: /images/http-xkcd.png
   :align: center

Translated into HTTP, the response sent back to the browser will look something
like this: 
将它翻译成HTTP语言，这个发送回浏览器的响应看起来会是这样的:

.. code-block:: text

    HTTP/1.1 200 OK
    Date: Sat, 02 Apr 2011 21:05:05 GMT
    Server: lighttpd/1.4.19
    Content-Type: text/html

    <html>
      <!-- HTML for the xkcd comic -->
    </html>

The HTTP response contains the requested resource (the HTML content in this
case), as well as other information about the response. The first line is
especially important and contains the HTTP response status code (200 in this
case). The status code communicates the overall outcome of the request back
to the client. Was the request successful? Was there an error? Different
status codes exist that indicate success, an error, or that the client needs
to do something (e.g. redirect to another page). A full list can be found
on Wikipedia's `List of HTTP status codes`_ article.
这个HTTP响应包括请求源（在这个例子中，源就是HTML内容），还有许多其他关于响应的信息。第一行是最重要的，它
包含了HTTP响应状态编码（即status code，在这个例子中是200）。状态编码将请求的结果返回给客户端。
请求是否成功？是否有错误？不同的状态编码代表了成功，或者错误，或者客户端需要做什么事情（比如重定向到另一个页面）。
所有的HTTP状态编码在`List of HTTP status codes`_中可以找到。

Like the request, an HTTP response contains additional pieces of information
known as HTTP headers. For example, one important HTTP response header is
``Content-Type``. The body of the same resource could be returned in multiple
different formats like HTML, XML, or JSON and the ``Content-Type`` header uses
Internet Media Types like ``text/html`` to tell the client which format is
being returned. A list of common media types can be found on Wikipedia's 
`List of common media types`_ article.
像请求一样，一个HTTP响应包含了其他HTTP头文件。比如一个重要的HTTP响应头文件是Content-Type。
源的内容可以以不同格式来返回，比如HTML,XML,或JSON，而``Content-Type``头文件则使用如text/html这样
的因特网媒体类型来告诉客户端要返回哪个格式。常用的媒体类型可以在`List of common media types`_中找到。

Many other headers exist, some of which are very powerful. For example, certain
headers can be used to create a powerful caching system.
还有很多其他头文件，有些非常有用。比如，有些头文件可以被用来创建缓存系统。

Requests, Responses and Web Development
请求，响应和web开发
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

This request-response conversation is the fundamental process that drives all
communication on the web. And as important and powerful as this process is,
it's inescapably simple.
请求-响应会话是web通信的基础。虽然它很重要，但是它很简单。

The most important fact is this: regardless of the language you use, the
type of application you build (web, mobile, JSON API), or the development
philosophy you follow, the end goal of an application is **always** to understand
each request and create and return the appropriate response.
最重要的事实是：不管你使用什么语言，创建什么类型的应用（web, mobile, JSON API），或者
基于什么开发原理，应用的最终目的都是解析每个请求并返回恰当的响应。

Symfony is architected to match this reality.
symfony就是根据这个原理来构建的。

.. tip::

    To learn more about the HTTP specification, read the original `HTTP 1.1 RFC`_
    or the `HTTP Bis`_, which is an active effort to clarify the original
    specification. A great tool to check both the request and response headers
    while browsing is the `Live HTTP Headers`_ extension for Firefox.
    要了解更多有关HTTP规则的信息，请参阅`HTTP 1.1 RFC`_或`HTTP Bis`_。如果你想在浏览网页的时候查找
    请求和响应的头文件，可以在火狐浏览器中安装`Live HTTP Headers`_扩展。

.. index::
   single: Symfony2 Fundamentals; Requests and responses

Requests and Responses in PHP
php中的请求和响应
-----------------------------

So how do you interact with the "request" and create a "response" when using
PHP? In reality, PHP abstracts you a bit from the whole process:
那么你如何使用php来与请求交互并创建一个响应呢？事实上，php已经将它从整个过程中抽象出来了：

.. code-block:: php

    <?php
    $uri = $_SERVER['REQUEST_URI'];
    $foo = $_GET['foo'];

    header('Content-type: text/html');
    echo 'The URI requested is: '.$uri;
    echo 'The value of the "foo" parameter is: '.$foo;

As strange as it sounds, this small application is in fact taking information
from the HTTP request and using it to create an HTTP response. Instead of
parsing the raw HTTP request message, PHP prepares superglobal variables
such as ``$_SERVER`` and ``$_GET`` that contain all the information from
the request. Similarly, instead of returning the HTTP-formatted text response,
you can use the ``header()`` function to create response headers and simply
print out the actual content that will be the content portion of the response
message. PHP will create a true HTTP response and return it to the client:
这个小应用实际上从HTTP请求中提取了信息并使用这个信息创建了一个HTTP响应。php使用包含了请求中信息的超全局变量如
``$_SERVER``和``$_GET``，而不是解析原始HTTP请求信息。相似的，你可以使用header()方法来创建响应头文件并
输出响应内容，而不是返回HTTP格式的文本响应。php会创建一个真正的HTTP响应并将它返回给客户端:

.. code-block:: text

    HTTP/1.1 200 OK
    Date: Sat, 03 Apr 2011 02:14:33 GMT
    Server: Apache/2.2.17 (Unix)
    Content-Type: text/html

    The URI requested is: /testing?foo=symfony
    The value of the "foo" parameter is: symfony

Requests and Responses in Symfony
symfony中的请求和响应
---------------------------------

Symfony provides an alternative to the raw PHP approach via two classes that
allow you to interact with the HTTP request and response in an easier way.
The :class:`Symfony\\Component\\HttpFoundation\\Request` class is a simple
object-oriented representation of the HTTP request message. With it, you
have all the request information at your fingertips::
symfony针对这种原始php方法而制定了一个可选方法，它使用两个类来与HTTP请求和HTTP响应交互。
:class:`Symfony\\Component\\HttpFoundation\\Request`类是一个简单的代表了HTTP请求信息的对象。你可以使用它
来获取所有请求信息::

    use Symfony\Component\HttpFoundation\Request;

    $request = Request::createFromGlobals();

    // the URI being requested (e.g. /about) minus any query parameters
    $request->getPathInfo();

    // retrieve GET and POST variables respectively
    $request->query->get('foo');
    $request->request->get('bar', 'default value if bar does not exist');

    // retrieve SERVER variables
    $request->server->get('HTTP_HOST');

    // retrieves an instance of UploadedFile identified by foo
    $request->files->get('foo');

    // retrieve a COOKIE value
    $request->cookies->get('PHPSESSID');

    // retrieve an HTTP request header, with normalized, lowercase keys
    $request->headers->get('host');
    $request->headers->get('content_type');

    $request->getMethod();          // GET, POST, PUT, DELETE, HEAD
    $request->getLanguages();       // an array of languages the client accepts

As a bonus, the ``Request`` class does a lot of work in the background that
you'll never need to worry about. For example, the ``isSecure()`` method
checks the *three* different values in PHP that can indicate whether or not
the user is connecting via a secured connection (i.e. ``https``).
Request对象还在后台做一些其他工作。比如，``isSecure()``方法会检查php中的值来确定
用户是否在使用一个安全连接（即https）。

.. sidebar:: ParameterBags and Request attributes

    As seen above, the ``$_GET`` and ``$_POST`` variables are accessible via
    the public ``query`` and ``request`` properties respectively. Each of
    these objects is a :class:`Symfony\\Component\\HttpFoundation\\ParameterBag`
    object, which has methods like
    :method:`Symfony\\Component\\HttpFoundation\\ParameterBag::get`,
    :method:`Symfony\\Component\\HttpFoundation\\ParameterBag::has`,
    :method:`Symfony\\Component\\HttpFoundation\\ParameterBag::all` and more.
    In fact, every public property used in the previous example is some instance
    of the ParameterBag.
    如上所示，``$_GET``和``$_POST``变量可以通过query和request属性来访问。这些对象每个都是一个
    :class:`Symfony\\Component\\HttpFoundation\\ParameterBag`对象，它有一些方法如
    :method:`Symfony\\Component\\HttpFoundation\\ParameterBag::get`,
    :method:`Symfony\\Component\\HttpFoundation\\ParameterBag::has`,
    :method:`Symfony\\Component\\HttpFoundation\\ParameterBag::all`等等。
    事实上，在以上范例中所用到的所有public属性都是ParameterBag的实例。
    
    .. _book-fundamentals-attributes:
    
    The Request class also has a public ``attributes`` property, which holds
    special data related to how the application works internally. For the
    Symfony2 framework, the ``attributes`` holds the values returned by the
    matched route, like ``_controller``, ``id`` (if you have an ``{id}``
    wildcard), and even the name of the matched route (``_route``). The
    ``attributes`` property exists entirely to be a place where you can
    prepare and store context-specific information about the request.
    Request类还有一个attributes属性，它存储了应用内部的一些数据。在symfony2框架中，attributes存储了
    从匹配的框架中返回的值，比如``_controller``, ``id`` （如果有的话），甚至是匹配路径的名称（``_route``）。
    attributes完全是用来存储关于请求的特定环境的信息的。
    

Symfony also provides a ``Response`` class: a simple PHP representation of
an HTTP response message. This allows your application to use an object-oriented
interface to construct the response that needs to be returned to the client::
symfony还有一个Response类：它是一个简单的代表了HTTP响应信息的php类。你可以使用它来创建一个
返回给用户的响应::

    use Symfony\Component\HttpFoundation\Response;
    $response = new Response();

    $response->setContent('<html><body><h1>Hello world!</h1></body></html>');
    $response->setStatusCode(200);
    $response->headers->set('Content-Type', 'text/html');

    // prints the HTTP headers followed by the content
    $response->send();

If Symfony offered nothing else, you would already have a toolkit for easily
accessing request information and an object-oriented interface for creating
the response. Even as you learn the many powerful features in Symfony, keep
in mind that the goal of your application is always *to interpret a request
and create the appropriate response based on your application logic*.
现在你已经有了一个工具来访问请求的信息以及创建响应。你还将学习symfony的其他功能，但请记住
你的应用的目的总是要解析一个请求并根据你的应用逻辑来创建响应。

.. tip::

    The ``Request`` and ``Response`` classes are part of a standalone component
    included with Symfony called ``HttpFoundation``. This component can be
    used entirely independent of Symfony and also provides classes for handling
    sessions and file uploads.
    ``Request``和``Response``类都包含在symfony中一个名叫``HttpFoundation``的独立的component中。
    这个component可以独立与symfony使用，并提供处理session和文件上传的类。

The Journey from the Request to the Response
从请求到响应
--------------------------------------------

Like HTTP itself, the ``Request`` and ``Response`` objects are pretty simple.
The hard part of building an application is writing what comes in between.
In other words, the real work comes in writing the code that interprets the
request information and creates the response.
像HTTP本身一样，``Request``和``Response``对象都非常简单。创建一个应用最难的部分就是编写它们之间的代码。
也就是说最难的部分就是编写解析请求和创建响应的代码。

Your application probably does many things, like sending emails, handling
form submissions, saving things to a database, rendering HTML pages and protecting
content with security. How can you manage all of this and still keep your
code organized and maintainable?
你的应用可能会做许多事情，像发送邮件，处理表单提交，将数据保存到数据库，提交HTML页面或
给内容设定权限。但你如何在做这许多事情的同时还保证你的代码的组织性和可维护性？

Symfony was created to solve these problems so that you don't have to.
symfony就是用来解决这些问题的。

The Front Controller
前端控制器
~~~~~~~~~~~~~~~~~~~~

Traditionally, applications were built so that each "page" of a site was
its own physical file:
传统上，网站的每个页面都有它自己的文件:

.. code-block:: text

    index.php
    contact.php
    blog.php

There are several problems with this approach, including the inflexibility
of the URLs (what if you wanted to change ``blog.php`` to ``news.php`` without
breaking all of your links?) and the fact that each file *must* manually
include some set of core files so that security, database connections and
the "look" of the site can remain consistent.
这个方法有许多缺陷，包括URL的灵活性问题（比如，如果你想将blog.php修改为news.php而不破坏所有链接该怎么办？），
还有在文件中必须手动包含一些核心文件来处理安全问题、数据库连接以及模板等。


A much better solution is to use a :term:`front controller`: a single PHP
file that handles every request coming into your application. For example:
一个更好的方法是使用一个前端控制器（:term:`front controller`）:它是一个单独的php文件，可以
处理传递到你的应用的所有请求,比如:

+------------------------+------------------------+
| ``/index.php``         | executes ``index.php`` |
+------------------------+------------------------+
| ``/index.php/contact`` | executes ``index.php`` |
+------------------------+------------------------+
| ``/index.php/blog``    | executes ``index.php`` |
+------------------------+------------------------+

.. tip::

    Using Apache's ``mod_rewrite`` (or equivalent with other web servers),
    the URLs can easily be cleaned up to be just ``/``, ``/contact`` and
    ``/blog``.
    通过使用Apache的``mod_rewrite``（或其他服务器的类似方法），URL可以被简化为``/``, ``/contact``和``/blog``。

Now, every request is handled exactly the same. Instead of individual URLs
executing different PHP files, the front controller is *always* executed,
and the routing of different URLs to different parts of your application
is done internally. This solves both problems with the original approach.
Almost all modern web apps do this - including apps like WordPress.
现在每个请求都通过相似的过程来处理。应用会始终执行前端控制器，而不用针对不同URL来执行
不同php文件，并且URL都是通过内部处理的。这样就解决了原先方法带来的问题。现在几乎所有的web app都
这样做——包括wordpress。

Stay Organized
保持组织性
~~~~~~~~~~~~~~

But inside your front controller, how do you know which page should
be rendered and how can you render each in a sane way? One way or another, you'll need to
check the incoming URI and execute different parts of your code depending
on that value. This can get ugly quickly:
但是在你的前端控制器内部，你如何知道哪个页面要被提交、并如何提交它呢？不管你用
什么方法，你都要检查URI并且根据这个URI的值来执行你代码中的不同部分。用原始的方法可以这样:

.. code-block:: php

    // index.php

    $request = Request::createFromGlobals();
    $path = $request->getPathInfo(); // the URI path being requested

    if (in_array($path, array('', '/')) {
        $response = new Response('Welcome to the homepage.');
    } elseif ($path == '/contact') {
        $response = new Response('Contact us');
    } else {
        $response = new Response('Page not found.', 404);
    }
    $response->send();

Solving this problem can be difficult. Fortunately it's *exactly* what Symfony
is designed to do.
要解决这个问题会比较难办，但幸好symfony就是用来解决这个问题的。

The Symfony Application Flow
symfony应用流程
~~~~~~~~~~~~~~~~~~~~~~~~~~~~

When you let Symfony handle each request, life is much easier. Symfony follows
the same simple pattern for every request:
当你使用symfony来处理每个请求时就容易多了。对于每个请求symfony都使用如下流程:

.. _request-flow-figure:

.. figure:: /images/request-flow.png
   :align: center
   :alt: Symfony2 request flow

   Incoming requests are interpreted by the routing and passed to controller
   functions that return ``Response`` objects.
   请求被路由系统解析并传递给将返回Response对象的控制器函数。

Each "page" of your site is defined in a routing configuration file that
maps different URLs to different PHP functions. The job of each PHP function,
called a :term:`controller`, is to use information from the request - along
with many other tools Symfony makes available - to create and return a ``Response``
object. In other words, the controller is where *your* code goes: it's where
you interpret the request and create a response.
你的站点的每个页面都在一个路径配置文件中被定义，这个文件可以将不同的URL映射到不同的php函数。
这些php函数被称作控制器（:term:`controller`），它们可以根据请求中的信息以及许多symfony提供的工具
来创建一个Response对象。换句话说，控制器就是你要编写代码的地方：你在其中解析请求并创建响应。

It's that easy! Let's review:
回顾一下:

* Each request executes a front controller file;
* 每个请求都执行一个前端控制器文件；

* The routing system determines which PHP function should be executed based
  on information from the request and routing configuration you've created;
* 路由系统根据请求信息和路径配置决定哪个php函数要被执行；

* The correct PHP function is executed, where your code creates and returns
  the appropriate ``Response`` object.
* 正确的php函数被执行（也就是你编写代码并返回恰当的Response对象的地方）；

A Symfony Request in Action
一个symfony请求范例
~~~~~~~~~~~~~~~~~~~~~~~~~~~

Without diving into too much detail, let's see this process in action. Suppose
you want to add a ``/contact`` page to your Symfony application. First, start
by adding an entry for ``/contact`` to your routing configuration file:
让我们举个例子。假设你需要在你的symfony应用中添加一个/contact页面。首先，在你的路径配置文件中
添加/contact路径:

.. code-block:: yaml

    contact:
        pattern:  /contact
        defaults: { _controller: AcmeDemoBundle:Main:contact }

.. note::

   This example uses :doc:`YAML</components/yaml>` to define the routing
   configuration. Routing configuration can also be written in other formats
   such as XML or PHP.
   本例使用了:doc:`YAML</components/yaml>`来定义路径配置。路径配置还可以使用其他格式来编写，如XML和PHP。

When someone visits the ``/contact`` page, this route is matched, and the
specified controller is executed. As you'll learn in the :doc:`routing chapter</book/routing>`,
the ``AcmeDemoBundle:Main:contact`` string is a short syntax that points to a
specific PHP method ``contactAction`` inside a class called ``MainController``:
当用户访问/contact页面时，路径被匹配，指定的控制器被执行。在:doc:`routing chapter</book/routing>`中
你将学到``AcmeDemoBundle:Main:contact``字符串的意义，它是一种简化语法，指向``MainController``类
中的一个名叫``contactAction``的php方法。

.. code-block:: php

    class MainController
    {
        public function contactAction()
        {
            return new Response('<h1>Contact us!</h1>');
        }
    }

In this very simple example, the controller simply creates a ``Response``
object with the HTML "<h1>Contact us!</h1>". In the :doc:`controller chapter</book/controller>`,
you'll learn how a controller can render templates, allowing your "presentation"
code (i.e. anything that actually writes out HTML) to live in a separate
template file. This frees up the controller to worry only about the hard
stuff: interacting with the database, handling submitted data, or sending
email messages. 
在这个简单的例子中，控制器创建了"<h1>Contact us!</h1>"这么一个Response对象。在
:doc:`controller chapter</book/controller>`中，你将学习控制器是如何提交模板的，它允许你在一个分离
的模板文件中编写模板的HTML代码。这样，控制器要处理的就仅仅是那些比较有难度的工作了，比如与数据库交互，
处理提交的数据，或发送邮件。

Symfony2: Build your App, not your Tools.
symfony2：创建你的App，而不是工具。
-----------------------------------------

You now know that the goal of any app is to interpret each incoming request
and create an appropriate response. As an application grows, it becomes more
difficult to keep your code organized and maintainable. Invariably, the same
complex tasks keep coming up over and over again: persisting things to the
database, rendering and reusing templates, handling form submissions, sending
emails, validating user input and handling security.
你现在已经知道任何app的目的都是解析请求并创建响应。当应用的内容增加时，它就会变得越来越难以组织和维护。
而且，同样的工作会一遍又一遍地重复：将数据载入数据库，提交和重用模板，处理表单提交，发送邮件，验证用户输入
信息，处理安全问题等。

The good news is that none of these problems is unique. Symfony provides
a framework full of tools that allow you to build your application, not your
tools. With Symfony2, nothing is imposed on you: you're free to use the full
Symfony framework, or just one piece of Symfony all by itself.
symfony框架提供了许多帮助你创建应用的工具。在symfony2中并不要求你必须使用这些工具，你可以
自由选择是使用整个symfony框架还是使用它的一部分。

.. index::
   single: Symfony2 Components

Standalone Tools: The Symfony2 *Components*
独立的工具：symfony2 component
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

So what *is* Symfony2? First, Symfony2 is a collection of over twenty independent
libraries that can be used inside *any* PHP project. These libraries, called
the *Symfony2 Components*, contain something useful for almost any situation,
regardless of how your project is developed. To name a few:
symfony2集合了二十多个独立库，任何php应用都可以使用这些库，它们被称作symfony2 component。以下列举了一些这样的component:

* `HttpFoundation`_ - Contains the ``Request`` and ``Response`` classes, as
  well as other classes for handling sessions and file uploads;
* `HttpFoundation`_  包含了Request和Response类，以及其他处理session和文件上传的类；

* `Routing`_ - Powerful and fast routing system that allows you to map a
  specific URI (e.g. ``/contact``) to some information about how that request
  should be handled (e.g. execute the ``contactAction()`` method);
* `Routing`_  强大的路由系统，允许你将特定的URI（如/contact）映射到控制器函数（如``contactAction()``）并执行这个函数；

* `Form`_ - A full-featured and flexible framework for creating forms and
  handling form submissions;
* `Form`_  一个创建表单并处理表单提交的框架；

* `Validator`_ A system for creating rules about data and then validating
  whether or not user-submitted data follows those rules;
* `Validator`_  创建关于用户提交数据的规则并确认提交的数据是否符合这些规则；

* `ClassLoader`_ An autoloading library that allows PHP classes to be used
  without needing to manually ``require`` the files containing those classes;
* `ClassLoader`_  一个autoload库，允许在不手动包含（require）php类的文件的情况下就能够使用这些类；

* `Templating`_ A toolkit for rendering templates, handling template inheritance
  (i.e. a template is decorated with a layout) and performing other common
  template tasks;
* `Templating`_  用于提交模板、处理模板继承以及其他模板的工作；

* `Security`_ - A powerful library for handling all types of security inside
  an application;
* `Security`_  用于处理应用中的所有安全类型；

* `Translation`_ A framework for translating strings in your application
* `Translation`_  一个用于翻译你的应用中的文本的框架。

Each and every one of these components is decoupled and can be used in *any*
PHP project, regardless of whether or not you use the Symfony2 framework.
Every part is made to be used if needed and replaced when necessary.
这些component都可以从symfony中独立出来而被任何php应用使用（即使你不使用symfony2也可以使用它）。

The Full Solution: The Symfony2 *Framework*
全面的解决方案：symfony2框架
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

So then, what *is* the Symfony2 *Framework*? The *Symfony2 Framework* is
a PHP library that accomplishes two distinct tasks:
那么什么是symfony2框架？symfony2框架就是一个php库，它可以做以下工作：

#. Provides a selection of components (i.e. the Symfony2 Components) and
   third-party libraries (e.g. ``Swiftmailer`` for sending emails);
#. 汇集了多个component（或称symfony2 component）以及第三方库（比如用于发送邮件的``Swiftmailer``）；

#. Provides sensible configuration and a "glue" library that ties all of these
   pieces together.
#. 提供便利的配置方法以及一个可以将这些部分都粘连起来的库。

The goal of the framework is to integrate many independent tools in order
to provide a consistent experience for the developer. Even the framework
itself is a Symfony2 bundle (i.e. a plugin) that can be configured or replaced
entirely.
这个框架的目的就是集合多个独立的工具，从而给开发者更好的体验。甚至这个框架本身也是一个symfony2 bundle（或称插件），
它也可以被配置或修改。

Symfony2 provides a powerful set of tools for rapidly developing web applications
without imposing on your application. Normal users can quickly start development
by using a Symfony2 distribution, which provides a project skeleton with
sensible defaults. For more advanced users, the sky is the limit.
symfony2提供了一系列工具来帮助迅速开发web应用，而且也不强制你使用这些工具。一般用户都可以迅速地开始运用symfony2来开发，因为它
已经提供了一个便利的框架。对于更高级的用户，他们还可以自由地扩展该框架，从而实现更多。

.. _`xkcd`: http://xkcd.com/
.. _`HTTP 1.1 RFC`: http://www.w3.org/Protocols/rfc2616/rfc2616.html
.. _`HTTP Bis`: http://datatracker.ietf.org/wg/httpbis/
.. _`Live HTTP Headers`: https://addons.mozilla.org/en-US/firefox/addon/live-http-headers/
.. _`List of HTTP status codes`: http://en.wikipedia.org/wiki/List_of_HTTP_status_codes
.. _`List of HTTP header fields`: http://en.wikipedia.org/wiki/List_of_HTTP_header_fields
.. _`List of common media types`: http://en.wikipedia.org/wiki/Internet_media_type#List_of_common_media_types
.. _`HttpFoundation`: https://github.com/symfony/HttpFoundation
.. _`Routing`: https://github.com/symfony/Routing
.. _`Form`: https://github.com/symfony/Form
.. _`Validator`: https://github.com/symfony/Validator
.. _`ClassLoader`: https://github.com/symfony/ClassLoader
.. _`Templating`: https://github.com/symfony/Templating
.. _`Security`: https://github.com/symfony/Security
.. _`Translation`: https://github.com/symfony/Translation
