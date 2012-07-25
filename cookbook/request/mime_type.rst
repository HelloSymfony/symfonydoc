.. index::
   single: Request; Add a request format and mime type

How to register a new Request Format and Mime Type
如何注册新的请求格式和mime类型
==================================================

Every ``Request`` has a "format" (e.g. ``html``, ``json``), which is used
to determine what type of content to return in the ``Response``. In fact,
the request format, accessible via
:method:`Symfony\\Component\\HttpFoundation\\Request::getRequestFormat`,
is used to set the MIME type of the ``Content-Type`` header on the ``Response``
object. Internally, Symfony contains a map of the most common formats (e.g.
``html``, ``json``) and their associated MIME types (e.g. ``text/html``,
``application/json``). Of course, additional format-MIME type entries can
easily be added. This document will show how you can add the ``jsonp`` format
and corresponding MIME type.
每个Request都有一个格式（比如html、json），它被用于决定Response会返回什么样类型的内容。你
可以通过:method:`Symfony\\Component\\HttpFoundation\\Request::getRequestFormat`访问请求格式，这个格式
被用来在Response的header中设置Content-Type的mime类型。在内部，symfony包含了最常用的格式（如html和json）以及
它们所相关联的mime类型（如``text/html``,``application/json``）。当然，你还可以添加格式的mime类型入口。
本章将讲述如何添加jsonp格式以及对应的mime类型。

Create an ``kernel.request`` Listener
创建一个kernel.request收听器
-------------------------------------

The key to defining a new MIME type is to create a class that will "listen" to
the ``kernel.request`` event dispatched by the Symfony kernel. The
``kernel.request`` event is dispatched early in Symfony's request handling
process and allows you to modify the request object.
要定义一个新的mime类型的关键就是要创建一个“收听”symfony核心所分配kernel.request事件
的类。这个kernel.request事件在symfony请求处理过程的早期阶段就被分配了，它允许你修改请求对象。

Create the following class, replacing the path with a path to a bundle in your
project::
创建以下的类，将路径修改为你自己的bundle的路径::

    // src/Acme/DemoBundle/RequestListener.php
    namespace Acme\DemoBundle;

    use Symfony\Component\HttpKernel\HttpKernelInterface;
    use Symfony\Component\HttpKernel\Event\GetResponseEvent;

    class RequestListener
    {
        public function onKernelRequest(GetResponseEvent $event)
        {
            $event->getRequest()->setFormat('jsonp', 'application/javascript');
        }
    }

Registering your Listener
注册你的收听器
-------------------------

As for any other listener, you need to add it in one of your configuration
file and register it as a listener by adding the ``kernel.event_listener`` tag:
像任何其他收听器（listener）一样，你需要在你的配置文件中添加它的信息，通过添加kernel.event_listener
标签来将它注册为一个收听器:

.. configuration-block::

    .. code-block:: xml

        <!-- app/config/config.xml -->
        <?xml version="1.0" ?>

        <container xmlns="http://symfony.com/schema/dic/services"
            xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
            xsi:schemaLocation="http://symfony.com/schema/dic/services http://symfony.com/schema/dic/services/services-1.0.xsd">
            <services>
            <service id="acme.demobundle.listener.request" class="Acme\DemoBundle\RequestListener">
                <tag name="kernel.event_listener" event="kernel.request" method="onKernelRequest" />
            </service>
            </services>
        </container>

    .. code-block:: yaml

        # app/config/config.yml
        services:
            acme.demobundle.listener.request:
                class: Acme\DemoBundle\RequestListener
                tags:
                    - { name: kernel.event_listener, event: kernel.request, method: onKernelRequest }

    .. code-block:: php

        # app/config/config.php
        $definition = new Definition('Acme\DemoBundle\RequestListener');
        $definition->addTag('kernel.event_listener', array('event' => 'kernel.request', 'method' => 'onKernelRequest'));
        $container->setDefinition('acme.demobundle.listener.request', $definition);

At this point, the ``acme.demobundle.listener.request`` service has been
configured and will be notified when the Symfony kernel dispatches the
``kernel.request`` event.
这时，``acme.demobundle.listener.request``服务已经被配置好，当symfony kernel分配kernel.request事件
时，它就会发出通知。

.. tip::

    You can also register the listener in a configuration extension class (see
    :ref:`service-container-extension-configuration` for more information).
    你还可以在一个配置扩展类中注册收听器（详情请见:ref:`service-container-extension-configuration`）。
