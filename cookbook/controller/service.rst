.. index::
   single: Controller; As Services

How to define Controllers as Services
如何将控制器定义为服务
=====================================

In the book, you've learned how easily a controller can be used when it
extends the base
:class:`Symfony\\Bundle\\FrameworkBundle\\Controller\\Controller` class. While
this works fine, controllers can also be specified as services.
你已经学习了如何通过扩展:class:`Symfony\\Bundle\\FrameworkBundle\\Controller\\Controller`类来
使用控制器。但控制器还可以被定义为服务。

To refer to a controller that's defined as a service, use the single colon (:)
notation. For example, suppose we've defined a service called
``my_controller`` and we want to forward to a method called ``indexAction()``
inside the service::
要访问一个被定义为服务的控制器，使用冒号（:）。比如，假设我们已经定义了一个名叫my_controller的
服务，现在我们要转到该服务中的一个名叫indexAction()的方法：

    $this->forward('my_controller:indexAction', array('foo' => $bar));

You need to use the same notation when defining the route ``_controller``
value:
在定义路径的_controller的值的时候，你需要使用同样的语句:

.. code-block:: yaml

    my_controller:
        pattern:   /
        defaults:  { _controller: my_controller:indexAction }

To use a controller in this way, it must be defined in the service container
configuration. For more information, see the :doc:`Service Container
</book/service_container>` chapter.
要像这样使用控制器，你必须在服务容器配置中定义它，参见</book/service_container>`。

When using a controller defined as a service, it will most likely not extend
the base ``Controller`` class. Instead of relying on its shortcut methods,
you'll interact directly with the services that you need. Fortunately, this is
usually pretty easy and the base ``Controller`` class itself is a great source
on how to perform many common tasks.
当使用一个被定义为服务的控制器时，它不会扩展基本的``Controller``类。你需要直接和你需要的服务
交互，而不是依赖于它的便捷方法。

.. note::

    Specifying a controller as a service takes a little bit more work. The
    primary advantage is that the entire controller or any services passed to
    the controller can be modified via the service container configuration.
    This is especially useful when developing an open-source bundle or any
    bundle that will be used in many different projects. So, even if you don't
    specify your controllers as services, you'll likely see this done in some
    open-source Symfony2 bundles.
