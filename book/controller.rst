.. index::
   single: Controller

Controller
控制器
==========

A controller is a PHP function you create that takes information from the
HTTP request and constructs and returns an HTTP response (as a Symfony2
``Response`` object). The response could be an HTML page, an XML document,
a serialized JSON array, an image, a redirect, a 404 error or anything else
you can dream up. The controller contains whatever arbitrary logic *your
application* needs to render the content of a page.
控制器是你创建的一个PHP方法，它能够从HTTP请求中提取信息、构建并返回一个
HTTP response（作为一个Symfony2``Response``对象）。这个response可以是一个HTML页面，
一个XML文档，一个序列化的JSON数组，一个图像，一个redirect，一个404错误或者其他
任何你能想象到的东西。这个控制器包含任意你的应用要提交页面所需要的逻辑。

To see how simple this is, let's look at a Symfony2 controller in action.
The following controller would render a page that simply prints ``Hello world!``::
要知道这有多容易，让我们看看symfony2的控制器应用。以下控制器会提交一个页面，这个页面
能输出``Hello world``::

    use Symfony\Component\HttpFoundation\Response;

    public function helloAction()
    {
        return new Response('Hello world!');
    }

The goal of a controller is always the same: create and return a ``Response``
object. Along the way, it might read information from the request, load a
database resource, send an email, or set information on the user's session.
But in all cases, the controller will eventually return the ``Response`` object
that will be delivered back to the client.
控制器的目的总是一样的：创建并返回一个``Response``对象。它可以读取请求中的信息，
加载数据库的源文件，发送email，或者在用户的session上设置信息。但是在所有这些情况
中，这个控制器最终都会返回``Response``对象，这个对象会被发送回客户端。

There's no magic and no other requirements to worry about! Here are a few
common examples:
这里没有魔术，也不需要顾虑其他的要求！下面是一些常见的例子：

* *Controller A* prepares a ``Response`` object representing the content
  for the homepage of the site.
  *控制器A*准备一个可以表示主页内容的``Response``对象。

* *Controller B* reads the ``slug`` parameter from the request to load a
  blog entry from the database and create a ``Response`` object displaying
  that blog. If the ``slug`` can't be found in the database, it creates and
  returns a ``Response`` object with a 404 status code.
  *控制器B* 读取请求中的``slug``参数，从而加载博客入口，并创建一个可以显示
  博客的``Response``对象。如果这个``slug``不能再数据库中找到，它就会创建并返回一个
  带有404 status 代码的``Response``对象。

* *Controller C* handles the form submission of a contact form. It reads
  the form information from the request, saves the contact information to
  the database and emails the contact information to the webmaster. Finally,
  it creates a ``Response`` object that redirects the client's browser to
  the contact form "thank you" page.
  *控制器C* 操作一个contact表单的提交。它从请求中读取表单的信息，将contact信息提交到
  数据库并将contact信息通过邮件发送给网站管理员。最后，它创建一个可以将用户浏览器
  重定向到contact表单的"thank you"页面的``Response``对象。

.. index::
   single: Controller; Request-controller-response lifecycle

Requests, Controller, Response Lifecycle
请求，控制器，响应流程
----------------------------------------

Every request handled by a Symfony2 project goes through the same simple lifecycle.
The framework takes care of the repetitive tasks and ultimately executes a
controller, which houses your custom application code:
每个Symfony2处理的请求都会经过同样简单的流程。本框架能应付这些重复的工作并最终执行
一个你定制的控制器：

#. Each request is handled by a single front controller file (e.g. ``app.php``
   or ``app_dev.php``) that bootstraps the application;
   每个请求都由一个单独的前端控制器处理（比如``app.php``或者``app_dev.php``）,它会
   引导这个应用;

#. The ``Router`` reads information from the request (e.g. the URI), finds
   a route that matches that information, and reads the ``_controller`` parameter
   from the route;
   ``路由``从请求（比如URI）中读取信息，查询与该信息匹配的路径，并从该路径中读取``_controller``
   参数;

#. The controller from the matched route is executed and the code inside the
   controller creates and returns a ``Response`` object;
   从这个匹配的路径导入的控制器被执行，并且控制器内部的代码创建并返回``Response``对象;

#. The HTTP headers and content of the ``Response`` object are sent back to
   the client.
   HTTP headers和``Response``的内容都被返回给客户端。

Creating a page is as easy as creating a controller (#3) and making a route that
maps a URL to that controller (#2).
创建一个页面就是创建一个控制器，并创建一个可以将一个URL映射到那个控制器的路径。

.. note::

    Though similarly named, a "front controller" is different from the
    "controllers" we'll talk about in this chapter. A front controller
    is a short PHP file that lives in your web directory and through which
    all requests are directed. A typical application will have a production
    front controller (e.g. ``app.php``) and a development front controller
    (e.g. ``app_dev.php``). You'll likely never need to edit, view or worry
    about the front controllers in your application.
    虽然名称相似，但“前端控制器”是不同于我们所讨论的“控制器”的。一个前端控制器是
    一个简短的PHP文件，它在你的web目录下，通过它所有的请求都被定向。一个典型应用会
    有一个生产前端控制器（比如``app.php``）和一个开发前端控制器（比如``app_dev.php``）。
    你可能永远不会编辑、查看或担心这个前端控制器。

.. index::
   single: Controller; Simple example

A Simple Controller
一个简单的控制器
-------------------

While a controller can be any PHP callable (a function, method on an object,
or a ``Closure``), in Symfony2, a controller is usually a single method inside
a controller object. Controllers are also called *actions*.
一个控制器可以是任何能够被执行的PHP代码段（一个方法，对象中的方法，或者一个``Closure``）,
但在symfony2中，一个控制器通常是一个控制器对象中单独的方法。控制器往往被称为*actions*。

.. code-block:: php
    :linenos:

    // src/Acme/HelloBundle/Controller/HelloController.php

    namespace Acme\HelloBundle\Controller;
    use Symfony\Component\HttpFoundation\Response;

    class HelloController
    {
        public function indexAction($name)
        {
          return new Response('<html><body>Hello '.$name.'!</body></html>');
        }
    }

.. tip::

    Note that the *controller* is the ``indexAction`` method, which lives
    inside a *controller class* (``HelloController``). Don't be confused
    by the naming: a *controller class* is simply a convenient way to group
    several controllers/actions together. Typically, the controller class
    will house several controllers/actions (e.g. ``updateAction``, ``deleteAction``,
    etc).
    注意这个控制器是在*控制器类*中的``indexAction``方法（``HelloController``）。
    不要因为名称而搞混：一个*控制器对象*只是一种将数个控制器集合起来的便捷方式。
    一般的，这个控制器对象会拥有好几个控制器（如``updateAction``,``deleteAction``等等）。

This controller is pretty straightforward, but let's walk through it:
这个控制器非常直观，我们来分析一下：

* *line 3*: Symfony2 takes advantage of PHP 5.3 namespace functionality to
  namespace the entire controller class. The ``use`` keyword imports the
  ``Response`` class, which our controller must return.
  *第3行*：symfony2利用了PHP5.3中的namespace功能来给整个控制器添加命名空间。这个``use``
  关键字导入了我们的控制器必须返回的``Response``类。

* *line 6*: The class name is the concatenation of a name for the controller
  class (i.e. ``Hello``) and the word ``Controller``. This is a convention
  that provides consistency to controllers and allows them to be referenced
  only by the first part of the name (i.e. ``Hello``) in the routing configuration.
  *第6行*：这个类名是控制器类名（如``Hello``）和单词``controller``的结合体。这是一个
  惯例，它可以保证控制器的一致性并且保证它们可以在路径配置中通过名称的第一部分（如``Hello``）
  被访问。

* *line 8*: Each action in a controller class is suffixed with ``Action``
  and is referenced in the routing configuration by the action's name (``index``).
  In the next section, you'll create a route that maps a URI to this action.
  You'll learn how the route's placeholders (``{name}``) become arguments
  to the action method (``$name``).
  *第8行*：每个控制器类中的控制器都有``Action``后缀，并通过控制器的名字（``index``）
  在路径配置中被访问。在下一节，你会创建一个将URI映射到这个控制器的路径。你将学习该路径的
  placeholders（``{name}``）是如何成为控制器方法（``$name``）的参数的。

* *line 10*: The controller creates and returns a ``Response`` object.
  *第10行*：控制器创建并返回一个``Response``对象。


.. index::
   single: Controller; Routes and controllers

Mapping a URL to a Controller
将一个URL映射到控制器
-----------------------------

The new controller returns a simple HTML page. To actually view this page
in your browser, you need to create a route, which maps a specific URL pattern
to the controller:
这个新的控制器返回一个简单的HTML页面。要在你的浏览器中查看这个页面，你还必须创建一个路径，
这个路径要将一个特定的URL pattern映射到这个控制器。

.. configuration-block::

    .. code-block:: yaml

        # app/config/routing.yml
        hello:
            pattern:      /hello/{name}
            defaults:     { _controller: AcmeHelloBundle:Hello:index }

    .. code-block:: xml

        <!-- app/config/routing.xml -->
        <route id="hello" pattern="/hello/{name}">
            <default key="_controller">AcmeHelloBundle:Hello:index</default>
        </route>

    .. code-block:: php

        // app/config/routing.php
        $collection->add('hello', new Route('/hello/{name}', array(
            '_controller' => 'AcmeHelloBundle:Hello:index',
        )));

Going to ``/hello/ryan`` now executes the ``HelloController::indexAction()``
controller and passes in ``ryan`` for the ``$name`` variable. Creating a
"page" means simply creating a controller method and associated route.
输入``/hello/ryan``会执行这个``HelloController::indexAction()``控制器并且将``ryan``
作为``$name``变量返回。创建一个页面仅仅意味着创建一个控制器方法以及一个相关路径。

Notice the syntax used to refer to the controller: ``AcmeHelloBundle:Hello:index``.
Symfony2 uses a flexible string notation to refer to different controllers.
This is the most common syntax and tells Symfony2 to look for a controller
class called ``HelloController`` inside a bundle named ``AcmeHelloBundle``. The
method ``indexAction()`` is then executed.
注意到这个访问控制器``AcmeHelloBundle:Helo:index``的语法。symfony2使用一个灵活的语法
访问不同的控制器。以上是一个最常用的语法，它告诉symfony2要查找一个``AcmeHelloBundle``
中的``HelloController``控制器。这个``indexAction()``方法最终被执行。

For more details on the string format used to reference different controllers,
see :ref:`controller-string-syntax`.
要了解更多的关于这种访问不同控制器的语法格式，请参阅:ref:`controller-string-syntax`。

.. note::

    This example places the routing configuration directly in the ``app/config/``
    directory. A better way to organize your routes is to place each route
    in the bundle it belongs to. For more information on this, see
    :ref:`routing-include-external-resources`.
    这个例子将路径配置直接放在``app/config``目录下。一个更好的组织你的路径的方法是将每个
    路径都放在它所属的bundle中。要了解更多相关信息，请参阅:ref:`routing-include-external-resources`。

.. tip::

    You can learn much more about the routing system in the :doc:`Routing chapter</book/routing>`.
    你可以从这个文档中学习更多有关路由系统的知识:doc:`Routing chapter</book/routing>`。

.. index::
   single: Controller; Controller arguments

.. _route-parameters-controller-arguments:

Route Parameters as Controller Arguments
将路径参数作为控制器参数
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

You already know that the ``_controller`` parameter ``AcmeHelloBundle:Hello:index``
refers to a ``HelloController::indexAction()`` method that lives inside the
``AcmeHelloBundle`` bundle. What's more interesting is the arguments that are
passed to that method:
你现在已经知道，这``_controller``参数``AcmeHelloBundle:Hello:index``会访问一个``AcmeHelloBundle``
中的``HelloController::indexAction()``方法。更有趣的是这个能够被传送到呢个方法的参数：

.. code-block:: php

    <?php
    // src/Acme/HelloBundle/Controller/HelloController.php

    namespace Acme\HelloBundle\Controller;
    use Symfony\Bundle\FrameworkBundle\Controller\Controller;

    class HelloController extends Controller
    {
        public function indexAction($name)
        {
          // ...
        }
    }

The controller has a single argument, ``$name``, which corresponds to the
``{name}`` parameter from the matched route (``ryan`` in our example). In
fact, when executing your controller, Symfony2 matches each argument of
the controller with a parameter from the matched route. Take the following
example:
这个控制器有一个单独的参数，``$name``，这个参数是相对应于其匹配的路径（在我们的
例子中是``ryan``）中的参数``{name}``的。实际上，当执行这个控制器时，symfony2会匹配
控制器和路径中的每个参数。比如：

.. configuration-block::

    .. code-block:: yaml

        # app/config/routing.yml
        hello:
            pattern:      /hello/{first_name}/{last_name}
            defaults:     { _controller: AcmeHelloBundle:Hello:index, color: green }

    .. code-block:: xml

        <!-- app/config/routing.xml -->
        <route id="hello" pattern="/hello/{first_name}/{last_name}">
            <default key="_controller">AcmeHelloBundle:Hello:index</default>
            <default key="color">green</default>
        </route>

    .. code-block:: php

        // app/config/routing.php
        $collection->add('hello', new Route('/hello/{first_name}/{last_name}', array(
            '_controller' => 'AcmeHelloBundle:Hello:index',
            'color'       => 'green',
        )));

The controller for this can take several arguments::
这个控制器会有数个参数::

    public function indexAction($first_name, $last_name, $color)
    {
        // ...
    }

Notice that both placeholder variables (``{first_name}``, ``{last_name}``)
as well as the default ``color`` variable are available as arguments in the
controller. When a route is matched, the placeholder variables are merged
with the ``defaults`` to make one array that's available to your controller.
注意每个placeholder变量（``{first_name}``,``{last_name}``）以及这个默认的变量
``color``都会被作为控制器中的参数。当一个路径被匹配时，这些placeholder变量会与``defaults``
变量合并，形成一个数组，这个数组能够在你的控制器中被使用。

Mapping route parameters to controller arguments is easy and flexible. Keep
the following guidelines in mind while you develop.
将路径参数映射到控制器参数非常容易。当你编写代码时，请记住以下几点。

* **The order of the controller arguments does not matter**
* **控制器参数的次序无关紧要**

    Symfony is able to match the parameter names from the route to the variable
    names in the controller method's signature. In other words, it realizes that
    the ``{last_name}`` parameter matches up with the ``$last_name`` argument.
    The arguments of the controller could be totally reordered and still work
    perfectly::
    symfony能够将路径的参数和控制器方法中的参数变量匹配起来。换句话说，它能够知道
    参数``{last_name}``能够与变量``$last_name``匹配。这些控制器中的变量可以被任意打乱次序
    并且正常工作::

        public function indexAction($last_name, $color, $first_name)
        {
            // ..
        }

* **Each required controller argument must match up with a routing parameter**
  **每个控制器中的参数都必须匹配路径中的参数**

    The following would throw a ``RuntimeException`` because there is no ``foo``
    parameter defined in the route::
    以下代码会抛出``RuntimeException``错误，因为在路径中没有定义``foo``参数::

        public function indexAction($first_name, $last_name, $color, $foo)
        {
            // ..
        }

    Making the argument optional, however, is perfectly ok. The following
    example would not throw an exception::
    但可以使这个参数成为可供选择的。以下例子不会抛出错误::

        public function indexAction($first_name, $last_name, $color, $foo = 'bar')
        {
            // ..
        }

* **Not all routing parameters need to be arguments on your controller**
  **不是每个路径参数都必须被作为控制器的参数**

    If, for example, the ``last_name`` weren't important for your controller,
    you could omit it entirely::
    比如，如果``last_name``对于你的控制器不那么重要，你可以忽略它::

        public function indexAction($first_name, $color)
        {
            // ..
        }

.. tip::

    Every route also has a special ``_route`` parameter, which is equal to
    the name of the route that was matched (e.g. ``hello``). Though not usually
    useful, this is equally available as a controller argument.
    每个路径都有一个特定的``_route``参数，这个参数等同于被匹配的路径名（如``hello``）。
    尽管不是经常用到，它同样能被作为控制器参数。
    

.. _book-controller-request-argument:

The ``Request`` as a Controller Argument
将``Request``作为控制器参数
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

For convenience, you can also have Symfony pass you the ``Request`` object
as an argument to your controller. This is especially convenient when you're
working with forms, for example::
方便起见，你也可以让symfony将``Request``对象作为参数传递给你的控制器。当你需要创建
表单的时候这样做尤其便利::


    use Symfony\Component\HttpFoundation\Request;

    public function updateAction(Request $request)
    {
        $form = $this->createForm(...);

        $form->bindRequest($request);
        // ...
    }

.. index::
   single: Controller; Base controller class

The Base Controller Class
基本的控制器类
-------------------------

For convenience, Symfony2 comes with a base ``Controller`` class that assists
with some of the most common controller tasks and gives your controller class
access to any resource it might need. By extending this ``Controller`` class,
you can take advantage of several helper methods.
简便起见，symfony2有一个基本的``控制器``类，它能帮助你做一些常用的控制器工作，并
使你的控制器类能够进入任何它所需要的资源。只要扩展这个``控制器``类，你就能利用一些
helper方法了。

Add the ``use`` statement atop the ``Controller`` class and then modify the
``HelloController`` to extend it:
在``控制器``类的上部加上``use``语句，并扩展``HelloController``:

.. code-block:: php

    // src/Acme/HelloBundle/Controller/HelloController.php

    namespace Acme\HelloBundle\Controller;
    use Symfony\Bundle\FrameworkBundle\Controller\Controller;
    use Symfony\Component\HttpFoundation\Response;

    class HelloController extends Controller
    {
        public function indexAction($name)
        {
          return new Response('<html><body>Hello '.$name.'!</body></html>');
        }
    }

This doesn't actually change anything about how your controller works. In
the next section, you'll learn about the helper methods that the base controller
class makes available. These methods are just shortcuts to using core Symfony2
functionality that's available to you with or without the use of the base
``Controller`` class. A great way to see the core functionality in action
is to look in the
这并不能改变你的控制器运作方式。在下一节中，你将学习基本类提供的helper方法。
这些方法都是运用symfony2核心功能的简易方法。一个了解这些核心功能如何运作的好办法是
直接查看
:class:`Symfony\\Bundle\\FrameworkBundle\\Controller\\Controller` 类。


.. tip::

    Extending the base class is *optional* in Symfony; it contains useful
    shortcuts but nothing mandatory. You can also extend
    ``Symfony\Component\DependencyInjection\ContainerAware``. The service
    container object will then be accessible via the ``container`` property.
    是否要扩展基本类在symfony中是可选的;它包含了一系列简易方法但并不强制。你也可以
    扩展``Symfony\Component\DependencyInjection\ContainerAware``。这个service container
    类就可以通过``container``属性进入了。

.. note::

    You can also define your :doc:`Controllers as Services
    </cookbook/controller/service>`.
    你也可以定义你的:doc:`Controllers as Services
    </cookbook/controller/service>`。

.. index::
   single: Controller; Common Tasks

Common Controller Tasks
常见的控制器任务
-----------------------

Though a controller can do virtually anything, most controllers will perform
the same basic tasks over and over again. These tasks, such as redirecting,
forwarding, rendering templates and accessing core services, are very easy
to manage in Symfony2.
虽然一个控制器可以做几乎任何工作，但大多数控制器都会重复做一些相同的工作。这些工作，
如重定向，转发，提交模板，使用核心服务，等等，在symfony2中都非常容易。

.. index::
   single: Controller; Redirecting

Redirecting
重定向
~~~~~~~~~~~

If you want to redirect the user to another page, use the ``redirect()`` method::
如果你想讲用户重定向到另一个页面，可以使用``redirect()``方法::

    public function indexAction()
    {
        return $this->redirect($this->generateUrl('homepage'));
    }

The ``generateUrl()`` method is just a helper function that generates the URL
for a given route. For more information, see the :doc:`Routing </book/routing>`
chapter.
``generateUrl()``方法只是一个helper方法，它可以用来集成一个给的路径的URL。要了解更多，
请参阅:doc:`Routing </book/routing>`。

By default, the ``redirect()`` method performs a 302 (temporary) redirect. To
perform a 301 (permanent) redirect, modify the second argument::
默认情况下，``redirect()``方法执行了一个302（temporary）重定向。要执行一个301（permanent）
重定向，改变第二个参数::

    public function indexAction()
    {
        return $this->redirect($this->generateUrl('homepage'), 301);
    }

.. tip::

    The ``redirect()`` method is simply a shortcut that creates a ``Response``
    object that specializes in redirecting the user. It's equivalent to:
    ``redirect()``方法只是一个创建能够对用户重定向的``Response``对象的简易方法。它等同于:

    .. code-block:: php

        use Symfony\Component\HttpFoundation\RedirectResponse;

        return new RedirectResponse($this->generateUrl('homepage'));

.. index::
   single: Controller; Forwarding

Forwarding
转发
~~~~~~~~~~

You can also easily forward to another controller internally with the ``forward()``
method. Instead of redirecting the user's browser, it makes an internal sub-request,
and calls the specified controller. The ``forward()`` method returns the ``Response``
object that's returned from that controller::
你也可以轻松地使用``forward()``方法在内部转发到另一个控制器。它发送了一个内部子请求，
执行了一个特定的控制器，而不是对用户浏览器重定向。``forward()``方法从那个控制器返回了``Response``对象。

    public function indexAction($name)
    {
        $response = $this->forward('AcmeHelloBundle:Hello:fancy', array(
            'name'  => $name,
            'color' => 'green'
        ));

        // further modify the response or return it directly

        return $response;
    }

Notice that the `forward()` method uses the same string representation of
the controller used in the routing configuration. In this case, the target
controller class will be ``HelloController`` inside some ``AcmeHelloBundle``.
The array passed to the method becomes the arguments on the resulting controller.
This same interface is used when embedding controllers into templates (see
:ref:`templating-embedding-controller`). The target controller method should
look something like the following::
注意这个`forward()`方法使用了与路径配置中相同的语法来指定控制器。在这个例子中，目标
控制器类就是``AcmeHelloBundle``中的``HelloController``。这个被传送的数组就成为
了目标控制器上的参数。同样的接口也可用于将控制器植入面板（参阅:ref:`templating-embedding-controller`）。
这个目标控制器应该像下面这样::

    public function fancyAction($name, $color)
    {
        // ... create and return a Response object
    }

And just like when creating a controller for a route, the order of the arguments
to ``fancyAction`` doesn't matter. Symfony2 matches the index key names
(e.g. ``name``) with the method argument names (e.g. ``$name``). If you
change the order of the arguments, Symfony2 will still pass the correct
value to each variable.
就像为一个路径创建一个控制器一样，传到``fancyAction``参数的次序无关紧要。symfony2将索引键
的名称（如``name``）与方法参数名称（如``$name``）相匹配。如果你改变参数的次序，symfony2仍会
将正确的值传送给每个变量。

.. tip::

    Like other base ``Controller`` methods, the ``forward`` method is just
    a shortcut for core Symfony2 functionality. A forward can be accomplished
    directly via the ``http_kernel`` service. A forward returns a ``Response``
    object::
    像其他基本控制器方法一样，``forward``方法只是运用symfony2核心方法的一个捷径。
    一个转发可以直接通过``http_Kernel``服务来完成。forwrd返回一个``Response``对象::

        $httpKernel = $this->container->get('http_kernel');
        $response = $httpKernel->forward('AcmeHelloBundle:Hello:fancy', array(
            'name'  => $name,
            'color' => 'green',
        ));

.. index::
   single: Controller; Rendering templates

.. _controller-rendering-templates:

Rendering Templates
提交模板
~~~~~~~~~~~~~~~~~~~

Though not a requirement, most controllers will ultimately render a template
that's responsible for generating the HTML (or other format) for the controller.
The ``renderView()`` method renders a template and returns its content. The
content from the template can be used to create a ``Response`` object::
大部分控制器最终会提交一个集成HTML（或其他格式）的模板。``renderView()``方法会提交一个
模板并返回它的内容。这些模板的内容可以用来创建一个``Response``对象::

    $content = $this->renderView('AcmeHelloBundle:Hello:index.html.twig', array('name' => $name));

    return new Response($content);

This can even be done in just one step with the ``render()`` method, which
returns a ``Response`` object containing the content from the template::
也可以只使用一个``render()``方法，它能够从模板返回一个``Response``对象::

    return $this->render('AcmeHelloBundle:Hello:index.html.twig', array('name' => $name));

In both cases, the ``Resources/views/Hello/index.html.twig`` template inside
the ``AcmeHelloBundle`` will be rendered.
大多数情况下，``AcmeHelloBundle``中的模板``Resources/views/Hello/index.html.twig``
会被提交。

The Symfony templating engine is explained in great detail in the
:doc:`Templating </book/templating>` chapter.
symfony模板引擎在文档中有详细描述:doc:`Templating </book/templating>`

.. tip::

    The ``renderView`` method is a shortcut to direct use of the ``templating``
    service. The ``templating`` service can also be used directly::
    ``renderView``方法是一个直接运用templating服务的捷径。templating服务也可以被直接运用::

        $templating = $this->get('templating');
        $content = $templating->render('AcmeHelloBundle:Hello:index.html.twig', array('name' => $name));

.. index::
   single: Controller; Accessing services

Accessing other Services
进入其他服务
~~~~~~~~~~~~~~~~~~~~~~~~

When extending the base controller class, you can access any Symfony2 service
via the ``get()`` method. Here are several common services you might need::
当扩展基本控制器类之后，你就可以通过``get()``方法使用任何Symfony2服务了。以下是一些通常
需要的服务::

    $request = $this->getRequest();

    $templating = $this->get('templating');

    $router = $this->get('router');

    $mailer = $this->get('mailer');

There are countless other services available and you are encouraged to define
your own. To list all available services, use the ``container:debug`` console
command:
还有许多其他的服务，你还可以定制你自己的服务。要列出所有的服务，使用console command
``container:debug``:

.. code-block:: bash

    php app/console container:debug

For more information, see the :doc:`/book/service_container` chapter.
详情请见:doc:`/book/service_container`。

.. index::
   single: Controller; Managing errors
   single: Controller; 404 pages

Managing Errors and 404 Pages
管理错误和404页面
-----------------------------

When things are not found, you should play well with the HTTP protocol and
return a 404 response. To do this, you'll throw a special type of exception.
If you're extending the base controller class, do the following::
当内容没有被找到，你必须管理好你的404响应。为了达到这个目的，你需要抛出一个特定
的错误类型。如果你扩展了基本类，就可以像下面这样:

    public function indexAction()
    {
        $product = // retrieve the object from database
        if (!$product) {
            throw $this->createNotFoundException('The product does not exist');
        }

        return $this->render(...);
    }

The ``createNotFoundException()`` method creates a special ``NotFoundHttpException``
object, which ultimately triggers a 404 HTTP response inside Symfony.
``createNotFoundException()``方法创建了一个特定的``NotFoundHttpException``对象，这个对象
最终会激发一个404 HTTP响应。

Of course, you're free to throw any ``Exception`` class in your controller -
Symfony2 will automatically return a 500 HTTP response code.
当然，你也可以在控制器中抛出任何``Exception``类，symfony2会自动返回一个500 HTTP响应。

.. code-block:: php

    throw new \Exception('Something went wrong!');

In every case, a styled error page is shown to the end user and a full debug
error page is shown to the developer (when viewing the page in debug mode).
Both of these error pages can be customized. For details, read the
":doc:`/cookbook/controller/error_pages`" cookbook recipe.
在所有情况中，一个特定的错误页面都会在客户端展示，而一个开发错误页面会展示给开发者
（当这个页面是在开发模式下）。这两种页面都可以被自定义。详情请见":doc:`/cookbook/controller/error_pages`"。

.. index::
   single: Controller; The session
   single: Session

Managing the Session
管理session
--------------------

Symfony2 provides a nice session object that you can use to store information
about the user (be it a real person using a browser, a bot, or a web service)
between requests. By default, Symfony2 stores the attributes in a cookie
by using the native PHP sessions.
symfony2提供了一个非常好的session对象，你可以用来存储用户信息（不管是真人，自动程序，
还是一个web服务）。默认情况下，symfony2利用本地PHP sessions来将信息存储至cookie中。

Storing and retrieving information from the session can be easily achieved
from any controller::
可以方便地利用任何控制器达到存储和获取session的目的::

    $session = $this->getRequest()->getSession();

    // store an attribute for reuse during a later user request
    $session->set('foo', 'bar');

    // in another controller for another request
    $foo = $session->get('foo');

    // use a default value of the key doesn't exist
    $filters = $session->set('filters', array());

These attributes will remain on the user for the remainder of that user's
session.
这些信息会被保留在用户session中。

.. index::
   single Session; Flash messages

Flash Messages
flash信息
~~~~~~~~~~~~~~

You can also store small messages that will be stored on the user's session
for exactly one additional request. This is useful when processing a form:
you want to redirect and have a special message shown on the *next* request.
These types of messages are called "flash" messages.
你还可以为某一个（仅仅是一个）额外的请求存放一些小的信息在用户的session中。尤其当运行表单
的时候:你想要重定向并发送一个特定的信息显示在*下一个*请求中。这种类型的信息就叫做falsh信息。

For example, imagine you're processing a form submit::
比如，当你在发送一个表单的时候::

    public function updateAction()
    {
        $form = $this->createForm(...);

        $form->bindRequest($this->getRequest());
        if ($form->isValid()) {
            // do some sort of processing

            $this->get('session')->getFlashBag()->add('notice', 'Your changes were saved!');

            return $this->redirect($this->generateUrl(...));
        }

        return $this->render(...);
    }

After processing the request, the controller sets a ``notice`` flash message
and then redirects. The name (``notice``) isn't significant - it's just what
you're using to identify the type of the message.
在运行这个请求后，这个控制器会设置一个``notice``flash信息并重定向。名称（``notice``）
并不重要，它仅仅是用来定义这个信息的类型罢了。

In the template of the next action, the following code could be used to render
the ``notice`` message:
在下一个控制器的模板中，以下的代码会用来提交一个``notice``信息:

.. configuration-block::

    .. code-block:: html+jinja

        {% for flashMessage in app.session.flashbag.get('notice') %}
            <div class="flash-notice">
                {{ flashMessage }}
            </div>
        {% endfor %}

    .. code-block:: php

        <?php foreach ($view['session']->getFlashBag()->get('notice') as $message): ?>
            <div class="flash-notice">
                <?php echo "<div class='flash-error'>$message</div>" ?>
            </div>
        <?php endforeach; ?>

By design, flash messages are meant to live for exactly one request (they're
"gone in a flash"). They're designed to be used across redirects exactly as
you've done in this example.
flash信息仅仅可以再一个请求中使用（它们一瞬间就会不见）。它们只在重定向的时候使用，像上例一样。

.. index::
   single: Controller; Response object

The Response Object
Response对象
-------------------

The only requirement for a controller is to return a ``Response`` object. The
:class:`Symfony\\Component\\HttpFoundation\\Response` class is a PHP
abstraction around the HTTP response - the text-based message filled with HTTP
headers and content that's sent back to the client::
对控制器唯一的要求就是要返回一个``Response``对象。:class:`Symfony\\Component\\HttpFoundation\\Response`
类是一个PHP抽象类，它包含了一个会被发送到客户端的已经填充了HTTP headers和内容的HTTP响应。

    // create a simple Response with a 200 status code (the default)
    $response = new Response('Hello '.$name, 200);

    // create a JSON-response with a 200 status code
    $response = new Response(json_encode(array('name' => $name)));
    $response->headers->set('Content-Type', 'application/json');

.. tip::

    The ``headers`` property is a
    :class:`Symfony\\Component\\HttpFoundation\\HeaderBag` object with several
    useful methods for reading and mutating the ``Response`` headers. The
    header names are normalized so that using ``Content-Type`` is equivalent
    to ``content-type`` or even ``content_type``.
    这个``headers``是一个:class:`Symfony\\Component\\HttpFoundation\\HeaderBag`对象，这个对象
    有一些很有用的方法能够读取和编译``Response`` headers。这个header名称都被标准化了，使用
    ``Content-Type``和使用``content-type``，``content_type``都是一样的。

.. index::
   single: Controller; Request object

The Request Object
Request对象
------------------

Besides the values of the routing placeholders, the controller also has access
to the ``Request`` object when extending the base ``Controller`` class::
除了路径placeholder的值，如果控制器已经扩展了基本控制器类，它也可以使用``Request``对象::

    $request = $this->getRequest();

    $request->isXmlHttpRequest(); // is it an Ajax request?

    $request->getPreferredLanguage(array('en', 'fr'));

    $request->query->get('page'); // get a $_GET parameter

    $request->request->get('page'); // get a $_POST parameter

Like the ``Response`` object, the request headers are stored in a ``HeaderBag``
object and are easily accessible.
像``Response``对象一样，这些请求header都存储在一个``HeaderBag``对象中，而且很容易使用。

Final Thoughts
总结
--------------

Whenever you create a page, you'll ultimately need to write some code that
contains the logic for that page. In Symfony, this is called a controller,
and it's a PHP function that can do anything it needs in order to return
the final ``Response`` object that will be returned to the user.
当你在编辑一个页面是，你最终会需要写一些包含那个页面逻辑的代码。在symfony中，
这种代码被称作控制器，它就是一个PHP方法，可以做任何工作来返回一个``Response``
对象，这个对象会被返回给用户。

To make life easier, you can choose to extend a base ``Controller`` class,
which contains shortcut methods for many common controller tasks. For example,
since you don't want to put HTML code in your controller, you can use
the ``render()`` method to render and return the content from a template.
简便起见，你可以选择是否扩展基本控制器类，这个基本类包含了一个简易方法来进行许多常用
工作。例如，如果你不想将HTML代码放在你的控制器中，你可以使用``render()``方法来提交模板。

In other chapters, you'll see how the controller can be used to persist and
fetch objects from a database, process form submissions, handle caching and
more.
在其他章节中，你会学习如何使用控制器来从数据库存入和取出数据，提交表单，管理缓存，等等。

Learn more from the Cookbook
从cookbook中学习更多
----------------------------

* :doc:`/cookbook/controller/error_pages`
* :doc:`/cookbook/controller/service`
