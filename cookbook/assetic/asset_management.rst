.. index::
   single: Assetic; Introduction

How to Use Assetic for Asset Management
如何使用assetic管理asset
=======================================

Assetic combines two major ideas: assets and filters. The assets are files
such as CSS, JavaScript and image files. The filters are things that can
be applied to these files before they are served to the browser. This allows
a separation between the asset files stored in the application and the files
actually presented to the user.
assetic包括了两个概念：asset和过滤器（filter）。asset是像css、javascript、图像那样的文件。
过滤器在将这些文件传到浏览器前对它们进行过滤。这使得存储在应用端的asset能与发送给客户端
的asset隔离。


Without Assetic, you just serve the files that are stored in the application
directly:
如果没有assetic，你只能直接发送存储在应用中的文件：

.. configuration-block::

    .. code-block:: html+jinja

        <script src="{{ asset('js/script.js') }}" type="text/javascript" />

    .. code-block:: php

        <script src="<?php echo $view['assets']->getUrl('js/script.js') ?>"
                type="text/javascript" />

But *with* Assetic, you can manipulate these assets however you want (or
load them from anywhere) before serving them. These means you can:
但是通过assetic，你可以使用你想要的方式来操作它们。这表示你可以：

* Minify and combine all of your CSS and JS files
* 缩小你的css和js文件大小

* Run all (or just some) of your CSS or JS files through some sort of compiler,
  such as LESS, SASS or CoffeeScript
* 通过某种编译器来运行你的css或js文件，比如LESS, SASS或CoffeeScript

* Run image optimizations on your images
* 优化你的图像设置

Assets
------

Using Assetic provides many advantages over directly serving the files.
The files do not need to be stored where they are served from and can be
drawn from various sources such as from within a bundle:
使用assetic能提供比直接发送这些文件更好的方式。这些文件不必被存储在它们所被发送的地方，
并且从一个bundle中的任何地方都可以提取。

.. configuration-block::

    .. code-block:: html+jinja

        {% javascripts
            '@AcmeFooBundle/Resources/public/js/*'
        %}
        <script type="text/javascript" src="{{ asset_url }}"></script>
        {% endjavascripts %}

    .. code-block:: html+php

        <?php foreach ($view['assetic']->javascripts(
            array('@AcmeFooBundle/Resources/public/js/*')) as $url): ?>
        <script type="text/javascript" src="<?php echo $view->escape($url) ?>"></script>
        <?php endforeach; ?>

.. tip::

    To bring in CSS stylesheets, you can use the same methodologies seen
    in this entry, except with the `stylesheets` tag:
    要使用css样式表，你可以使用相同的方法，但要使用stylesheets标签：

    .. configuration-block::

        .. code-block:: html+jinja

            {% stylesheets
                '@AcmeFooBundle/Resources/public/css/*'
            %}
            <link rel="stylesheet" href="{{ asset_url }}" />
            {% endstylesheets %}

        .. code-block:: html+php

            <?php foreach ($view['assetic']->stylesheets(
                array('@AcmeFooBundle/Resources/public/css/*')) as $url): ?>
            <link rel="stylesheet" href="<?php echo $view->escape($url) ?>" />
            <?php endforeach; ?>

In this example, all of the files in the ``Resources/public/js/`` directory
of the ``AcmeFooBundle`` will be loaded and served from a different location.
The actual rendered tag might simply look like:
在这个例子中，所有在``AcmeFooBundle``中``Resources/public/js/``目录中的文件会被从一个不同的地方
加载。这会提交一个这样的标签：

.. code-block:: html

    <script src="/app_dev.php/js/abcd123.js"></script>

.. note::

    This is a key point: once you let Assetic handle your assets, the files are
    served from a different location. This *can* cause problems with CSS files
    that reference images by their relative path. However, this can be fixed
    by using the ``cssrewrite`` filter, which updates paths in CSS files
    to reflect their new location.
    这是关键：一旦你让assetic来操作你的asset，这些文件就会被从一个不同的地方加载。如果css文件
    会通过相对路径索引到某个图像，这可能会导致问题。但是只要使用``cssrewrite``过滤器就可以了，
    它可以在css文件中更新路径，从而指向它们的新地点。

Combining Assets
合并asset
~~~~~~~~~~~~~~~~

You can also combine several files into one. This helps to reduce the number
of HTTP requests, which is great for front end performance. It also allows
you to maintain the files more easily by splitting them into manageable parts.
This can help with re-usability as you can easily split project-specific
files from those which can be used in other applications, but still serve
them as a single file:
你还可以将几个文件合并成一个。这样做可以帮助减少HTTP请求的数量，提高前端性能。它也使你能够通过
将文件分成数个部分来简化文件管理。你可以将某个应用特有的文件和能够在其他应用中使用的文件分隔开来，
但仍将它们一起发送，从而使文件能可重复使用：

.. configuration-block::

    .. code-block:: html+jinja

        {% javascripts
            '@AcmeFooBundle/Resources/public/js/*'
            '@AcmeBarBundle/Resources/public/js/form.js'
            '@AcmeBarBundle/Resources/public/js/calendar.js'
        %}
        <script src="{{ asset_url }}"></script>
        {% endjavascripts %}

    .. code-block:: html+php

        <?php foreach ($view['assetic']->javascripts(
            array('@AcmeFooBundle/Resources/public/js/*',
                  '@AcmeBarBundle/Resources/public/js/form.js',
                  '@AcmeBarBundle/Resources/public/js/calendar.js')) as $url): ?>
        <script src="<?php echo $view->escape($url) ?>"></script>
        <?php endforeach; ?>

In the `dev` environment, each file is still served individually, so that
you can debug problems more easily. However, in the `prod` environment, this
will be rendered as a single `script` tag.
在dev环境中，每个文件仍然是被分别发送的，这样你可以更方便地调试错误。但是在prod环境中，这些文件
依然会被放在一个单个的script标签里发送。

.. tip::

    If you're new to Assetic and try to use your application in the ``prod``
    environment (by using the ``app.php`` controller), you'll likely see
    that all of your CSS and JS breaks. Don't worry! This is on purpose.
    For details on using Assetic in the `prod` environment, see :ref:`cookbook-assetic-dumping`.
    如果你不熟悉assetic，却想要在prod环境中使用你的应用（通过使用app.php控制器），你可能会发现
    自己的css和js都不能使用。不要着急！这其实是有目的的。要了解更多关于在prod中使用assetic的信息请参阅:ref:`cookbook-assetic-dumping`。

And combining files doesn't only apply to *your* files. You can also use Assetic to
combine third party assets, such as jQuery, with your own into a single file:
而且你不止可以合并你自己的文件，你还可以使用assetic来合并第三方文件，比如说jQuery：

.. configuration-block::

    .. code-block:: html+jinja

        {% javascripts
            '@AcmeFooBundle/Resources/public/js/thirdparty/jquery.js'
            '@AcmeFooBundle/Resources/public/js/*'
        %}
        <script src="{{ asset_url }}"></script>
        {% endjavascripts %}

    .. code-block:: html+php

        <?php foreach ($view['assetic']->javascripts(
            array('@AcmeFooBundle/Resources/public/js/thirdparty/jquery.js',
                  '@AcmeFooBundle/Resources/public/js/*')) as $url): ?>
        <script src="<?php echo $view->escape($url) ?>"></script>
        <?php endforeach; ?>

Filters
过滤器
-------

Once they're managed by Assetic, you can apply filters to your assets before
they are served. This includes filters that compress the output of your assets
for smaller file sizes (and better front-end optimization). Other filters
can compile JavaScript file from CoffeeScript files and process SASS into CSS.
In fact, Assetic has a long list of available filters.
如果你使用了assetic来管理文件，你可以在发送你的asset之前用过滤器过滤它们。包括能够将
你的文件压缩得更小的过滤器（以及更好的前端体验）。其他过滤器可以从CoffeeScript文件编译javascript文件，
以及将SASS编译成CSS。实际上，assetic有许多的过滤器可以使用。

Many of the filters do not do the work directly, but use existing third-party
libraries to do the heavy-lifting. This means that you'll often need to install
a third-party library to use a filter.  The great advantage of using Assetic
to invoke these libraries (as opposed to using them directly) is that instead
of having to run them manually after you work on the files, Assetic will
take care of this for you and remove this step altogether from your development
and deployment processes.
许多过滤器都不直接做它们的工作，而是使用已经存在的第三方库。这表示如果你要使用过滤器，你
必须安装第三方库。使用assetic的一个好处就是，在你编写完那些程序后，你不必手动执行它们，assetic
会自动帮你加载，你在开发过程中不需要走这一步。

To use a filter, you first need to specify it in the Assetic configuration.
Adding a filter here doesn't mean it's being used - it just means that it's
available to use (we'll use the filter below).
要使用一个过滤器，首先你需要在assetic配置中指定它。在配置中添加它并不意味着它正在被使用——
只意味着它能够被使用（下面我们就会使用过滤器）。

For example to use the JavaScript YUI Compressor the following config should
be added:
比如要使用javascript YUI压缩器，你需要添加以下配置：

.. configuration-block::

    .. code-block:: yaml

        # app/config/config.yml
        assetic:
            filters:
                yui_js:
                    jar: "%kernel.root_dir%/Resources/java/yuicompressor.jar"

    .. code-block:: xml

        <!-- app/config/config.xml -->
        <assetic:config>
            <assetic:filter
                name="yui_js"
                jar="%kernel.root_dir%/Resources/java/yuicompressor.jar" />
        </assetic:config>

    .. code-block:: php

        // app/config/config.php
        $container->loadFromExtension('assetic', array(
            'filters' => array(
                'yui_js' => array(
                    'jar' => '%kernel.root_dir%/Resources/java/yuicompressor.jar',
                ),
            ),
        ));

Now, to actually *use* the filter on a group of JavaScript files, add it
into your template:
现在，要对一系列的javascript文件使用这个过滤器，在你的模板中添加它：

.. configuration-block::

    .. code-block:: html+jinja

        {% javascripts
            '@AcmeFooBundle/Resources/public/js/*'
            filter='yui_js'
        %}
        <script src="{{ asset_url }}"></script>
        {% endjavascripts %}

    .. code-block:: html+php

        <?php foreach ($view['assetic']->javascripts(
            array('@AcmeFooBundle/Resources/public/js/*'),
            array('yui_js')) as $url): ?>
        <script src="<?php echo $view->escape($url) ?>"></script>
        <?php endforeach; ?>

A more detailed guide about configuring and using Assetic filters as well as
details of Assetic's debug mode can be found in :doc:`/cookbook/assetic/yuicompressor`.
更详细的关于assetic过滤器配置以及assetic调试模式的信息请参见:doc:`/cookbook/assetic/yuicompressor`。

Controlling the URL used
控制要使用的URL
------------------------

If you wish to, you can control the URLs that Assetic produces. This is
done from the template and is relative to the public document root:
如果你需要，你可以控制assetic产生的URL。这是在模板中完成的，并且路径与公共文件根目录相对：

.. configuration-block::

    .. code-block:: html+jinja

        {% javascripts
            '@AcmeFooBundle/Resources/public/js/*'
            output='js/compiled/main.js'
        %}
        <script src="{{ asset_url }}"></script>
        {% endjavascripts %}

    .. code-block:: html+php

        <?php foreach ($view['assetic']->javascripts(
            array('@AcmeFooBundle/Resources/public/js/*'),
            array(),
            array('output' => 'js/compiled/main.js')
        ) as $url): ?>
        <script src="<?php echo $view->escape($url) ?>"></script>
        <?php endforeach; ?>

.. note::

    Symfony also contains a method for cache *busting*, where the final URL
    generated by Assetic contains a query parameter that can be incremented
    via configuration on each deployment. For more information, see the
    :ref:`ref-framework-assets-version` configuration option.
    symfony还有一个缓存取消方法，这个方法可以使assetic集成的URL包含一个请求参数，可以通过配置使这个
    参数在每次开发过程中递增。详情请见:ref:`ref-framework-assets-version`。

.. _cookbook-assetic-dumping:

Dumping Asset Files
转储asset文件
-------------------

In the ``dev`` environment, Assetic generates paths to CSS and JavaScript
files that don't physically exist on your computer. But they render nonetheless
because an internal Symfony controller opens the files and serves back the
content (after running any filters).
在dev环境中，assetic会集成一个在你的电脑上不存在的通往css和javascript文件的路径。
但是这个路径依然会访问这些文件，因为一个symfony的内部控制器打开了这些文件并
在执行过滤器后发送。

This kind of dynamic serving of processed assets is great because it means
that you can immediately see the new state of any asset files you change.
It's also bad, because it can be quite slow. If you're using a lot of filters,
it might be downright frustrating.
这种动态发送asset文件的方法很好，因为它意味着你可以在改变这些文件之后马上见到效果。
它也有不好，因为它会很慢。特别是你用到很多过滤器的时候。

Fortunately, Assetic provides a way to dump your assets to real files, instead
of being generated dynamically.
assetic提供一种方法，使你能将asset转储到真正文件夹，而不是动态集成。

Dumping Asset Files in the ``prod`` environment
在prod环境中转储asset文件
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

In the ``prod`` environment, your JS and CSS files are represented by a single
tag each. In other words, instead of seeing each JavaScript file you're including
in your source, you'll likely just see something like this:
在prod环境中，你的js和css文件都由一个单独的标签表示。换句话说，你会看到以下语句，而不是
每个你包含在模板内的javascript文件：

.. code-block:: html

    <script src="/app_dev.php/js/abcd123.js"></script>

Moreover, that file does **not** actually exist, nor is it dynamically rendered
by Symfony (as the asset files are in the ``dev`` environment). This is on
purpose - letting Symfony generate these files dynamically in a production
environment is just too slow.
并且，这个文件并不存在，也不是被symfony动态发送的（如在dev环境中）。这是有意为之的——让
symfony在prod环境中动态集成文件太慢了。

Instead, each time you use your app in the ``prod`` environment (and therefore,
each time you deploy), you should run the following task:
相反的，每次你在prod环境中使用你的应用，你都要运行以下命令行：

.. code-block:: bash

    php app/console assetic:dump --env=prod --no-debug

This will physically generate and write each file that you need (e.g. ``/js/abcd123.js``).
If you update any of your assets, you'll need to run this again to regenerate
the file.
这会集成所有你需要的文件(e.g. ``/js/abcd123.js``)。如果你改变你的asset，你必须重新运行这个命令来重新集成这个文件。

Dumping Asset Files in the ``dev`` environment
在dev环境中转储asset文件
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

By default, each asset path generated in the ``dev`` environment is handled
dynamically by Symfony. This has no disadvantage (you can see your changes
immediately), except that assets can load noticeably slow. If you feel like
your assets are loading too slowly, follow this guide.
默认情况下，在dev中集成的每个asset路径都由symfony动态集成。除了有时候太慢，这没有什么不好。
但如果你觉得太慢了，你可以：

First, tell Symfony to stop trying to process these files dynamically. Make
the following change in your ``config_dev.yml`` file:
首先，告诉symfony停止动态集成这些文件。在你的``config_dev.yml``文件里按照以下代码修改配置：

.. configuration-block::

    .. code-block:: yaml

        # app/config/config_dev.yml
        assetic:
            use_controller: false

    .. code-block:: xml

        <!-- app/config/config_dev.xml -->
        <assetic:config use-controller="false" />

    .. code-block:: php

        // app/config/config_dev.php
        $container->loadFromExtension('assetic', array(
            'use_controller' => false,
        ));

Next, since Symfony is no longer generating these assets for you, you'll
need to dump them manually. To do so, run the following:
第二步，由于symfony已经没有再为你集成这些asset了，你还要手动转储它们。要达到这个目的，执行命令行：

.. code-block:: bash

    php app/console assetic:dump

This physically writes all of the asset files you need for your ``dev``
environment. The big disadvantage is that you need to run this each time
you update an asset. Fortunately, by passing the ``--watch`` option, the
command will automatically regenerate assets *as they change*:
这可以在dev环境中输出所有的asset文件。但不方便的就是每次当你修改asset文件代码后，
都必须执行这个命令。不过，通过使用``--watch``选项，这个命令会在asset改变时自动重新集成：

.. code-block:: bash

    php app/console assetic:dump --watch

Since running this command in the ``dev`` environment may generate a bunch
of files, it's usually a good idea to point your generated assets files to
some isolated directory (e.g. ``/js/compiled``), to keep things organized:
由于在dev环境中运行这个命令可能会集成一堆文件，最好将这些集成的文件放到一个独立的目录下(e.g. ``/js/compiled``)，
从而保证良好的组织性：

.. configuration-block::

    .. code-block:: html+jinja

        {% javascripts
            '@AcmeFooBundle/Resources/public/js/*'
            output='js/compiled/main.js'
        %}
        <script src="{{ asset_url }}"></script>
        {% endjavascripts %}

    .. code-block:: html+php

        <?php foreach ($view['assetic']->javascripts(
            array('@AcmeFooBundle/Resources/public/js/*'),
            array(),
            array('output' => 'js/compiled/main.js')
        ) as $url): ?>
        <script src="<?php echo $view->escape($url) ?>"></script>
        <?php endforeach; ?>
