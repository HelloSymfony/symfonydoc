.. index::
   single: Internals

Internals
=========

Looks like you want to understand how Symfony2 works and how to extend it.
That makes me very happy! This section is an in-depth explanation of the
Symfony2 internals.
本章将深入解释symfony2的内部结构。

.. note::

    You need to read this section only if you want to understand how Symfony2
    works behind the scene, or if you want to extend Symfony2.
    如果你需要理解symfony2背后是如何工作的，或想要扩展symfony2，请阅读本章。

Overview
概览
--------

The Symfony2 code is made of several independent layers. Each layer is built
on top of the previous one.
symfony2由数个独立的层组成。每个层都在先前的层的基础上建立。

.. tip::

    Autoloading is not managed by the framework directly; it's done
    independently with the help of the
    :class:`Symfony\\Component\\ClassLoader\\UniversalClassLoader` class
    and the ``src/autoload.php`` file. Read the :doc:`dedicated chapter
    </components/class_loader>` for more information.
    本框架不支持自动载入；你可以使用:class:`Symfony\\Component\\ClassLoader\\UniversalClassLoader`
    类和``src/autoload.php``。请参阅:doc:`dedicated chapter
    </components/class_loader>`。

``HttpFoundation`` Component
~~~~~~~~~~~~~~~~~~~~~~~~~~~~

The deepest level is the :namespace:`Symfony\\Component\\HttpFoundation`
component. HttpFoundation provides the main objects needed to deal with HTTP.
It is an Object-Oriented abstraction of some native PHP functions and
variables:
最底层的是:namespace:`Symfony\\Component\\HttpFoundation` component。HttpFoundation提供处理HTTP所需要的
最主要的对象。它是一个包含php函数和变量的面向对象抽象层:

* The :class:`Symfony\\Component\\HttpFoundation\\Request` class abstracts
  the main PHP global variables like ``$_GET``, ``$_POST``, ``$_COOKIE``,
  ``$_FILES``, and ``$_SERVER``;
  :class:`Symfony\\Component\\HttpFoundation\\Request`类将主要php全局变量抽象出来，
  如``$_GET``, ``$_POST``, ``$_COOKIE``,
  ``$_FILES``, 和``$_SERVER``;

* The :class:`Symfony\\Component\\HttpFoundation\\Response` class abstracts
  some PHP functions like ``header()``, ``setcookie()``, and ``echo``;
  :class:`Symfony\\Component\\HttpFoundation\\Response`类抽象出了一些php函数如header(),setcookie()和echo();

* The :class:`Symfony\\Component\\HttpFoundation\\Session` class and
  :class:`Symfony\\Component\\HttpFoundation\\SessionStorage\\SessionStorageInterface`
  interface abstract session management ``session_*()`` functions.
  :class:`Symfony\\Component\\HttpFoundation\\Session`类和:class:`Symfony\\Component\\HttpFoundation\\SessionStorage\\SessionStorageInterface`
  接口抽象出了管理session的函数``session_*()``。

``HttpKernel`` Component
~~~~~~~~~~~~~~~~~~~~~~~~

On top of HttpFoundation is the :namespace:`Symfony\\Component\\HttpKernel`
component. HttpKernel handles the dynamic part of HTTP; it is a thin wrapper
on top of the Request and Response classes to standardize the way requests are
handled. It also provides extension points and tools that makes it the ideal
starting point to create a Web framework without too much overhead.
在HttpFoundation顶层的是:namespace:`Symfony\\Component\\HttpKernel` component。
HttpKernel可以处理http的动态部分；它是一个位于Request和Response类顶层的包装，用于使处理请求的方法
标准化。它还提供扩展的接入点和工具来使得创建web框架变得轻便。

It also optionally adds configurability and extensibility, thanks to the
Dependency Injection component and a powerful plugin system (bundles).
它还选择性地添加配置和扩展，这在Dependency Injection component和插件系统中（bundle）。

.. seealso::

    阅读更多关于:doc:`Dependency Injection </book/service_container>`和
    :doc:`Bundles </cookbook/bundles/best_practices>`的信息.

``FrameworkBundle`` Bundle
~~~~~~~~~~~~~~~~~~~~~~~~~~

The :namespace:`Symfony\\Bundle\\FrameworkBundle` bundle is the bundle that
ties the main components and libraries together to make a lightweight and fast
MVC framework. It comes with a sensible default configuration and conventions
to ease the learning curve.
:namespace:`Symfony\\Bundle\\FrameworkBundle`这个bundle将主要component和库联系在一起，从而
创造轻量级的MVC框架。它预装了一个可行的默认配置，使你的学习过程变得简单。

.. index::
   single: Internals; Kernel

Kernel
------

The :class:`Symfony\\Component\\HttpKernel\\HttpKernel` class is the central
class of Symfony2 and is responsible for handling client requests. Its main
goal is to "convert" a :class:`Symfony\\Component\\HttpFoundation\\Request`
object to a :class:`Symfony\\Component\\HttpFoundation\\Response` object.
:class:`Symfony\\Component\\HttpKernel\\HttpKernel`类是symfony2中的核心类，它用于处理客户端请求。
它的主要目的是将:class:`Symfony\\Component\\HttpFoundation\\Request`对象转化为
:class:`Symfony\\Component\\HttpFoundation\\Response`对象。

每个symfony2 kernel都植入了
:class:`Symfony\\Component\\HttpKernel\\HttpKernelInterface`::

    function handle(Request $request, $type = self::MASTER_REQUEST, $catch = true)

.. index::
   single: Internals; Controller Resolver

Controllers
控制器
~~~~~~~~~~~

To convert a Request to a Response, the Kernel relies on a "Controller". A
Controller can be any valid PHP callable.
要将一个请求转换为一个响应，kernel依于控制器。控制器可以是任何php功能。

kernel通过植入
:class:`Symfony\\Component\\HttpKernel\\Controller\\ControllerResolverInterface`这个代理来确定哪个
控制器应该被执行::

    public function getController(Request $request);

    public function getArguments(Request $request, $controller);

The
:method:`Symfony\\Component\\HttpKernel\\Controller\\ControllerResolverInterface::getController`
method returns the Controller (a PHP callable) associated with the given
Request. The default implementation
(:class:`Symfony\\Component\\HttpKernel\\Controller\\ControllerResolver`)
looks for a ``_controller`` request attribute that represents the controller
name (a "class::method" string, like
``Bundle\BlogBundle\PostController:indexAction``).
:method:`Symfony\\Component\\HttpKernel\\Controller\\ControllerResolverInterface::getController`方法返回与请求相对应
的控制器。默认的接口(:class:`Symfony\\Component\\HttpKernel\\Controller\\ControllerResolver`)查询一个
_controller请求参数，该参数代表了控制器的名称（一个class::method字符串，如``Bundle\BlogBundle\PostController:indexAction``）。

.. tip::

    The default implementation uses the
    :class:`Symfony\\Bundle\\FrameworkBundle\\EventListener\\RouterListener`
    to define the ``_controller`` Request attribute (see :ref:`kernel-core-request`).
    默认接口使用:class:`Symfony\\Bundle\\FrameworkBundle\\EventListener\\RouterListener`来定义_controller
    请求参数（参见:ref:`kernel-core-request`）。

The
:method:`Symfony\\Component\\HttpKernel\\Controller\\ControllerResolverInterface::getArguments`
method returns an array of arguments to pass to the Controller callable. The
default implementation automatically resolves the method arguments, based on
the Request attributes.
:method:`Symfony\\Component\\HttpKernel\\Controller\\ControllerResolverInterface::getArguments`方法
返回一个包含参数的数组来给控制器传递参数。默认的接口会自动根据请求参数决定方法的参数。

.. sidebar:: Matching Controller method arguments from Request attributes

    For each method argument, Symfony2 tries to get the value of a Request
    attribute with the same name. If it is not defined, the argument default
    value is used if defined::
    对于每个方法的参数，symfony2都会试图获取与该参数名称相同的请求参数的值。如果它没有被定义，则使用
    已经被定义的默认的值::

        // Symfony2 will look for an 'id' attribute (mandatory)
        // and an 'admin' one (optional)
        public function showAction($id, $admin = true)
        {
            // ...
        }

.. index::
  single: Internals; Request Handling

Handling Requests
处理请求
~~~~~~~~~~~~~~~~~

The ``handle()`` method takes a ``Request`` and *always* returns a ``Response``.
To convert the ``Request``, ``handle()`` relies on the Resolver and an ordered
chain of Event notifications (see the next section for more information about
each Event):
handle()方法会接收Request并返回Response。要转换Request，handle()依赖于Resolver以及
一个有序的事件触发链（参见下一章）:

1. Before doing anything else, the ``kernel.request`` event is notified -- if
   one of the listeners returns a ``Response``, it jumps to step 8 directly;
   首先，kernel.request事件被触发——如果listener中的一个返回一个Response，它会直接跳到第8步；

2. The Resolver is called to determine the Controller to execute;
2. Resolver确定要执行的控制器；

3. Listeners of the ``kernel.controller`` event can now manipulate the
   Controller callable the way they want (change it, wrap it, ...);
   kernel.contoller事件的listener会根据需要来操作控制器（修改，打包，等等）；

4. The Kernel checks that the Controller is actually a valid PHP callable;
4. Kernel检查控制器是否可用；

5. The Resolver is called to determine the arguments to pass to the Controller;
5. Resolver决定要传递给控制器的参数；

6. The Kernel calls the Controller;
6. 控制器被执行；

7. If the Controller does not return a ``Response``, listeners of the
   ``kernel.view`` event can convert the Controller return value to a ``Response``;
   如果控制器不返回一个Response，kernel.view的listener会将控制器的返回值转换为Response；

8. Listeners of the ``kernel.response`` event can manipulate the ``Response``
   (content and headers);
   kernel.response事件的listener操作Resonse（内容和头部）；

9. The Response is returned.
9. 返回Response。

If an Exception is thrown during processing, the ``kernel.exception`` is
notified and listeners are given a chance to convert the Exception to a
Response. If that works, the ``kernel.response`` event is notified; if not, the
Exception is re-thrown.
如果这个过程中一个Exception被抛出，则kernel.exception被触发，并且listener给你一个机会将Exception转换为Response。
如果成功，则kernel.response事件会被触发；如果不成功，则Exception会被抛出。

If you don't want Exceptions to be caught (for embedded requests for
instance), disable the ``kernel.exception`` event by passing ``false`` as the
third argument to the ``handle()`` method.
如果你不想要Exception被catch（比如对于嵌入的请求），可以通过将false作为第三个参数传递给handle()方法来实现。

.. index::
  single: Internals; Internal Requests

Internal Requests
内部请求
~~~~~~~~~~~~~~~~~

At any time during the handling of a request (the 'master' one), a sub-request
can be handled. You can pass the request type to the ``handle()`` method (its
second argument):
当处理请求时（主要的那个），一个子请求可以被处理。你可以将请求类型传递给handle()方法（它的第二个参数）:

* ``HttpKernelInterface::MASTER_REQUEST``;
* ``HttpKernelInterface::SUB_REQUEST``.

The type is passed to all events and listeners can act accordingly (some
processing must only occur on the master request).
这个类型可以被传递给所有的事件，并且listener可以根据它来动作（有些过程只在主要请求中发生）。

.. index::
   pair: Kernel; Event

Events
事件
~~~~~~

Each event thrown by the Kernel is a subclass of
:class:`Symfony\\Component\\HttpKernel\\Event\\KernelEvent`. This means that
each event has access to the same basic information:
Kernel抛出的事件是:class:`Symfony\\Component\\HttpKernel\\Event\\KernelEvent`类的子类。这
表示每个事件都可以访问同样的基本信息：

* ``getRequestType()`` - 返回请求的类型
  (``HttpKernelInterface::MASTER_REQUEST``或``HttpKernelInterface::SUB_REQUEST``);

* ``getKernel()`` - 返回处理请求的Kernel;

* ``getRequest()`` - 返回要被处理的Request。

``getRequestType()``
....................

The ``getRequestType()`` method allows listeners to know the type of the
request. For instance, if a listener must only be active for master requests,
add the following code at the beginning of your listener method::
``getRequestType()``方法使listener知道请求的类型。比如，如果一个listener必须只对于主要请求活跃，
你可以在listener方法的开始添加以下代码::

    use Symfony\Component\HttpKernel\HttpKernelInterface;

    if (HttpKernelInterface::MASTER_REQUEST !== $event->getRequestType()) {
        // return immediately
        return;
    }

.. tip::

    If you are not yet familiar with the Symfony2 Event Dispatcher, read the
    :doc:`Event Dispatcher Component Documentation</components/event_dispatcher/introduction>`
    section first.
    如果你对symfony2的事件分配器（Event Dispatcher）还不熟悉，请参阅:doc:`Event Dispatcher Component Documentation</components/event_dispatcher/introduction>`。

.. index::
   single: Event; kernel.request

.. _kernel-core-request:

``kernel.request``事件
........................

*Event Class*: :class:`Symfony\\Component\\HttpKernel\\Event\\GetResponseEvent`

The goal of this event is to either return a ``Response`` object immediately
or setup variables so that a Controller can be called after the event. Any
listener can return a ``Response`` object via the ``setResponse()`` method on
the event. In this case, all other listeners won't be called.
这个事件类的目的是返回一个Response对象或设置变量，从而在该事件后可以执行一个控制器。
所有listener都可以通过setResponse()方法来返回一个该事件的Response对象。在这种情况下，所有其他的listner都不会
被触发。

This event is used by ``FrameworkBundle`` to populate the ``_controller``
``Request`` attribute, via the
:class:`Symfony\\Bundle\\FrameworkBundle\\EventListener\\RouterListener`. RequestListener
uses a :class:`Symfony\\Component\\Routing\\RouterInterface` object to match
the ``Request`` and determine the Controller name (stored in the
``_controller`` ``Request`` attribute).
这个事件被``FrameworkBundle``用来通过:class:`Symfony\\Bundle\\FrameworkBundle\\EventListener\\RouterListener`将Request参数传递给_controller。


.. index::
   single: Event; kernel.controller

``kernel.controller``事件
...........................

*Event Class*: :class:`Symfony\\Component\\HttpKernel\\Event\\FilterControllerEvent`

This event is not used by ``FrameworkBundle``, but can be an entry point used
to modify the controller that should be executed:
这个事件不被``FrameworkBundle``使用，但可以作为修改控制器的入口：

.. code-block:: php

    use Symfony\Component\HttpKernel\Event\FilterControllerEvent;

    public function onKernelController(FilterControllerEvent $event)
    {
        $controller = $event->getController();
        // ...

        // the controller can be changed to any PHP callable
        $event->setController($controller);
    }

.. index::
   single: Event; kernel.view

``kernel.view``事件
.....................

*Event Class*: :class:`Symfony\\Component\\HttpKernel\\Event\\GetResponseForControllerResultEvent`

This event is not used by ``FrameworkBundle``, but it can be used to implement
a view sub-system. This event is called *only* if the Controller does *not*
return a ``Response`` object. The purpose of the event is to allow some other
return value to be converted into a ``Response``.
这个事件不被``FrameworkBundle``使用，但它可以被用来植入一个视图子系统。该事件仅在控制器不返回Response对象时被触发。
这个事件的目的是使其他返回的值能够被转换为Response。

The value returned by the Controller is accessible via the
``getControllerResult`` method::
控制器返回的值可以通过``getControllerResult``方法来访问::

    use Symfony\Component\HttpKernel\Event\GetResponseForControllerResultEvent;
    use Symfony\Component\HttpFoundation\Response;

    public function onKernelView(GetResponseForControllerResultEvent $event)
    {
        $val = $event->getControllerResult();
        $response = new Response();
        // some how customize the Response from the return value

        $event->setResponse($response);
    }

.. index::
   single: Event; kernel.response

``kernel.response``事件
.........................

*Event Class*: :class:`Symfony\\Component\\HttpKernel\\Event\\FilterResponseEvent`

The purpose of this event is to allow other systems to modify or replace the
``Response`` object after its creation:
这个事件的目的是允许在Response对象创建后，其他系统能修改或覆盖该对象:

.. code-block:: php

    public function onKernelResponse(FilterResponseEvent $event)
    {
        $response = $event->getResponse();
        // .. modify the response object
    }

The ``FrameworkBundle`` registers several listeners:
``FrameworkBundle``有多个listener:

* :class:`Symfony\\Component\\HttpKernel\\EventListener\\ProfilerListener`:
  为当前请求收集数据;

* :class:`Symfony\\Bundle\\WebProfilerBundle\\EventListener\\WebDebugToolbarListener`:
  植入调试栏（Web Debug Toolbar）;

* :class:`Symfony\\Component\\HttpKernel\\EventListener\\ResponseListener`: 根据请求格式
  调整响应的Content-Type;

* :class:`Symfony\\Component\\HttpKernel\\EventListener\\EsiListener`: adds a
  ``Surrogate-Control`` HTTP header when the Response needs to be parsed for
  ESI tags.
* :class:`Symfony\\Component\\HttpKernel\\EventListener\\EsiListener`:当需要使用ESI标签而解析Response时，
  添加一个``Surrogate-Control`` HTTP头部。

.. index::
   single: Event; kernel.exception

.. _kernel-kernel.exception:

``kernel.exception``事件
..........................

*Event Class*: :class:`Symfony\\Component\\HttpKernel\\Event\\GetResponseForExceptionEvent`

``FrameworkBundle`` registers an
:class:`Symfony\\Component\\HttpKernel\\EventListener\\ExceptionListener` that
forwards the ``Request`` to a given Controller (the value of the
``exception_listener.controller`` parameter -- must be in the
``class::method`` notation).
:class:`Symfony\\Component\\HttpKernel\\EventListener\\ExceptionListener`可以被``FrameworkBundle``所使用。
它将Request定向到一个给定的控制器（``exception_listener.controller``参数的值，它必须在``class::method``标记中）。

A listener on this event can create and set a ``Response`` object, create
and set a new ``Exception`` object, or do nothing:
这个事件的listener可以创建和设置一个Response对象，创建和设置一个新的Exception对象，或什么都不做:

.. code-block:: php

    use Symfony\Component\HttpKernel\Event\GetResponseForExceptionEvent;
    use Symfony\Component\HttpFoundation\Response;

    public function onKernelException(GetResponseForExceptionEvent $event)
    {
        $exception = $event->getException();
        $response = new Response();
        // setup the Response object based on the caught exception
        $event->setResponse($response);

        // you can alternatively set a new Exception
        // $exception = new \Exception('Some special exception');
        // $event->setException($exception);
    }

.. index::
   single: Event Dispatcher

The Event Dispatcher
事件分配器
--------------------

The event dispatcher is a standalone component that is responsible for much
of the underlying logic and flow behind a Symfony request. For more information,
see the :doc:`Event Dispatcher Component Documentation</components/event_dispatcher/introduction>`.
事件分配器（event dispatcher）是一个独立的component，它负责许多背后的逻辑，并针对每个请求。详情请见
:doc:`Event Dispatcher Component Documentation</components/event_dispatcher/introduction>`。

.. index::
   single: Profiler

.. _internals-profiler:

Profiler
--------

When enabled, the Symfony2 profiler collects useful information about each
request made to your application and store them for later analysis. Use the
profiler in the development environment to help you to debug your code and
enhance performance; use it in the production environment to explore problems
after the fact.
当被激活的时候，symfony2 profiler会收集对于每个请求的有用的信息并存储起来。你可以在开发环境中
使用它来帮助调试代码和提高性能，在生成环境中发现潜在问题。

You rarely have to deal with the profiler directly as Symfony2 provides
visualizer tools like the Web Debug Toolbar and the Web Profiler. If you use
the Symfony2 Standard Edition, the profiler, the web debug toolbar, and the
web profiler are all already configured with sensible settings.
你基本不需要直接处理profiler，因为symfony2已经提供了可视化工具，如web调试工具条和web profiler。
如果你使用的是symfony2标准版本，则profiler、web调试工具条和web profiler都已经被配置好了。

.. note::

    The profiler collects information for all requests (simple requests,
    redirects, exceptions, Ajax requests, ESI requests; and for all HTTP
    methods and all formats). It means that for a single URL, you can have
    several associated profiling data (one per external request/response
    pair).
    profiler会收集所有请求的信息（简单请求、重定向、错误、ajax请求、ESI请求或其他任何格式和http方法）。
    它表示对于一个单独的URL，你可以有多个相联系的profiling数据。

.. index::
   single: Profiler; Visualizing

Visualizing Profiling Data
可视化profiling数据
~~~~~~~~~~~~~~~~~~~~~~~~~~

Using the Web Debug Toolbar
使用web调试工具条
...........................

In the development environment, the web debug toolbar is available at the
bottom of all pages. It displays a good summary of the profiling data that
gives you instant access to a lot of useful information when something does
not work as expected.
在开发环境下，web调试工具条在所有页面的底部。它会展示许多简短形式的有用信息。

If the summary provided by the Web Debug Toolbar is not enough, click on the
token link (a string made of 13 random characters) to access the Web Profiler.
如果这些简短信息对你还不够，可以单击token链接（那个由13个数字组成的字符串）来访问完整的web profiler。

.. note::

    If the token is not clickable, it means that the profiler routes are not
    registered (see below for configuration information).
    如果那个token不能被点击，这表示profiler的路径没有被注册（参见以下章节）。

Analyzing Profiling data with the Web Profiler
根据web profiler来解析profiling数据
..............................................

The Web Profiler is a visualization tool for profiling data that you can use
in development to debug your code and enhance performance; but it can also be
used to explore problems that occur in production. It exposes all information
collected by the profiler in a web interface.
web profiler是一个用于显示数据的可视化工具，你可以在开发环境中使用它，但你也可以在生成环境中使用它。

.. index::
   single: Profiler; Using the profiler service

Accessing the Profiling information
访问profiling信息
...................................

You don't need to use the default visualizer to access the profiling
information. But how can you retrieve profiling information for a specific
request after the fact? When the profiler stores data about a Request, it also
associates a token with it; this token is available in the ``X-Debug-Token``
HTTP header of the Response::
你不必要使用默认的可视化工具条来访问profiling数据。但你如何在返回请求的响应后获取profiling信息呢？
当profiler存储关于一个请求的信息时，它还与一个token联系起来了；这个token在响应的``X-Debug-Token`` HTTP头部中::

    $profile = $container->get('profiler')->loadProfileFromResponse($response);

    $profile = $container->get('profiler')->loadProfile($token);

.. tip::

    When the profiler is enabled but not the web debug toolbar, or when you
    want to get the token for an Ajax request, use a tool like Firebug to get
    the value of the ``X-Debug-Token`` HTTP header.
    当激活了profiler而没有激活web调试工具条的时候，或你想要获取一个ajax请求的token，你可以使用Firebug那样
    的工具来获取``X-Debug-Token`` HTTP头部的值。

Use the ``find()`` method to access tokens based on some criteria::
使用find()方法根据一些参数来访问token::

    // get the latest 10 tokens
    $tokens = $container->get('profiler')->find('', '', 10);

    // get the latest 10 tokens for all URL containing /admin/
    $tokens = $container->get('profiler')->find('', '/admin/', 10);

    // get the latest 10 tokens for local requests
    $tokens = $container->get('profiler')->find('127.0.0.1', '', 10);

If you want to manipulate profiling data on a different machine than the one
where the information were generated, use the ``export()`` and ``import()``
methods::
如果你想要在一个不同的机器上操纵profiling数据，可以使用export()和import()方法::

    // on the production machine
    $profile = $container->get('profiler')->loadProfile($token);
    $data = $profiler->export($profile);

    // on the development machine
    $profiler->import($data);

.. index::
   single: Profiler; Visualizing

Configuration
配置
.............

The default Symfony2 configuration comes with sensible settings for the
profiler, the web debug toolbar, and the web profiler. Here is for instance
the configuration for the development environment:
symfony2对于web调试工具条、profiler和web profiler都有着合理的默认配置。以下是开发环境下配置的范例:

.. configuration-block::

    .. code-block:: yaml

        # load the profiler
        framework:
            profiler: { only_exceptions: false }

        # enable the web profiler
        web_profiler:
            toolbar: true
            intercept_redirects: true
            verbose: true

    .. code-block:: xml

        <!-- xmlns:webprofiler="http://symfony.com/schema/dic/webprofiler" -->
        <!-- xsi:schemaLocation="http://symfony.com/schema/dic/webprofiler http://symfony.com/schema/dic/webprofiler/webprofiler-1.0.xsd"> -->

        <!-- load the profiler -->
        <framework:config>
            <framework:profiler only-exceptions="false" />
        </framework:config>

        <!-- enable the web profiler -->
        <webprofiler:config
            toolbar="true"
            intercept-redirects="true"
            verbose="true"
        />

    .. code-block:: php

        // load the profiler
        $container->loadFromExtension('framework', array(
            'profiler' => array('only-exceptions' => false),
        ));

        // enable the web profiler
        $container->loadFromExtension('web_profiler', array(
            'toolbar' => true,
            'intercept-redirects' => true,
            'verbose' => true,
        ));

When ``only-exceptions`` is set to ``true``, the profiler only collects data
when an exception is thrown by the application.
当only-exception被设置为true时，profiler会仅在错误被抛出时才收集数据。

When ``intercept-redirects`` is set to ``true``, the web profiler intercepts
the redirects and gives you the opportunity to look at the collected data
before following the redirect.
当intercept-redirects被设置为true时，web profiler会截取重定向并给你机会在重定向之前查看收集的数据。

When ``verbose`` is set to ``true``, the Web Debug Toolbar displays a lot of
information. Setting ``verbose`` to ``false`` hides some secondary information
to make the toolbar shorter.
当verbose被设置为true的时候，web调试工具条会展示许多信息。若将它设置为false会隐藏许多次要信息，工具条也会更短。

If you enable the web profiler, you also need to mount the profiler routes:
如果你激活了web profiler，你还要指定profiler的路径:

.. configuration-block::

    .. code-block:: yaml

        _profiler:
            resource: @WebProfilerBundle/Resources/config/routing/profiler.xml
            prefix:   /_profiler

    .. code-block:: xml

        <import resource="@WebProfilerBundle/Resources/config/routing/profiler.xml" prefix="/_profiler" />

    .. code-block:: php

        $collection->addCollection($loader->import("@WebProfilerBundle/Resources/config/routing/profiler.xml"), '/_profiler');

As the profiler adds some overhead, you might want to enable it only under
certain circumstances in the production environment. The ``only-exceptions``
settings limits profiling to 500 pages, but what if you want to get
information when the client IP comes from a specific address, or for a limited
portion of the website? You can use a request matcher:
由于profiler会增加负载，当处于生成环境中时，你也许想要仅在某些情况下激活它。only-exceptions设置
将profiling限定在500个页面以内，但如果你需要在客户端的IP从某些特殊地址传来时获取信息，或仅对网站的一部分
使用profiler怎么办呢？你可以使用一个请求匹配器（metcher）:

.. configuration-block::

    .. code-block:: yaml

        # enables the profiler only for request coming for the 192.168.0.0 network
        framework:
            profiler:
                matcher: { ip: 192.168.0.0/24 }

        # enables the profiler only for the /admin URLs
        framework:
            profiler:
                matcher: { path: "^/admin/" }

        # combine rules
        framework:
            profiler:
                matcher: { ip: 192.168.0.0/24, path: "^/admin/" }

        # use a custom matcher instance defined in the "custom_matcher" service
        framework:
            profiler:
                matcher: { service: custom_matcher }

    .. code-block:: xml

        <!-- enables the profiler only for request coming for the 192.168.0.0 network -->
        <framework:config>
            <framework:profiler>
                <framework:matcher ip="192.168.0.0/24" />
            </framework:profiler>
        </framework:config>

        <!-- enables the profiler only for the /admin URLs -->
        <framework:config>
            <framework:profiler>
                <framework:matcher path="^/admin/" />
            </framework:profiler>
        </framework:config>

        <!-- combine rules -->
        <framework:config>
            <framework:profiler>
                <framework:matcher ip="192.168.0.0/24" path="^/admin/" />
            </framework:profiler>
        </framework:config>

        <!-- use a custom matcher instance defined in the "custom_matcher" service -->
        <framework:config>
            <framework:profiler>
                <framework:matcher service="custom_matcher" />
            </framework:profiler>
        </framework:config>

    .. code-block:: php

        // enables the profiler only for request coming for the 192.168.0.0 network
        $container->loadFromExtension('framework', array(
            'profiler' => array(
                'matcher' => array('ip' => '192.168.0.0/24'),
            ),
        ));

        // enables the profiler only for the /admin URLs
        $container->loadFromExtension('framework', array(
            'profiler' => array(
                'matcher' => array('path' => '^/admin/'),
            ),
        ));

        // combine rules
        $container->loadFromExtension('framework', array(
            'profiler' => array(
                'matcher' => array('ip' => '192.168.0.0/24', 'path' => '^/admin/'),
            ),
        ));

        # use a custom matcher instance defined in the "custom_matcher" service
        $container->loadFromExtension('framework', array(
            'profiler' => array(
                'matcher' => array('service' => 'custom_matcher'),
            ),
        ));

Learn more from the Cookbook
----------------------------

* :doc:`/cookbook/testing/profiling`
* :doc:`/cookbook/profiler/data_collector`
* :doc:`/cookbook/event_dispatcher/class_extension`
* :doc:`/cookbook/event_dispatcher/method_behavior`

.. _`Symfony2 Dependency Injection component`: https://github.com/symfony/DependencyInjection
