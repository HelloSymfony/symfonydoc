Doctrine Extensions: Timestampable, Sluggable, Translatable, etc.
doctrine扩展：Timestampable, Sluggable, Translatable,等等
=================================================================

Doctrine2 is very flexible, and the community has already created a series
of useful Doctrine extensions to help you with common entity-related tasks.
doctrine2非常灵活，社区已经创建了一系列有用的doctrine扩展来帮助你实现关于实体类的任务。

One library in particular - the `DoctrineExtensions`_ library - provides integration
functionality for `Sluggable`_, `Translatable`_, `Timestampable`_, `Loggable`_,
`Tree`_ and `Sortable`_ behaviors.
`DoctrineExtensions`_库提供了对`Sluggable`_, `Translatable`_, `Timestampable`_, `Loggable`_,
`Tree`_ 和`Sortable`_等行为的功能融合。

The usage for each of these extensions is explained in that repository.
每个扩展的使用方法都在那里有解释。

However, to install/activate each extension you must register and activate an
:doc:`Event Listener</cookbook/doctrine/event_listeners_subscribers>`.
To do this, you have two options:
但要激活或安装扩展你必须注册并激活一个:doc:`Event Listener</cookbook/doctrine/event_listeners_subscribers>`。
要达到这个目的，有两种方法:

#. Use the `StofDoctrineExtensionsBundle`_, which integrates the above library.
#. 使用`StofDoctrineExtensionsBundle`_，它结合了所有以上的库。

#. Implement this services directly by following the documentation for integration
   with Symfony2: `Install Gedmo Doctrine2 extensions in Symfony2`_
#. 根据symfony2的文档直接植入这个服务：`Install Gedmo Doctrine2 extensions in Symfony2`_

.. _`DoctrineExtensions`: https://github.com/l3pp4rd/DoctrineExtensions
.. _`StofDoctrineExtensionsBundle`: https://github.com/stof/StofDoctrineExtensionsBundle
.. _`Sluggable`: https://github.com/l3pp4rd/DoctrineExtensions/blob/master/doc/sluggable.md
.. _`Translatable`: https://github.com/l3pp4rd/DoctrineExtensions/blob/master/doc/translatable.md
.. _`Timestampable`: https://github.com/l3pp4rd/DoctrineExtensions/blob/master/doc/timestampable.md
.. _`Loggable`: https://github.com/l3pp4rd/DoctrineExtensions/blob/master/doc/loggable.md
.. _`Tree`: https://github.com/l3pp4rd/DoctrineExtensions/blob/master/doc/tree.md
.. _`Sortable`: https://github.com/l3pp4rd/DoctrineExtensions/blob/master/doc/sortable.md
.. _`Install Gedmo Doctrine2 extensions in Symfony2`: https://github.com/l3pp4rd/DoctrineExtensions/blob/master/doc/symfony2.md