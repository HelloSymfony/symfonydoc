.. index::
   single: Assetic; YUI Compressor

How to Minify JavaScripts and Stylesheets with YUI Compressor
如何使用YUI压缩器使javascript和样式表最小化
=============================================================

Yahoo! provides an excellent utility for minifying JavaScripts and stylesheets
so they travel over the wire faster, the `YUI Compressor`_. Thanks to Assetic,
you can take advantage of this tool very easily.
`YUI Compressor`_提供一个非常好的方法来使javascript和样式表最小化，这样它们可以发送得更快。
通过assetic，你可以非常方便地使用这个工具。

Download the YUI Compressor JAR
下载YUI压缩器JAR
-------------------------------

The YUI Compressor is written in Java and distributed as a JAR. `Download the JAR`_
from the Yahoo! site and save it to ``app/Resources/java/yuicompressor.jar``.
YUI压缩器是用Java写的并以JAR的格式提供。从yahoo！下载`Download the JAR`_并将它保存在``app/Resources/java/yuicompressor.jar``。

Configure the YUI Filters
配置YUI过滤器
-------------------------

Now you need to configure two Assetic filters in your application, one for
minifying JavaScripts with the YUI Compressor and one for minifying
stylesheets:
现在你需要在你的应用中配置两个assetic过滤器，一个是使javascript最小化的，另一个是使样式表最小化的：

.. configuration-block::

    .. code-block:: yaml

        # app/config/config.yml
        assetic:
            filters:
                yui_css:
                    jar: "%kernel.root_dir%/Resources/java/yuicompressor.jar"
                yui_js:
                    jar: "%kernel.root_dir%/Resources/java/yuicompressor.jar"

    .. code-block:: xml

        <!-- app/config/config.xml -->
        <assetic:config>
            <assetic:filter
                name="yui_css"
                jar="%kernel.root_dir%/Resources/java/yuicompressor.jar" />
            <assetic:filter
                name="yui_js"
                jar="%kernel.root_dir%/Resources/java/yuicompressor.jar" />
        </assetic:config>

    .. code-block:: php

        // app/config/config.php
        $container->loadFromExtension('assetic', array(
            'filters' => array(
                'yui_css' => array(
                    'jar' => '%kernel.root_dir%/Resources/java/yuicompressor.jar',
                ),
                'yui_js' => array(
                    'jar' => '%kernel.root_dir%/Resources/java/yuicompressor.jar',
                ),
            ),
        ));

You now have access to two new Assetic filters in your application:
``yui_css`` and ``yui_js``. These will use the YUI Compressor to minify
stylesheets and JavaScripts, respectively.
你现在可以访问你的应用中的两个新的assetic过滤器了：``yui_css`` and ``yui_js``。
它们可以分别用YUI来使javascript和css最小化。

Minify your Assets
使你的asset最小化
------------------

You have YUI Compressor configured now, but nothing is going to happen until
you apply one of these filters to an asset. Since your assets are a part of
the view layer, this work is done in your templates:
你现在已经配置好了YUI压缩器，但是要应用它们你还必须在模板中：

.. configuration-block::

    .. code-block:: html+jinja

        {% javascripts '@AcmeFooBundle/Resources/public/js/*' filter='yui_js' %}
        <script src="{{ asset_url }}"></script>
        {% endjavascripts %}

    .. code-block:: html+php

        <?php foreach ($view['assetic']->javascripts(
            array('@AcmeFooBundle/Resources/public/js/*'),
            array('yui_js')) as $url): ?>
        <script src="<?php echo $view->escape($url) ?>"></script>
        <?php endforeach; ?>

.. note::

    The above example assumes that you have a bundle called ``AcmeFooBundle``
    and your JavaScript files are in the ``Resources/public/js`` directory under
    your bundle. This isn't important however - you can include your Javascript
    files no matter where they are.
    以上的例子假设你有一个名叫``AcmeFooBundle``的bundle，并且你的javascript文件都在``Resources/public/js``
    目录下。这并不重要——不论你的javascript文件在哪儿，你都可以包含它。

With the addition of the ``yui_js`` filter to the asset tags above, you should
now see minified JavaScripts coming over the wire much faster. The same process
can be repeated to minify your stylesheets.
当你像asset标签添加``yui_js``过滤器后，你可以看见javascript文件传输得更快了。对于样式表也一样。

.. configuration-block::

    .. code-block:: html+jinja

        {% stylesheets '@AcmeFooBundle/Resources/public/css/*' filter='yui_css' %}
        <link rel="stylesheet" type="text/css" media="screen" href="{{ asset_url }}" />
        {% endstylesheets %}

    .. code-block:: html+php

        <?php foreach ($view['assetic']->stylesheets(
            array('@AcmeFooBundle/Resources/public/css/*'),
            array('yui_css')) as $url): ?>
        <link rel="stylesheet" type="text/css" media="screen" href="<?php echo $view->escape($url) ?>" />
        <?php endforeach; ?>

Disable Minification in Debug Mode
在调试模式下禁止最小化
----------------------------------

Minified JavaScripts and Stylesheets are very difficult to read, let alone
debug. Because of this, Assetic lets you disable a certain filter when your
application is in debug mode. You can do this be prefixing the filter name
in your template with a question mark: ``?``. This tells Assetic to only
apply this filter when debug mode is off.
最小化后的javascript和css文件很难读，更不要说调试了。所以当你的应用在调试模式下，assetic允许你
禁止某个过滤器。要达到这个目的，你可以在你的模板的过滤器名称前加上一个问号：？。这告诉assetic
只在当调试模式关闭时才允许运用过滤器。

.. configuration-block::

    .. code-block:: html+jinja

        {% javascripts '@AcmeFooBundle/Resources/public/js/*' filter='?yui_js' %}
        <script src="{{ asset_url }}"></script>
        {% endjavascripts %}

    .. code-block:: html+php

        <?php foreach ($view['assetic']->javascripts(
            array('@AcmeFooBundle/Resources/public/js/*'),
            array('?yui_js')) as $url): ?>
        <script src="<?php echo $view->escape($url) ?>"></script>
        <?php endforeach; ?>

.. _`YUI Compressor`: http://developer.yahoo.com/yui/compressor/
.. _`Download the JAR`: http://yuilibrary.com/downloads/#yuicompressor