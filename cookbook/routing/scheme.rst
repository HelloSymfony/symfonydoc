.. index::
   single: Routing; Scheme requirement

How to force routes to always use HTTPS or HTTP
如何强制路径总是使用HTTPS或HTTP
===============================================

Sometimes, you want to secure some routes and be sure that they are always
accessed via the HTTPS protocol. The Routing component allows you to enforce
the URI scheme via the ``_scheme`` requirement:
有时候你需要加强路径的安全，确保它们总是使用HTTPS协议。路由系统允许你通过_scheme参数
来加强URI的模式:

.. configuration-block::

    .. code-block:: yaml

        secure:
            pattern:  /secure
            defaults: { _controller: AcmeDemoBundle:Main:secure }
            requirements:
                _scheme:  https

    .. code-block:: xml

        <?xml version="1.0" encoding="UTF-8" ?>

        <routes xmlns="http://symfony.com/schema/routing"
            xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
            xsi:schemaLocation="http://symfony.com/schema/routing http://symfony.com/schema/routing/routing-1.0.xsd">

            <route id="secure" pattern="/secure">
                <default key="_controller">AcmeDemoBundle:Main:secure</default>
                <requirement key="_scheme">https</requirement>
            </route>
        </routes>

    .. code-block:: php

        use Symfony\Component\Routing\RouteCollection;
        use Symfony\Component\Routing\Route;

        $collection = new RouteCollection();
        $collection->add('secure', new Route('/secure', array(
            '_controller' => 'AcmeDemoBundle:Main:secure',
        ), array(
            '_scheme' => 'https',
        )));

        return $collection;

The above configuration forces the ``secure`` route to always use HTTPS.
以上配置强制使得secure路径使用HTTPS。

When generating the ``secure`` URL, and if the current scheme is HTTP, Symfony
will automatically generate an absolute URL with HTTPS as the scheme:
当集成secure URL时，并且如果当前模式是HTTP，symfony会自动集成HTTPS模式的scheme:

.. code-block:: text

    # If the current scheme is HTTPS
    {{ path('secure') }}
    # generates /secure

    # If the current scheme is HTTP
    {{ path('secure') }}
    # generates https://example.com/secure

The requirement is also enforced for incoming requests. If you try to access
the ``/secure`` path with HTTP, you will automatically be redirected to the
same URL, but with the HTTPS scheme.
如果你试图使用HTTP访问/secure路径，你会被自动重定向到HTTPS模式的相同路径。

The above example uses ``https`` for the ``_scheme``, but you can also force a
URL to always use ``http``.
以上范例使用https作为_scheme的参数，但你也可以强制URL使用http。

.. note::

    The Security component provides another way to enforce HTTP or HTTPs via
    the ``requires_channel`` setting. This alternative method is better suited
    to secure an "area" of your website (all URLs under ``/admin``) or when
    you want to secure URLs defined in a third party bundle.
    Security component提供了另一种方式来强制使用HTTPS或HTTP——通过requires_channel配置。
    这个方法可以使你的网站的某一部分（比如所有在/admin下的URL）、或第三方bundle中定义的URL被强制使用某协议。
