.. index::
   single: Tests

Performance
性能
===========

Symfony2 is fast, right out of the box. Of course, if you really need speed,
there are many ways that you can make Symfony even faster. In this chapter,
you'll explore many of the most common and powerful ways to make your Symfony
application even faster.
symfony2非常快。当然，有许多方法可以使它变得更快。本章将讲述一些可以使symfony应用更快捷的方法。

.. index::
   single: Performance; Byte code cache

Use a Byte Code Cache (e.g. APC)
使用Byte Code Cache（或称APC）
--------------------------------

One the best (and easiest) things that you should do to improve your performance
is to use a "byte code cache". The idea of a byte code cache is to remove
the need to constantly recompile the PHP source code. There are a number of
`byte code caches`_ available, some of which are open source. The most widely
used byte code cache is probably `APC`_
要想提高性能，最佳方法（且最容易）就是使用一个Byte Code Cache。Byte Code Cache的原理就是
不重复编译php源代码。有许多可用的`byte code caches`_，有些是开源的。最常用的就是`APC`_。

Using a byte code cache really has no downside, and Symfony2 has been architected
to perform really well in this type of environment.
使用byte code cache只有好处没有坏处，而且symfony2能够很好地在这种类型的环境下运作。

Further Optimizations
更多优化
~~~~~~~~~~~~~~~~~~~~~

Byte code caches usually monitor the source files for changes. This ensures
that if the source of a file changes, the byte code is recompiled automatically.
This is really convenient, but obviously adds overhead.
Byte code cache会监视源文件，看是否有修改。这保证了当源文件被修改时，byte code会自动编译。
这样很便利，但是会增加负载。

For this reason, some byte code caches offer an option to disable these checks.
Obviously, when disabling these checks, it will be up to the server admin
to ensure that the cache is cleared whenever any source files change. Otherwise,
the updates you've made won't be seen.
由于这个原因，有些byte code会提供选项来禁止监视。显然，当禁止监视时，服务器管理员就必须
保证当源文件被修改后，缓存被清理了。否则，更新信息不会被用户看见。

For example, to disable these checks in APC, simply add ``apc.stat=0`` to
your php.ini configuration.
比如，要在APC中禁止监视，只要在你的php.ini配置中添加``apc.stat=0``就可以了。

.. index::
   single: Performance; Autoloader

Use an Autoloader that caches (e.g. ``ApcUniversalClassLoader``)
使用一个可缓存的Autoloader（或称``ApcUniversalClassLoader``）
----------------------------------------------------------------

By default, the Symfony2 standard edition uses the ``UniversalClassLoader``
in the `autoloader.php`_ file. This autoloader is easy to use, as it will
automatically find any new classes that you've placed in the registered
directories.
symfony2标准版本默认在`autoloader.php`_文件中使用``UniversalClassLoader``。这个autoloader很容易
使用，因为它会自动查找你在注册目录中新添加的类。

Unfortunately, this comes at a cost, as the loader iterates over all configured
namespaces to find a particular file, making ``file_exists`` calls until it
finally finds the file it's looking for.
不幸的是，这是有代价的，因为loader会通过遍历所有被配置好的命名空间来寻找一个文件，直到
找到这个文件，它才会执行file_exists函数。

The simplest solution is to cache the location of each class after it's located
the first time. Symfony comes with a class - ``ApcUniversalClassLoader`` -
loader that extends the ``UniversalClassLoader`` and stores the class locations
in APC.
最简单的方法就是缓存所有类的位置。symfony有一个类——``ApcUniversalClassLoader``——它扩展了
``UniversalClassLoader``并在APC中存储了类的位置。

To use this class loader, simply adapt your ``autoloader.php`` as follows:
要使用这个loader，只要修改你的autoloader.php:

.. code-block:: php

    // app/autoload.php
    require __DIR__.'/../vendor/symfony/symfony/src/Symfony/Component/ClassLoader/ApcUniversalClassLoader.php';

    use Symfony\Component\ClassLoader\ApcUniversalClassLoader;

    $loader = new ApcUniversalClassLoader('some caching unique prefix');
    // ...

.. note::

    When using the APC autoloader, if you add new classes, they will be found
    automatically and everything will work the same as before (i.e. no
    reason to "clear" the cache). However, if you change the location of a
    particular namespace or prefix, you'll need to flush your APC cache. Otherwise,
    the autoloader will still be looking at the old location for all classes
    inside that namespace.
    当使用APC autoloader时，如果你添加一个新的类，它会自动被找到，所有的东西都会像以前
    一样工作（也就是说没有清空缓存）。但是，如果你修改了一个特定的命名空间或前缀时，你就需要
    刷新你的APC缓存了。否则autoloader会在旧的命名空间中查看类。

.. index::
   single: Performance; Bootstrap files

Use Bootstrap Files
使用Bootstrap文件
-------------------

To ensure optimal flexibility and code reuse, Symfony2 applications leverage
a variety of classes and 3rd party components. But loading all of these classes
from separate files on each request can result in some overhead. To reduce
this overhead, the Symfony2 Standard Edition provides a script to generate
a so-called `bootstrap file`_, consisting of multiple classes definitions
in a single file. By including this file (which contains a copy of many of
the core classes), Symfony no longer needs to include any of the source files
containing those classes. This will reduce disc IO quite a bit.
symfony2应用中有许多类和第三方component，要在每个请求中从不同文件加载所有的这些类会增加负载。
symfony2标准版本提供了一个脚本来集成一个`bootstrap file`_，它将多个类的定义都包含在一个文件中，
通过包含这个文件（它包含对许多核心类的一个复制），symfony就不需要包含那些类所在的文件了。
这会极大减少disc IO。

If you're using the Symfony2 Standard Edition, then you're probably already
using the bootstrap file. To be sure, open your front controller (usually
``app.php``) and check to make sure that the following line exists::
如果你使用symfony2标准版本，你可能就已经在使用bootstrap文件了。你可以打开前端控制器（通常是app.php）
并检查以下代码行是否存在::

    require_once __DIR__.'/../app/bootstrap.php.cache';

Note that there are two disadvantages when using a bootstrap file:
注意使用bootstrap有两个不利方面:

* the file needs to be regenerated whenever any of the original sources change
  (i.e. when you update the Symfony2 source or vendor libraries);
  当你修改源文件时（也就是说当你更新symfony2或vendor库时），这个文件需要被重新集成；

* when debugging, one will need to place break points inside the bootstrap file.
* 在调试的时候，你需要在bootstrap文件中设置断点。

If you're using Symfony2 Standard Edition, the bootstrap file is automatically
rebuilt after updating the vendor libraries via the ``php composer.phar install``
command.
如果你使用的是symfony2标准版本，bootstrap文件会通过``php composer.phar install``命令行在更新
vendor库后自动重建。

Bootstrap Files and Byte Code Caches
bootstrap文件和Byte Code Cache
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Even when using a byte code cache, performance will improve when using a bootstrap
file since there will be less files to monitor for changes. Of course if this
feature is disabled in the byte code cache (e.g. ``apc.stat=0`` in APC), there
is no longer a reason to use a bootstrap file.
在使用Byte Code Cache时，同时使用bootstrap文件也是会提高性能的，因为当更新时，更少的文件会被监视。
当然如果在Byte Code Cache中禁止了监视功能（在APC中设置``apc.stat=0``），也就没必要使用bootstrap文件了。

.. _`byte code caches`: http://en.wikipedia.org/wiki/List_of_PHP_accelerators
.. _`APC`: http://php.net/manual/en/book.apc.php
.. _`autoloader.php`: https://github.com/symfony/symfony-standard/blob/master/app/autoload.php
.. _`bootstrap file`: https://github.com/sensio/SensioDistributionBundle/blob/master/Composer/ScriptHandler.php
