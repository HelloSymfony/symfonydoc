How to customize Error Pages
如何定制错误页面
============================

When any exception is thrown in Symfony2, the exception is caught inside the
``Kernel`` class and eventually forwarded to a special controller,
``TwigBundle:Exception:show`` for handling. This controller, which lives
inside the core ``TwigBundle``, determines which error template to display and
the status code that should be set for the given exception.
当symfony2抛出任何错误时，都由Kernel类进行接并最终转发给一个特定的控制器（``TwigBundle:Exception:show``）来处理。
这个控制器被放置在核心``TwigBundle``中，它会决定要显示哪个错误模板以及要为这个给定的exception设置哪个status code。

Error pages can be customized in two different ways, depending on how much
control you need:
错误页面可以通过两个不同方法来定制：

1. Customize the error templates of the different error pages (explained below);
1. 定制不同错误页面的错误模板（以下将解释）；

2. Replace the default exception controller ``TwigBundle::Exception:show``
   with your own controller and handle it however you want (see
   :ref:`exception_controller in the Twig reference<config-twig-exception-controller>`);
2. 将默认的exception控制器``TwigBundle::Exception:show``替换为你自己的控制器并由你来处理它（参见
   :ref:`exception_controller in the Twig reference<config-twig-exception-controller>`）。

.. tip::

    The customization of exception handling is actually much more powerful
    than what's written here. An internal event, ``kernel.exception``, is thrown
    which allows complete control over exception handling. For more
    information, see :ref:`kernel-kernel.exception`.
    当错误发生时，一个``kernel.exception``内部事件会被抛出，它允许你完全控制错误处理。
    详情请见:ref:`kernel-kernel.exception`。

All of the error templates live inside ``TwigBundle``. To override the
templates, we simply rely on the standard method for overriding templates that
live inside a bundle. For more information, see
:ref:`overriding-bundle-templates`.
所欲的错误模板都被放置在``TwigBundle``中。要覆盖这些模板，我们可以使用标准覆盖方法，参见
:ref:`overriding-bundle-templates`。

For example, to override the default error template that's shown to the
end-user, create a new template located at
``app/Resources/TwigBundle/views/Exception/error.html.twig``:
比如，要覆盖默认的错误模板，可以在``app/Resources/TwigBundle/views/Exception/error.html.twig``目录下创建一个新的模板：

.. code-block:: html+jinja

    <!DOCTYPE html>
    <html>
    <head>
        <meta http-equiv="Content-Type" content="text/html; charset=utf-8" />
        <title>An Error Occurred: {{ status_text }}</title>
    </head>
    <body>
        <h1>Oops! An Error Occurred</h1>
        <h2>The server returned a "{{ status_code }} {{ status_text }}".</h2>
    </body>
    </html>

.. tip::

    If you're not familiar with Twig, don't worry. Twig is a simple, powerful
    and optional templating engine that integrates with ``Symfony2``. For more
    information about Twig see :doc:`/book/templating`.
    如果你对twig不熟悉，请参见:doc:`/book/templating`。

In addition to the standard HTML error page, Symfony provides a default error
page for many of the most common response formats, including JSON
(``error.json.twig``), XML, (``error.xml.twig``), and even Javascript
(``error.js.twig``), to name a few. To override any of these templates, just
create a new file with the same name in the
``app/Resources/TwigBundle/views/Exception`` directory. This is the standard
way of overriding any template that lives inside a bundle.
除了标准的HTML错误页面，symfony2还提供多种格式的默认错误页面，包括JSON(``error.json.twig``)，XML(``error.xml.twig``)，
甚至javascript(``error.js.twig``)，等等。要覆盖这些模板，只要在``app/Resources/TwigBundle/views/Exception``目录下
创建一个有同样名称的新文件就可以了。


.. _cookbook-error-pages-by-status-code:

Customizing the 404 Page and other Error Pages
定制404页面和其他错误页面
----------------------------------------------

You can also customize specific error templates according to the HTTP status
code. For instance, create a
``app/Resources/TwigBundle/views/Exception/error404.html.twig`` template to
display a special page for 404 (page not found) errors.
你还可以根据HTTP status code来定制错误页面。比如，创建一个``app/Resources/TwigBundle/views/Exception/error404.html.twig``
模板来显示一个404错误页面（page not found)。

Symfony uses the following algorithm to determine which template to use:
symfony使用以下流程来决定要使用哪个模板：

* First, it looks for a template for the given format and status code (like
  ``error404.json.twig``);
  首先，它会查找给定格式和status code的模板（比如``error404.json.twig``）。

* If it does not exist, it looks for a template for the given format (like
  ``error.json.twig``);
  如果不存在，它会查找给定格式的模板（如``error.json.twig``）；

* If it does not exist, it falls back to the HTML template (like
  ``error.html.twig``).
  如果还不存在，它会使用备选的HTML模板（如``error.html.twig``）。

.. tip::

    To see the full list of default error templates, see the
    ``Resources/views/Exception`` directory of the ``TwigBundle``. In a
    standard Symfony2 installation, the ``TwigBundle`` can be found at
    ``vendor/symfony/symfony/src/Symfony/Bundle/TwigBundle``. Often, the easiest way
    to customize an error page is to copy it from the ``TwigBundle`` into
    ``app/Resources/TwigBundle/views/Exception`` and then modify it.
    所有默认错误模板都在``TwigBundle``中的``Resources/views/Exception``目录下。
    如果你使用symfony2标准安装，``TwigBundle``就在``vendor/symfony/symfony/src/Symfony/Bundle/TwigBundle``
    目录下。定制错误页面最便捷的方式就是将它从``TwigBundle``中复制并粘贴到``app/Resources/TwigBundle/views/Exception``中，
    然后修改它。

.. note::

    The debug-friendly exception pages shown to the developer can even be
    customized in the same way by creating templates such as
    ``exception.html.twig`` for the standard HTML exception page or
    ``exception.json.twig`` for the JSON exception page.
    对开发者的调试错误页面也可以使用同样方法定制，为HTML错误页面创建``exception.html.twig``
    或为JSON错误页面创建``exception.json.twig``。
