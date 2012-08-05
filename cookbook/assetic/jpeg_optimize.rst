.. index::
   single: Assetic; Image Optimization

How to Use Assetic For Image Optimization with Twig Functions
如何在twig中使用assetic进行图像优化
=============================================================

Amongst its many filters, Assetic has four filters which can be used for on-the-fly
image optimization. This allows you to get the benefits of smaller file sizes
without having to use an image editor to process each image. The results
are cached and can be dumped for production so there is no performance hit
for your end users.
在assetic的过滤器中，有四个可以用来在传输中优化图像的过滤器。它允许你缩小图像大小，
而不必使用图像编辑器。这些结果被缓存下来了，并且可以被转储，对你的客户端表现没有影响。

Using Jpegoptim
使用Jpegoptim
---------------

`Jpegoptim`_ is a utility for optimizing JPEG files. To use it with Assetic,
add the following to the Assetic config:
`Jpegoptim`_ 是一个优化JPEG文件的设备。将以下代码添加到assetic配置中，从而你可以在assetic中使用它：

.. configuration-block::

    .. code-block:: yaml

        # app/config/config.yml
        assetic:
            filters:
                jpegoptim:
                    bin: path/to/jpegoptim

    .. code-block:: xml

        <!-- app/config/config.xml -->
        <assetic:config>
            <assetic:filter
                name="jpegoptim"
                bin="path/to/jpegoptim" />
        </assetic:config>

    .. code-block:: php

        // app/config/config.php
        $container->loadFromExtension('assetic', array(
            'filters' => array(
                'jpegoptim' => array(
                    'bin' => 'path/to/jpegoptim',
                ),
            ),
        ));

.. note::

    Notice that to use jpegoptim, you must have it already installed on your
    system. The ``bin`` option points to the location of the compiled binary.
    如果你要使用jpegoptim，必须确保在你的系统中已经安装它了。bin这个选项指向它的安装地点。

It can now be used from a template:
现在可以从模板中使用它了：

.. configuration-block::

    .. code-block:: html+jinja

        {% image '@AcmeFooBundle/Resources/public/images/example.jpg'
            filter='jpegoptim' output='/images/example.jpg'
        %}
        <img src="{{ asset_url }}" alt="Example"/>
        {% endimage %}

    .. code-block:: html+php

        <?php foreach ($view['assetic']->images(
            array('@AcmeFooBundle/Resources/public/images/example.jpg'),
            array('jpegoptim')) as $url): ?>
        <img src="<?php echo $view->escape($url) ?>" alt="Example"/>
        <?php endforeach; ?>

Removing all EXIF Data
移除所有的EXIF数据
~~~~~~~~~~~~~~~~~~~~~~

By default, running this filter only removes some of the meta information
stored in the file. Any EXIF data and comments are not removed, but you can
remove these by using the ``strip_all`` option:
默认情况下，执行这个过滤器只会移除一些文件中的meta信息。所有EXIF数据和注释都没有被移除，
但你可以通过strip_all选项来移除它们：

.. configuration-block::

    .. code-block:: yaml

        # app/config/config.yml
        assetic:
            filters:
                jpegoptim:
                    bin: path/to/jpegoptim
                    strip_all: true

    .. code-block:: xml

        <!-- app/config/config.xml -->
        <assetic:config>
            <assetic:filter
                name="jpegoptim"
                bin="path/to/jpegoptim"
                strip_all="true" />
        </assetic:config>

    .. code-block:: php

        // app/config/config.php
        $container->loadFromExtension('assetic', array(
            'filters' => array(
                'jpegoptim' => array(
                    'bin' => 'path/to/jpegoptim',
                    'strip_all' => 'true',
                ),
            ),
        ));

Lowering Maximum Quality
减小最高质量
~~~~~~~~~~~~~~~~~~~~~~~~

The quality level of the JPEG is not affected by default. You can gain
further file size reductions by setting the max quality setting lower than
the current level of the images. This will of course be at the expense of
image quality:
默认情况下，JPEG的质量没有受到影响。你可以通过将最大质量设置得比当前质量低来缩减文件大小。
当然这对图像质量有影响：

.. configuration-block::

    .. code-block:: yaml

        # app/config/config.yml
        assetic:
            filters:
                jpegoptim:
                    bin: path/to/jpegoptim
                    max: 70

    .. code-block:: xml

        <!-- app/config/config.xml -->
        <assetic:config>
            <assetic:filter
                name="jpegoptim"
                bin="path/to/jpegoptim"
                max="70" />
        </assetic:config>

    .. code-block:: php

        // app/config/config.php
        $container->loadFromExtension('assetic', array(
            'filters' => array(
                'jpegoptim' => array(
                    'bin' => 'path/to/jpegoptim',
                    'max' => '70',
                ),
            ),
        ));

Shorter syntax: Twig Function
便捷语法：twig方法
-----------------------------

If you're using Twig, it's possible to achieve all of this with a shorter
syntax by enabling and using a special Twig function. Start by adding the
following config:
如果你使用twig，那么只要激活并使用一个特定twig方法，就可以通过一个便捷语法来达到这个目的了。
添加以下配置：

.. configuration-block::

    .. code-block:: yaml

        # app/config/config.yml
        assetic:
            filters:
                jpegoptim:
                    bin: path/to/jpegoptim
            twig:
                functions:
                    jpegoptim: ~

    .. code-block:: xml

        <!-- app/config/config.xml -->
        <assetic:config>
            <assetic:filter
                name="jpegoptim"
                bin="path/to/jpegoptim" />
            <assetic:twig>
                <assetic:twig_function
                    name="jpegoptim" />
            </assetic:twig>
        </assetic:config>

    .. code-block:: php

        // app/config/config.php
        $container->loadFromExtension('assetic', array(
            'filters' => array(
                'jpegoptim' => array(
                    'bin' => 'path/to/jpegoptim',
                ),
            ),
            'twig' => array(
                'functions' => array('jpegoptim'),
                ),
            ),
        ));

The Twig template can now be changed to the following:
twig模板可以这样改变：

.. code-block:: html+jinja

    <img src="{{ jpegoptim('@AcmeFooBundle/Resources/public/images/example.jpg') }}"
         alt="Example"/>

You can specify the output directory in the config in the following way:
你还可以指定输出的目录：

.. configuration-block::

    .. code-block:: yaml

        # app/config/config.yml
        assetic:
            filters:
                jpegoptim:
                    bin: path/to/jpegoptim
            twig:
                functions:
                    jpegoptim: { output: images/*.jpg }

    .. code-block:: xml

        <!-- app/config/config.xml -->
        <assetic:config>
            <assetic:filter
                name="jpegoptim"
                bin="path/to/jpegoptim" />
            <assetic:twig>
                <assetic:twig_function
                    name="jpegoptim"
                    output="images/*.jpg" />
            </assetic:twig>
        </assetic:config>

    .. code-block:: php

        // app/config/config.php
        $container->loadFromExtension('assetic', array(
            'filters' => array(
                'jpegoptim' => array(
                    'bin' => 'path/to/jpegoptim',
                ),
            ),
            'twig' => array(
                'functions' => array(
                    'jpegoptim' => array(
                        output => 'images/*.jpg'
                    ),
                ),
            ),
        ));

.. _`Jpegoptim`: http://www.kokkonen.net/tjko/projects.html