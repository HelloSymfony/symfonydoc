.. index::
   single: Installation

安装和配置Symfony
=================

本章的目的是让你可以很快地开始并创建一个可用的Symfony应用。方便的是
Symfony提供了一个“标准发布版本”，它是一个可用的Symfony“起步”项目，
你可以下载这个“标准发布版本”立刻开始开发。

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

到目前为止，所有必须的第三方库都在``vendor/``目录下。同时在``app/``目录下有一个默认的应用配置，
还有一些示例代码在``src/``目录下。

Symfony2有一个可视化的配置测试工具来帮助你确保你的web服务器和PHP正确配置，以便使用Symfony。用下面的URL来
检查你的配置：

.. code-block:: text
    
    http://localhost/Symfony/web/config.php

 如果检查到有问题，更正后，我们继续。

 .. sidebar:: 设置权限

      一个最常见的问题是web服务器和命令行用户必须对``app/cache``和``app/logs``有写权限。
      在UNIX系统上，如果你的web服务器用户跟命令行用户不是同一个用户，你可以在你的项目中
      运行一次一下的命令，以确保正确设置了权限。把``www-data``改为你的web服务器用户：

      **1. 在支持chmod +a的系统上用ACL**

      许多系统允许你用命令``chmod +a``。先试下这个，如果有错误，尝试下一个的方法：

      .. code-block:: bash
          rm -rf app/cache/*
          rm -rf app/logs/*

        sudo chmod +a "www-data allow delete,write,append,file_inherit,directory_inherit" app/cache app/logs
        sudo chmod +a "`whoami` allow delete,write,append,file_inherit,directory_inherit" app/cache app/log

      **2. 在不支持chmod +a的系统上用ACL**

      一些系统不支持``chmod +a``，但支持另外一种叫``setfacl``的工具。你可能需要在你的硬盘分区上`打开ACL支持`_，
      安装setfacl，然后如下所示来使用：

      .. code-block:: bash

          sudo setfacl -R -m u:www-data:rwx -m u:`whoami`:rwx app/cache app/logs
          sudo setfacl -dR -m u:www-data:rwx -m u:`whoami`:rwx app/cache app/logs

      **3. 不用ACL**

      如果你没有权限改变文件夹的ACL，你需要修改umash，使cache和log文件夹和对整个
      用户组或全部用户可写（取决于web服务器和命令行用户是否是同一个用户组）。要做到这一点，
      把下行放到``app/console``，``web/app.php``和```web/app_dev.php``的开头。

      .. code-block:: php

          umash(0002); // 这个会让权限变为0775

          // 或

          umash(0000); // 这个会让权限变为0777

       需要注意的是，如果你有权限更改文件夹权限，推荐使用ACL，因为改变umash不是thread-safe的。

当一切都没问题时，点击"Go to welcome page"来请求你的第一个真正的Symfony2网页：

.. code-block:: text

    http://localhost/Symfony/web/app_dev.php/

你应该会看到Symfony2欢迎你的信息。

.. image:: /images/quick_tour/welcome.jpg


开始开发
--------

现在你有了一个具有完成功能的Symfony2应用，你可以开始继续开发了！你的下载的发布版本应该包含以下
示例代码 - 可以通过查看随同发布版本的``README.rst``文件（以txt格式打开）来了解示例代码，并了解
如何在后续开发中删除这些代码。

使用版本控制系统
----------------

如果你使用类似于``Git``和``Subversion``的版本控制系统，你可以稍加设置，开始提交你的代码。
Symfony标准发布版本*是*你新项目的起点。

想要了解怎样更好的使用git来管理你的项目， 参照:doc:`/cookbook/workflow/new_project_git`。

忽略``vendor/``目录
~~~~~~~~~~~~~~~~~~~

如果你下载了不含vendors的压缩包，你可以忽略整个``vendor/``文件夹，避免其被提交到版本控制系统。
如果你用的是``Git``，可以创建一个``.gitignore``的文件，计入下面一行：

.. code-block:: text
    
    vendor/

现在，vendor文件夹不会被错误的提交到版本控制。这样很好（其实，棒极了！），因为其他人clone或
check out这个项目时，他/她可以简单的运行``php bin/vendors install``脚本来下载所有必须的vendor
库。

.. _`enable ACL support`: https://help.ubuntu.com/community/FilePermissions#ACLs
.. _`http://symfony.com/download`: http://symfony.com/download
.. _`Git`: http://git-scm.com/
.. _`GitHub Bootcamp`: http://help.github.com/set-up-git-redirec

