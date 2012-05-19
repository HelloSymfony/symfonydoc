.. index::
   single: Assetic; Apply Filters

How to Apply an Assetic Filter to a Specific File Extension
如何将assetic过滤器应用到有特定后缀名的文件
===========================================================

Assetic filters can be applied to individual files, groups of files or even,
as you'll see here, files that have a specific extension. To show you how
to handle each option, let's suppose that you want to use Assetic's CoffeeScript
filter, which compiles CoffeeScript files into Javascript.
assetic过滤器可以被应用到单个文件，多个文件，或者你在这儿见到的，有特定后缀名的文件。
举例来说，假设你想要使用assetic的CoffeeScript过滤器将CoffeeScript文件编译到javascript文件。

The main configuration is just the paths to coffee and node. These default
respectively to ``/usr/bin/coffee`` and ``/usr/bin/node``:
主要的配置仅仅是到达coffe和node的路径。默认情况下分别为``/usr/bin/coffee``和``/usr/bin/node``:

.. configuration-block::

    .. code-block:: yaml

        # app/config/config.yml
        assetic:
            filters:
                coffee:
                    bin: /usr/bin/coffee
                    node: /usr/bin/node

    .. code-block:: xml

        <!-- app/config/config.xml -->
        <assetic:config>
            <assetic:filter
                name="coffee"
                bin="/usr/bin/coffee"
                node="/usr/bin/node" />
        </assetic:config>

    .. code-block:: php

        // app/config/config.php
        $container->loadFromExtension('assetic', array(
            'filters' => array(
                'coffee' => array(
                    'bin' => '/usr/bin/coffee',
                    'node' => '/usr/bin/node',
                ),
            ),
        ));

Filter a Single File
过滤单个文件
--------------------

You can now serve up a single CoffeeScript file as JavaScript from within your
templates:
你现在可以从你的模板中将CoffeeScript文件编译成javascript文件：

.. configuration-block::

    .. code-block:: html+jinja

        {% javascripts '@AcmeFooBundle/Resources/public/js/example.coffee'
            filter='coffee'
        %}
        <script src="{{ asset_url }}" type="text/javascript"></script>
        {% endjavascripts %}

    .. code-block:: html+php

        <?php foreach ($view['assetic']->javascripts(
            array('@AcmeFooBundle/Resources/public/js/example.coffee'),
            array('coffee')) as $url): ?>
        <script src="<?php echo $view->escape($url) ?>" type="text/javascript"></script>
        <?php endforeach; ?>

This is all that's needed to compile this CoffeeScript file and server it
as the compiled JavaScript.
仅使用以上代码就可以编译CoffeeScript文件，并将它作为javascript输出。

Filter Multiple Files
过滤多个文件
---------------------

You can also combine multiple CoffeeScript files into a single output file:
你还可以将多个CoffeeScript文件合并成一个单独的输出文件：

.. configuration-block::

    .. code-block:: html+jinja

        {% javascripts '@AcmeFooBundle/Resources/public/js/example.coffee'
                       '@AcmeFooBundle/Resources/public/js/another.coffee'
            filter='coffee'
        %}
        <script src="{{ asset_url }}" type="text/javascript"></script>
        {% endjavascripts %}

    .. code-block:: html+php

        <?php foreach ($view['assetic']->javascripts(
            array('@AcmeFooBundle/Resources/public/js/example.coffee',
                  '@AcmeFooBundle/Resources/public/js/another.coffee'),
            array('coffee')) as $url): ?>
        <script src="<?php echo $view->escape($url) ?>" type="text/javascript"></script>
        <?php endforeach; ?>

Both the files will now be served up as a single file compiled into regular
JavaScript.
这两个文件会被编译成javascript文件并合并成一个文件输出。

Filtering based on a File Extension
根据文件后缀名来进行过滤
-----------------------------------

One of the great advantages of using Assetic is reducing the number of asset
files to lower HTTP requests. In order to make full use of this, it would
be good to combine *all* your JavaScript and CoffeeScript files together
since they will ultimately all be served as JavaScript. Unfortunately just
adding the JavaScript files to the files to be combined as above will not
work as the regular JavaScript files will not survive the CoffeeScript compilation.
使用assetic的一个极大好处就是减少asset文件从而减少HTTP请求。要达到这个目的，应该将所有的javascript和
CoffeeScript文件都合并在一起，并作为javascript输出。不幸的是，仅仅将javascript文件添加到以上的合并文件中
是不够的，CoffeeScript编译器不知道该怎样编译它们。

This problem can be avoided by using the ``apply_to`` option in the config,
which allows you to specify that a filter should always be applied to particular
file extensions. In this case you can specify that the Coffee filter is
applied to all ``.coffee`` files:
要解决这个问题，可以在配置中使用``apply_to``选项，它允许过滤器只针对某一类型后缀名的文件进行过滤。
于是，你可以指定coffee过滤器只应用于``.coffee``文件：

.. configuration-block::

    .. code-block:: yaml

        # app/config/config.yml
        assetic:
            filters:
                coffee:
                    bin: /usr/bin/coffee
                    node: /usr/bin/node
                    apply_to: "\.coffee$"

    .. code-block:: xml

        <!-- app/config/config.xml -->
        <assetic:config>
            <assetic:filter
                name="coffee"
                bin="/usr/bin/coffee"
                node="/usr/bin/node"
                apply_to="\.coffee$" />
        </assetic:config>

    .. code-block:: php

        // app/config/config.php
        $container->loadFromExtension('assetic', array(
            'filters' => array(
                'coffee' => array(
                    'bin' => '/usr/bin/coffee',
                    'node' => '/usr/bin/node',
                    'apply_to' => '\.coffee$',
                ),
            ),
        ));

With this, you no longer need to specify the ``coffee`` filter in the template.
You can also list regular JavaScript files, all of which will be combined
and rendered as a single JavaScript file (with only the ``.coffee`` files
being run through the CoffeeScript filter):
现在，你不必再在模板中指定coffee过滤器了。你还可以列出javascript文件，所有这些文件都会被
作为一个单独的javascript文件输出（并且只有``.coffee``文件被过滤）：

.. configuration-block::

    .. code-block:: html+jinja

        {% javascripts '@AcmeFooBundle/Resources/public/js/example.coffee'
                       '@AcmeFooBundle/Resources/public/js/another.coffee'
                       '@AcmeFooBundle/Resources/public/js/regular.js'
        %}
        <script src="{{ asset_url }}" type="text/javascript"></script>
        {% endjavascripts %}

    .. code-block:: html+php

        <?php foreach ($view['assetic']->javascripts(
            array('@AcmeFooBundle/Resources/public/js/example.coffee',
                  '@AcmeFooBundle/Resources/public/js/another.coffee',
                  '@AcmeFooBundle/Resources/public/js/regular.js'),
            as $url): ?>
        <script src="<?php echo $view->escape($url) ?>" type="text/javascript"></script>
        <?php endforeach; ?>
