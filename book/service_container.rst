.. index::
   single: Service Container
   single: Dependency Injection Container

Service Container
服务容器
=================

A modern PHP application is full of objects. One object may facilitate the
delivery of email messages while another may allow you to persist information
into a database. In your application, you may create an object that manages
your product inventory, or another object that processes data from a third-party
API. The point is that a modern application does many things and is organized
into many objects that handle each task.
一个php应用会包含许多对象。一个对象可能会用于收发邮件，另一个可能会用于向数据库载入信息。
在你的应用中，你可以创建一个具有某种功能的对象。

In this chapter, we'll talk about a special PHP object in Symfony2 that helps
you instantiate, organize and retrieve the many objects of your application.
This object, called a service container, will allow you to standardize and
centralize the way objects are constructed in your application. The container
makes your life easier, is super fast, and emphasizes an architecture that
promotes reusable and decoupled code. And since all core Symfony2 classes
use the container, you'll learn how to extend, configure and use any object
in Symfony2. In large part, the service container is the biggest contributor
to the speed and extensibility of Symfony2.
在本章中，我们将讲述symfony2中一个特别的php对象，它可以帮助你初始化、组织以及获取应用中的对象。
这个对象就叫做服务容器（service container），它能使应用中创建的对象更组织化。服务容器使代码编写
更加容易，它强调一个代码重用、隔离的结构。由于所有的symfony2类都使用容器，你将学习如何在symfony中扩展、配置并
使用对象。服务容器是symfony2中贡献最大的部分。

Finally, configuring and using the service container is easy. By the end
of this chapter, you'll be comfortable creating your own objects via the
container and customizing objects from any third-party bundle. You'll begin
writing code that is more reusable, testable and decoupled, simply because
the service container makes writing good code so easy.
最后，配置和使用服务容器都非常容易。本章末尾，你将学到如何通过容器创建你自己的对象，以及
修改第三方bundle中的对象。你将学会如何编写重用性、测试性、组织性更好的代码，因为服务容器
将这一切都变得非常简单。

.. index::
   single: Service Container; What is a service?

What is a Service?
什么是服务？
------------------

Put simply, a :term:`Service` is any PHP object that performs some sort of
"global" task. It's a purposefully-generic name used in computer science
to describe an object that's created for a specific purpose (e.g. delivering
emails). Each service is used throughout your application whenever you need
the specific functionality it provides. You don't have to do anything special
to make a service: simply write a PHP class with some code that accomplishes
a specific task. Congratulations, you've just created a service!
服务（:term:`Service`）就是任何具有某种功能的、并可以在全局使用的对象。这个名称在
计算机领域被用来表示一个为了某个特定目的而创建的对象（比如发送邮件）。只要你需要这个服务提供的特定功能，
它都可以在应用的任何地方被使用。

.. note::

    As a rule, a PHP object is a service if it is used globally in your
    application. A single ``Mailer`` service is used globally to send
    email messages whereas the many ``Message`` objects that it delivers
    are *not* services. Similarly, a ``Product`` object is not a service,
    but an object that persists ``Product`` objects to a database *is* a service.
    作为一个规则，如果一个php对象能够在你的应用中的任何地方被使用，它就是一个服务。
    一个Mailer服务可以在任何地方被用于发送邮件，但它发送的Message对象就不是服务。相似的，
    一个Product对象不是服务，但那个将Product载入数据库的对象就是服务。

So what's the big deal then? The advantage of thinking about "services" is
that you begin to think about separating each piece of functionality in your
application into a series of services. Since each service does just one job,
you can easily access each service and use its functionality wherever you
need it. Each service can also be more easily tested and configured since
it's separated from the other functionality in your application. This idea
is called `service-oriented architecture`_ and is not unique to Symfony2
or even PHP. Structuring your application around a set of independent service
classes is a well-known and trusted object-oriented best-practice. These skills
are key to being a good developer in almost any language.
创建服务的理念就是将每个功能分开而打包在服务中，每个服务都只做一件事情，当你需要那个服务的时候，
你随时可以访问和调用它的功能。这个理念被称作`service-oriented architecture`_，它并不是symfony或php所独有的，
而是一个被普遍证实的良好的面向对象处理方法。

.. index::
   single: Service Container; What is?

What is a Service Container?
什么是服务容器？
----------------------------

A :term:`Service Container` (or *dependency injection container*) is simply
a PHP object that manages the instantiation of services (i.e. objects).
For example, suppose we have a simple PHP class that delivers email messages.
Without a service container, we must manually create the object whenever
we need it:
服务容器（:term:`Service Container`，或称*dependency injection container*）只是一个
可以管理服务初始化的php对象。比如，假设我们有一个简单的发送邮件的php类，如果没有服务容器，当我们
需要这个服务时，就必须手动创建这个对象:

.. code-block:: php

    use Acme\HelloBundle\Mailer;

    $mailer = new Mailer('sendmail');
    $mailer->send('ryan@foobar.net', ... );

This is easy enough. The imaginary ``Mailer`` class allows us to configure
the method used to deliver the email messages (e.g. ``sendmail``, ``smtp``, etc).
But what if we wanted to use the mailer service somewhere else? We certainly
don't want to repeat the mailer configuration *every* time we need to use
the ``Mailer`` object. What if we needed to change the ``transport`` from
``sendmail`` to ``smtp`` everywhere in the application? We'd need to hunt
down every place we create a ``Mailer`` service and change it.
这很简单。Mailer类允许我们配置发送邮件的方法（比如sendmail，smtp，等等）。
但是假如我们想使用别的发送邮件的方法呢？我们当然不愿意在每次需要使用Mailer对象的时候
来配置它。而且如果我们需要将整个应用中的邮件发送方法从sendmail改成smtp怎么办呢？这样的话
我们就必须修改每个创建了Mailer服务的地方。

.. index::
   single: Service Container; Configuring services

Creating/Configuring Services in the Container
在容器中创建/配置服务
----------------------------------------------

A better answer is to let the service container create the ``Mailer`` object
for you. In order for this to work, we must *teach* the container how to
create the ``Mailer`` service. This is done via configuration, which can
be specified in YAML, XML or PHP:
一个更好的方法就是让服务容器为你创建Mailer对象。我们可以通过配置（YAML,XML,PHP都可以）来“教”容器如何创建Mailer服务:

.. configuration-block::

    .. code-block:: yaml

        # app/config/config.yml
        services:
            my_mailer:
                class:        Acme\HelloBundle\Mailer
                arguments:    [sendmail]

    .. code-block:: xml

        <!-- app/config/config.xml -->
        <services>
            <service id="my_mailer" class="Acme\HelloBundle\Mailer">
                <argument>sendmail</argument>
            </service>
        </services>

    .. code-block:: php

        // app/config/config.php
        use Symfony\Component\DependencyInjection\Definition;

        $container->setDefinition('my_mailer', new Definition(
            'Acme\HelloBundle\Mailer',
            array('sendmail')
        ));

.. note::

    When Symfony2 initializes, it builds the service container using the
    application configuration (``app/config/config.yml`` by default). The
    exact file that's loaded is dictated by the ``AppKernel::registerContainerConfiguration()``
    method, which loads an environment-specific configuration file (e.g.
    ``config_dev.yml`` for the ``dev`` environment or ``config_prod.yml``
    for ``prod``).
    当symfony2初始化之后，它使用应用配置（``app/config/config.yml``）来创建服务容器。但到底
    使用哪个文件是通过``AppKernel::registerContainerConfiguration()``指定的，它会载入一个基于环境的配置文件
    （如dev环境下使用``config_dev.yml``文件，而prod环境下使用``config_prod.yml``文件）。

An instance of the ``Acme\HelloBundle\Mailer`` object is now available via
the service container. The container is available in any traditional Symfony2
controller where you can access the services of the container via the ``get()``
shortcut method::
通过服务容器，一个``Acme\HelloBundle\Mailer``对象的实例已经可用了。这个容器在symfony2中的所有控制器
都可用，你可以通过get()方法访问它::

    class HelloController extends Controller
    {
        // ...

        public function sendEmailAction()
        {
            // ...
            $mailer = $this->get('my_mailer');
            $mailer->send('ryan@foobar.net', ... );
        }
    }

When we ask for the ``my_mailer`` service from the container, the container
constructs the object and returns it. This is another major advantage of
using the service container. Namely, a service is *never* constructed until
it's needed. If you define a service and never use it on a request, the service
is never created. This saves memory and increases the speed of your application.
This also means that there's very little or no performance hit for defining
lots of services. Services that are never used are never constructed.
当我们向容器请求my_mailer服务时，容器会创建这个对象并返回它。这也是服务容器的一个优点，
如果你定义了一个服务但是不请求它，这个服务就不会被创建。这样做可以节省内存并提高应用的速度。
即使你定义了很多服务也不会导致性能问题。如果你不使用服务，服务就不会被创建。

As an added bonus, the ``Mailer`` service is only created once and the same
instance is returned each time you ask for the service. This is almost always
the behavior you'll need (it's more flexible and powerful), but we'll learn
later how you can configure a service that has multiple instances.
还有，Mailer服务只被创建一次，当你再需要它时，同样的实例会被返回。这样做会更灵活和方便。下面我们还要学到
如何配置一个服务，使它有多个实例。

.. _book-service-container-parameters:

Service Parameters
服务参数
------------------

The creation of new services (i.e. objects) via the container is pretty
straightforward. Parameters make defining services more organized and flexible:
以上通过容器创建服务（或者说对象）的方法都非常直接，但还可以用参数方法来定义服务，
这样能使它更有组织性且更灵活:

.. configuration-block::

    .. code-block:: yaml

        # app/config/config.yml
        parameters:
            my_mailer.class:      Acme\HelloBundle\Mailer
            my_mailer.transport:  sendmail

        services:
            my_mailer:
                class:        %my_mailer.class%
                arguments:    [%my_mailer.transport%]

    .. code-block:: xml

        <!-- app/config/config.xml -->
        <parameters>
            <parameter key="my_mailer.class">Acme\HelloBundle\Mailer</parameter>
            <parameter key="my_mailer.transport">sendmail</parameter>
        </parameters>

        <services>
            <service id="my_mailer" class="%my_mailer.class%">
                <argument>%my_mailer.transport%</argument>
            </service>
        </services>

    .. code-block:: php

        // app/config/config.php
        use Symfony\Component\DependencyInjection\Definition;

        $container->setParameter('my_mailer.class', 'Acme\HelloBundle\Mailer');
        $container->setParameter('my_mailer.transport', 'sendmail');

        $container->setDefinition('my_mailer', new Definition(
            '%my_mailer.class%',
            array('%my_mailer.transport%')
        ));

The end result is exactly the same as before - the difference is only in
*how* we defined the service. By surrounding the ``my_mailer.class`` and
``my_mailer.transport`` strings in percent (``%``) signs, the container knows
to look for parameters with those names. When the container is built, it
looks up the value of each parameter and uses it in the service definition.
最后的结果还是和原来的一样，不同点就是我们如何定义服务。通过用百分号包围``my_mailer.class``和
``my_mailer.transport``，容器就知道要寻找具有那个名称的参数了。

.. note::

    The percent sign inside a parameter or argument, as part of the string, must 
    be escaped with another percent sign:
    
    .. code-block:: xml

        <argument type="string">http://symfony.com/?foo=%%s&bar=%%d</argument>

The purpose of parameters is to feed information into services. Of course
there was nothing wrong with defining the service without using any parameters.
Parameters, however, have several advantages:
使用参数的目的就是要将信息传入服务中。当然也可以不用参数来定义服务，不过使用参数有几个好处:

* separation and organization of all service "options" under a single
  ``parameters`` key;
  将所有服务的选项都放置在一个parameter参数下；

* parameter values can be used in multiple service definitions;
  参数可以被多个服务定义使用；

* when creating a service in a bundle (we'll show this shortly), using parameters
  allows the service to be easily customized in your application.
  当在bundle中创建服务时（下面将讲到），使用参数可以让这个服务在应用中更容易被修改。

The choice of using or not using parameters is up to you. High-quality
third-party bundles will *always* use parameters as they make the service
stored in the container more configurable. For the services in your application,
however, you may not need the flexibility of parameters.
高质量的第三方bundle都会使用参数，因为参数使得容器中的服务更容易配置。

Array Parameters
数组参数
~~~~~~~~~~~~~~~~

Parameters do not need to be flat strings, they can also be arrays. For the XML
format, you need to use the type="collection" attribute for all parameters that are
arrays.
参数不一定是字符串，它们也可以是数组。如果使用XML格式，你需要对所有是数组的参数使用type="collection"属性。

.. configuration-block::

    .. code-block:: yaml

        # app/config/config.yml
        parameters:
            my_mailer.gateways:
                - mail1
                - mail2
                - mail3
            my_multilang.language_fallback:
                en:
                    - en
                    - fr
                fr:
                    - fr
                    - en

    .. code-block:: xml

        <!-- app/config/config.xml -->
        <parameters>
            <parameter key="my_mailer.gateways" type="collection">
                <parameter>mail1</parameter>
                <parameter>mail2</parameter>
                <parameter>mail3</parameter>
            </parameter>
            <parameter key="my_multilang.language_fallback" type="collection">
                <parameter key="en" type="collection">
                    <parameter>en</parameter>
                    <parameter>fr</parameter>
                </parameter>
                <parameter key="fr" type="collection">
                    <parameter>fr</parameter>
                    <parameter>en</parameter>
                </parameter>
            </parameter>
        </parameters>

    .. code-block:: php

        // app/config/config.php
        use Symfony\Component\DependencyInjection\Definition;

        $container->setParameter('my_mailer.gateways', array('mail1', 'mail2', 'mail3'));
        $container->setParameter('my_multilang.language_fallback',
                                 array('en' => array('en', 'fr'),
                                       'fr' => array('fr', 'en'),
                                ));


Importing other Container Configuration Resources
导入其他容器配置文件
-------------------------------------------------

.. tip::

    In this section, we'll refer to service configuration files as *resources*.
    This is to highlight that fact that, while most configuration resources
    will be files (e.g. YAML, XML, PHP), Symfony2 is so flexible that configuration
    could be loaded from anywhere (e.g. a database or even via an external
    web service).
    在这一节中，我们把服务配置文件称作源（resource）。这是为了表明，虽然大部分的配置都是使用文件的，
    但symfony2却可以从任何地方载入配置（比如数据库或外部web服务）。

The service container is built using a single configuration resource
(``app/config/config.yml`` by default). All other service configuration
(including the core Symfony2 and third-party bundle configuration) must
be imported from inside this file in one way or another. This gives you absolute
flexibility over the services in your application.
服务容器使用了仅仅一个源（``app/config/config.yml``）。所有其他服务配置（包括symfony2核心和第三方bundle配置）
都必须从这个文件中导入。

External service configuration can be imported in two different ways. First,
we'll talk about the method that you'll use most commonly in your application:
the ``imports`` directive. In the following section, we'll introduce the
second method, which is the flexible and preferred method for importing service
configuration from third-party bundles.
外部服务配置可以用两个方法来导入。首先我们将讲述你会最常用到的方法：imports方法。接下来
我们将介绍第二种方法，我们推荐用这个方法来导入第三方bundle的服务配置。

.. index::
   single: Service Container; imports

.. _service-container-imports-directive:

Importing Configuration with ``imports``
使用imports导入配置
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

So far, we've placed our ``my_mailer`` service container definition directly
in the application configuration file (e.g. ``app/config/config.yml``). Of
course, since the ``Mailer`` class itself lives inside the ``AcmeHelloBundle``,
it makes more sense to put the ``my_mailer`` container definition inside the
bundle as well.
现在，我们已经将my_mailer服务容器的定义直接放置在应用配置文件中了（``app/config/config.yml``）。
当然，由于Mailer类就是``AcmeHelloBundle``中的，如果能将my_mailer容器配置放在这个bundle中就更好了。

First, move the ``my_mailer`` container definition into a new container resource
file inside ``AcmeHelloBundle``. If the ``Resources`` or ``Resources/config``
directories don't exist, create them.
首先，将my_mailer容器配置移至``AcmeHelloBundle``中一个新的源文件中。如果``Resources``或``Resources/config``
不存在，就创建它们。

.. configuration-block::

    .. code-block:: yaml

        # src/Acme/HelloBundle/Resources/config/services.yml
        parameters:
            my_mailer.class:      Acme\HelloBundle\Mailer
            my_mailer.transport:  sendmail

        services:
            my_mailer:
                class:        %my_mailer.class%
                arguments:    [%my_mailer.transport%]

    .. code-block:: xml

        <!-- src/Acme/HelloBundle/Resources/config/services.xml -->
        <parameters>
            <parameter key="my_mailer.class">Acme\HelloBundle\Mailer</parameter>
            <parameter key="my_mailer.transport">sendmail</parameter>
        </parameters>

        <services>
            <service id="my_mailer" class="%my_mailer.class%">
                <argument>%my_mailer.transport%</argument>
            </service>
        </services>

    .. code-block:: php

        // src/Acme/HelloBundle/Resources/config/services.php
        use Symfony\Component\DependencyInjection\Definition;

        $container->setParameter('my_mailer.class', 'Acme\HelloBundle\Mailer');
        $container->setParameter('my_mailer.transport', 'sendmail');

        $container->setDefinition('my_mailer', new Definition(
            '%my_mailer.class%',
            array('%my_mailer.transport%')
        ));

The definition itself hasn't changed, only its location. Of course the service
container doesn't know about the new resource file. Fortunately, we can
easily import the resource file using the ``imports`` key in the application
configuration.
这个配置并没有改变，只不过地点改变了。当然这个服务容器并不知道这个新的源文件。我们可以使用在应用配置中使用imports：

.. configuration-block::

    .. code-block:: yaml

        # app/config/config.yml
        imports:
            - { resource: @AcmeHelloBundle/Resources/config/services.yml }

    .. code-block:: xml

        <!-- app/config/config.xml -->
        <imports>
            <import resource="@AcmeHelloBundle/Resources/config/services.xml"/>
        </imports>

    .. code-block:: php

        // app/config/config.php
        $this->import('@AcmeHelloBundle/Resources/config/services.php');

The ``imports`` directive allows your application to include service container
configuration resources from any other location (most commonly from bundles).
The ``resource`` location, for files, is the absolute path to the resource
file. The special ``@AcmeHello`` syntax resolves the directory path of
the ``AcmeHelloBundle`` bundle. This helps you specify the path to the resource
without worrying later if you move the ``AcmeHelloBundle`` to a different
directory.
imports允许你的应用从任何地点包含服务容器配置源（通常是bundle中）。resource，就是指指向能够源的
绝对路径。``@AcmeHello``语法指代的就是``AcmeHelloBundle``的目录路径。这样如果你将``AcmeHelloBundle``
移至不同目录，你也不必担心它的路径。

.. index::
   single: Service Container; Extension configuration

.. _service-container-extension-configuration:

Importing Configuration via Container Extensions
通过容器扩展来导入配置
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

When developing in Symfony2, you'll most commonly use the ``imports`` directive
to import container configuration from the bundles you've created specifically
for your application. Third-party bundle container configuration, including
Symfony2 core services, are usually loaded using another method that's more
flexible and easy to configure in your application.
当使用symfony2开发时，你常常会使用imports来从你创建的bundle中导入容器配置。但第三方bundle，包括
symfony2的核心服务的容器配置，通常都使用另一个方法来载入。

Here's how it works. Internally, each bundle defines its services very much
like we've seen so far. Namely, a bundle uses one or more configuration
resource files (usually XML) to specify the parameters and services for that
bundle. However, instead of importing each of these resources directly from
your application configuration using the ``imports`` directive, you can simply
invoke a *service container extension* inside the bundle that does the work for
you. A service container extension is a PHP class created by the bundle author
to accomplish two things:
在内部，每个bundle都像我们所见到的那样来定义它的服务。也就是一个bundle使用一个或多个
配置源文件（通常是XML）来指定那个bundle的参数和服务。但是，你可以在bundle的内部调用
服务容器扩展来导入配置，而不用imports方法。服务容器扩展就是bundle的作者创建的一个php类，它可以做
两个工作：

* import all service container resources needed to configure the services for
  the bundle;
  导入服务容器的源来配置这个bundle的服务；

* provide semantic, straightforward configuration so that the bundle can
  be configured without interacting with the flat parameters of the bundle's
  service container configuration.
  提供直接明了的配置，这样那个bundle就不必和它的服务容器配置的直接参数交互了。

In other words, a service container extension configures the services for
a bundle on your behalf. And as we'll see in a moment, the extension provides
a sensible, high-level interface for configuring the bundle.

Take the ``FrameworkBundle`` - the core Symfony2 framework bundle - as an
example. The presence of the following code in your application configuration
invokes the service container extension inside the ``FrameworkBundle``:
拿``FrameworkBundle``作为例子，它是symfony2框架的核心bundle。以下的代码调用了``FrameworkBundle``
中的服务容器扩展:

.. configuration-block::

    .. code-block:: yaml

        # app/config/config.yml
        framework:
            secret:          xxxxxxxxxx
            charset:         UTF-8
            form:            true
            csrf_protection: true
            router:        { resource: "%kernel.root_dir%/config/routing.yml" }
            # ...

    .. code-block:: xml

        <!-- app/config/config.xml -->
        <framework:config charset="UTF-8" secret="xxxxxxxxxx">
            <framework:form />
            <framework:csrf-protection />
            <framework:router resource="%kernel.root_dir%/config/routing.xml" />
            <!-- ... -->
        </framework>

    .. code-block:: php

        // app/config/config.php
        $container->loadFromExtension('framework', array(
            'secret'          => 'xxxxxxxxxx',
            'charset'         => 'UTF-8',
            'form'            => array(),
            'csrf-protection' => array(),
            'router'          => array('resource' => '%kernel.root_dir%/config/routing.php'),
            // ...
        ));

When the configuration is parsed, the container looks for an extension that
can handle the ``framework`` configuration directive. The extension in question,
which lives in the ``FrameworkBundle``, is invoked and the service configuration
for the ``FrameworkBundle`` is loaded. If you remove the ``framework`` key
from your application configuration file entirely, the core Symfony2 services
won't be loaded. The point is that you're in control: the Symfony2 framework
doesn't contain any magic or perform any actions that you don't have control
over.
当配置被解析时，容器会查找一个可以处理framework配置的扩展。这个扩展是``FrameworkBundle``中的，
它被调用，然后``FrameworkBundle``的服务配置就载入了。如果你将framework去掉，symfony2核心服务就
不会加载。你对symfony有着控制权，symfony2框架的所有逻辑都可以由你控制。

Of course you can do much more than simply "activate" the service container
extension of the ``FrameworkBundle``. Each extension allows you to easily
customize the bundle, without worrying about how the internal services are
defined.
当然你还可以做比简单的调用``FrameworkBundle``的服务容器扩展更多的工作。每个扩展都允许你修改bundle，
而不必担心内部的服务是如何定义的。

In this case, the extension allows you to customize the ``charset``, ``error_handler``,
``csrf_protection``, ``router`` configuration and much more. Internally,
the ``FrameworkBundle`` uses the options specified here to define and configure
the services specific to it. The bundle takes care of creating all the necessary
``parameters`` and ``services`` for the service container, while still allowing
much of the configuration to be easily customized. As an added bonus, most
service container extensions are also smart enough to perform validation -
notifying you of options that are missing or the wrong data type.
在这个例子中，这个扩展允许你修改charset、error_handler、csrf_protection、router等配置。在内部，``FrameworkBundle``
使用这里指定的选项来定义和配置相关服务。bundle自己会创建所有的parameters和services，但与此同时你又可以自己修改这些配置。
还有，服务容器扩展还能够验证在你修改后，选项是否丢失或者值是否是正确类型。

When installing or configuring a bundle, see the bundle's documentation for
how the services for the bundle should be installed and configured. The options
available for the core bundles can be found inside the :doc:`Reference Guide</reference/index>`.
当安装或者配置一个bundle时，请查看bundle的文档以了解bundle的服务如何被安装和配置。
核心bundle的可用选项可以在:doc:`Reference Guide</reference/index>`中找到。

.. note::

   Natively, the service container only recognizes the ``parameters``,
   ``services``, and ``imports`` directives. Any other directives
   are handled by a service container extension.
   服务容器只能够识别parameters、services和imports这几个选项。其他选项都有服务容器扩展处理。

If you want to expose user friendly configuration in your own bundles, read the
":doc:`/cookbook/bundles/extension`" cookbook recipe.
如果你想在你自己的bundle中创建用户友好配置，参阅":doc:`/cookbook/bundles/extension`"。

.. index::
   single: Service Container; Referencing services

Referencing (Injecting) Services
引用（注入）服务
--------------------------------

So far, our original ``my_mailer`` service is simple: it takes just one argument
in its constructor, which is easily configurable. As you'll see, the real
power of the container is realized when you need to create a service that
depends on one or more other services in the container.
目前为止，我们的my_mailer服务很简单：它只在constructor中引入一个参数，这很容易配置。
但容器的最强大的用途在于根据容器中一个或多个其他服务来创建一个服务。

Let's start with an example. Suppose we have a new service, ``NewsletterManager``,
that helps to manage the preparation and delivery of an email message to
a collection of addresses. Of course the ``my_mailer`` service is already
really good at delivering email messages, so we'll use it inside ``NewsletterManager``
to handle the actual delivery of the messages. This pretend class might look
something like this::
举个例子，假设我们有一个新服务，``NewsletterManager``，它可以管理一个要到达多个地址的邮件的
准备和发送工作。my_mailer是用于发送邮件的，所以我们可以在``NewsletterManager``内部使用它来处理
邮件发送。这个类看起来会是这样::

    namespace Acme\HelloBundle\Newsletter;

    use Acme\HelloBundle\Mailer;

    class NewsletterManager
    {
        protected $mailer;

        public function __construct(Mailer $mailer)
        {
            $this->mailer = $mailer;
        }

        // ...
    }

Without using the service container, we can create a new ``NewsletterManager``
fairly easily from inside a controller::
如果不使用服务容器的话，我们可以很容易地从控制器中创建一个新的``NewsletterManager``::

    public function sendNewsletterAction()
    {
        $mailer = $this->get('my_mailer');
        $newsletter = new Acme\HelloBundle\Newsletter\NewsletterManager($mailer);
        // ...
    }

This approach is fine, but what if we decide later that the ``NewsletterManager``
class needs a second or third constructor argument? What if we decide to
refactor our code and rename the class? In both cases, you'd need to find every
place where the ``NewsletterManager`` is instantiated and modify it. Of course,
the service container gives us a much more appealing option:
看起来很好，但是如果我们想要在``NewsletterManager``类中添加一个或两个constructor参数呢？
如果我们想重构代码、重命名这个类呢？那么就又要寻找所有``NewsletterManager``被实例化了的地方并修改它。
如果使用服务容器解决这个问题就方便得多:

.. configuration-block::

    .. code-block:: yaml

        # src/Acme/HelloBundle/Resources/config/services.yml
        parameters:
            # ...
            newsletter_manager.class: Acme\HelloBundle\Newsletter\NewsletterManager

        services:
            my_mailer:
                # ...
            newsletter_manager:
                class:     %newsletter_manager.class%
                arguments: [@my_mailer]

    .. code-block:: xml

        <!-- src/Acme/HelloBundle/Resources/config/services.xml -->
        <parameters>
            <!-- ... -->
            <parameter key="newsletter_manager.class">Acme\HelloBundle\Newsletter\NewsletterManager</parameter>
        </parameters>

        <services>
            <service id="my_mailer" ... >
              <!-- ... -->
            </service>
            <service id="newsletter_manager" class="%newsletter_manager.class%">
                <argument type="service" id="my_mailer"/>
            </service>
        </services>

    .. code-block:: php

        // src/Acme/HelloBundle/Resources/config/services.php
        use Symfony\Component\DependencyInjection\Definition;
        use Symfony\Component\DependencyInjection\Reference;

        // ...
        $container->setParameter('newsletter_manager.class', 'Acme\HelloBundle\Newsletter\NewsletterManager');

        $container->setDefinition('my_mailer', ... );
        $container->setDefinition('newsletter_manager', new Definition(
            '%newsletter_manager.class%',
            array(new Reference('my_mailer'))
        ));

In YAML, the special ``@my_mailer`` syntax tells the container to look for
a service named ``my_mailer`` and to pass that object into the constructor
of ``NewsletterManager``. In this case, however, the specified service ``my_mailer``
must exist. If it does not, an exception will be thrown. You can mark your
dependencies as optional - this will be discussed in the next section.
在YAML中，@my_mailer语句告诉容器寻找一个名叫my_mailer的服务并将那个对象传递到``NewsletterManager``
的constructor中。在这个例子中，指定的my_mailer必须存在。如果不存在，一个错误会被抛出。
下面的章节我们将介绍如果要使引用可选怎么做。

Using references is a very powerful tool that allows you to create independent service
classes with well-defined dependencies. In this example, the ``newsletter_manager``
service needs the ``my_mailer`` service in order to function. When you define
this dependency in the service container, the container takes care of all
the work of instantiating the objects.
使用引用允许你根据某个服务来创建另一个服务。在这个例子中，``newsletter_manager``服务需要``my_mailer``服务
才能运行。当你在服务容器中定义引用之后，容器就会帮助你对类进行初始化。

Optional Dependencies: Setter Injection
可选引用：Setter注入
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Injecting dependencies into the constructor in this manner is an excellent
way of ensuring that the dependency is available to use. If you have optional
dependencies for a class, then "setter injection" may be a better option. This
means injecting the dependency using a method call rather than through the
constructor. The class would look like this::
注入引用（inject dependency）到constructor是一个很好的保证引用可用的方法。但是如果你要使
引用可选，就要使用setter injection。意思是使用一个函数而不是通过constructor来注入引用。它
会像这样::

    namespace Acme\HelloBundle\Newsletter;

    use Acme\HelloBundle\Mailer;

    class NewsletterManager
    {
        protected $mailer;

        public function setMailer(Mailer $mailer)
        {
            $this->mailer = $mailer;
        }

        // ...
    }

Injecting the dependency by the setter method just needs a change of syntax:
通过setter注入引用只需要修改一些语句：

.. configuration-block::

    .. code-block:: yaml

        # src/Acme/HelloBundle/Resources/config/services.yml
        parameters:
            # ...
            newsletter_manager.class: Acme\HelloBundle\Newsletter\NewsletterManager

        services:
            my_mailer:
                # ...
            newsletter_manager:
                class:     %newsletter_manager.class%
                calls:
                    - [ setMailer, [ @my_mailer ] ]

    .. code-block:: xml

        <!-- src/Acme/HelloBundle/Resources/config/services.xml -->
        <parameters>
            <!-- ... -->
            <parameter key="newsletter_manager.class">Acme\HelloBundle\Newsletter\NewsletterManager</parameter>
        </parameters>

        <services>
            <service id="my_mailer" ... >
              <!-- ... -->
            </service>
            <service id="newsletter_manager" class="%newsletter_manager.class%">
                <call method="setMailer">
                     <argument type="service" id="my_mailer" />
                </call>
            </service>
        </services>

    .. code-block:: php

        // src/Acme/HelloBundle/Resources/config/services.php
        use Symfony\Component\DependencyInjection\Definition;
        use Symfony\Component\DependencyInjection\Reference;

        // ...
        $container->setParameter('newsletter_manager.class', 'Acme\HelloBundle\Newsletter\NewsletterManager');

        $container->setDefinition('my_mailer', ... );
        $container->setDefinition('newsletter_manager', new Definition(
            '%newsletter_manager.class%'
        ))->addMethodCall('setMailer', array(
            new Reference('my_mailer')
        ));

.. note::

    The approaches presented in this section are called "constructor injection"
    and "setter injection". The Symfony2 service container also supports
    "property injection".
    这节展示的是constructor注入和setter注入。symfony2服务还支持property注入。

Making References Optional
使引用可选
--------------------------

Sometimes, one of your services may have an optional dependency, meaning
that the dependency is not required for your service to work properly. In
the example above, the ``my_mailer`` service *must* exist, otherwise an exception
will be thrown. By modifying the ``newsletter_manager`` service definition,
you can make this reference optional. The container will then inject it if
it exists and do nothing if it doesn't:
有时候你需要使引用可选，也就是这个引用不是必须的。在以上的例子中，my_mailer服务必须存在，否则一个错误会
被抛出。要想使这个引用可选，可以修改``newsletter_manager``服务的设置。这样如果它存在，就会被
注入，如果不存在，就被忽略:

.. configuration-block::

    .. code-block:: yaml

        # src/Acme/HelloBundle/Resources/config/services.yml
        parameters:
            # ...

        services:
            newsletter_manager:
                class:     %newsletter_manager.class%
                arguments: [@?my_mailer]

    .. code-block:: xml

        <!-- src/Acme/HelloBundle/Resources/config/services.xml -->

        <services>
            <service id="my_mailer" ... >
              <!-- ... -->
            </service>
            <service id="newsletter_manager" class="%newsletter_manager.class%">
                <argument type="service" id="my_mailer" on-invalid="ignore" />
            </service>
        </services>

    .. code-block:: php

        // src/Acme/HelloBundle/Resources/config/services.php
        use Symfony\Component\DependencyInjection\Definition;
        use Symfony\Component\DependencyInjection\Reference;
        use Symfony\Component\DependencyInjection\ContainerInterface;

        // ...
        $container->setParameter('newsletter_manager.class', 'Acme\HelloBundle\Newsletter\NewsletterManager');

        $container->setDefinition('my_mailer', ... );
        $container->setDefinition('newsletter_manager', new Definition(
            '%newsletter_manager.class%',
            array(new Reference('my_mailer', ContainerInterface::IGNORE_ON_INVALID_REFERENCE))
        ));

In YAML, the special ``@?`` syntax tells the service container that the dependency
is optional. Of course, the ``NewsletterManager`` must also be written to
allow for an optional dependency:
在YAML中，``@?``语句告诉服务容器这个引用是可选的。当然，还要在``NewsletterManager``中允许
引用可选：

.. code-block:: php

        public function __construct(Mailer $mailer = null)
        {
            // ...
        }

Core Symfony and Third-Party Bundle Services
symfony核心和第三方bundle服务
--------------------------------------------

Since Symfony2 and all third-party bundles configure and retrieve their services
via the container, you can easily access them or even use them in your own
services. To keep things simple, Symfony2 by default does not require that
controllers be defined as services. Furthermore Symfony2 injects the entire
service container into your controller. For example, to handle the storage of
information on a user's session, Symfony2 provides a ``session`` service,
which you can access inside a standard controller as follows::
由于symfony2和所有的第三方bundle都会通过容器来配置和获取它们的服务，你可以很容易地访问这些服务或在你创建
的服务中使用它们。为了让事情简单化，symfony2不要求将控制器定义为服务，但是symfony将整个服务容器都
注入你的控制器。比如，要处理在用户session上的信息存储，symfony2提供一个session服务，你可以通过
控制器来访问::

    public function indexAction($bar)
    {
        $session = $this->get('session');
        $session->set('foo', $bar);

        // ...
    }

In Symfony2, you'll constantly use services provided by the Symfony core or
other third-party bundles to perform tasks such as rendering templates (``templating``),
sending emails (``mailer``), or accessing information on the request (``request``).
在symfony2中，你会一直使用symfony核心或者其他第三方bundle提供的服务来做一些工作，比如提交模板（templating）、
发送邮件（mailer）、或访问请求上的信息（request）。

We can take this a step further by using these services inside services that
you've created for your application. Let's modify the ``NewsletterManager``
to use the real Symfony2 ``mailer`` service (instead of the pretend ``my_mailer``).
Let's also pass the templating engine service to the ``NewsletterManager``
so that it can generate the email content via a template::
还可以在你自己创建的服务中使用这些服务。下面我们将修改``NewsletterManager``，使它能够使用
真正的symfony2 mailer服务（而不是那个假设的my_mailer）。我们还要将模板引擎服务传递到``NewsletterManager``中，
这样它就能通过模板来集成email了::

    namespace Acme\HelloBundle\Newsletter;

    use Symfony\Component\Templating\EngineInterface;

    class NewsletterManager
    {
        protected $mailer;

        protected $templating;

        public function __construct(\Swift_Mailer $mailer, EngineInterface $templating)
        {
            $this->mailer = $mailer;
            $this->templating = $templating;
        }

        // ...
    }

Configuring the service container is easy:

.. configuration-block::

    .. code-block:: yaml

        services:
            newsletter_manager:
                class:     %newsletter_manager.class%
                arguments: [@mailer, @templating]

    .. code-block:: xml

        <service id="newsletter_manager" class="%newsletter_manager.class%">
            <argument type="service" id="mailer"/>
            <argument type="service" id="templating"/>
        </service>

    .. code-block:: php

        $container->setDefinition('newsletter_manager', new Definition(
            '%newsletter_manager.class%',
            array(
                new Reference('mailer'),
                new Reference('templating')
            )
        ));

The ``newsletter_manager`` service now has access to the core ``mailer``
and ``templating`` services. This is a common way to create services specific
to your application that leverage the power of different services within
the framework.
``newsletter_manager``服务现在已经可以访问核心mailer和templating服务了。这是一个在你的应用中创建服务
的常用方法，它可以利用框架中的不同服务的功能。

.. tip::

    Be sure that ``swiftmailer`` entry appears in your application
    configuration. As we mentioned in :ref:`service-container-extension-configuration`,
    the ``swiftmailer`` key invokes the service extension from the
    ``SwiftmailerBundle``, which registers the ``mailer`` service.
    确保swiftmailer在你的应用配置中。像我们在:ref:`service-container-extension-configuration`中
    提到的，swiftmailer从``SwiftmailerBundle``中调用了服务扩展并注册了mailer服务。

.. index::
   single: Service Container; Advanced configuration

Advanced Container Configuration
高级容器配置
--------------------------------

As we've seen, defining services inside the container is easy, generally
involving a ``service`` configuration key and a few parameters. However,
the container has several other tools available that help to *tag* services
for special functionality, create more complex services, and perform operations
after the container is built.
在容器内定义服务大体上就是设置服务配置和parameters。容器还有其他功能，比如添加标签、创建更复杂服务、
在容器创建后执行动作等等。

Marking Services as public / private
将访问标记为public/private
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

When defining services, you'll usually want to be able to access these definitions
within your application code. These services are called ``public``. For example,
the ``doctrine`` service registered with the container when using the DoctrineBundle
is a public service as you can access it via::
当定义服务时，你往往需要在你的应用代码中访问这些定义。这些服务被称作public。比如，当使用
DoctrineBundle时，doctrine服务被容器注册，它就是一个public服务，你可以通过它::

   $doctrine = $container->get('doctrine');

However, there are use-cases when you don't want a service to be public. This
is common when a service is only defined because it could be used as an
argument for another service.
但有些时候你不想一个服务公用，比如当一个服务仅仅被创建来作为另一个服务的参数。

.. note::

    If you use a private service as an argument to more than one other service,
    this will result in two different instances being used as the instantiation
    of the private service is done inline (e.g. ``new PrivateFooBar()``).
    如果你使用一个private服务作为两个或以上服务的参数，那么就会有多个实例，因为实例化
    private服务是内联的（e.g. ``new PrivateFooBar()``）。

Simply said: A service will be private when you do not want to access it
directly from your code.

Here is an example:
以下是一个范例：

.. configuration-block::

    .. code-block:: yaml

        services:
           foo:
             class: Acme\HelloBundle\Foo
             public: false

    .. code-block:: xml

        <service id="foo" class="Acme\HelloBundle\Foo" public="false" />

    .. code-block:: php

        $definition = new Definition('Acme\HelloBundle\Foo');
        $definition->setPublic(false);
        $container->setDefinition('foo', $definition);

Now that the service is private, you *cannot* call::
现在这个服务是private的了，你不能::

    $container->get('foo');

However, if a service has been marked as private, you can still alias it (see
below) to access this service (via the alias).
但是如果一个服务被标记为private，你仍可以通过别名来访问它。（见下一节）

.. note::

   Services are by default public.
   服务默认是public的。

Aliasing
别名
~~~~~~~~

When using core or third party bundles within your application, you may want
to use shortcuts to access some services. You can do so by aliasing them and,
furthermore, you can even alias non-public services.
当在你的应用中使用核心或第三方bundle时，你可能需要使用便捷方式来访问服务。你可以使用别名。

.. configuration-block::

    .. code-block:: yaml

        services:
           foo:
             class: Acme\HelloBundle\Foo
           bar:
             alias: foo

    .. code-block:: xml

        <service id="foo" class="Acme\HelloBundle\Foo"/>

        <service id="bar" alias="foo" />

    .. code-block:: php

        $definition = new Definition('Acme\HelloBundle\Foo');
        $container->setDefinition('foo', $definition);

        $containerBuilder->setAlias('bar', 'foo');

This means that when using the container directly, you can access the ``foo``
service by asking for the ``bar`` service like this::
这表示当直接使用这个容器时，你可以通过bar来访问foo服务::

    $container->get('bar'); // Would return the foo service

Requiring files
请求文件
~~~~~~~~~~~~~~~

There might be use cases when you need to include another file just before
the service itself gets loaded. To do so, you can use the ``file`` directive.
有时候你可能需要在服务载入之前包含另一个文件。你可以使用file选项。

.. configuration-block::

    .. code-block:: yaml

        services:
           foo:
             class: Acme\HelloBundle\Foo\Bar
             file: %kernel.root_dir%/src/path/to/file/foo.php

    .. code-block:: xml

        <service id="foo" class="Acme\HelloBundle\Foo\Bar">
            <file>%kernel.root_dir%/src/path/to/file/foo.php</file>
        </service>

    .. code-block:: php

        $definition = new Definition('Acme\HelloBundle\Foo\Bar');
        $definition->setFile('%kernel.root_dir%/src/path/to/file/foo.php');
        $container->setDefinition('foo', $definition);

Notice that symfony will internally call the PHP function require_once
which means that your file will be included only once per request.
symfony内部会执行php函数require_once，这样你的文件只会被包含一次。

.. _book-service-container-tags:

Tags (``tags``)
标签（tags）
~~~~~~~~~~~~~~~

In the same way that a blog post on the Web might be tagged with things such
as "Symfony" or "PHP", services configured in your container can also be
tagged. In the service container, a tag implies that the service is meant
to be used for a specific purpose. Take the following example:
在你的容器中配置的服务可以被添加标签。在服务容器中，标签意味着这个服务被用于某个
特定目的。比如:

.. configuration-block::

    .. code-block:: yaml

        services:
            foo.twig.extension:
                class: Acme\HelloBundle\Extension\FooExtension
                tags:
                    -  { name: twig.extension }

    .. code-block:: xml

        <service id="foo.twig.extension" class="Acme\HelloBundle\Extension\FooExtension">
            <tag name="twig.extension" />
        </service>

    .. code-block:: php

        $definition = new Definition('Acme\HelloBundle\Extension\FooExtension');
        $definition->addTag('twig.extension');
        $container->setDefinition('foo.twig.extension', $definition);

The ``twig.extension`` tag is a special tag that the ``TwigBundle`` uses
during configuration. By giving the service this ``twig.extension`` tag,
the bundle knows that the ``foo.twig.extension`` service should be registered
as a Twig extension with Twig. In other words, Twig finds all services tagged
with ``twig.extension`` and automatically registers them as extensions.
``twig.extension``标签是一个特定标签，TwigBundle会在配置时用到的。通过给这个服务``twig.extension``
标签，bundle就知道了foo.twig.extension服务应该被注册为一个twig扩展。换句话说，twig查找所有有
twig.extension标签的服务并自动将它们注册为扩展。

Tags, then, are a way to tell Symfony2 or other third-party bundles that
your service should be registered or used in some special way by the bundle.
标签能够告诉symfony或其他第三方bundle，你的服务可以被bundle注册或被bundle运用。

The following is a list of tags available with the core Symfony2 bundles.
Each of these has a different effect on your service and many tags require
additional arguments (beyond just the ``name`` parameter).
以下是一系列核心symfony2 bundle的可用标签。每个都有不同作用，并且许多标签都要求添加参数（除了name参数）。

* assetic.filter
* assetic.templating.php
* data_collector
* form.field_factory.guesser
* kernel.cache_warmer
* kernel.event_listener
* monolog.logger
* routing.loader
* security.listener.factory
* security.voter
* templating.helper
* twig.extension
* translation.loader
* validator.constraint_validator

Learn more from the Cookbook
----------------------------

* :doc:`/cookbook/service_container/factories`
* :doc:`/cookbook/service_container/parentservices`
* :doc:`/cookbook/controller/service`

.. _`service-oriented architecture`: http://wikipedia.org/wiki/Service-oriented_architecture
