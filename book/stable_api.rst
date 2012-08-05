The Symfony2 Stable API
symfony2稳定API
=======================

The Symfony2 stable API is a subset of all Symfony2 published public methods
(components and core bundles) that share the following properties:
symfony2稳定API（stable API）是symfony2中的公共方法（component和核心bundle）的子设置，它们
都有以下属性:

* The namespace and class name won't change;
* 命名空间和类名不会改变；
* The method name won't change;
* 方法名称不会改变；
* The method signature (arguments and return value type) won't change;
* 方法的特征不会改变（参数和返回值的类型）；
* The semantic of what the method does won't change.
* 方法的语义不会改变。

The implementation itself can change though. The only valid case for a change
in the stable API is in order to fix a security issue.
API本身不会改变。在稳定API中唯一可能改变的就是针对安全问题所做的改变。

The stable API is based on a whitelist, tagged with `@api`. Therefore,
everything not tagged explicitly is not part of the stable API.
所有稳定API都有`@api`标签。没有显性标签的都不是稳定API。

.. tip::

    Any third party bundle should also publish its own stable API.
    所有第三方bundle都有它自己的API。

As of Symfony 2.0, the following components have a public tagged API:
symfony2.0中，以下的component有有标签的API：

* BrowserKit
* ClassLoader
* Console
* CssSelector
* DependencyInjection
* DomCrawler
* EventDispatcher
* Finder
* HttpFoundation
* HttpKernel
* Locale
* Process
* Routing
* Templating
* Translation
* Validator
* Yaml
