.. index::
   single: Security

Security
安全系统
========

Security is a two-step process whose goal is to prevent a user from accessing
a resource that he/she should not have access to.
安全系统的目的是为了防止一个未授权用户访问页面。它分为两步。

In the first step of the process, the security system identifies who the user
is by requiring the user to submit some sort of identification. This is called
**authentication**, and it means that the system is trying to find out who
you are.
在第一步中，安全系统通过要求用户提交一些身份验证来识别用户。这个步骤被称作验证（**authentication**），
它表示系统想知道你是谁。

Once the system knows who you are, the next step is to determine if you should
have access to a given resource. This part of the process is called **authorization**,
and it means that the system is checking to see if you have privileges to
perform a certain action.
在系统知道你是谁之后，第二步就是确定你是否有权进入某个页面。这一步骤被称作授权（**authorization**），
它表示系统要检查你是否能做某个动作。

.. image:: /images/book/security_authentication_authorization.png
   :align: center

.. note::

    Symfony's `security component`_ is available as a standalone PHP library
    for use inside any PHP project.
    symfony的`security component`_ 是一个独立的php库。

Basic Example: HTTP Authentication
基本范例：HTTP验证
----------------------------------

The security component can be configured via your application configuration.
In fact, most standard security setups are just a matter of using the right
configuration. The following configuration tells Symfony to secure any URL
matching ``/admin/*`` and to ask the user for credentials using basic HTTP
authentication (i.e. the old-school username/password box):
security component能够通过你的应用配置来配置。实际上，多数安全系统的设置只不过关乎你是否在使用
正确的配置。以下的配置告诉symfony要禁止进入任何匹配``/admin/*``的URL，并且向用户请求HTTP验证（老式的username/password验证框）：

.. configuration-block::

    .. code-block:: yaml

        # app/config/security.yml
        security:
            firewalls:
                secured_area:
                    pattern:    ^/
                    anonymous: ~
                    http_basic:
                        realm: "Secured Demo Area"

            access_control:
                - { path: ^/admin, roles: ROLE_ADMIN }

            providers:
                in_memory:
                    memory:
                        users:
                            ryan:  { password: ryanpass, roles: 'ROLE_USER' }
                            admin: { password: kitten, roles: 'ROLE_ADMIN' }

            encoders:
                Symfony\Component\Security\Core\User\User: plaintext

    .. code-block:: xml

        <?xml version="1.0" encoding="UTF-8"?>

        <srv:container xmlns="http://symfony.com/schema/dic/security"
            xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
            xmlns:srv="http://symfony.com/schema/dic/services"
            xsi:schemaLocation="http://symfony.com/schema/dic/services http://symfony.com/schema/dic/services/services-1.0.xsd">

            <!-- app/config/security.xml -->

            <config>
                <firewall name="secured_area" pattern="^/">
                    <anonymous />
                    <http-basic realm="Secured Demo Area" />
                </firewall>

                <access-control>
                    <rule path="^/admin" role="ROLE_ADMIN" />
                </access-control>

                <provider name="in_memory">
                    <memory>
                        <user name="ryan" password="ryanpass" roles="ROLE_USER" />
                        <user name="admin" password="kitten" roles="ROLE_ADMIN" />
                    </memory>
                </provider>

                <encoder class="Symfony\Component\Security\Core\User\User" algorithm="plaintext" />
            </config>
        </srv:container>

    .. code-block:: php

        // app/config/security.php
        $container->loadFromExtension('security', array(
            'firewalls' => array(
                'secured_area' => array(
                    'pattern' => '^/',
                    'anonymous' => array(),
                    'http_basic' => array(
                        'realm' => 'Secured Demo Area',
                    ),
                ),
            ),
            'access_control' => array(
                array('path' => '^/admin', 'role' => 'ROLE_ADMIN'),
            ),
            'providers' => array(
                'in_memory' => array(
                    'memory' => array(
                        'users' => array(
                            'ryan' => array('password' => 'ryanpass', 'roles' => 'ROLE_USER'),
                            'admin' => array('password' => 'kitten', 'roles' => 'ROLE_ADMIN'),
                        ),
                    ),
                ),
            ),
            'encoders' => array(
                'Symfony\Component\Security\Core\User\User' => 'plaintext',
            ),
        ));

.. tip::

    A standard Symfony distribution separates the security configuration
    into a separate file (e.g. ``app/config/security.yml``). If you don't
    have a separate security file, you can put the configuration directly
    into your main config file (e.g. ``app/config/config.yml``).
    一个标准的symfony代码结构应该是将安全配置分隔到一个单独文件（``app/config/security.yml``），
    如果你没有一个单独文件，可以将配置代码直接放置到你的主要配置文件（``app/config/config.yml``）。

The end result of this configuration is a fully-functional security system
that looks like the following:
这个配置是一个功能完整的安全系统：

* There are two users in the system (``ryan`` and ``admin``);
* 系统中有两个用户（``ryan``和``admin``）；
* Users authenticate themselves via the basic HTTP authentication prompt;
* 用户通过basic HTTP验证；
* Any URL matching ``/admin/*`` is secured, and only the ``admin`` user
  can access it;
* 所有匹配``/admin/*``都被禁止进入，只有admin用户可以进入；
* All URLs *not* matching ``/admin/*`` are accessible by all users (and the
  user is never prompted to login).
* 不匹配``/admin/*``的所有URL，所有用户都可以进入。

Let's look briefly at how security works and how each part of the configuration
comes into play.
让我们简单的看一下安全系统以及配置中的每个部分是如何工作的。

How Security Works: Authentication and Authorization
安全系统如何工作：验证和授权
----------------------------------------------------

Symfony's security system works by determining who a user is (i.e. authentication)
and then checking to see if that user should have access to a specific resource
or URL.
symfony的安全系统通过确定用户是谁（验证），然后检查那个用户能否进入一个URL或使用某个功能。

Firewalls (Authentication)
防火墙（验证）
~~~~~~~~~~~~~~~~~~~~~~~~~~

When a user makes a request to a URL that's protected by a firewall, the
security system is activated. The job of the firewall is to determine whether
or not the user needs to be authenticated, and if he does, to send a response
back to the user initiating the authentication process.
当一个用户请求访问一个防火墙保护的URL时，安全系统就启动了。防火墙的工作就是确定这个用户是否
需要被验证，如果需要，就发送一个响应给用户，从而使验证系统初始化。

A firewall is activated when the URL of an incoming request matches the configured
firewall's regular expression ``pattern`` config value. In this example, the
``pattern`` (``^/``) will match *every* incoming request. The fact that the
firewall is activated does *not* mean, however, that the HTTP authentication
username and password box is displayed for every URL. For example, any user
can access ``/foo`` without being prompted to authenticate.
当一个请求的URL匹配了配置的防火墙的正则表达式``pattern``配置值后，防火墙就启动了。在这个例子中，
``pattern`` (``^/``)会匹配所有请求，但是防火墙被启动并不意味着HTTP验证对于每个URL都显示。比如，
所有用户都可以进入``/foo``而不会有弹出框来要求验证。

.. image:: /images/book/security_anonymous_user_access.png
   :align: center

This works first because the firewall allows *anonymous users* via the ``anonymous``
configuration parameter. In other words, the firewall doesn't require the
user to fully authenticate immediately. And because no special ``role`` is
needed to access ``/foo`` (under the ``access_control`` section), the request
can be fulfilled without ever asking the user to authenticate.
这是由于防火墙通过``anonymous``参数允许匿名用户。换句话说，防火墙并不要求用户马上验证。并且
在``access_control``中，也没有设定一个role来允许进入/foo。所以用户可以直接进入这个URL。

If you remove the ``anonymous`` key, the firewall will *always* make a user
fully authenticate immediately.
如果你去掉``anonymous``参数，防火墙会要求每个用户访问每个URL时都验证。

Access Controls (Authorization)
访问控制（授权）
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

If a user requests ``/admin/foo``, however, the process behaves differently.
This is because of the ``access_control`` configuration section that says
that any URL matching the regular expression pattern ``^/admin`` (i.e. ``/admin``
or anything matching ``/admin/*``) requires the ``ROLE_ADMIN`` role. Roles
are the basis for most authorization: a user can access ``/admin/foo`` only
if it has the ``ROLE_ADMIN`` role.
如果一个用户请求``/admin/foo``，事情就不同了。这是因为在``access_control``配置中要求了
所有匹配正则表达式pattern ``^/admin``（即``/admin``或所有匹配``/admin/*``的URL）都要求
``ROLE_ADMIN``角色。角色是授权的基础：仅当一个用户有``ROLE_ADMIN``角色时，他才可以进入``/admin/foo``。

.. image:: /images/book/security_anonymous_user_denied_authorization.png
   :align: center

Like before, when the user originally makes the request, the firewall doesn't
ask for any identification. However, as soon as the access control layer
denies the user access (because the anonymous user doesn't have the ``ROLE_ADMIN``
role), the firewall jumps into action and initiates the authentication process.
The authentication process depends on the authentication mechanism you're
using. For example, if you're using the form login authentication method,
the user will be redirected to the login page. If you're using HTTP authentication,
the user will be sent an HTTP 401 response so that the user sees the username
and password box.
当用户最开始请求时，防火墙并没有要求确定，但是一旦访问控制层拒绝了用户访问（由于匿名用户没有``ROLE_ADMIN``角色），
防火墙就会启动，并初始化验证。如何验证取决于你所使用的验证方法。比如，如果你使用的是
表单注册验证，用户就会被重定向到登陆页面；如果你使用的是HTTP验证，就会发送一个HTTP 401响应，
用户就会看到一个username和password框。

The user now has the opportunity to submit its credentials back to the application.
If the credentials are valid, the original request can be re-tried.
现在用户就可以将他的验证信息发送给应用。如果匹配，就重新发送请求。

.. image:: /images/book/security_ryan_no_role_admin_access.png
   :align: center

In this example, the user ``ryan`` successfully authenticates with the firewall.
But since ``ryan`` doesn't have the ``ROLE_ADMIN`` role, he's still denied
access to ``/admin/foo``. Ultimately, this means that the user will see some
sort of message indicating that access has been denied.
在这个例子中，用户ryan成功地验证并通过了防火墙，但是由于ryan没有``ROLE_ADMIN``角色，他仍然不能访问
``/admin/foo``。这表示这个用户可能会看到一些表示他不能访问页面的信息。

.. tip::

    When Symfony denies the user access, the user sees an error screen and
    receives a 403 HTTP status code (``Forbidden``). You can customize the
    access denied error screen by following the directions in the
    :ref:`Error Pages<cookbook-error-pages-by-status-code>` cookbook entry
    to customize the 403 error page.
    当symfony禁止用户访问时，用户会看见一个错误页面并接收一个403 HTTP status code(forbidden)。
    如何定制这个错误页面，请参阅:ref:`Error Pages<cookbook-error-pages-by-status-code>`。

Finally, if the ``admin`` user requests ``/admin/foo``, a similar process
takes place, except now, after being authenticated, the access control layer
will let the request pass through:
最后，如果admin用户请求``/admin/foo``，类似的过程会触发，但是这次在验证后，访问控制层
会让请求通过：

.. image:: /images/book/security_admin_role_access.png
   :align: center

The request flow when a user requests a protected resource is straightforward,
but incredibly flexible. As you'll see later, authentication can be handled
in any number of ways, including via a form login, X.509 certificate, or by
authenticating the user via Twitter. Regardless of the authentication method,
the request flow is always the same:
当用户请求一个受保护的页面时，请求流程很简单而且灵活。验证可以通过一系列方法来进行处理，包括
表单注册，X.509验证，或者通过twitter。不管验证方法是什么，请求流程都是这样的：

#. 用户进入一个受保护的页面；
#. 应用将用户重定向到一个表单登陆页面；
#. 用户提交信息(e.g. username/password);
#. 防火墙验证用户;
#. 通过验证的用户重新发送请求.

.. note::

    The *exact* process actually depends a little bit on which authentication
    mechanism you're using. For example, when using form login, the user
    submits its credentials to one URL that processes the form (e.g. ``/login_check``)
    and then is redirected back to the originally requested URL (e.g. ``/admin/foo``).
    But with HTTP authentication, the user submits its credentials directly
    to the original URL (e.g. ``/admin/foo``) and then the page is returned
    to the user in that same request (i.e. no redirect).
    实际上对于不同的验证方法，这个流程还是有些不同的。比如，当使用表单验证时，用户信息会被提交
    到一个处理该表单的URL（即``/login_check``），然后被重定向到原先请求的URL（即``/admin/foo``）。
    但是如果使用HTTP验证，用户会将信息直接提交到原先请求的URL（即``/admin/foo``），并且这个页面
    会通过同样的请求返回给用户（没有重定向）。

    These types of idiosyncrasies shouldn't cause you any problems, but they're
    good to keep in mind.
    这种特点不会带来问题，但最好记在心上。

.. tip::

    You'll also learn later how *anything* can be secured in Symfony2, including
    specific controllers, objects, or even PHP methods.
    你还将学习别的东西如何被保护，如控制器，对象，或php方法。

.. _book-security-form-login:

Using a Traditional Login Form
使用传统的表单验证
------------------------------

.. tip::

    In this section, you'll learn how to create a basic login form that continues
    to use the hard-coded users that are defined in the ``security.yml`` file.
    在这一节中，你将学习如何创建一个使用``security.yml``中硬编码的用户来验证的表单。

    To load users from the database, please read :doc:`/cookbook/security/entity_provider`.
    By reading that article and this section, you can create a full login form
    system that loads users from the database.
    如果要使用数据库中的用户，请参阅:doc:`/cookbook/security/entity_provider`。

So far, you've seen how to blanket your application beneath a firewall and
then protect access to certain areas with roles. By using HTTP Authentication,
you can effortlessly tap into the native username/password box offered by
all browsers. However, Symfony supports many authentication mechanisms out
of the box. For details on all of them, see the
:doc:`Security Configuration Reference</reference/configuration/security>`.
现在，你已经知道如何用防火墙来覆盖你的应用并使用roles参数来控制某些区域的访问。
通过使用HTTP验证，你可以直接输入浏览器提供的username/password框。symfony还支持许多验证
方法，请参阅:doc:`Security Configuration Reference</reference/configuration/security>`。

In this section, you'll enhance this process by allowing the user to authenticate
via a traditional HTML login form.
在本节你将学习如何用传统HTML登陆表单来验证。

First, enable form login under your firewall:
首先，在你的防火墙下配置表单：

.. configuration-block::

    .. code-block:: yaml

        # app/config/security.yml
        security:
            firewalls:
                secured_area:
                    pattern:    ^/
                    anonymous: ~
                    form_login:
                        login_path:  /login
                        check_path:  /login_check

    .. code-block:: xml

        <?xml version="1.0" encoding="UTF-8"?>

        <srv:container xmlns="http://symfony.com/schema/dic/security"
            xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
            xmlns:srv="http://symfony.com/schema/dic/services"
            xsi:schemaLocation="http://symfony.com/schema/dic/services http://symfony.com/schema/dic/services/services-1.0.xsd">

            <!-- app/config/security.xml -->

            <config>
                <firewall name="secured_area" pattern="^/">
                    <anonymous />
                    <form-login login_path="/login" check_path="/login_check" />
                </firewall>
            </config>
        </srv:container>

    .. code-block:: php

        // app/config/security.php
        $container->loadFromExtension('security', array(
            'firewalls' => array(
                'secured_area' => array(
                    'pattern' => '^/',
                    'anonymous' => array(),
                    'form_login' => array(
                        'login_path' => '/login',
                        'check_path' => '/login_check',
                    ),
                ),
            ),
        ));

.. tip::

    If you don't need to customize your ``login_path`` or ``check_path``
    values (the values used here are the default values), you can shorten
    your configuration:
    如果你不必定制你的``login_path``或``check_path``值（在这里使用的是默认的值），你可以简化你的配置：

    .. configuration-block::

        .. code-block:: yaml

            form_login: ~

        .. code-block:: xml

            <form-login />

        .. code-block:: php

            'form_login' => array(),

Now, when the security system initiates the authentication process, it will
redirect the user to the login form (``/login`` by default). Implementing
this login form visually is your job. First, create two routes: one that
will display the login form (i.e. ``/login``) and one that will handle the
login form submission (i.e. ``/login_check``):
现在，当安全系统初始化验证过程时，它会将用户重定向到表单（默认``/login``）。
如何植入这个表单是你的事：首先，创建两个路径，一个会显示这个表单（比如``/login``），
一个会处理这个表单提交的数据（比如``/login_check``）：

.. configuration-block::

    .. code-block:: yaml

        # app/config/routing.yml
        login:
            pattern:   /login
            defaults:  { _controller: AcmeSecurityBundle:Security:login }
        login_check:
            pattern:   /login_check

    .. code-block:: xml

        <!-- app/config/routing.xml -->
        <?xml version="1.0" encoding="UTF-8" ?>

        <routes xmlns="http://symfony.com/schema/routing"
            xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
            xsi:schemaLocation="http://symfony.com/schema/routing http://symfony.com/schema/routing/routing-1.0.xsd">

            <route id="login" pattern="/login">
                <default key="_controller">AcmeSecurityBundle:Security:login</default>
            </route>
            <route id="login_check" pattern="/login_check" />

        </routes>

    ..  code-block:: php

        // app/config/routing.php
        use Symfony\Component\Routing\RouteCollection;
        use Symfony\Component\Routing\Route;

        $collection = new RouteCollection();
        $collection->add('login', new Route('/login', array(
            '_controller' => 'AcmeDemoBundle:Security:login',
        )));
        $collection->add('login_check', new Route('/login_check', array()));

        return $collection;

.. note::

    You will *not* need to implement a controller for the ``/login_check``
    URL as the firewall will automatically catch and process any form submitted
    to this URL.
    你不必为``/login_check`` URL创建一个控制器，因为防火墙会自动获取并处理表单信息。

.. versionadded:: 2.1
    As of Symfony 2.1, you *must* have routes configured for your ``login_path``
    (e.g. ``/login``) and ``check_path`` (e.g. ``/login_check``) URLs.
    在symfony2.1版本中，你必须为你的``login_path``
    (e.g. ``/login``)和``check_path`` (e.g. ``/login_check``)配置路径！

Notice that the name of the ``login`` route isn't important. What's important
is that the URL of the route (``/login``) matches the ``login_path`` config
value, as that's where the security system will redirect users that need
to login.
注意login路径的名称并不重要。重要的是路径(``/login``)的URL匹配login_path的配置值，
因为这个值会将用户定向到表单页面。

Next, create the controller that will display the login form:
接着创建显示表单的控制器：

.. code-block:: php

    // src/Acme/SecurityBundle/Controller/SecurityController.php;
    namespace Acme\SecurityBundle\Controller;

    use Symfony\Bundle\FrameworkBundle\Controller\Controller;
    use Symfony\Component\Security\Core\SecurityContext;

    class SecurityController extends Controller
    {
        public function loginAction()
        {
            $request = $this->getRequest();
            $session = $request->getSession();

            // get the login error if there is one
            if ($request->attributes->has(SecurityContext::AUTHENTICATION_ERROR)) {
                $error = $request->attributes->get(SecurityContext::AUTHENTICATION_ERROR);
            } else {
                $error = $session->get(SecurityContext::AUTHENTICATION_ERROR);
                $session->remove(SecurityContext::AUTHENTICATION_ERROR);
            }

            return $this->render('AcmeSecurityBundle:Security:login.html.twig', array(
                // last username entered by the user
                'last_username' => $session->get(SecurityContext::LAST_USERNAME),
                'error'         => $error,
            ));
        }
    }

Don't let this controller confuse you. As you'll see in a moment, when the
user submits the form, the security system automatically handles the form
submission for you. If the user had submitted an invalid username or password,
this controller reads the form submission error from the security system so
that it can be displayed back to the user.
其实这个控制器没有做检查表单的工作。当用户提交表单时，安全系统会自动处理表单。当用户
提交了一个不可用的username或password，这个控制器会从安全系统中读取错误信息并将它返回给用户。

In other words, your job is to display the login form and any login errors
that may have occurred, but the security system itself takes care of checking
the submitted username and password and authenticating the user.
换句话说，你的工作只是编写显示表单和错误信息的代码，而安全系统会检查提交的username和password是否可用，并
验证这个用户。

Finally, create the corresponding template:
最后，创建相应的模板：

.. configuration-block::

    .. code-block:: html+jinja

        {# src/Acme/SecurityBundle/Resources/views/Security/login.html.twig #}
        {% if error %}
            <div>{{ error.message }}</div>
        {% endif %}

        <form action="{{ path('login_check') }}" method="post">
            <label for="username">Username:</label>
            <input type="text" id="username" name="_username" value="{{ last_username }}" />

            <label for="password">Password:</label>
            <input type="password" id="password" name="_password" />

            {#
                If you want to control the URL the user is redirected to on success (more details below)
                <input type="hidden" name="_target_path" value="/account" />
            #}

            <button type="submit">login</button>
        </form>

    .. code-block:: html+php

        <?php // src/Acme/SecurityBundle/Resources/views/Security/login.html.php ?>
        <?php if ($error): ?>
            <div><?php echo $error->getMessage() ?></div>
        <?php endif; ?>

        <form action="<?php echo $view['router']->generate('login_check') ?>" method="post">
            <label for="username">Username:</label>
            <input type="text" id="username" name="_username" value="<?php echo $last_username ?>" />

            <label for="password">Password:</label>
            <input type="password" id="password" name="_password" />

            <!--
                If you want to control the URL the user is redirected to on success (more details below)
                <input type="hidden" name="_target_path" value="/account" />
            -->

            <button type="submit">login</button>
        </form>

.. tip::

    The ``error`` variable passed into the template is an instance of
    :class:`Symfony\\Component\\Security\\Core\\Exception\\AuthenticationException`.
    It may contain more information - or even sensitive information - about
    the authentication failure, so use it wisely!
    传递进模板的error变量是:class:`Symfony\\Component\\Security\\Core\\Exception\\AuthenticationException`
    类的一个实例。它可能包含更多信息——或敏感信息——关于验证失败的，所以使用时要注意！

The form has very few requirements. First, by submitting the form to ``/login_check``
(via the ``login_check`` route), the security system will intercept the form
submission and process the form for you automatically. Second, the security
system expects the submitted fields to be called ``_username`` and ``_password``
(these field names can be :ref:`configured<reference-security-firewall-form-login>`).
表单的要求很少。首先，通过将表单提交到``/login_check``（通过``login_check``路径），安全系统会
进入表单提交的过程并处理该表单。第二，安全系统要求提交字段的名称是``_username``和``_password``
（这些字段名称可以被配置（:ref:`configured<reference-security-firewall-form-login>`））。

And that's it! When you submit the form, the security system will automatically
check the user's credentials and either authenticate the user or send the
user back to the login form where the error can be displayed.
于是，当你提交表单的时候，安全系统会自动检查用户的信息，然后或者使其通过验证，或者将
用户定向到显示错误的表单页面。

Let's review the whole process:
整个的流程是这样的：

#. 用户试图进入被保护的页面;
#. 防火墙初始化验证过程，将用户定向到表单登陆页面 (``/login``);
#. ``/login`` 页面通过在本例中创建的路径和控制器输出表单;
#. 用户将表单提交到``/login_check``;
#. 安全系统介入，验证用户提交的信息，要么使用户通过验证，要么将用户返回表单页面。

By default, if the submitted credentials are correct, the user will be redirected
to the original page that was requested (e.g. ``/admin/foo``). If the user
originally went straight to the login page, he'll be redirected to the homepage.
This can be highly customized, allowing you to, for example, redirect the
user to a specific URL.
默认情况下，如果提交的信息是正确的，用户会被重定向到原先请求的页面（e.g. ``/admin/foo``）。
如果用户原先访问的是登陆表单页面，则他会被重定向到主页。

For more details on this and how to customize the form login process in general,
see :doc:`/cookbook/security/form_login`.
要了解如何定制表单登陆过程，参见:doc:`/cookbook/security/form_login`。

.. _book-security-common-pitfalls:

.. sidebar:: Avoid Common Pitfalls

    When setting up your login form, watch out for a few common pitfalls.
    当设置你的表单时，注意以下几点：

    **1. 创建正确的路径**

    First, be sure that you've defined the ``/login`` and ``/login_check``
    routes correctly and that they correspond to the ``login_path`` and
    ``check_path`` config values. A misconfiguration here can mean that you're
    redirected to a 404 page instead of the login page, or that submitting
    the login form does nothing (you just see the login form over and over
    again).
    首先，确定你已经正确定义了``/login``和``/login_check``路径，并确保它们都对应于 
    配置中的``login_path``和``check_path``的值。如果你配置错误，那么你就会被重定向到一个
    404页面，而不是登陆页面，或者提交表单的时候不做任何事情。

    **2. 确保登陆页面没有被禁止访问**

    Also, be sure that the login page does *not* require any roles to be
    viewed. For example, the following configuration - which requires the
    ``ROLE_ADMIN`` role for all URLs (including the ``/login`` URL), will
    cause a redirect loop:
    确保登陆页面的访问不要求任何角色。比如，以下配置要求访问所有URL都要有``ROLE_ADMIN``角色（包括``/login``路径），
    这就会导致一个重定向循环：

    .. configuration-block::

        .. code-block:: yaml

            access_control:
                - { path: ^/, roles: ROLE_ADMIN }

        .. code-block:: xml

            <access-control>
                <rule path="^/" role="ROLE_ADMIN" />
            </access-control>

        .. code-block:: php

            'access_control' => array(
                array('path' => '^/', 'role' => 'ROLE_ADMIN'),
            ),

    Removing the access control on the ``/login`` URL fixes the problem:
    要解决这个问题，将``/login``路径的access control移除：

    .. configuration-block::

        .. code-block:: yaml

            access_control:
                - { path: ^/login, roles: IS_AUTHENTICATED_ANONYMOUSLY }
                - { path: ^/, roles: ROLE_ADMIN }

        .. code-block:: xml

            <access-control>
                <rule path="^/login" role="IS_AUTHENTICATED_ANONYMOUSLY" />
                <rule path="^/" role="ROLE_ADMIN" />
            </access-control>

        .. code-block:: php

            'access_control' => array(
                array('path' => '^/login', 'role' => 'IS_AUTHENTICATED_ANONYMOUSLY'),
                array('path' => '^/', 'role' => 'ROLE_ADMIN'),
            ),

    Also, if your firewall does *not* allow for anonymous users, you'll need
    to create a special firewall that allows anonymous users for the login
    page:并且，如果你的防火墙不允许匿名用户，你还要创建一个特定的防火墙来允许登陆页面的匿名用户：

    .. configuration-block::

        .. code-block:: yaml

            firewalls:
                login_firewall:
                    pattern:    ^/login$
                    anonymous:  ~
                secured_area:
                    pattern:    ^/
                    form_login: ~

        .. code-block:: xml

            <firewall name="login_firewall" pattern="^/login$">
                <anonymous />
            </firewall>
            <firewall name="secured_area" pattern="^/">
                <form_login />
            </firewall>

        .. code-block:: php

            'firewalls' => array(
                'login_firewall' => array(
                    'pattern' => '^/login$',
                    'anonymous' => array(),
                ),
                'secured_area' => array(
                    'pattern' => '^/',
                    'form_login' => array(),
                ),
            ),

    **3. 确保``/login_check``在防火墙的后面**

    Next, make sure that your ``check_path`` URL (e.g. ``/login_check``)
    is behind the firewall you're using for your form login (in this example,
    the single firewall matches *all* URLs, including ``/login_check``). If
    ``/login_check`` doesn't match any firewall, you'll receive a ``Unable
    to find the controller for path "/login_check"`` exception.
    确保你的check_path URL（即/login_check）在你为你的登陆表单所设置的防火墙的后面
    （在这个例子中，防火墙匹配所有的URL，包括了/login_check）。如果/login_check不匹配任何
    防火墙，你会接收到一个``Unable
    to find the controller for path "/login_check"``错误信息。

    **4. 多个防火墙不共享安全环境**

    If you're using multiple firewalls and you authenticate against one firewall,
    you will *not* be authenticated against any other firewalls automatically.
    Different firewalls are like different security systems. That's why,
    for most applications, having one main firewall is enough.
    如果你使用多个防火墙并且通过一个防火墙验证，你不会自动的通过其他防火墙进行验证。
    不同防火墙就好比不同安全系统。这就是为什么大多数应用只有一个主要防火墙就足够了。

Authorization
授权
-------------

The first step in security is always authentication: the process of verifying
who the user is. With Symfony, authentication can be done in any way - via
a form login, basic HTTP Authentication, or even via Facebook.
安全系统的第一步就是验证：确定用户是谁。symfony允许验证通过表单、HTTP验证、或facebook验证。

Once the user has been authenticated, authorization begins. Authorization
provides a standard and powerful way to decide if a user can access any resource
(a URL, a model object, a method call, ...). This works by assigning specific
roles to each user, and then requiring different roles for different resources.
一旦用户被验证，授权系统便启动了。授权系统通过一个标准方式来确定用户是否可以进入某个页面
（一个URL，一个model对象，一个方法……），即，为每个用户都确定一个角色，然后对页面确定
某个角色能够访问。

The process of authorization has two different sides:
授权的过程有两个方面：

#. 用户有不同角色;
#. 要进入某个页面，需要特定的角色。

In this section, you'll focus on how to secure different resources (e.g. URLs,
method calls, etc) with different roles. Later, you'll learn more about how
roles are created and assigned to users.
在本节中，你将学到如何限制访问不同页面（URL或方法，等等）。你还将学习如何创建角色并指定给用户。

Securing Specific URL Patterns
限制访问特定的URL pattern
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

The most basic way to secure part of your application is to secure an entire
URL pattern. You've seen this already in the first example of this chapter,
where anything matching the regular expression pattern ``^/admin`` requires
the ``ROLE_ADMIN`` role.
限制访问应用的一个基本方法就是限制访问整个URL pattern。在本章第一节中你已经见到了，
所有匹配正则表达式pattern ``^/admin`` 的页面都要求ROLE_ADMIN角色。

You can define as many URL patterns as you need - each is a regular expression.
你可以任意多地指定URL pattern——每个都是正则表达式。

.. configuration-block::

    .. code-block:: yaml

        # app/config/security.yml
        security:
            # ...
            access_control:
                - { path: ^/admin/users, roles: ROLE_SUPER_ADMIN }
                - { path: ^/admin, roles: ROLE_ADMIN }

    .. code-block:: xml

        <!-- app/config/security.xml -->
        <config>
            <!-- ... -->
            <rule path="^/admin/users" role="ROLE_SUPER_ADMIN" />
            <rule path="^/admin" role="ROLE_ADMIN" />
        </config>

    .. code-block:: php

        // app/config/security.php
        $container->loadFromExtension('security', array(
            // ...
            'access_control' => array(
                array('path' => '^/admin/users', 'role' => 'ROLE_SUPER_ADMIN'),
                array('path' => '^/admin', 'role' => 'ROLE_ADMIN'),
            ),
        ));

.. tip::

    Prepending the path with ``^`` ensures that only URLs *beginning* with
    the pattern are matched. For example, a path of simply ``/admin`` (without
    the ``^``) would correctly match ``/admin/foo`` but would also match URLs
    like ``/foo/admin``.
    在路径前面加上``^``能够确保唯有以这个pattern开头的URL能够匹配。比如，仅仅是一个
    ``/admin`` URL（没有^）会匹配``/admin/foo``，同时也会匹配``/foo/admin``。

For each incoming request, Symfony2 tries to find a matching access control
rule (the first one wins). If the user isn't authenticated yet, the authentication
process is initiated (i.e. the user is given a chance to login). However,
if the user *is* authenticated but doesn't have the required role, an
:class:`Symfony\\Component\\Security\\Core\\Exception\\AccessDeniedException`
exception is thrown, which you can handle and turn into a nice "access denied"
error page for the user. See :doc:`/cookbook/controller/error_pages` for
more information.
对于每个请求，symfony2都会查询匹配的access_control规则（第一个是优先的）。如果用户还没有被
验证，那么验证程序就会启动（比如会让用户登陆）。但是，如果这个用户已经通过验证而角色不匹配，
就会抛出一个:class:`Symfony\\Component\\Security\\Core\\Exception\\AccessDeniedException`错误，
你可以自定义这个错误页面。参见:doc:`/cookbook/controller/error_pages`。

Since Symfony uses the first access control rule it matches, a URL like ``/admin/users/new``
will match the first rule and require only the ``ROLE_SUPER_ADMIN`` role.
Any URL like ``/admin/blog`` will match the second rule and require ``ROLE_ADMIN``.
由于symfony使用第一个匹配的access_control规则，一个像``/admin/users/new``这样的URL会匹配
第一个规则并要求``ROLE_SUPER_ADMIN``角色。任何像``/admin/blog``这样的URL则会匹配第二个规则，并要求ROLE_ADMIN角色。

.. _book-security-securing-ip:

Securing by IP
根据IP限制访问
~~~~~~~~~~~~~~

Certain situations may arise when you may need to restrict access to a given
route based on IP. This is particularly relevant in the case of :ref:`Edge Side Includes<edge-side-includes>`
(ESI), for example, which utilize a route named "_internal". When
ESI is used, the _internal route is required by the gateway cache to enable
different caching options for subsections within a given page. This route
comes with the ^/_internal prefix by default in the standard edition (assuming
you've uncommented those lines from the routing file).
有些时候你可能需要根据IP地址来限制访问某个特定的路径。尤其在:ref:`Edge Side Includes<edge-side-includes>`（ESI）中
会用到，它会使用一个名叫_internal的路径。当使用ESI时，gateway缓存会通过请求这个路径来激活
不同部分的缓存选项。在标准版本中，这个路径有一个^/_internal前缀（假设你在路由文件中没有将这一行注释掉）。

Here is an example of how you might secure this route from outside access:
以下是一个可以使这个路径禁止外部访问的范例：

.. configuration-block::

    .. code-block:: yaml

        # app/config/security.yml
        security:
            # ...
            access_control:
                - { path: ^/_internal, roles: IS_AUTHENTICATED_ANONYMOUSLY, ip: 127.0.0.1 }

    .. code-block:: xml

            <access-control>
                <rule path="^/_internal" role="IS_AUTHENTICATED_ANONYMOUSLY" ip="127.0.0.1" />
            </access-control>

    .. code-block:: php

            'access_control' => array(
                array('path' => '^/_internal', 'role' => 'IS_AUTHENTICATED_ANONYMOUSLY', 'ip' => '127.0.0.1'),
            ),

.. _book-security-securing-channel:

Securing by Channel
根据channel限制访问
~~~~~~~~~~~~~~~~~~~

Much like securing based on IP, requiring the use of SSL is as simple as
adding a new access_control entry:
像基于IP的安全访问一样，使用SSL也是添加一个新的access_control：

.. configuration-block::

    .. code-block:: yaml

        # app/config/security.yml
        security:
            # ...
            access_control:
                - { path: ^/cart/checkout, roles: IS_AUTHENTICATED_ANONYMOUSLY, requires_channel: https }

    .. code-block:: xml

            <access-control>
                <rule path="^/cart/checkout" role="IS_AUTHENTICATED_ANONYMOUSLY" requires_channel="https" />
            </access-control>

    .. code-block:: php

            'access_control' => array(
                array('path' => '^/cart/checkout', 'role' => 'IS_AUTHENTICATED_ANONYMOUSLY', 'requires_channel' => 'https'),
            ),

.. _book-security-securing-controller:

Securing a Controller
根据控制器限制访问
~~~~~~~~~~~~~~~~~~~~~

Protecting your application based on URL patterns is easy, but may not be
fine-grained enough in certain cases. When necessary, you can easily force
authorization from inside a controller:
根据URL来限制访问有时候还不能满足所有要求，有时，你还可以在控制器中设置授权：

.. code-block:: php

    use Symfony\Component\Security\Core\Exception\AccessDeniedException;
    // ...

    public function helloAction($name)
    {
        if (false === $this->get('security.context')->isGranted('ROLE_ADMIN')) {
            throw new AccessDeniedException();
        }

        // ...
    }

.. _book-security-securing-controller-annotations:

You can also choose to install and use the optional ``JMSSecurityExtraBundle``,
which can secure your controller using annotations:
你还可以选择使用``JMSSecurityExtraBundle``，它可以使用注释来在控制器中设置访问限制：

.. code-block:: php

    use JMS\SecurityExtraBundle\Annotation\Secure;

    /**
     * @Secure(roles="ROLE_ADMIN")
     */
    public function helloAction($name)
    {
        // ...
    }

For more information, see the `JMSSecurityExtraBundle`_ documentation. If you're
using Symfony's Standard Distribution, this bundle is available by default.
If not, you can easily download and install it.
要了解更多请参阅`JMSSecurityExtraBundle`_。如果你使用的是symfony标准版，这个bundle已经默认安装了。
如果不是，请自行下载并安装。

Securing other Services
限制访问其他服务
~~~~~~~~~~~~~~~~~~~~~~~

In fact, anything in Symfony can be protected using a strategy similar to
the one seen in the previous section. For example, suppose you have a service
(i.e. a PHP class) whose job is to send emails from one user to another.
You can restrict use of this class - no matter where it's being used from -
to users that have a specific role.
事实上，任何东西都可以像以上方法那样被限制访问。比如，假设你有一个服务（如一个php类），这个服务可以
将邮件从一个用户发送到另一个用户。你可以对拥有某个角色的用户限制使用这个类——不管是在哪里使用它的。

For more information on how you can use the security component to secure
different services and methods in your application, see :doc:`/cookbook/security/securing_services`.
更多关于限制访问你的服务和方法的信息请参阅:doc:`/cookbook/security/securing_services`。

Access Control Lists (ACLs): Securing Individual Database Objects
Access Control Lists (ACLs)：限制访问个人数据库实体类
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Imagine you are designing a blog system where your users can comment on your
posts. Now, you want a user to be able to edit his own comments, but not
those of other users. Also, as the admin user, you yourself want to be able
to edit *all* comments.
假设你要设计一个博客系统，在这个系统中，用户可以对你的文章发表评论。现在，你想使这个用户可以
修改他的自己的评论，但是不能修改别人的评论。并且，作为一个管理员，你可以修改所有的评论。

The security component comes with an optional access control list (ACL) system
that you can use when you need to control access to individual instances
of an object in your system. *Without* ACL, you can secure your system so that
only certain users can edit blog comments in general. But *with* ACL, you
can restrict or allow access on a comment-by-comment basis.
安全系统有一个可选的Access Control Lists (ACLs)系统，你可以使用它来控制访问数据库的实体类（个人信息）。
假如没有ACL，你可以允许某些用户来编辑整个的博客评论。但是通过ACL，你可以根据用户的数据库信息来
决定他是否被允许访问某个评论页面。

For more information, see the cookbook article: :doc:`/cookbook/security/acl`.
请参阅:doc:`/cookbook/security/acl`。

Users
用户
-----

In the previous sections, you learned how you can protect different resources
by requiring a set of *roles* for a resource. In this section we'll explore
the other side of authorization: users.
在以上章节中，你学习了如何通过请求一系列的角色来确定用户是否能够访问某个页面。在这一节中我们
将探索授权的另一方面：用户。

Where do Users come from? (*User Providers*)
用户来自哪里？（用户提供方）
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

During authentication, the user submits a set of credentials (usually a username
and password). The job of the authentication system is to match those credentials
against some pool of users. So where does this list of users come from?
在验证过程中，用户提交了一系列的用户信息（通常是username和password）。验证的工作就是将
提交的用户信息与一个信息库中的用户信息相匹配。那么这个信息库中的用户从哪里来？

In Symfony2, users can come from anywhere - a configuration file, a database
table, a web service, or anything else you can dream up. Anything that provides
one or more users to the authentication system is known as a "user provider".
Symfony2 comes standard with the two most common user providers: one that
loads users from a configuration file and one that loads users from a database
table.
在symfony2中，用户可以来自于一个配置文件，一个数据库，一个web服务，等等。所有向验证系统提供了
用户的系统都被称为“用户提供方”。symfony2中最常用的用户提供方有两个：一个从配置文件中载入用户，一个
从数据库中载入用户。

Specifying Users in a Configuration File
在配置文件中指定用户
........................................

The easiest way to specify your users is directly in a configuration file.
In fact, you've seen this already in the example in this chapter.
最方便的方法就是在配置文件中直接指定用户。实际上，在以上章节中你已经看到了这种方法。

.. configuration-block::

    .. code-block:: yaml

        # app/config/security.yml
        security:
            # ...
            providers:
                default_provider:
                    memory:
                        users:
                            ryan:  { password: ryanpass, roles: 'ROLE_USER' }
                            admin: { password: kitten, roles: 'ROLE_ADMIN' }

    .. code-block:: xml

        <!-- app/config/security.xml -->
        <config>
            <!-- ... -->
            <provider name="default_provider">
                <memory>
                    <user name="ryan" password="ryanpass" roles="ROLE_USER" />
                    <user name="admin" password="kitten" roles="ROLE_ADMIN" />
                </memory>
            </provider>
        </config>

    .. code-block:: php

        // app/config/security.php
        $container->loadFromExtension('security', array(
            // ...
            'providers' => array(
                'default_provider' => array(
                    'memory' => array(
                        'users' => array(
                            'ryan' => array('password' => 'ryanpass', 'roles' => 'ROLE_USER'),
                            'admin' => array('password' => 'kitten', 'roles' => 'ROLE_ADMIN'),
                        ),
                    ),
                ),
            ),
        ));

This user provider is called the "in-memory" user provider, since the users
aren't stored anywhere in a database. The actual user object is provided
by Symfony (:class:`Symfony\\Component\\Security\\Core\\User\\User`).
这个用户提供方被称作in-memory用户提供方，这是因为用户没有被存储在数据库中。实际上的用户是由symfony提供的
（:class:`Symfony\\Component\\Security\\Core\\User\\User`）。

.. tip::
    Any user provider can load users directly from configuration by specifying
    the ``users`` configuration parameter and listing the users beneath it.
    所有的用户提供方都可以直接从配置中载入用户，只要设定``users``参数并将用户列在下面。

.. caution::

    If your username is completely numeric (e.g. ``77``) or contains a dash
    (e.g. ``user-name``), you should use that alternative syntax when specifying
    users in YAML:
    如果你的username是数字（比如77）或者有横杠（比如user-name），你应该在YAML中按照如下格式写：

    .. code-block:: yaml

        users:
            - { name: 77, password: pass, roles: 'ROLE_USER' }
            - { name: user-name, password: pass, roles: 'ROLE_USER' }

For smaller sites, this method is quick and easy to setup. For more complex
systems, you'll want to load your users from the database.
对于小的网站，这个方法很方便。但是对于大的网站，你需要从数据库中载入用户。

.. _book-security-user-entity:

Loading Users from the Database
从数据库中载入用户
...............................

If you'd like to load your users via the Doctrine ORM, you can easily do
this by creating a ``User`` class and configuring the ``entity`` provider.
如果你想从doctrine ORM中载入用户，可以创建一个User类并配置entity提供方。

.. tip:

    A high-quality open source bundle is available that allows your users
    to be stored via the Doctrine ORM or ODM. Read more about the `FOSUserBundle`_
    on GitHub.
    现在有一个开源bundle允许你将用户存储在doctrine ORM或ODM中。参阅`FOSUserBundle`_。

With this approach, you'll first create your own ``User`` class, which will
be stored in the database.
首先，你需要创建你的User类：

.. code-block:: php

    // src/Acme/UserBundle/Entity/User.php
    namespace Acme\UserBundle\Entity;

    use Symfony\Component\Security\Core\User\UserInterface;
    use Doctrine\ORM\Mapping as ORM;

    /**
     * @ORM\Entity
     */
    class User implements UserInterface
    {
        /**
         * @ORM\Column(type="string", length=255)
         */
        protected $username;

        // ...
    }

As far as the security system is concerned, the only requirement for your
custom user class is that it implements the :class:`Symfony\\Component\\Security\\Core\\User\\UserInterface`
interface. This means that your concept of a "user" can be anything, as long
as it implements this interface.
唯一的要求就是需要你在该类中植入:class:`Symfony\\Component\\Security\\Core\\User\\UserInterface`这个interface。
这表示你的user可以是任何东西，只要植入这个interface就可以了。

.. versionadded:: 2.1

    In Symfony 2.1, the ``equals`` method was removed from ``UserInterface``.
    If you need to override the default implementation of comparison logic,
    implement the new :class:`Symfony\\Component\\Security\\Core\\User\\EquatableInterface`
    interface.
    在symfony2.1中，equals方法从``UserInterface``中被移除了。如果你需要覆盖其中植入的默认的比较逻辑，
    请植入新的:class:`Symfony\\Component\\Security\\Core\\User\\EquatableInterface` interface。

.. note::

    The user object will be serialized and saved in the session during requests,
    therefore it is recommended that you `implement the \Serializable interface`_
    in your user object. This is especially important if your ``User`` class
    has a parent class with private properties.
    在请求过程中，user对象会被序列化并存储在session中，所以推荐在你的user对象中`implement the \Serializable interface`_。
    如果你的User类有一个父类，并这个父类有private属性，这一点尤其重要。

Next, configure an ``entity`` user provider, and point it to your ``User``
class:
下面，配置一个entity用户提供方，并将它指向你的User类：

.. configuration-block::

    .. code-block:: yaml

        # app/config/security.yml
        security:
            providers:
                main:
                    entity: { class: Acme\UserBundle\Entity\User, property: username }

    .. code-block:: xml

        <!-- app/config/security.xml -->
        <config>
            <provider name="main">
                <entity class="Acme\UserBundle\Entity\User" property="username" />
            </provider>
        </config>

    .. code-block:: php

        // app/config/security.php
        $container->loadFromExtension('security', array(
            'providers' => array(
                'main' => array(
                    'entity' => array('class' => 'Acme\UserBundle\Entity\User', 'property' => 'username'),
                ),
            ),
        ));

With the introduction of this new provider, the authentication system will
attempt to load a ``User`` object from the database by using the ``username``
field of that class.
由于指定了用户提供方，验证系统会从数据库中通过username这个字段载入User对象。

.. note::
    This example is just meant to show you the basic idea behind the ``entity``
    provider. For a full working example, see :doc:`/cookbook/security/entity_provider`.
    这个例子仅仅向你展示了entity提供方背后最基本的逻辑。更多范例请参阅:doc:`/cookbook/security/entity_provider`。

For more information on creating your own custom provider (e.g. if you needed
to load users via a web service), see :doc:`/cookbook/security/custom_provider`.
要创建你自己的提供方，请参阅:doc:`/cookbook/security/custom_provider`。

.. _book-security-encoding-user-password:

Encoding the User's Password
给用户密码加密
~~~~~~~~~~~~~~~~~~~~~~~~~~~~

So far, for simplicity, all the examples have stored the users' passwords
in plain text (whether those users are stored in a configuration file or in
a database somewhere). Of course, in a real application, you'll want to encode
your users' passwords for security reasons. This is easily accomplished by
mapping your User class to one of several built-in "encoders". For example,
to store your users in memory, but obscure their passwords via ``sha1``,
do the following:
目前为止，为了简化，所有的范例都使用普通文本来存储用户的password（不管用户被存储在配置文件或数据库中）。当然，
在实际应用中，为了安全考虑，你需要给用户的password加密。只要将你的User对象映射到encoders中就可以了。
比如，要将用户存储在memory中，但要通过sha1加密他们的password，只要：

.. configuration-block::

    .. code-block:: yaml

        # app/config/security.yml
        security:
            # ...
            providers:
                in_memory:
                    memory:
                        users:
                            ryan:  { password: bb87a29949f3a1ee0559f8a57357487151281386, roles: 'ROLE_USER' }
                            admin: { password: 74913f5cd5f61ec0bcfdb775414c2fb3d161b620, roles: 'ROLE_ADMIN' }

            encoders:
                Symfony\Component\Security\Core\User\User:
                    algorithm:   sha1
                    iterations: 1
                    encode_as_base64: false

    .. code-block:: xml

        <!-- app/config/security.xml -->
        <config>
            <!-- ... -->
            <provider name="in_memory">
                <memory>
                    <user name="ryan" password="bb87a29949f3a1ee0559f8a57357487151281386" roles="ROLE_USER" />
                    <user name="admin" password="74913f5cd5f61ec0bcfdb775414c2fb3d161b620" roles="ROLE_ADMIN" />
                </memory>
            </provider>

            <encoder class="Symfony\Component\Security\Core\User\User" algorithm="sha1" iterations="1" encode_as_base64="false" />
        </config>

    .. code-block:: php

        // app/config/security.php
        $container->loadFromExtension('security', array(
            // ...
            'providers' => array(
                'in_memory' => array(
                    'memory' => array(
                        'users' => array(
                            'ryan' => array('password' => 'bb87a29949f3a1ee0559f8a57357487151281386', 'roles' => 'ROLE_USER'),
                            'admin' => array('password' => '74913f5cd5f61ec0bcfdb775414c2fb3d161b620', 'roles' => 'ROLE_ADMIN'),
                        ),
                    ),
                ),
            ),
            'encoders' => array(
                'Symfony\Component\Security\Core\User\User' => array(
                    'algorithm'         => 'sha1',
                    'iterations'        => 1,
                    'encode_as_base64'  => false,
                ),
            ),
        ));

By setting the ``iterations`` to ``1`` and the ``encode_as_base64`` to false,
the password is simply run through the ``sha1`` algorithm one time and without
any extra encoding. You can now calculate the hashed password either programmatically
(e.g. ``hash('sha1', 'ryanpass')``) or via some online tool like `functions-online.com`_
通过将iterations设置为1并将encode_as_base64设置为false，password就会通过sha1加密一次。要
解密password，可以通过程序（e.g. ``hash('sha1', 'ryanpass')``），或在线工具，如`functions-online.com`_。

If you're creating your users dynamically (and storing them in a database),
you can use even tougher hashing algorithms and then rely on an actual password
encoder object to help you encode passwords. For example, suppose your User
object is ``Acme\UserBundle\Entity\User`` (like in the above example). First,
configure the encoder for that user:
如果你动态创建用户（并将他们存储在数据库），你可以使用更复杂的方法和encoder对象来加密。比如，
假设你的User对象是``Acme\UserBundle\Entity\User``，首先，为那个用户配置encoders：

.. configuration-block::

    .. code-block:: yaml

        # app/config/security.yml
        security:
            # ...

            encoders:
                Acme\UserBundle\Entity\User: sha512

    .. code-block:: xml

        <!-- app/config/security.xml -->
        <config>
            <!-- ... -->

            <encoder class="Acme\UserBundle\Entity\User" algorithm="sha512" />
        </config>

    .. code-block:: php

        // app/config/security.php
        $container->loadFromExtension('security', array(
            // ...

            'encoders' => array(
                'Acme\UserBundle\Entity\User' => 'sha512',
            ),
        ));

In this case, you're using the stronger ``sha512`` algorithm. Also, since
you've simply specified the algorithm (``sha512``) as a string, the system
will default to hashing your password 5000 times in a row and then encoding
it as base64. In other words, the password has been greatly obfuscated so
that the hashed password can't be decoded (i.e. you can't determine the password
from the hashed password).
在这个例子中，你使用了一种更复杂的sha512方法。系统会默认将你的密码加密5000次并解析为base64。
换句话说，密码不能被解密了。

If you have some sort of registration form for users, you'll need to be able
to determine the hashed password so that you can set it on your user. No
matter what algorithm you configure for your user object, the hashed password
can always be determined in the following way from a controller:
如果用户通过表单注册，你需要确定被加密后的密码，从而能添加到数据库。不管你配置了什么
加密方法，在控制器中，你可以用以下方法来确定加密密码：

.. code-block:: php

    $factory = $this->get('security.encoder_factory');
    $user = new Acme\UserBundle\Entity\User();

    $encoder = $factory->getEncoder($user);
    $password = $encoder->encodePassword('ryanpass', $user->getSalt());
    $user->setPassword($password);

Retrieving the User Object
获取User对象
~~~~~~~~~~~~~~~~~~~~~~~~~~

After authentication, the ``User`` object of the current user can be accessed
via the ``security.context`` service. From inside a controller, this will
look like:
在验证后，当前用户的User对象可以通过``security.context``服务访问。在控制器中，像这样：

.. code-block:: php

    public function indexAction()
    {
        $user = $this->get('security.context')->getToken()->getUser();
    }

In a controller this can be shortcut to:
以下是便捷方法：

.. code-block:: php

    public function indexAction()
    {
        $user = $this->getUser();
    }


.. note::

    Anonymous users are technically authenticated, meaning that the ``isAuthenticated()``
    method of an anonymous user object will return true. To check if your
    user is actually authenticated, check for the ``IS_AUTHENTICATED_FULLY``
    role.
    匿名用户技术上是被验证过的，这表示``isAuthenticated()``方法对于一个匿名用户对象会返回true。
    要检查你的用户是否确实通过了验证，检查``IS_AUTHENTICATED_FULLY``角色。
    
In a Twig Template this object can be accessed via the ``app.user`` key,
which calls the :method:`GlobalVariables::getUser()<Symfony\\Bundle\\FrameworkBundle\\Templating\\GlobalVariables::getUser>`
method:
在twig模板中这个对象可以通过app.user访问，它会执行:method:`GlobalVariables::getUser()<Symfony\\Bundle\\FrameworkBundle\\Templating\\GlobalVariables::getUser>`。

.. configuration-block::

    .. code-block:: html+jinja

        <p>Username: {{ app.user.username }}</p>


Using Multiple User Providers
使用多个用户提供方
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Each authentication mechanism (e.g. HTTP Authentication, form login, etc)
uses exactly one user provider, and will use the first declared user provider
by default. But what if you want to specify a few users via configuration
and the rest of your users in the database? This is possible by creating
a new provider that chains the two together:
每个验证体系（e.g. HTTP Authentication, form login, etc）使用仅仅一个用户提供方，并且会默认使用
第一个用户提供方。但是如果你需要通过配置指定一些用户，而剩下的通过数据库指定呢？要达到这个目的，
可以创建一个提供方将两个串联在一起：

.. configuration-block::

    .. code-block:: yaml

        # app/config/security.yml
        security:
            providers:
                chain_provider:
                    chain:
                        providers: [in_memory, user_db]
                in_memory:
                    users:
                        foo: { password: test }
                user_db:
                    entity: { class: Acme\UserBundle\Entity\User, property: username }

    .. code-block:: xml

        <!-- app/config/security.xml -->
        <config>
            <provider name="chain_provider">
                <chain>
                    <provider>in_memory</provider>
                    <provider>user_db</provider>
                </chain>
            </provider>
            <provider name="in_memory">
                <user name="foo" password="test" />
            </provider>
            <provider name="user_db">
                <entity class="Acme\UserBundle\Entity\User" property="username" />
            </provider>
        </config>

    .. code-block:: php

        // app/config/security.php
        $container->loadFromExtension('security', array(
            'providers' => array(
                'chain_provider' => array(
                    'chain' => array(
                        'providers' => array('in_memory', 'user_db'),
                    ),
                ),
                'in_memory' => array(
                    'users' => array(
                        'foo' => array('password' => 'test'),
                    ),
                ),
                'user_db' => array(
                    'entity' => array('class' => 'Acme\UserBundle\Entity\User', 'property' => 'username'),
                ),
            ),
        ));

Now, all authentication mechanisms will use the ``chain_provider``, since
it's the first specified. The ``chain_provider`` will, in turn, try to load
the user from both the ``in_memory`` and ``user_db`` providers.
现在，所有的验证系统都会使用``chain_provider``，因为它是第一个被指定的。现在``chain_provider``
会试图从``in_memory``和``user_db``两处提供方载入用户。

.. tip::

    If you have no reasons to separate your ``in_memory`` users from your
    ``user_db`` users, you can accomplish this even more easily by combining
    the two sources into a single provider:
    如果你的``in_memory``和``user_db``的用户不分开，你可以更容易地将两者结合：

    .. configuration-block::

        .. code-block:: yaml

            # app/config/security.yml
            security:
                providers:
                    main_provider:
                        memory:
                            users:
                                foo: { password: test }
                        entity:
                            class: Acme\UserBundle\Entity\User,
                            property: username

        .. code-block:: xml

            <!-- app/config/security.xml -->
            <config>
                <provider name=="main_provider">
                    <memory>
                        <user name="foo" password="test" />
                    </memory>
                    <entity class="Acme\UserBundle\Entity\User" property="username" />
                </provider>
            </config>

        .. code-block:: php

            // app/config/security.php
            $container->loadFromExtension('security', array(
                'providers' => array(
                    'main_provider' => array(
                        'memory' => array(
                            'users' => array(
                                'foo' => array('password' => 'test'),
                            ),
                        ),
                        'entity' => array('class' => 'Acme\UserBundle\Entity\User', 'property' => 'username'),
                    ),
                ),
            ));

You can also configure the firewall or individual authentication mechanisms
to use a specific provider. Again, unless a provider is specified explicitly,
the first provider is always used:
你还可以配置防火墙，使它使用指定的提供方。再说明一次，除非一个提供方被显性地指明，否则总是
使用第一个提供方：

.. configuration-block::

    .. code-block:: yaml

        # app/config/security.yml
        security:
            firewalls:
                secured_area:
                    # ...
                    provider: user_db
                    http_basic:
                        realm: "Secured Demo Area"
                        provider: in_memory
                    form_login: ~

    .. code-block:: xml

        <!-- app/config/security.xml -->
        <config>
            <firewall name="secured_area" pattern="^/" provider="user_db">
                <!-- ... -->
                <http-basic realm="Secured Demo Area" provider="in_memory" />
                <form-login />
            </firewall>
        </config>

    .. code-block:: php

        // app/config/security.php
        $container->loadFromExtension('security', array(
            'firewalls' => array(
                'secured_area' => array(
                    // ...
                    'provider' => 'user_db',
                    'http_basic' => array(
                        // ...
                        'provider' => 'in_memory',
                    ),
                    'form_login' => array(),
                ),
            ),
        ));

In this example, if a user tries to login via HTTP authentication, the authentication
system will use the ``in_memory`` user provider. But if the user tries to
login via the form login, the ``user_db`` provider will be used (since it's
the default for the firewall as a whole).
在这个例子中，如果一个用户要通过HTTP验证登陆，验证系统会使用in_memory用户提供方。
但是如果用户想要通过表单登陆，验证系统就会使用user_db提供方（这是整个防火墙默认的提供方）。

For more information about user provider and firewall configuration, see
the :doc:`/reference/configuration/security`.
要了解更多有关用户提供方和防火墙配置的信息请参阅:doc:`/reference/configuration/security`。

Roles
角色
-----

The idea of a "role" is key to the authorization process. Each user is assigned
a set of roles and then each resource requires one or more roles. If the user
has the required roles, access is granted. Otherwise access is denied.
角色是授权过程的关键。每个用户都被指定了一系列的角色，并且每个页面都要求一个或多个角色。
如果用户拥有要求的角色，他就允许访问，否则就不被允许。

Roles are pretty simple, and are basically strings that you can invent and
use as needed (though roles are objects internally). For example, if you
need to start limiting access to the blog admin section of your website,
you could protect that section using a ``ROLE_BLOG_ADMIN`` role. This role
doesn't need to be defined anywhere - you can just start using it.
roles仅仅是字符串，你可以根据需要任意指定（虽然roles在内部其实是对象）。比如，如果你
想对博客系统的admin部分设置访问权限，你可以使用一个``ROLE_BLOG_ADMIN``角色。
这个角色不必在别的地方定义，只要使用它就可以了。

.. note::

    All roles **must** begin with the ``ROLE_`` prefix to be managed by
    Symfony2. If you define your own roles with a dedicated ``Role`` class
    (more advanced), don't use the ``ROLE_`` prefix.
    所有的角色都必须以``ROLE_``作为前缀，这样才能被symfony2处理。如果你要使用一个Role类来定义你
    自己的角色，就不要使用``ROLE_``前缀。

Hierarchical Roles
角色继承
~~~~~~~~~~~~~~~~~~

Instead of associating many roles to users, you can define role inheritance
rules by creating a role hierarchy:
你可以定义角色继承：

.. configuration-block::

    .. code-block:: yaml

        # app/config/security.yml
        security:
            role_hierarchy:
                ROLE_ADMIN:       ROLE_USER
                ROLE_SUPER_ADMIN: [ROLE_ADMIN, ROLE_ALLOWED_TO_SWITCH]

    .. code-block:: xml

        <!-- app/config/security.xml -->
        <config>
            <role id="ROLE_ADMIN">ROLE_USER</role>
            <role id="ROLE_SUPER_ADMIN">ROLE_ADMIN, ROLE_ALLOWED_TO_SWITCH</role>
        </config>

    .. code-block:: php

        // app/config/security.php
        $container->loadFromExtension('security', array(
            'role_hierarchy' => array(
                'ROLE_ADMIN'       => 'ROLE_USER',
                'ROLE_SUPER_ADMIN' => array('ROLE_ADMIN', 'ROLE_ALLOWED_TO_SWITCH'),
            ),
        ));

In the above configuration, users with ``ROLE_ADMIN`` role will also have the
``ROLE_USER`` role. The ``ROLE_SUPER_ADMIN`` role has ``ROLE_ADMIN``, ``ROLE_ALLOWED_TO_SWITCH``
and ``ROLE_USER`` (inherited from ``ROLE_ADMIN``).
在以上范例中，拥有``ROLE_ADMIN``同时也就有了``ROLE_USER``角色。``ROLE_SUPER_ADMIN``角色同时有
``ROLE_ADMIN``, ``ROLE_ALLOWED_TO_SWITCH``和``ROLE_USER``角色（继承至``ROLE_ADMIN``）。

Logging Out
登出
-----------

Usually, you'll also want your users to be able to log out. Fortunately,
the firewall can handle this automatically for you when you activate the
``logout`` config parameter:
用户还要能够登出。防火墙可以自动处理这类工作，只要你激活logout配置参数：

.. configuration-block::

    .. code-block:: yaml

        # app/config/security.yml
        security:
            firewalls:
                secured_area:
                    # ...
                    logout:
                        path:   /logout
                        target: /
            # ...

    .. code-block:: xml

        <!-- app/config/security.xml -->
        <config>
            <firewall name="secured_area" pattern="^/">
                <!-- ... -->
                <logout path="/logout" target="/" />
            </firewall>
            <!-- ... -->
        </config>

    .. code-block:: php

        // app/config/security.php
        $container->loadFromExtension('security', array(
            'firewalls' => array(
                'secured_area' => array(
                    // ...
                    'logout' => array('path' => 'logout', 'target' => '/'),
                ),
            ),
            // ...
        ));

Once this is configured under your firewall, sending a user to ``/logout``
(or whatever you configure the ``path`` to be), will un-authenticate the
current user. The user will then be sent to the homepage (the value defined
by the ``target`` parameter). Both the ``path`` and ``target`` config parameters
default to what's specified here. In other words, unless you need to customize
them, you can omit them entirely and shorten your configuration:
一旦你的防火墙有了这个配置，将一个用户定向至/logout（或者任何你在path中配置的路径）会使用户登出。
然后用户会被定向至主页（在target中配置的值）。这里定义的就是默认的path和target的值，所以，你可以直接用以下的
便捷方式，除非你需要自定义它们：

.. configuration-block::

    .. code-block:: yaml

        logout: ~

    .. code-block:: xml

        <logout />

    .. code-block:: php

        'logout' => array(),

Note that you will *not* need to implement a controller for the ``/logout``
URL as the firewall takes care of everything. You may, however, want to create
a route so that you can use it to generate the URL:
注意你不需要为/logout植入一个控制器，防火墙会自动完成一切工作。你需要创建一个路径来集成URL：

.. configuration-block::

    .. code-block:: yaml

        # app/config/routing.yml
        logout:
            pattern:   /logout

    .. code-block:: xml

        <!-- app/config/routing.xml -->
        <?xml version="1.0" encoding="UTF-8" ?>

        <routes xmlns="http://symfony.com/schema/routing"
            xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
            xsi:schemaLocation="http://symfony.com/schema/routing http://symfony.com/schema/routing/routing-1.0.xsd">

            <route id="logout" pattern="/logout" />

        </routes>

    ..  code-block:: php

        // app/config/routing.php
        use Symfony\Component\Routing\RouteCollection;
        use Symfony\Component\Routing\Route;

        $collection = new RouteCollection();
        $collection->add('logout', new Route('/logout', array()));

        return $collection;

Once the user has been logged out, he will be redirected to whatever path
is defined by the ``target`` parameter above (e.g. the ``homepage``). For
more information on configuring the logout, see the
:doc:`Security Configuration Reference</reference/configuration/security>`.
一旦用户登出，他会被定向至target中定义的路径。要了解更多有关logout的信息请参阅
:doc:`Security Configuration Reference</reference/configuration/security>`。

Access Control in Templates
模板中的访问权限
---------------------------

If you want to check if the current user has a role inside a template, use
the built-in helper function:
如果你需要检查当前的用户在一个模板中是否拥有角色，可以使用内置的helper方法：

.. configuration-block::

    .. code-block:: html+jinja

        {% if is_granted('ROLE_ADMIN') %}
            <a href="...">Delete</a>
        {% endif %}

    .. code-block:: html+php

        <?php if ($view['security']->isGranted('ROLE_ADMIN')): ?>
            <a href="...">Delete</a>
        <?php endif; ?>

.. note::

    If you use this function and are *not* at a URL where there is a firewall
    active, an exception will be thrown. Again, it's almost always a good
    idea to have a main firewall that covers all URLs (as has been shown
    in this chapter).
    如果你在一个防火墙没有激活的URL中使用这个方法，一个错误会被抛出。所以，最好是有
    一个防火墙覆盖所有的URL（本章已列举该范例）。

Access Control in Controllers
控制器中的访问权限
-----------------------------

If you want to check if the current user has a role in your controller, use
the ``isGranted`` method of the security context:
如果你需要检查当前用户在控制器中是否有权限，使用security context中的``isGranted``方法：

.. code-block:: php

    public function indexAction()
    {
        // show different content to admin users
        if ($this->get('security.context')->isGranted('ROLE_ADMIN')) {
            // Load admin content here
        }
        // load other regular content here
    }

.. note::

    A firewall must be active or an exception will be thrown when the ``isGranted``
    method is called. See the note above about templates for more details.
    像上述模板中一样，该URL中的防火墙应该被激活，否则会抛出错误。

Impersonating a User
切换用户
--------------------

Sometimes, it's useful to be able to switch from one user to another without
having to logout and login again (for instance when you are debugging or trying
to understand a bug a user sees that you can't reproduce). This can be easily
done by activating the ``switch_user`` firewall listener:
有时候你需要从一个用户切换到另一个用户，而不必登陆以及登出（比如当你调试的时候）。
要达到这个目的，可以配置防火墙中的``switch_user``参数：

.. configuration-block::

    .. code-block:: yaml

        # app/config/security.yml
        security:
            firewalls:
                main:
                    # ...
                    switch_user: true

    .. code-block:: xml

        <!-- app/config/security.xml -->
        <config>
            <firewall>
                <!-- ... -->
                <switch-user />
            </firewall>
        </config>

    .. code-block:: php

        // app/config/security.php
        $container->loadFromExtension('security', array(
            'firewalls' => array(
                'main'=> array(
                    // ...
                    'switch_user' => true
                ),
            ),
        ));

To switch to another user, just add a query string with the ``_switch_user``
parameter and the username as the value to the current URL:
要想切换到另一个用户，只要在当前URL后面添加一个具有``_switch_user``参数和username的请求语句：

    http://example.com/somewhere?_switch_user=thomas

To switch back to the original user, use the special ``_exit`` username:
要切换会原来的用户，只要将``_exit``作为username使用：

    http://example.com/somewhere?_switch_user=_exit

Of course, this feature needs to be made available to a small group of users.
By default, access is restricted to users having the ``ROLE_ALLOWED_TO_SWITCH``
role. The name of this role can be modified via the ``role`` setting. For
extra security, you can also change the query parameter name via the ``parameter``
setting:
当然，这个功能要对一个用户群有效。默认情况下，拥有``ROLE_ALLOWED_TO_SWITCH``角色的用户
才被允许访问。这个角色的名称可以通过role参数修改。你还可以通过parameter修改请求参数的名称：

.. configuration-block::

    .. code-block:: yaml

        # app/config/security.yml
        security:
            firewalls:
                main:
                    // ...
                    switch_user: { role: ROLE_ADMIN, parameter: _want_to_be_this_user }

    .. code-block:: xml

        <!-- app/config/security.xml -->
        <config>
            <firewall>
                <!-- ... -->
                <switch-user role="ROLE_ADMIN" parameter="_want_to_be_this_user" />
            </firewall>
        </config>

    .. code-block:: php

        // app/config/security.php
        $container->loadFromExtension('security', array(
            'firewalls' => array(
                'main'=> array(
                    // ...
                    'switch_user' => array('role' => 'ROLE_ADMIN', 'parameter' => '_want_to_be_this_user'),
                ),
            ),
        ));

Stateless Authentication
stateless验证
------------------------

By default, Symfony2 relies on a cookie (the Session) to persist the security
context of the user. But if you use certificates or HTTP authentication for
instance, persistence is not needed as credentials are available for each
request. In that case, and if you don't need to store anything else between
requests, you can activate the stateless authentication (which means that no
cookie will be ever created by Symfony2):
默认情况下，symfony2通过cookie（session）来存储用户的security context。但是如果你使用HTTP验证的话，
这一般是不必的，因为对于每个请求这个信息都存在。这样，如果你不需要在请求之间存储任何其他信息，你可以激活
stateless验证（这表示symfony2不会创建cookie）。

.. configuration-block::

    .. code-block:: yaml

        # app/config/security.yml
        security:
            firewalls:
                main:
                    http_basic: ~
                    stateless:  true

    .. code-block:: xml

        <!-- app/config/security.xml -->
        <config>
            <firewall stateless="true">
                <http-basic />
            </firewall>
        </config>

    .. code-block:: php

        // app/config/security.php
        $container->loadFromExtension('security', array(
            'firewalls' => array(
                'main' => array('http_basic' => array(), 'stateless' => true),
            ),
        ));

.. note::

    If you use a form login, Symfony2 will create a cookie even if you set
    ``stateless`` to ``true``.
    如果你使用一个表单登陆验证的话，即使你将stateless设置为true，symfony2也会创建一个cookie。

Final Words
总结
-----------

Security can be a deep and complex issue to solve correctly in your application.
Fortunately, Symfony's security component follows a well-proven security
model based around *authentication* and *authorization*. Authentication,
which always happens first, is handled by a firewall whose job is to determine
the identity of the user through several different methods (e.g. HTTP authentication,
login form, etc). In the cookbook, you'll find examples of other methods
for handling authentication, including how to implement a "remember me" cookie
functionality.
symfony安全系统遵循了一个良好的验证和授权模式。验证是第一步，通过防火墙来确定一个用户的身份。
在cookbook中，你将会发现其他处理验证的方法，包括如何植入一个“remember me” cookie功能。

Once a user is authenticated, the authorization layer can determine whether
or not the user should have access to a specific resource. Most commonly,
*roles* are applied to URLs, classes or methods and if the current user
doesn't have that role, access is denied. The authorization layer, however,
is much deeper, and follows a system of "voting" so that multiple parties
can determine if the current user should have access to a given resource.
Find out more about this and other topics in the cookbook.
一旦用户通过验证，授权系统会确定该用户是否有进入某个页面的权限。roles被应用于URL、类、
或方法。如果当前用户没有那个角色，他将不能进入页面。当然，授权层还有更多值得发掘的东西。
请参阅cookbook。

Learn more from the Cookbook
----------------------------

* :doc:`Forcing HTTP/HTTPS </cookbook/security/force_https>`
* :doc:`Blacklist users by IP address with a custom voter </cookbook/security/voters>`
* :doc:`Access Control Lists (ACLs) </cookbook/security/acl>`
* :doc:`/cookbook/security/remember_me`

.. _`security component`: https://github.com/symfony/Security
.. _`JMSSecurityExtraBundle`: https://github.com/schmittjoh/JMSSecurityExtraBundle
.. _`FOSUserBundle`: https://github.com/FriendsOfSymfony/FOSUserBundle
.. _`implement the \Serializable interface`: http://php.net/manual/en/class.serializable.php
.. _`functions-online.com`: http://www.functions-online.com/sha1.html
