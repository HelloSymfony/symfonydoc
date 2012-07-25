.. index::
   single: Translations

Translations
翻译
============

The term "internationalization" (often abbreviated `i18n`_) refers to the process
of abstracting strings and other locale-specific pieces out of your application
and into a layer where they can be translated and converted based on the user's
locale (i.e. language and country). For text, this means wrapping each with a
function capable of translating the text (or "message") into the language of
the user::
术语"internationalization"（简称`i18n`_）表示将你的应用中的字符串和其他本地片段抽象到一个可以
将它们根据用户的locale（也就是语言和国家）来来进行翻译的层。对于文本而言，就是将每个文本通过
一个翻译函数包装起来::

    // text will *always* print out in English
    echo 'Hello World';

    // text can be translated into the end-user's language or default to English
    echo $translator->trans('Hello World');

.. note::

    The term *locale* refers roughly to the user's language and country. It
    can be any string that your application uses to manage translations
    and other format differences (e.g. currency format). We recommended the
    `ISO639-1`_ *language* code, an underscore (``_``), then the `ISO3166 Alpha-2`_ *country*
    code (e.g. ``fr_FR`` for French/France).
    locale大略地表示了用户的语言和国家，它可以是任何字符串，你的应用会使用这个字符串来管理翻译
    以及其他格式问题（比如说货币格式）。我们推荐使用`ISO639-1`_编码，下划线（_），以及`ISO3166 Alpha-2`_
    国家编码(e.g. ``fr_FR`` for French/France)。

In this chapter, we'll learn how to prepare an application to support multiple
locales and then how to create translations for multiple locales. Overall,
the process has several common steps:
在本章中，我们将学习如何使应用支持多个locale，以及如何针对多个locale来创建翻译。总体来说，
流程分四部:

1. Enable and configure Symfony's ``Translation`` component;
1. 配置symfony的Translation component；

2. Abstract strings (i.e. "messages") by wrapping them in calls to the ``Translator``;
2. 通过将字符串（即“messages”）包装在Translator函数中来抽象它们；

3. Create translation resources for each supported locale that translate
   each message in the application;
3. 为每个被支持的locale创建翻译源文件，这个源文件会翻译应用中的所有message；

4. Determine, set and manage the user's locale for the request and optionally
   on the user's entire session.
4. 根据请求在用户的整个session中确定、设置和管理用户的locale。

.. index::
   single: Translations; Configuration

Configuration
配置
-------------

Translations are handled by a ``Translator`` :term:`service` that uses the
user's locale to lookup and return translated messages. Before using it,
enable the ``Translator`` in your configuration:
翻译是通过``Translator`` :term:`service`来处理的，它通过用户的locale来查找并返回被翻译的信息。
在使用它之前，你需要在配置中激活Translator:

.. configuration-block::

    .. code-block:: yaml

        # app/config/config.yml
        framework:
            translator: { fallback: en }

    .. code-block:: xml

        <!-- app/config/config.xml -->
        <framework:config>
            <framework:translator fallback="en" />
        </framework:config>

    .. code-block:: php

        // app/config/config.php
        $container->loadFromExtension('framework', array(
            'translator' => array('fallback' => 'en'),
        ));

The ``fallback`` option defines the fallback locale when a translation does
not exist in the user's locale.
fallback选项定义了当用户locale不存在时的备选项。

.. tip::

    When a translation does not exist for a locale, the translator first tries
    to find the translation for the language (``fr`` if the locale is
    ``fr_FR`` for instance). If this also fails, it looks for a translation
    using the fallback locale.
    当一个locale的翻译不存在时，translator首先会查找这个语言的转换形式（比如locale是fr_FR，就会查找fr）。
    如果找不到，它才会使用备选的locale。

The locale used in translations is the one stored on the request. This is
typically set via a ``_locale`` attribute on your routes (see :ref:`book-translation-locale-url`).
杂翻译中使用的locale就是请求中存储的locale。这是通过路由上的``_locale``属性设置的（参见:ref:`book-translation-locale-url`）。

.. index::
   single: Translations; Basic translation

Basic Translation
基础翻译
-----------------

Translation of text is done through the  ``translator`` service
(:class:`Symfony\\Component\\Translation\\Translator`). To translate a block
of text (called a *message*), use the
:method:`Symfony\\Component\\Translation\\Translator::trans` method. Suppose,
for example, that we're translating a simple message from inside a controller:
文本的翻译是通过``translator``服务(:class:`Symfony\\Component\\Translation\\Translator`)完成的。
要翻译一段文本（也称message），需要使用:method:`Symfony\\Component\\Translation\\Translator::trans`方法。
假设我们要从控制器中翻译一段message:

.. code-block:: php

    public function indexAction()
    {
        $t = $this->get('translator')->trans('Symfony2 is great');

        return new Response($t);
    }

When this code is executed, Symfony2 will attempt to translate the message
"Symfony2 is great" based on the ``locale`` of the user. For this to work,
we need to tell Symfony2 how to translate the message via a "translation
resource", which is a collection of message translations for a given locale.
This "dictionary" of translations can be created in several different formats,
XLIFF being the recommended format:
当这段代码被执行的时候，symfony2会试图根据用户的locale来翻译"Symfony2 is great"。
我们还要告诉symfony2如何通过一个翻译源文件来翻译这段话，翻译源文件就是对一个给定locale的
message翻译的集合。这个源文件可以被通过不同格式创建，我们推荐使用XLIFF格式:

.. configuration-block::

    .. code-block:: xml

        <!-- messages.fr.xliff -->
        <?xml version="1.0"?>
        <xliff version="1.2" xmlns="urn:oasis:names:tc:xliff:document:1.2">
            <file source-language="en" datatype="plaintext" original="file.ext">
                <body>
                    <trans-unit id="1">
                        <source>Symfony2 is great</source>
                        <target>J'aime Symfony2</target>
                    </trans-unit>
                </body>
            </file>
        </xliff>

    .. code-block:: php

        // messages.fr.php
        return array(
            'Symfony2 is great' => 'J\'aime Symfony2',
        );

    .. code-block:: yaml

        # messages.fr.yml
        Symfony2 is great: J'aime Symfony2

Now, if the language of the user's locale is French (e.g. ``fr_FR`` or ``fr_BE``),
the message will be translated into ``J'aime Symfony2``.
现在，如果用户的locale是French（e.g. ``fr_FR``或``fr_BE``），这段message会被翻译为``J'aime Symfony2``。

The Translation Process
翻译流程
~~~~~~~~~~~~~~~~~~~~~~~

To actually translate the message, Symfony2 uses a simple process:
要翻译一段话，symfony2使用了一个简单流程:

* The ``locale`` of the current user, which is stored on the request (or
  stored as ``_locale`` on the session), is determined;
  确定存储在请求上（或存储在session上）的用户的locale；

* A catalog of translated messages is loaded from translation resources defined
  for the ``locale`` (e.g. ``fr_FR``). Messages from the fallback locale are
  also loaded and added to the catalog if they don't already exist. The end
  result is a large "dictionary" of translations. See `Message Catalogues`_
  for more details;
  从为locale（如fr_FR）定义的翻译源文件载入被翻译message的目录。如果它们不存在，从备选locale中的message也被载入并
  添加到目录。最后结果是一个大的翻译“词典”。详情参见`Message Catalogues`_；

* If the message is located in the catalog, the translation is returned. If
  not, the translator returns the original message.
  如果message在目录中存在，翻译就会被返回，如果不存在，就会返回原来的message。

When using the ``trans()`` method, Symfony2 looks for the exact string inside
the appropriate message catalog and returns it (if it exists).
当使用trans()方法时，symfony2查找message目录中的字符串并返回它（如果存在的话）。

.. index::
   single: Translations; Message placeholders

Message Placeholders
~~~~~~~~~~~~~~~~~~~~

Sometimes, a message containing a variable needs to be translated:
有时候，一个包含了变量的message需要被翻译:

.. code-block:: php

    public function indexAction($name)
    {
        $t = $this->get('translator')->trans('Hello '.$name);

        return new Response($t);
    }

However, creating a translation for this string is impossible since the translator
will try to look up the exact message, including the variable portions
(e.g. "Hello Ryan" or "Hello Fabien"). Instead of writing a translation
for every possible iteration of the ``$name`` variable, we can replace the
variable with a "placeholder":
但是对这样一个字符串创建翻译是不可能的，因为symfony2会查找确切的message，包括这个变量部分（比如"Hello Ryan"或"Hello Fabien"）。
我们可以将这个变量替换为一个placeholder，而不是对每个可能出现的$name变量编写翻译:

.. code-block:: php

    public function indexAction($name)
    {
        $t = $this->get('translator')->trans('Hello %name%', array('%name%' => $name));

        new Response($t);
    }

Symfony2 will now look for a translation of the raw message (``Hello %name%``)
and *then* replace the placeholders with their values. Creating a translation
is done just as before:
symfony2会对未被处理的message(``Hello %name%``)查找一个翻译。你可以像刚才一样创建翻译:

.. configuration-block::

    .. code-block:: xml

        <!-- messages.fr.xliff -->
        <?xml version="1.0"?>
        <xliff version="1.2" xmlns="urn:oasis:names:tc:xliff:document:1.2">
            <file source-language="en" datatype="plaintext" original="file.ext">
                <body>
                    <trans-unit id="1">
                        <source>Hello %name%</source>
                        <target>Bonjour %name%</target>
                    </trans-unit>
                </body>
            </file>
        </xliff>

    .. code-block:: php

        // messages.fr.php
        return array(
            'Hello %name%' => 'Bonjour %name%',
        );

    .. code-block:: yaml

        # messages.fr.yml
        'Hello %name%': Hello %name%

.. note::

    The placeholders can take on any form as the full message is reconstructed
    using the PHP `strtr function`_. However, the ``%var%`` notation is
    required when translating in Twig templates, and is overall a sensible
    convention to follow.
    placeholder可以是任何格式，因为整个message会使用PHP `strtr function`_来重构。但是在twig模板中
    翻译时，``%var%``注释是必须的。

As we've seen, creating a translation is a two-step process:
如上所述，创建一个翻译有两个过程:

1. Abstract the message that needs to be translated by processing it through
   the ``Translator``.
1. 将需要翻译的message传递到Translator中，从而抽象它。

2. Create a translation for the message in each locale that you choose to
   support.
2. 在你要支持的每个locale中都创建一个翻译。

The second step is done by creating message catalogues that define the translations
for any number of different locales.
第二步是通过创建定义了所有不同locale翻译的message目录完成的。

.. index::
   single: Translations; Message catalogues

Message Catalogues
message目录
------------------

When a message is translated, Symfony2 compiles a message catalogue for the
user's locale and looks in it for a translation of the message. A message
catalogue is like a dictionary of translations for a specific locale. For
example, the catalogue for the ``fr_FR`` locale might contain the following
translation:
当一个message被翻译后，symfony2会为用户的locale编译一个message catalog，并在其中查找
这个message的翻译。一个message目录就好比对一个特定的locale翻译的字典。比如，fr_FR的目录就
可能包含以下的翻译:

    Symfony2 is Great => J'aime Symfony2

It's the responsibility of the developer (or translator) of an internationalized
application to create these translations. Translations are stored on the
filesystem and discovered by Symfony, thanks to some conventions.
创建这些翻译就是开发者的事情了。翻译会被存储在文件系统上，并能够被symfony查找。

.. tip::

    Each time you create a *new* translation resource (or install a bundle
    that includes a translation resource), be sure to clear your cache so
    that Symfony can discover the new translation resource:
    每当你创建了一个新的翻译源文件时（或创建了一个包含翻译源文件的bundle），一定要
    清空缓存，从而确保symfony能够发现这个新的翻译源文件:
    
    .. code-block:: bash
    
        php app/console cache:clear

.. index::
   single: Translations; Translation resource locations

Translation Locations and Naming Conventions
翻译源文件的放置和命名规则
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Symfony2 looks for message files (i.e. translations) in the following locations:
symfony2在以下目录查找message文件（即翻译源文件）:

* ``<kernel root directory>/Resources/translations``目录;

* ``<kernel root directory>/Resources/<bundle name>/translations``目录;

* bundle的``Resources/translations/``目录.

The locations are listed with the highest priority first. That is you can
override the translation messages of a bundle in any of the top 2 directories.
第一个是有最高优先级的。这样你可以在头2个目录中覆盖bundle的翻译信息。

The override mechanism works at a key level: only the overriden keys need
to be listed in a higher priority message file. When a key is not found
in a message file, the translator will automatically fallback to the lower
priority message files.
覆盖机制在key层面上工作：只有被覆盖的key需要在更高优先级的message文件中列出。当一个key在
message文件中没有找到时，translator会自动使用更低优先级的message文件。

The filename of the translations is also important as Symfony2 uses a convention
to determine details about the translations. Each message file must be named
according to the following pattern: ``domain.locale.loader``:
翻译源文件的名称也很重要，因为symfony2使用它来决定翻译的详情。每个message文件都必须按``domain.locale.loader``
的模式来命名:

* **domain**: An optional way to organize messages into groups (e.g. ``admin``,
  ``navigation`` or the default ``messages``) - see `Using Message Domains`_;
* **domain**: 一个可任选的方法来组织message（如``admin``,``navigation``或者默认的``messages``），参见`Using Message Domains`_；

* **locale**: The locale that the translations are for (e.g. ``en_GB``, ``en``, etc);
* **locale**: 是针对什么locale翻译的（e.g. ``en_GB``, ``en``, etc）；

* **loader**: How Symfony2 should load and parse the file (e.g. ``xliff``,
  ``php`` or ``yml``).
* **loader**: symfony2如何载入并解析这个文件（e.g. ``xliff``,``php`` or ``yml``）

The loader can be the name of any registered loader. By default, Symfony
provides the following loaders:
loader可以是任何被注册的loader的名称。默认情况下，symfony提供以下loader:

* ``xliff``: XLIFF file;
* ``php``:   PHP file;
* ``yml``:  YAML file.

The choice of which loader to use is entirely up to you and is a matter of
taste.
要用哪个loader完全取决于你。

.. note::

    You can also store translations in a database, or any other storage by
    providing a custom class implementing the
    :class:`Symfony\\Component\\Translation\\Loader\\LoaderInterface` interface.
    你可以将翻译存储在数据库中，或通过一个植入了:class:`Symfony\\Component\\Translation\\Loader\\LoaderInterface`
    的类来存储。

.. index::
   single: Translations; Creating translation resources

Creating Translations
创建翻译
~~~~~~~~~~~~~~~~~~~~~

The act of creating translation files is an important part of "localization"
(often abbreviated `L10n`_). Translation files consist of a series of
id-translation pairs for the given domain and locale. The source is the identifier
for the individual translation, and can be the message in the main locale (e.g.
"Symfony is great") of your application or a unique identifier (e.g.
"symfony2.great" - see the sidebar below):
创建翻译源文件是"本地化"（localization，也称`L10n`_）的重要部分。翻译源文件包含了一系列给定
domain和locale的翻译组（用id分组）。源就是单个翻译的指示器，它可以是应用的主要locale（比如"Symfony is great"）
中的message，也可以是一个唯一的指示器（如"symfony2.great"，参见以下部分）的message。

.. configuration-block::

    .. code-block:: xml

        <!-- src/Acme/DemoBundle/Resources/translations/messages.fr.xliff -->
        <?xml version="1.0"?>
        <xliff version="1.2" xmlns="urn:oasis:names:tc:xliff:document:1.2">
            <file source-language="en" datatype="plaintext" original="file.ext">
                <body>
                    <trans-unit id="1">
                        <source>Symfony2 is great</source>
                        <target>J'aime Symfony2</target>
                    </trans-unit>
                    <trans-unit id="2">
                        <source>symfony2.great</source>
                        <target>J'aime Symfony2</target>
                    </trans-unit>
                </body>
            </file>
        </xliff>

    .. code-block:: php

        // src/Acme/DemoBundle/Resources/translations/messages.fr.php
        return array(
            'Symfony2 is great' => 'J\'aime Symfony2',
            'symfony2.great'    => 'J\'aime Symfony2',
        );

    .. code-block:: yaml

        # src/Acme/DemoBundle/Resources/translations/messages.fr.yml
        Symfony2 is great: J'aime Symfony2
        symfony2.great:    J'aime Symfony2

Symfony2 will discover these files and use them when translating either
"Symfony2 is great" or "symfony2.great" into a French language locale (e.g.
``fr_FR`` or ``fr_BE``).
symfony2会查询这些文件并使用它们将"Symfony2 is great"或"symfony2.great"翻译成法文（如``fr_FR`` or ``fr_BE``）。

.. sidebar:: Using Real or Keyword Messages

    This example illustrates the two different philosophies when creating
    messages to be translated:
    这个例子描述了在创建message翻译的时候的两个方法:

    .. code-block:: php

        $t = $translator->trans('Symfony2 is great');

        $t = $translator->trans('symfony2.great');

    In the first method, messages are written in the language of the default
    locale (English in this case). That message is then used as the "id"
    when creating translations.
    在第一种方法中使用的是默认locale（这里是英文）来写这个message，在创建翻译的时候它就会被作为id。

    In the second method, messages are actually "keywords" that convey the
    idea of the message. The keyword message is then used as the "id" for
    any translations. In this case, translations must be made for the default
    locale (i.e. to translate ``symfony2.great`` to ``Symfony2 is great``).
    在第二个方法中，message实际上是被传递了的“关键字”。关键字message被作为翻译的id使用。
    在这个例子中，翻译必须被转换为默认locale（也就是把``symfony2.great``翻译成``Symfony2 is great``）。

    The second method is handy because the message key won't need to be changed
    in every translation file if we decide that the message should actually
    read "Symfony2 is really great" in the default locale.
    推荐使用第二种方法，因为关键字message不必在每个翻译源文件中被修改。比如当我们要把
    Symfony2 is great修改为Symfony2 is really great时，我们就不必修改关键字message了。

    The choice of which method to use is entirely up to you, but the "keyword"
    format is often recommended. 
    你要选择哪种方法都取决于你自己，但我们推荐第二种方法。

    Additionally, the ``php`` and ``yaml`` file formats support nested ids to
    avoid repeating yourself if you use keywords instead of real text for your
    ids:
    还有，如果你使用关键字而不是实际文本，php和yaml文件格式都支持你嵌套id，这样可以避免重复。

    .. configuration-block::

        .. code-block:: yaml

            symfony2:
                is:
                    great: Symfony2 is great
                    amazing: Symfony2 is amazing
                has:
                    bundles: Symfony2 has bundles
            user:
                login: Login

        .. code-block:: php

            return array(
                'symfony2' => array(
                    'is' => array(
                        'great' => 'Symfony2 is great',
                        'amazing' => 'Symfony2 is amazing',
                    ),
                    'has' => array(
                        'bundles' => 'Symfony2 has bundles',
                    ),
                ),
                'user' => array(
                    'login' => 'Login',
                ),
            );

    The multiple levels are flattened into single id/translation pairs by
    adding a dot (.) between every level, therefore the above examples are
    equivalent to the following:
    以上这个嵌套层也可以在每个id和翻译之间加一个点（.），从而写成以下格式:

    .. configuration-block::

        .. code-block:: yaml

            symfony2.is.great: Symfony2 is great
            symfony2.is.amazing: Symfony2 is amazing
            symfony2.has.bundles: Symfony2 has bundles
            user.login: Login

        .. code-block:: php

            return array(
                'symfony2.is.great' => 'Symfony2 is great',
                'symfony2.is.amazing' => 'Symfony2 is amazing',
                'symfony2.has.bundles' => 'Symfony2 has bundles',
                'user.login' => 'Login',
            );

.. index::
   single: Translations; Message domains

Using Message Domains
使用message domain
---------------------

As we've seen, message files are organized into the different locales that
they translate. The message files can also be organized further into "domains".
When creating message files, the domain is the first portion of the filename.
The default domain is ``messages``. For example, suppose that, for organization,
translations were split into three different domains: ``messages``, ``admin``
and ``navigation``. The French translation would have the following message
files:
如你所见，message文件都被组织在不同的locale中，这些message文件还可以被组织在domain中。
在创建message文件的时候，domain就是文件名的第一个部分。默认的domain是messages。比如，如果翻译源文件被
分隔在三个不同的domain中：messages、admin和navigation。法文翻译源文件就是以下三个:

* ``messages.fr.xliff``
* ``admin.fr.xliff``
* ``navigation.fr.xliff``

When translating strings that are not in the default domain (``messages``),
you must specify the domain as the third argument of ``trans()``:
如果你要翻译不存放在默认的domain（messages）下，你必须将domain作为trans()的第三个参数指定:

.. code-block:: php

    $this->get('translator')->trans('Symfony2 is great', array(), 'admin');

Symfony2 will now look for the message in the ``admin`` domain of the user's
locale.
现在symfony2就会在admin这个domain中查找用户的本地message。

.. index::
   single: Translations; User's locale

Handling the User's Locale
处理用户的locale
--------------------------

The locale of the current user is stored in the request and is accessible
via the ``request`` object:
当前用户的locale被存储在请求中，并可以通过request对象访问:

.. code-block:: php

    // access the reqest object in a standard controller
    $request = $this->getRequest();

    $locale = $request->getLocale();

    $request->setLocale('en_US');

.. index::
   single: Translations; Fallback and default locale

It is also possible to store the locale in the session instead of on a per 
request basis. If you do this, each subsequent request will have this locale.
还可以将locale存储在session中，而不是请求中。如果你这样做，每个请求都会使用这个locale。

.. code-block:: php

    $this->get('session')->set('_locale', 'en_US');

See the :ref:`book-translation-locale-url` section below about setting the
locale via routing.
要了解如何通过路由设置locale请参阅:ref:`book-translation-locale-url`部分。

Fallback and Default Locale
备选和默认locale
~~~~~~~~~~~~~~~~~~~~~~~~~~~

If the locale hasn't been set explicitly in the session, the ``fallback_locale``
configuration parameter will be used by the ``Translator``. The parameter
defaults to ``en`` (see `Configuration`_).
如果locale没有在session中显性地设置，Translator就会使用``fallback_locale``配置参数。这个参数
默认为en（参见`Configuration`_）。

Alternatively, you can guarantee that a locale is set on each user's request
by defining a ``default_locale`` for the framework:
你还可以通过定义``default_locale``来确保用户的每个请求都有某个locale:

.. configuration-block::

    .. code-block:: yaml

        # app/config/config.yml
        framework:
            default_locale: en

    .. code-block:: xml

        <!-- app/config/config.xml -->
        <framework:config>
            <framework:default-locale>en</framework:default-locale>
        </framework:config>

    .. code-block:: php

        // app/config/config.php
        $container->loadFromExtension('framework', array(
            'default_locale' => 'en',
        ));

.. versionadded:: 2.1

     The ``default_locale`` parameter was defined under the session key
     originally, however, as of 2.1 this has been moved. This is because the 
     locale is now set on the request instead of the session.
     ``default_locale``已经默认在session参数下设置了，但是，在symfony2.1版本中这个设置被去掉了。
     这是因为locale现在被设置在请求上而不是session上。

.. _book-translation-locale-url:

The Locale and the URL
locale和URL
~~~~~~~~~~~~~~~~~~~~~~

Since you can store the locale of the user is in the session, it may be tempting
to use the same URL to display a resource in many different languages based
on the user's locale. For example, ``http://www.example.com/contact`` could
show content in English for one user and French for another user. Unfortunately,
this violates a fundamental rule of the Web: that a particular URL returns
the same resource regardless of the user. To further muddy the problem, which
version of the content would be indexed by search engines?
由于你将用户的locale存储在session上，你可能想要使用相同的URL来显示不同语言的页面。比如，
``http://www.example.com/contact``可以为一个用户展示英文页面，也可以为另一个用户展示法文页面。
但是这样做违反了一个web的基本法则：不管针对什么用户，一个URL都只能返回相同的源。还有，
搜索引擎将索引页面的哪个内容版本呢？

A better policy is to include the locale in the URL. This is fully-supported
by the routing system using the special ``_locale`` parameter:
更好的方法是将locale包含在URL中。你可以在路由系统中使用``_locale``参数来达到这个目的:

.. configuration-block::

    .. code-block:: yaml

        contact:
            pattern:   /{_locale}/contact
            defaults:  { _controller: AcmeDemoBundle:Contact:index, _locale: en }
            requirements:
                _locale: en|fr|de

    .. code-block:: xml

        <route id="contact" pattern="/{_locale}/contact">
            <default key="_controller">AcmeDemoBundle:Contact:index</default>
            <default key="_locale">en</default>
            <requirement key="_locale">en|fr|de</requirement>
        </route>

    .. code-block:: php

        use Symfony\Component\Routing\RouteCollection;
        use Symfony\Component\Routing\Route;

        $collection = new RouteCollection();
        $collection->add('contact', new Route('/{_locale}/contact', array(
            '_controller' => 'AcmeDemoBundle:Contact:index',
            '_locale'     => 'en',
        ), array(
            '_locale'     => 'en|fr|de'
        )));

        return $collection;

When using the special `_locale` parameter in a route, the matched locale
will *automatically be set on the user's session*. In other words, if a user
visits the URI ``/fr/contact``, the locale ``fr`` will automatically be set
as the locale for the user's session.
当在路径中使用`_locale`参数时，这个匹配的locale会被自动设置在用户的session上。换句话说，如何一个用户访问
URI ``/fr/contact``，fr这个locale会被自动设置在用户的session上。

You can now use the user's locale to create routes to other translated pages
in your application.
现在你可以使用用户的locale来创建通向其他翻译页面的路径了。

.. index::
   single: Translations; Pluralization

Pluralization
复数形式
-------------

Message pluralization is a tough topic as the rules can be quite complex. For
instance, here is the mathematic representation of the Russian pluralization
rules::
message复数形式是一个较难课题，因为规则可能会很复杂。比如以下是俄罗斯的复数规则:

    (($number % 10 == 1) && ($number % 100 != 11)) ? 0 : ((($number % 10 >= 2) && ($number % 10 <= 4) && (($number % 100 < 10) || ($number % 100 >= 20))) ? 1 : 2);

As you can see, in Russian, you can have three different plural forms, each
given an index of 0, 1 or 2. For each form, the plural is different, and
so the translation is also different.
在俄罗斯语言中有三个不同的复数形式，对于每个形式的复数都是不同的，翻译也是不同的。

When a translation has different forms due to pluralization, you can provide
all the forms as a string separated by a pipe (``|``)::
当一个复数的翻译有不同格式时，可以使用管状符号（|）::

    'There is one apple|There are %count% apples'

To translate pluralized messages, use the
:method:`Symfony\\Component\\Translation\\Translator::transChoice` method:
要翻译复数形式的message，使用:method:`Symfony\\Component\\Translation\\Translator::transChoice`:

.. code-block:: php

    $t = $this->get('translator')->transChoice(
        'There is one apple|There are %count% apples',
        10,
        array('%count%' => 10)
    );

The second argument (``10`` in this example), is the *number* of objects being
described and is used to determine which translation to use and also to populate
the ``%count%`` placeholder.
第二个参数（在这个例子中是10）是被描述对象的数目，它将决定哪个翻译要使用，也被用来填充``%count%`` placeholder。

Based on the given number, the translator chooses the right plural form.
In English, most words have a singular form when there is exactly one object
and a plural form for all other numbers (0, 2, 3...). So, if ``count`` is
``1``, the translator will use the first string (``There is one apple``)
as the translation. Otherwise it will use ``There are %count% apples``.
根据给定的数字，translator会选择正确的复数形式。在英语中，大部分单词都对一个对象使用单数形式，
对其他数目则使用复数形式（0,2,3……）。所以如果这里count是1的话，translator就会使用第一个字符串（``There is one apple``），
否则就使用第二个字符串（``There are %count% apples``）。

Here is the French translation::
以下是法文翻译::

    'Il y a %count% pomme|Il y a %count% pommes'

Even if the string looks similar (it is made of two sub-strings separated by a
pipe), the French rules are different: the first form (no plural) is used when
``count`` is ``0`` or ``1``. So, the translator will automatically use the
first string (``Il y a %count% pomme``) when ``count`` is ``0`` or ``1``.
即使这个字符串看起来一样（它有两个由管状符号分隔的子字符串组成），法语规则是不同的：当count是0或1时，
第一个形式（不是复数）被使用。所以当count是0或时，translator会自动使用第一个字符串（``Il y a %count% pomme``）。

Each locale has its own set of rules, with some having as many as six different
plural forms with complex rules behind which numbers map to which plural form.
The rules are quite simple for English and French, but for Russian, you'd
may want a hint to know which rule matches which string. To help translators,
you can optionally "tag" each string::
每个locale都有它自己的规则，有的多达六个不同的复数形式。对于英语和法语很简单，但对俄语就难了。
你要指明哪个字符串对应哪个规则。你可以为每个字符串加标签::

    'one: There is one apple|some: There are %count% apples'

    'none_or_one: Il y a %count% pomme|some: Il y a %count% pommes'

The tags are really only hints for translators and don't affect the logic
used to determine which plural form to use. The tags can be any descriptive
string that ends with a colon (``:``). The tags also do not need to be the
same in the original message as in the translated one.
这些标签仅仅是指明translator如何翻译，它不影响决定要使用哪种复数形式的逻辑。标签可以
是任何以冒号结尾（:）的描述性字符串。标签在源message中不需要和翻译中的一样。

.. tip:

    As tags are optional, the translator doesn't use them (the translator will
    only get a string based on its position in the string).
    标签是可选的，translator不使用它们（translator只根据位置来获取字符串）。

Explicit Interval Pluralization
显性内部复数表示
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

The easiest way to pluralize a message is to let Symfony2 use internal logic
to choose which string to use based on a given number. Sometimes, you'll
need more control or want a different translation for specific cases (for
``0``, or when the count is negative, for example). For such cases, you can
use explicit math intervals::
将一个message转换为复数形式最简单的方法就是让symfony2使用内部逻辑来根据一个给定数字来选择
哪个字符串要使用。有时候你需要对于特殊的情况来翻译（比如0或是负数）。对于这种情况，你可以使用
显性数学形式分隔::

    '{0} There are no apples|{1} There is one apple|]1,19] There are %count% apples|[20,Inf] There are many apples'

The intervals follow the `ISO 31-11`_ notation. The above string specifies
four different intervals: exactly ``0``, exactly ``1``, ``2-19``, and ``20``
and higher.
这个分隔形式遵循`ISO 31-11`_标记。以上字符串指定了四个不同的分隔：0、1、2-19、20及以上。

You can also mix explicit math rules and standard rules. In this case, if
the count is not matched by a specific interval, the standard rules take
effect after removing the explicit rules::
你还可以混合显性规则和标准规则。在这个例子中，如何数量不匹配任何一个分隔，则在移除显性规则后，
标准规则就起作用了::

    '{0} There are no apples|[20,Inf] There are many apples|There is one apple|a_few: There are %count% apples'

For example, for ``1`` apple, the standard rule  will
be used. For ``2-19`` apples, the second standard rule ``There are %count%
apples`` will be selected.
比如，对于1个苹果，标准规则``There is one apple``会被使用。对于2-19个苹果，第二个标准规则``There are %count%
apples``会被使用。

An :class:`Symfony\\Component\\Translation\\Interval` can represent a finite set
of numbers::
:class:`Symfony\\Component\\Translation\\Interval`能对有限数量进行设置::

    {1,2,3,4}

Or numbers between two other numbers::
或两个数量之间的数字::

    [1, +Inf[
    ]-1,2[

The left delimiter can be ``[`` (inclusive) or ``]`` (exclusive). The right
delimiter can be ``[`` (exclusive) or ``]`` (inclusive). Beside numbers, you
can use ``-Inf`` and ``+Inf`` for the infinite.
左边的定界符可以是[（包含）或]（不包含）。右边的定界符可以是[(不包含)或]（包含）。
除了数字，你还可以使用-Inf和+Inf来表示无限。

.. index::
   single: Translations; In templates

Translations in Templates
模板中的翻译
-------------------------

Most of the time, translation occurs in templates. Symfony2 provides native
support for both Twig and PHP templates.
多数情况下，翻译都会在模板中发生。symfony2对php和twig模板都提供翻译支持。

Twig Templates
twig模板
~~~~~~~~~~~~~~

Symfony2 provides specialized Twig tags (``trans`` and ``transchoice``) to
help with message translation of *static blocks of text*:
symfony2提供特定的twig标签（``trans``和``transchoice``）来翻译文本block中的message：

.. code-block:: jinja

    {% trans %}Hello %name%{% endtrans %}

    {% transchoice count %}
        {0} There are no apples|{1} There is one apple|]1,Inf] There are %count% apples
    {% endtranschoice %}

The ``transchoice`` tag automatically gets the ``%count%`` variable from
the current context and passes it to the translator. This mechanism only
works when you use a placeholder following the ``%var%`` pattern.
transchoice标签会自动从当前文本中获取``%count%``变量并将它传递给translator。
你必须写成``%var%``这样的形式。

.. tip::

    If you need to use the percent character (``%``) in a string, escape it by
    doubling it: ``{% trans %}Percent: %percent%%%{% endtrans %}``
    如果你要在字符串中使用百分号（%），要将它写两遍来escape它：``{% trans %}Percent: %percent%%%{% endtrans %}``

You can also specify the message domain and pass some additional variables:
你还可以指定message domain：

.. code-block:: jinja

    {% trans with {'%name%': 'Fabien'} from "app" %}Hello %name%{% endtrans %}

    {% trans with {'%name%': 'Fabien'} from "app" into "fr" %}Hello %name%{% endtrans %}

    {% transchoice count with {'%name%': 'Fabien'} from "app" %}
        {0} There is no apples|{1} There is one apple|]1,Inf] There are %count% apples
    {% endtranschoice %}

The ``trans`` and ``transchoice`` filters can be used to translate *variable
texts* and complex expressions:
过滤器``trans``和``transchoice``也可以用来翻译:

.. code-block:: jinja

    {{ message|trans }}

    {{ message|transchoice(5) }}

    {{ message|trans({'%name%': 'Fabien'}, "app") }}

    {{ message|transchoice(5, {'%name%': 'Fabien'}, 'app') }}

.. tip::

    Using the translation tags or filters have the same effect, but with
    one subtle difference: automatic output escaping is only applied to
    variables translated using a filter. In other words, if you need to
    be sure that your translated variable is *not* output escaped, you must
    apply the raw filter after the translation filter:
    使用标签和过滤器的效果是一样的，但有一点不同：输出的自动escape仅应用于通过过滤器来翻译的变量上。
    如果你不要翻译的变量被escape，你必须在翻译过滤器后面使用raw过滤器：

    .. code-block:: jinja

            {# text translated between tags is never escaped #}
            {% trans %}
                <h3>foo</h3>
            {% endtrans %}

            {% set message = '<h3>foo</h3>' %}

            {# a variable translated via a filter is escaped by default #}
            {{ message|trans|raw }}

            {# but static strings are never escaped #}
            {{ '<h3>foo</h3>'|trans }}

.. versionadded:: 2.1

     You can now set the translation domain for an entire Twig template with a
     single tag:
     你可以为整个twig模板设置一个domain，只要加上一个标签：

     .. code-block:: jinja

            {% trans_default_domain "app" %}

     Note that this only influences the current template, not any "included"
     templates (in order to avoid side effects).
     注意这只会影响当前的模板，而不会影响被包含的模板。

PHP Templates
php模板
~~~~~~~~~~~~~

The translator service is accessible in PHP templates through the
``translator`` helper:
你可以通过translator helper来访问php模板中的translator服务：

.. code-block:: html+php

    <?php echo $view['translator']->trans('Symfony2 is great') ?>

    <?php echo $view['translator']->transChoice(
        '{0} There is no apples|{1} There is one apple|]1,Inf[ There are %count% apples',
        10,
        array('%count%' => 10)
    ) ?>

Forcing the Translator Locale
强制Translator本地化
-----------------------------

When translating a message, Symfony2 uses the locale from the current request
or the ``fallback`` locale if necessary. You can also manually specify the
locale to use for translation:
当翻译一个message的时候，symfony2会使用当前请求的locale或“备选”的locale。你还可以手动设置
要使用的locale：

.. code-block:: php

    $this->get('translator')->trans(
        'Symfony2 is great',
        array(),
        'messages',
        'fr_FR',
    );

    $this->get('translator')->trans(
        '{0} There are no apples|{1} There is one apple|]1,Inf[ There are %count% apples',
        10,
        array('%count%' => 10),
        'messages',
        'fr_FR',
    );

Translating Database Content
翻译数据库内容
----------------------------

The translation of database content should be handled by Doctrine through
the `Translatable Extension`_. For more information, see the documentation
for that library.
对数据库内容的翻译应该由doctrine的`Translatable Extension`_来处理，详情请见该文档。

.. _book-translation-constraint-messages:

Translating Constraint Messages
翻译需验证的信息
-------------------------------

The best way to understand constraint translation is to see it in action. To start,
suppose you've created a plain-old-PHP object that you need to use somewhere in
your application:
假设你已经创建了一个普通php对象：

.. code-block:: php

    // src/Acme/BlogBundle/Entity/Author.php
    namespace Acme\BlogBundle\Entity;

    class Author
    {
        public $name;
    }

Add constraints though any of the supported methods. Set the message option to the
translation source text. For example, to guarantee that the $name property is not
empty, add the following:
添加验证规则。对翻译源文本添加message选项。比如，要保证$name属性不是空的，添加以下代码:

.. configuration-block::

    .. code-block:: yaml

        # src/Acme/BlogBundle/Resources/config/validation.yml
        Acme\BlogBundle\Entity\Author:
            properties:
                name:
                    - NotBlank: { message: "author.name.not_blank" }

    .. code-block:: php-annotations

        // src/Acme/BlogBundle/Entity/Author.php
        use Symfony\Component\Validator\Constraints as Assert;

        class Author
        {
            /**
             * @Assert\NotBlank(message = "author.name.not_blank")
             */
            public $name;
        }

    .. code-block:: xml

        <!-- src/Acme/BlogBundle/Resources/config/validation.xml -->
        <?xml version="1.0" encoding="UTF-8" ?>
        <constraint-mapping xmlns="http://symfony.com/schema/dic/constraint-mapping"
            xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
            xsi:schemaLocation="http://symfony.com/schema/dic/constraint-mapping http://symfony.com/schema/dic/constraint-mapping/constraint-mapping-1.0.xsd">

            <class name="Acme\BlogBundle\Entity\Author">
                <property name="name">
                    <constraint name="NotBlank">
                        <option name="message">author.name.not_blank</option>
                    </constraint>
                </property>
            </class>
        </constraint-mapping>

    .. code-block:: php

        // src/Acme/BlogBundle/Entity/Author.php

        use Symfony\Component\Validator\Mapping\ClassMetadata;
        use Symfony\Component\Validator\Constraints\NotBlank;

        class Author
        {
            public $name;

            public static function loadValidatorMetadata(ClassMetadata $metadata)
            {
                $metadata->addPropertyConstraint('name', new NotBlank(array(
                    'message' => 'author.name.not_blank'
                )));
            }
        }

Create a translation file under the ``validators`` catalog for the constraint messages,
 typically in the ``Resources/translations/`` directory of the bundle. See `Message Catalogues`_ for more details.
在目录validators下为有验证规则的message创建翻译源文件，仍放置在``Resources/translations/``目录下。
详情请参见`Message Catalogues`_。

.. configuration-block::

    .. code-block:: xml

        <!-- validators.fr.xliff -->
        <?xml version="1.0"?>
        <xliff version="1.2" xmlns="urn:oasis:names:tc:xliff:document:1.2">
            <file source-language="en" datatype="plaintext" original="file.ext">
                <body>
                    <trans-unit id="1">
                        <source>author.name.not_blank</source>
                        <target>Please enter an author name.</target>
                    </trans-unit>
                </body>
            </file>
        </xliff>

    .. code-block:: php

        // validators.fr.php
        return array(
            'author.name.not_blank' => 'Please enter an author name.',
        );

    .. code-block:: yaml

        # validators.fr.yml
        author.name.not_blank: Please enter an author name.

Summary
总结
-------

With the Symfony2 Translation component, creating an internationalized application
no longer needs to be a painful process and boils down to just a few basic
steps:
通过symfony2 Translation component，创建一个国际性的应用很简单，只要通过以下几步：

* Abstract messages in your application by wrapping each in either the
  :method:`Symfony\\Component\\Translation\\Translator::trans` or
  :method:`Symfony\\Component\\Translation\\Translator::transChoice` methods;
  将你的应用中的message通过:method:`Symfony\\Component\\Translation\\Translator::trans`或者
  :method:`Symfony\\Component\\Translation\\Translator::transChoice`方法抽象出来；

* Translate each message into multiple locales by creating translation message
  files. Symfony2 discovers and processes each file because its name follows
  a specific convention;
  通过创建翻译源文件，将每个message都根据不同locale翻译出来。symfony2会根据每个文件的名称来查找和执行他们；

* Manage the user's locale, which is stored on the request, but can also
  be set once the user's session.
  管理用户的locale，它可能存储在请求上，或在用户的session上。

.. _`i18n`: http://en.wikipedia.org/wiki/Internationalization_and_localization
.. _`L10n`: http://en.wikipedia.org/wiki/Internationalization_and_localization
.. _`strtr function`: http://www.php.net/manual/en/function.strtr.php
.. _`ISO 31-11`: http://en.wikipedia.org/wiki/Interval_%28mathematics%29#The_ISO_notation
.. _`Translatable Extension`: https://github.com/l3pp4rd/DoctrineExtensions
.. _`ISO3166 Alpha-2`: http://en.wikipedia.org/wiki/ISO_3166-1#Current_codes
.. _`ISO639-1`: http://en.wikipedia.org/wiki/List_of_ISO_639-1_codes
