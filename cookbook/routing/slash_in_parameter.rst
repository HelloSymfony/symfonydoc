.. index::
   single: Routing; Allow / in route parameter

How to allow a "/" character in a route parameter
如何允许一个"/"字符被作为路径参数
=================================================

Sometimes, you need to compose URLs with parameters that can contain a slash 
``/``. For example, take the classic ``/hello/{name}`` route. By default,
``/hello/Fabien`` will match this route but not ``/hello/Fabien/Kris``. This
is because Symfony uses this character as separator between route parts.
有时候你需要构成包含"/"参数的URL。比如/hello/{name}路径，默认情况下，/hello/Fabien能够匹配这个路径，
但/hello/Febien/Kris不会匹配。因为symfony使用/作为路径各个部分的分隔。

This guide covers how you can modify a route so that ``/hello/Fabien/Kris``
matches the ``/hello/{name}`` route, where ``{name}`` equals ``Fabien/Kris``.
本章讲述如何修改路径，使得/hello/Fabien/Kris也能够匹配/hello/{name},其中{name}等于/Fabien/Kris。

Configure the Route
配置路径
-------------------

By default, the symfony routing components requires that the parameters 
match the following regex pattern: ``[^/]+``. This means that all characters 
are allowed except ``/``. 
默认情况下，symfony的routing component要求参数匹配以下正则表达式：``[^/]+``。这表示
所有字符串都被允许，除了/。

You must explicitly allow ``/`` to be part of your parameter by specifying 
a more permissive regex pattern.
你必须修改这个正则表达式来允许/作为参数。

.. configuration-block::

    .. code-block:: yaml

        _hello:
            pattern: /hello/{name}
            defaults: { _controller: AcmeDemoBundle:Demo:hello }
            requirements:
                name: ".+"

    .. code-block:: xml

        <?xml version="1.0" encoding="UTF-8" ?>

        <routes xmlns="http://symfony.com/schema/routing"
            xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
            xsi:schemaLocation="http://symfony.com/schema/routing http://symfony.com/schema/routing/routing-1.0.xsd">

            <route id="_hello" pattern="/hello/{name}">
                <default key="_controller">AcmeDemoBundle:Demo:hello</default>
                <requirement key="name">.+</requirement>
            </route>
        </routes>

    .. code-block:: php

        use Symfony\Component\Routing\RouteCollection;
        use Symfony\Component\Routing\Route;

        $collection = new RouteCollection();
        $collection->add('_hello', new Route('/hello/{name}', array(
            '_controller' => 'AcmeDemoBundle:Demo:hello',
        ), array(
            'name' => '.+',
        )));

        return $collection;

    .. code-block:: php-annotations

        use Sensio\Bundle\FrameworkExtraBundle\Configuration\Route;

        class DemoController
        {
            /**
             * @Route("/hello/{name}", name="_hello", requirements={"name" = ".+"})
             */
            public function helloAction($name)
            {
                // ...
            }
        }

That's it! Now, the ``{name}`` parameter can contain the ``/`` character.
这样{name}参数就可以包含/字符了。