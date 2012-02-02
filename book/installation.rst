.. index::
   single: Installation

安装和配置Symfony
=================

本章的目的是让你可以很快地开始并创建一个可用的Symfony应用。方便的是
Symfony提供了一个“标准发布版本”，它是一个可用的Symfony“起步”项目，
你可以下载现在这个“标准发布版本”立刻开始开发。

.. tip::
   
   如果你在寻找如何以最佳方式开始一个新项目，并把它置入版本控制，请参照
   `Using Source Controll`_。

下载Symfony2发布版本
--------------------

.. tip::

   首先，请确保你已经安装并配置好了一个web服务器（如Apache），同时安装了PHP
   5.3.2或更高版本。想了解更多Symfony2的系统要求，参照 :doc:`requirments reference</reference/requirments>`。

Symfony2打包了一个“发布版本”，这是一个具有完整功能的应用，他包含了Symfony2 核心（core）库，一些精选的bundle，
一个合理的目录结构，还有一些默认的配置。在你下载一个Symfony2发布版本的同时你也下载了一个具有完整功能的应用框架，
你可以基于这个应用框架立刻开始开发你自己的应用。

我们从访问Symfony2下载页面`http://symfony.com/download`_开始。在这个页面，你会看到一个*Symfony 标准版*，这是Symfony2
的主要分发版本。在这里你要做两个选择：

* 是下载``.tgz``还是下载``.zip``格式的压缩包 - 这两个是一样的，下载任何一个你用起来方便的格式。
* 是否下载一个包含vendors的发布版本。如果你的电脑上已经安装了`Git`_，你
  应该下载不含vendors的Symfony2，因为这样给你使用第三方库的时候增加了一些弹性。

下载任何一个压缩包，解压到你的web服务器根目录下。在UNIX命令行环境下，用任何一个下面的命令可以解压
（把``###``换成真正的文件名）：

.. code-block:: bash
    
    # .tgz格式
    tar zxvf Symfony_Standard_Vendors_2.0.###.tgz

    # .zip格式
    unzip Symfony_Standard_Vendors_2.0.###.zip    

解压后，你应该有一个``Symfony/``的文件夹，目录结构如下：

.. code-block:: text

    www/ <- 你的web根目录
        Symfony/ <- 解压后的压缩包
            app/
                cache/
                config/
                logs/
            src/
                ...
            vendor/
                ...
            web/
                app.php
                ...

更新Vendors
~~~~~~~~~~~

最后，如果你下载了不包含vendors的压缩包，从命令行运行下面的命令安装vendors：

.. code-block:: bash
  
    php bin/vendors install

 这个命令会下载所有必需的的vendor库 - 包括Symfony自己 - 到 ``vendor/``目录下。
 要了解Symfony2怎么管理第三方库，请参考
 ":ref:`cookbook-managing-vendor-libraries`"。

配置和安装
~~~~~~~~~~

到目前为止，所有必须的第三方库都在``vendor/``目录下。

