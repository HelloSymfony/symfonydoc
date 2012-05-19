.. index::
   single: Form; Custom form rendering

How to customize Form Rendering
如何定制表单样式
===============================

Symfony gives you a wide variety of ways to customize how a form is rendered.
In this guide, you'll learn how to customize every possible part of your
form with as little effort as possible whether you use Twig or PHP as your
templating engine.
symfony提供许多方法来让你自定义表单样式。在本节中，你将学习如何使用极少的代码来定制表单的
每个部分，不管是用Twig还是PHP。

Form Rendering Basics
表单样式基础
---------------------

Recall that the label, error and HTML widget of a form field can easily
be rendered by using the ``form_row`` Twig function or the ``row`` PHP helper
method:
以前提到，通过twig的form_row方法或者PHP的row方法能够很轻松地改变表单的label、error和HTML widget：

.. configuration-block::

    .. code-block:: jinja

        {{ form_row(form.age) }}

    .. code-block:: php

        <?php echo $view['form']->row($form['age']) }} ?>

You can also render each of the three parts of the field individually:
你也可以将这三部分分别提交：

.. configuration-block::

    .. code-block:: jinja

        <div>
            {{ form_label(form.age) }}
            {{ form_errors(form.age) }}
            {{ form_widget(form.age) }}
        </div>

    .. code-block:: php

        <div>
            <?php echo $view['form']->label($form['age']) }} ?>
            <?php echo $view['form']->errors($form['age']) }} ?>
            <?php echo $view['form']->widget($form['age']) }} ?>
        </div>

In both cases, the form label, errors and HTML widget are rendered by using
a set of markup that ships standard with Symfony. For example, both of the
above templates would render:
在两个例子中，表单label，error和HTML widget都可以用symfony自带的样式。比如，以上两个模板会提交：

.. code-block:: html

    <div>
        <label for="form_age">Age</label>
        <ul>
            <li>This field is required</li>
        </ul>
        <input type="number" id="form_age" name="form[age]" />
    </div>

To quickly prototype and test a form, you can render the entire form with
just one line:
想要快速测试表单，你可以用一行代码将表单整个提交：

.. configuration-block::

    .. code-block:: jinja

        {{ form_widget(form) }}

    .. code-block:: php

        <?php echo $view['form']->widget($form) }} ?>

The remainder of this recipe will explain how every part of the form's markup
can be modified at several different levels. For more information about form
rendering in general, see :ref:`form-rendering-template`.以下章节会解释表单的各个部分可以如何
被更改。更多关于常用表单样式请参阅:ref:`form-rendering-template`。

.. _cookbook-form-customization-form-themes:

What are Form Themes?
表单主题是什么？
---------------------

Symfony uses form fragments - a small piece of a template that renders just
one part of a form - to render every part of a form - - field labels, errors,
``input`` text fields, ``select`` tags, etc
symfony使用小的表单片段——一个仅提交部分表单的模板——来提交表单的每个部分。比如，字段label，error，
input text字段，select标签，等等。

The fragments are defined as blocks in Twig and as template files in PHP.
片段在twig中被定义为block，而在php中则是模板文件。

A *theme* is nothing more than a set of fragments that you want to use when
rendering a form. In other words, if you want to customize one portion of
how a form is rendered, you'll import a *theme* which contains a customization
of the appropriate form fragments.
一个主题仅仅是你想在提交表单时使用的一系列片段而已。换句话说，如果你想定制提交的表单的某部分，
你要导入一个主题，这个主题包含了你所定制的表单片段。

Symfony comes with a default theme (`form_div_layout.html.twig`_ in Twig and
``FrameworkBundle:Form`` in PHP) that defines each and every fragment needed
to render every part of a form.
symfony有一个默认的主题（在twig中这个主题是`form_div_layout.html.twig`_，在PHP中这个主题是``FrameworkBundle:Form``），
这个主题定义了所有提交表单时需要的片段。

In the next section you will learn how to customize a theme by overriding
some or all of its fragments.
在下一节中你将学习如何通过覆盖某个表单的片段来定制它的样式。

For example, when the widget of a ``integer`` type field is rendered, an ``input``
``number`` field is generated
比如，当一个类型为integer的字段被提交时，一个input number的字段被自动生成：

.. configuration-block::

    .. code-block:: html+jinja

        {{ form_widget(form.age) }}

    .. code-block:: php

        <?php echo $view['form']->widget($form['age']) ?>

renders:

.. code-block:: html

    <input type="number" id="form_age" name="form[age]" required="required" value="33" />

Internally, Symfony uses the ``integer_widget`` fragment  to render the field.
This is because the field type is ``integer`` and you're rendering its ``widget``
(as opposed to its ``label`` or ``errors``).
在内部，symfony使用``integer_widget``片段来提交字段。这是因为这个字段类型是integer并且你是在提交它的widget（而不是label或
errors）。

In Twig that would default to the block ``integer_widget`` from the `form_div_layout.html.twig`_
template.
在twig中那默认是`form_div_layout.html.twig`_中的``integer_widget``这个block。

In PHP it would rather be the ``integer_widget.html.php`` file located in ``FrameworkBundle/Resources/views/Form``
folder.
但在PHP中则是``FrameworkBundle/Resources/views/Form``中的``integer_widget.html.php``文件。

The default implementation of the ``integer_widget`` fragment looks like this:
默认的``integer_widget``片段植入会像这样：

.. configuration-block::

    .. code-block:: jinja

        {% block integer_widget %}
            {% set type = type|default('number') %}
            {{ block('field_widget') }}
        {% endblock integer_widget %}

    .. code-block:: html+php

        <!-- integer_widget.html.php -->

        <?php echo $view['form']->renderBlock('field_widget', array('type' => isset($type) ? $type : "number")) ?>

As you can see, this fragment itself renders another fragment - ``field_widget``:
如你所见，这个片段自己又提交了另一个片段——``field_widget``:

.. configuration-block::

    .. code-block:: html+jinja

        {% block field_widget %}
            {% set type = type|default('text') %}
            <input type="{{ type }}" {{ block('widget_attributes') }} value="{{ value }}" />
        {% endblock field_widget %}

    .. code-block:: html+php

        <!-- FrameworkBundle/Resources/views/Form/field_widget.html.php -->

        <input
            type="<?php echo isset($type) ? $view->escape($type) : "text" ?>"
            value="<?php echo $view->escape($value) ?>"
            <?php echo $view['form']->renderBlock('attributes') ?>
        />

The point is, the fragments dictate the HTML output of each part of a form. To
customize the form output, you just need to identify and override the correct
fragment. A set of these form fragment customizations is known as a form "theme".
When rendering a form, you can choose which form theme(s) you want to apply.
关键是，这个片段决定了表单的每个部分的HTML输出。要想定制表单输出，你只要定制这个片段并
使它覆盖原相对应的片段就行了。一系列的表单定制被称作一个表单主题。当输出表单时，你可以选择主题。

In Twig a theme is a single template file and the fragments are the blocks defined
in this file.
在twig中一个主题是一个单独的模板文件，而片段就是该文件中的block。

In PHP a theme is a folder and the the fragments are individual template files in
this folder.
在php中一个主题就是一个文件，片段则是该文件中的单独模板文件。

.. _cookbook-form-customization-sidebar:

.. sidebar:: Knowing which block to customize

    In this example, the customized fragment name is ``integer_widget`` because
    you want to override the HTML ``widget`` for all ``integer`` field types. If
    you need to customize textarea fields, you would customize ``textarea_widget``.
    在这个例子中，这个被定制的片段名为``integer_widget``，因为你想将所有的integer字段类型
    所输出的widget用HTML标签覆盖。如果你要定制textarea字段，你就要定制``textarea_widget``。

    As you can see, the fragment name is a combination of the field type and
    which part of the field is being rendered (e.g. ``widget``, ``label``,
    ``errors``, ``row``). As such, to customize how errors are rendered for
    just input ``text`` fields, you should customize the ``text_errors`` fragment.
    片段名称是字段类型和表单被输出的部分的结合(e.g. ``widget``, ``label``,
    ``errors``, ``row``)。于是，要定制error在text input中如何输出，你必须定制``text_errors``
    这个片段。

    More commonly, however, you'll want to customize how errors are displayed
    across *all* fields. You can do this by customizing the ``field_errors``
    fragment. This takes advantage of field type inheritance. Specifically,
    since the ``text`` type extends from the ``field`` type, the form component
    will first look for the type-specific fragment (e.g. ``text_errors``) before
    falling back to its parent fragment name if it doesn't exist (e.g. ``field_errors``).
    更经常的，你要定制errors如何在所有的片段中输出。你可以使用``field_errors``片段。这是利用了字段类型继承。
    尤其是，由于text字段是由field字段扩展的，表单输出机制会首先查看特定片段(e.g. ``text_errors``)，如果没有的话
    就会查看它的父片段名(e.g. ``field_errors``)。

    For more information on this topic, see :ref:`form-template-blocks`.
    更多请参阅:ref:`form-template-blocks`。

.. _cookbook-form-theming-methods:

Form Theming
表单主题
------------

To see the power of form theming, suppose you want to wrap every input ``number``
field with a ``div`` tag. The key to doing this is to customize the
``integer_widget`` fragment.
比如，假设你想用div标签包围每个input number字段。要达到这个目的，你可以定制``integer_widget``片段。

Form Theming in Twig
twig中的表单主题
--------------------

When customizing the form field block in Twig, you have two options on *where*
the customized form block can live:
当在twig中定制表单block的时候，关于要把你定制的block放在哪儿，有两个选择：

+--------------------------------------+-----------------------------------+-------------------------------------------+
| 方法                                 | 有利方面                          | 不利方面                                  |
+======================================+===================================+===========================================+
| 在相同表单的模板中                   | 快，方便                          | 不能被重复使用                            |
+--------------------------------------+-----------------------------------+-------------------------------------------+
| 在一个不同的文件里                   | 能重复使用                        |一个额外的模板要被创建                     |
+--------------------------------------+-----------------------------------+-------------------------------------------+

Both methods have the same effect but are better in different situations.
两种方法都有相同效果，但是要视情况而利用。

.. _cookbook-form-twig-theming-self:

Method 1: Inside the same Template as the Form
方法1：在表单的同一模板中
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

The easiest way to customize the ``integer_widget`` block is to customize it
directly in the template that's actually rendering the form.
定制这个``integer_widget`` block最容易的方法就是直接在与要输出的表单同一个模板中定制它。

.. code-block:: html+jinja

    {% extends '::base.html.twig' %}

    {% form_theme form _self %}

    {% block integer_widget %}
        <div class="integer_widget">
            {% set type = type|default('number') %}
            {{ block('field_widget') }}
        </div>
    {% endblock %}

    {% block content %}
        {# render the form #}

        {{ form_row(form.age) }}
    {% endblock %}

By using the special ``{% form_theme form _self %}`` tag, Twig looks inside
the same template for any overridden form blocks. Assuming the ``form.age``
field is an ``integer`` type field, when its widget is rendered, the customized
``integer_widget`` block will be used.
通过使用这个``{% form_theme form _self %}``标签，twig会在相同的模板中查找所有被覆盖的
表单block。假设form.age字段是一个integer类型字段，当它的widget被输出的时候，这个被定制的
``integer_widget`` block会被运用。

The disadvantage of this method is that the customized form block can't be
reused when rendering other forms in other templates. In other words, this method
is most useful when making form customizations that are specific to a single
form in your application. If you want to reuse a form customization across
several (or all) forms in your application, read on to the next section.
这个方法的弊端是定制的表单block不能被其他模板使用。换句话说，这个方法适用于某个单独的表单的特殊定制。
如果你想要对数个（或全部）表单都使用这个定制，请阅读下面章节。

.. _cookbook-form-twig-separate-template:

Method 2: Inside a Separate Template
方法2：使用单独的模板
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

You can also choose to put the customized ``integer_widget`` form block in a
separate template entirely. The code and end-result are the same, but you
can now re-use the form customization across many templates:
你也可以选择将定制的``integer_widget``表单block放在一个单独的模板中。代码和最终结果都是一样的，
但你可以重复使用它：

.. code-block:: html+jinja

    {# src/Acme/DemoBundle/Resources/views/Form/fields.html.twig #}

    {% block integer_widget %}
        <div class="integer_widget">
            {% set type = type|default('number') %}
            {{ block('field_widget') }}
        </div>
    {% endblock %}

Now that you've created the customized form block, you need to tell Symfony
to use it. Inside the template where you're actually rendering your form,
tell Symfony to use the template via the ``form_theme`` tag:
现在你已经创建了这个定制的表单block，你还要告诉symfony使用它。在你要输出表单的那个
模板的内部，告诉symfony你要通过``form_theme``标签来使用定制的模板：

.. _cookbook-form-twig-theme-import-template:

.. code-block:: html+jinja

    {% form_theme form 'AcmeDemoBundle:Form:fields.html.twig' %}

    {{ form_widget(form.age) }}

When the ``form.age`` widget is rendered, Symfony will use the ``integer_widget``
block from the new template and the ``input`` tag will be wrapped in the
``div`` element specified in the customized block.
当``form.age`` widget被输出时，symfony会使用新模板中的``integer_widget`` block，并且
这个input标签会被包围在div标签中。

.. _cookbook-form-php-theming:

Form Theming in PHP
php中的表单主题
-------------------

When using PHP as a templating engine, the only method to customize a fragment
is to create a new template file - this is similar to the second method used by
Twig.
当使用php作为模板引擎时，定制一个片段的唯一方法就是创建一个新的模板文件——这跟上面讲的twig中的
第二种方法很相似。

The template file must be named after the fragment. You must create a ``integer_widget.html.php``
file in order to customize the ``integer_widget`` fragment.
模板文件必须和片段名一样。你必须创建一个``integer_widget.html.php``文件，从而来定制``integer_widget``片段。

.. code-block:: html+php

    <!-- src/Acme/DemoBundle/Resources/views/Form/integer_widget.html.php -->

    <div class="integer_widget">
        <?php echo $view['form']->renderBlock('field_widget', array('type' => isset($type) ? $type : "number")) ?>
    </div>

Now that you've created the customized form template, you need to tell Symfony
to use it. Inside the template where you're actually rendering your form,
tell Symfony to use the theme via the ``setTheme`` helper method:
现在你已经创建好了表单模板，你需要告诉symfony使用它。在你要输出表单的模板的内部，告诉symfony
通过使用``setTheme``方法来使用这个主题：

.. _cookbook-form-php-theme-import-template:

.. code-block:: php

    <?php $view['form']->setTheme($form, array('AcmeDemoBundle:Form')) ;?>

    <?php $view['form']->widget($form['age']) ?>

When the ``form.age`` widget is rendered, Symfony will use the customized
``integer_widget.html.php`` template and the ``input`` tag will be wrapped in
the ``div`` element.
当``form.age``的widget被输出时，symfony会使用这个定制的``integer_widget.html.php``模板并且
input标签会被div标签包围。

.. _cookbook-form-twig-import-base-blocks:

Referencing Base Form Blocks (Twig specific)
访问基本表单block（twig特有）
--------------------------------------------

So far, to override a particular form block, the best method is to copy
the default block from `form_div_layout.html.twig`_, paste it into a different template,
and the customize it. In many cases, you can avoid doing this by referencing
the base block when customizing it.
目前为止，要覆盖一个特定表单block，最有效的方法就是复制默认的表单样式文件`form_div_layout.html.twig`_代码
然后粘贴到另外一个模板中并修改它。但你可以通过访问基本表单block来避免这样做。

This is easy to do, but varies slightly depending on if your form block customizations
are in the same template as the form or a separate template.
这很容易，但是根据你的表单定制的block是在与所要输出的表单在同一文件中或是在不同文件中而有所不同。

Referencing Blocks from inside the same Template as the Form
从相同的模板内部访问基本表单block
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Import the blocks by adding a ``use`` tag in the template where you're rendering
the form:
通过在你要输出的表单模板中添加一个use标签，可以导入基本表单的block：

.. code-block:: jinja

    {% use 'form_div_layout.html.twig' with integer_widget as base_integer_widget %}

Now, when the blocks from `form_div_layout.html.twig`_ are imported, the
``integer_widget`` block is called ``base_integer_widget``. This means that when
you redefine the ``integer_widget`` block, you can reference the default markup
via ``base_integer_widget``:
现在，当`form_div_layout.html.twig`_中的block被导入时，原来的``integer_widget``被称作是``base_integer_widget``。
这表示当你重新定义``integer_widget``的时候，你可以通过访问``base_integer_widget``来访问原来那个。

.. code-block:: html+jinja

    {% block integer_widget %}
        <div class="integer_widget">
            {{ block('base_integer_widget') }}
        </div>
    {% endblock %}

Referencing Base Blocks from an External Template
从外部模板中访问基本表单
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

If your form customizations live inside an external template, you can reference
the base block by using the ``parent()`` Twig function:
如果你的表单定制在外部模板中，你可以通过``parent()``这个twig的方法来访问基本表单block：

.. code-block:: html+jinja

    {# src/Acme/DemoBundle/Resources/views/Form/fields.html.twig #}

    {% extends 'form_div_layout.html.twig' %}

    {% block integer_widget %}
        <div class="integer_widget">
            {{ parent() }}
        </div>
    {% endblock %}

.. note::

    It is not possible to reference the base block when using PHP as the
    templating engine. You have to manually copy the content from the base block
    to your new template file.
    如果使用php作为模板引擎，访问基本表单block是不可能的。你必须手动复制基本表单中的内容到新的模板。

.. _cookbook-form-global-theming:

Making Application-wide Customizations
创建整个应用内的定制
--------------------------------------

If you'd like a certain form customization to be global to your application,
you can accomplish this by making the form customizations in an external
template and then importing it inside your application configuration:
如果你想要某个表单定制对于你的整个应用都可以使用，你可以将这个定制放在一个外部文件中
并将它导入你的应用配置：

Twig
~~~~

By using the following configuration, any customized form blocks inside the
``AcmeDemoBundle:Form:fields.html.twig`` template will be used globally when a
form is rendered.
通过以下配置，所有在``AcmeDemoBundle:Form:fields.html.twig``中
的表单block都会在整个应用中被使用。

.. configuration-block::

    .. code-block:: yaml

        # app/config/config.yml

        twig:
            form:
                resources:
                    - 'AcmeDemoBundle:Form:fields.html.twig'
            # ...

    .. code-block:: xml

        <!-- app/config/config.xml -->

        <twig:config ...>
                <twig:form>
                    <resource>AcmeDemoBundle:Form:fields.html.twig</resource>
                </twig:form>
                <!-- ... -->
        </twig:config>

    .. code-block:: php

        // app/config/config.php

        $container->loadFromExtension('twig', array(
            'form' => array('resources' => array(
                'AcmeDemoBundle:Form:fields.html.twig',
             ))
            // ...
        ));

By default, Twig uses a *div* layout when rendering forms. Some people, however,
may prefer to render forms in a *table* layout. Use the ``form_table_layout.html.twig``
resource to use such a layout:
默认地，twig使用一个div来布局表单。但有些人可能更喜欢用table标签。使用``form_table_layout.html.twig``：

.. configuration-block::

    .. code-block:: yaml

        # app/config/config.yml

        twig:
            form:
                resources: ['form_table_layout.html.twig']
            # ...

    .. code-block:: xml

        <!-- app/config/config.xml -->

        <twig:config ...>
                <twig:form>
                    <resource>form_table_layout.html.twig</resource>
                </twig:form>
                <!-- ... -->
        </twig:config>

    .. code-block:: php

        // app/config/config.php

        $container->loadFromExtension('twig', array(
            'form' => array('resources' => array(
                'form_table_layout.html.twig',
             ))
            // ...
        ));

If you only want to make the change in one template, add the following line to
your template file rather than adding the template as a resource:
如果你只想把它应用到一个模板中，只需要将以下语句添加到你的模板文件中，而不是加到配置文件中：

.. code-block:: html+jinja

	{% form_theme form 'form_table_layout.html.twig' %}

Note that the ``form`` variable in the above code is the form view variable
that you passed to your template.
注意上面的这个form变量是你从控制器中传递到模板的form view变量。

PHP
~~~

By using the following configuration, any customized form fragments inside the
``src/Acme/DemoBundle/Resources/views/Form`` folder will be used globally when a
form is rendered.
通过使用下面的配置，所有在``src/Acme/DemoBundle/Resources/views/Form``中的定制的片段会在整个应用中使用。

.. configuration-block::

    .. code-block:: yaml

        # app/config/config.yml

        framework:
            templating:
                form:
                    resources:
                        - 'AcmeDemoBundle:Form'
            # ...


    .. code-block:: xml

        <!-- app/config/config.xml -->

        <framework:config ...>
            <framework:templating>
                <framework:form>
                    <resource>AcmeDemoBundle:Form</resource>
                </framework:form>
            </framework:templating>
            <!-- ... -->
        </framework:config>


    .. code-block:: php

        // app/config/config.php

        // PHP
        $container->loadFromExtension('framework', array(
            'templating' => array('form' =>
                array('resources' => array(
                    'AcmeDemoBundle:Form',
             )))
            // ...
        ));

By default, the PHP engine uses a *div* layout when rendering forms. Some people,
however, may prefer to render forms in a *table* layout. Use the ``FrameworkBundle:FormTable``
resource to use such a layout:
默认地，php引擎会使用div，如果想用table标签，请使用``FrameworkBundle:FormTable``：

.. configuration-block::

    .. code-block:: yaml

        # app/config/config.yml

        framework:
            templating:
                form:
                    resources:
                        - 'FrameworkBundle:FormTable'

    .. code-block:: xml

        <!-- app/config/config.xml -->

        <framework:config ...>
            <framework:templating>
                <framework:form>
                    <resource>FrameworkBundle:FormTable</resource>
                </framework:form>
            </framework:templating>
            <!-- ... -->
        </framework:config>

    .. code-block:: php

        // app/config/config.php

        $container->loadFromExtension('framework', array(
            'templating' => array('form' =>
                array('resources' => array(
                    'FrameworkBundle:FormTable',
             )))
            // ...
        ));

If you only want to make the change in one template, add the following line to
your template file rather than adding the template as a resource:
如果你只想在某一个模板中修改，在模板中加上以下语句：

.. code-block:: html+php

	<?php $view['form']->setTheme($form, array('FrameworkBundle:FormTable')); ?>

Note that the ``$form`` variable in the above code is the form view variable
that you passed to your template.
注意这个$form变量是你传递到你的模板中的变量。

How to customize an Individual field
如何定制字段
------------------------------------

So far, you've seen the different ways you can customize the widget output
of all text field types. You can also customize individual fields. For example,
suppose you have two ``text`` fields - ``first_name`` and ``last_name`` - but
you only want to customize one of the fields. This can be accomplished by
customizing a fragment whose name is a combination of the field id attribute and
which part of the field is being customized. For example:
目前为止，你已经了解了你能够定制所有text类型的widget。比如，如果你有两个text字段——``first_name``和``last_name``，
但是你只想定制其中一个。要达到这个目的，你只要定制一个片段，这个片段的名称是这个字段的id属性和
该字段的那个被定制的部分的名字（如widget，label，errors）的结合：

.. configuration-block::

    .. code-block:: html+jinja

        {% form_theme form _self %}

        {% block _product_name_widget %}
            <div class="text_widget">
                {{ block('field_widget') }}
            </div>
        {% endblock %}

        {{ form_widget(form.name) }}

    .. code-block:: html+php

        <!-- Main template -->

        <?php echo $view['form']->setTheme($form, array('AcmeDemoBundle:Form')); ?>

        <?php echo $view['form']->widget($form['name']); ?>

        <!-- src/Acme/DemoBundle/Resources/views/Form/_product_name_widget.html.php -->

        <div class="text_widget">
              echo $view['form']->renderBlock('field_widget') ?>
        </div>

Here, the ``_product_name_widget`` fragment defines the template to use for the
field whose *id* is ``product_name`` (and name is ``product[name]``).
在这里，``_product_name_widget``片段定义了这个id是``product_name`` (name是``product[name]``)的字段的模板。

.. tip::

   The ``product`` portion of the field is the form name, which may be set
   manually or generated automatically based on your form type name (e.g.
   ``ProductType`` equates to ``product``). If you're not sure what your
   form name is, just view the source of your generated form.
   这个字段的product部分是表单名，它可以被在你的表单类型名称上被手动或者自动添加（e.g.
   ``ProductType``等同于``product``）。如果你不确定你的表单名称，请参照你自己集成的表单。

You can also override the markup for an entire field row using the same method:
你也可以使用相同方法覆盖整个字段：

.. configuration-block::

    .. code-block:: html+jinja

        {% form_theme form _self %}

        {% block _product_name_row %}
            <div class="name_row">
                {{ form_label(form) }}
                {{ form_errors(form) }}
                {{ form_widget(form) }}
            </div>
        {% endblock %}

    .. code-block:: html+php

        <!-- _product_name_row.html.php -->

        <div class="name_row">
            <?php echo $view['form']->label($form) ?>
            <?php echo $view['form']->errors($form) ?>
            <?php echo $view['form']->widget($form) ?>
        </div>

Other Common Customizations
其它常用的定制
---------------------------

So far, this recipe has shown you several different ways to customize a single
piece of how a form is rendered. The key is to customize a specific fragment that
corresponds to the portion of the form you want to control (see
:ref:`naming form blocks<cookbook-form-customization-sidebar>`).
现在，本章已经展示给你如何使用不同的方法来定制一个表单的某部分。关键点是定制一个与你想要修改的表单中的
部分相应的特定的片段（参考:ref:`naming form blocks<cookbook-form-customization-sidebar>`）。

In the next sections, you'll see how you can make several common form customizations.
To apply these customizations, use one of the methods described in the
:ref:`cookbook-form-theming-methods` section.
下一节你将学习如何创建数个常用表单定制。要应用这些定制，参见本节:ref:`cookbook-form-theming-methods`。

Customizing Error Output
定制错误输出
~~~~~~~~~~~~~~~~~~~~~~~~

.. note::
   The form component only handles *how* the validation errors are rendered,
   and not the actual validation error messages. The error messages themselves
   are determined by the validation constraints you apply to your objects.
   For more information, see the chapter on :doc:`validation</book/validation>`.
   本节讲述验证如何被输出，而不是如何验证。这些错误信息都是由你自己通过验证规则决定的。
   更多信息请参见:doc:`validation</book/validation>`。

There are many different ways to customize how errors are rendered when a
form is submitted with errors. The error messages for a field are rendered
when you use the ``form_errors`` helper:
有许多不同的方法可以定制错误信息。当你使用``form_errors``方法的时候，错误信息就会被提交了：

.. configuration-block::

    .. code-block:: jinja

        {{ form_errors(form.age) }}

    .. code-block:: php

        <?php echo $view['form']->errors($form['age']); ?>

By default, the errors are rendered inside an unordered list:
默认情况下，错误信息是一个无序列表：

.. code-block:: html

    <ul>
        <li>This field is required</li>
    </ul>

To override how errors are rendered for *all* fields, simply copy, paste
and customize the ``field_errors`` fragment.
要想覆盖所有字段的错误信息，只要粘贴、复制并定制``field_errors``片段就可以了。

.. configuration-block::

    .. code-block:: html+jinja

        {% block field_errors %}
        {% spaceless %}
            {% if errors|length > 0 %}
            <ul class="error_list">
                {% for error in errors %}
                    <li>{{ error.messageTemplate|trans(error.messageParameters, 'validators') }}</li>
                {% endfor %}
            </ul>
            {% endif %}
        {% endspaceless %}
        {% endblock field_errors %}

    .. code-block:: html+php

        <!-- fields_errors.html.php -->

        <?php if ($errors): ?>
            <ul class="error_list">
                <?php foreach ($errors as $error): ?>
                    <li><?php echo $view['translator']->trans(
                        $error->getMessageTemplate(),
                        $error->getMessageParameters(),
                        'validators'
                    ) ?></li>
                <?php endforeach; ?>
            </ul>
        <?php endif ?>

.. tip::
    See :ref:`cookbook-form-theming-methods` for how to apply this customization.
    应用该定制请参见:ref:`cookbook-form-theming-methods`。

You can also customize the error output for just one specific field type.
For example, certain errors that are more global to your form (i.e. not specific
to just one field) are rendered separately, usually at the top of your form:
你也可以只针对某一个字段类型定制错误。比如，一个对于你的表单来说比较全局性的错误（而不是针对某个
特定字段）可以被分开输出，往往是在你的表单的头部：

.. configuration-block::

    .. code-block:: jinja

        {{ form_errors(form) }}

    .. code-block:: php

        <?php echo $view['form']->render($form); ?>

To customize *only* the markup used for these errors, follow the same directions
as above, but now call the block ``form_errors`` (Twig) / the file ``form_errors.html.php``
(PHP). Now, when errors for the ``form`` type are rendered, your customized
fragment will be used instead of the default ``field_errors``.
要仅仅定制这种错误，方法和上面讲的一样，但是要使用``form_errors`` (Twig) / the file ``form_errors.html.php``
(PHP)。现在，当错误被输出时，你定制的片段会被应用。

Customizing the "Form Row"
定制"Form Row"
~~~~~~~~~~~~~~~~~~~~~~~~~~

When you can manage it, the easiest way to render a form field is via the
``form_row`` function, which renders the label, errors and HTML widget of
a field. To customize the markup used for rendering *all* form field rows,
override the ``field_row`` fragment. For example, suppose you want to add a
class to the ``div`` element around each row:
最方便的方法就是使用``form_row``来输出表单，它输出字段的label，errors，以及HTML widget。
要定制所有的表单字段的row，你需要覆盖``field_row``片段。比如，假设你想给所有row的div标签添加一个类：

.. configuration-block::

    .. code-block:: html+jinja

        {% block field_row %}
            <div class="form_row">
                {{ form_label(form) }}
                {{ form_errors(form) }}
                {{ form_widget(form) }}
            </div>
        {% endblock field_row %}

    .. code-block:: html+php

        <!-- field_row.html.php -->

        <div class="form_row">
            <?php echo $view['form']->label($form) ?>
            <?php echo $view['form']->errors($form) ?>
            <?php echo $view['form']->widget($form) ?>
        </div>

.. tip::
    See :ref:`cookbook-form-theming-methods` for how to apply this customization.
    如何应用这个定制请参见:ref:`cookbook-form-theming-methods`。

Adding a "Required" Asterisk to Field Labels
给字段标签加星号
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

If you want to denote all of your required fields with a required asterisk (``*``),
you can do this by customizing the ``field_label`` fragment.
如果你想要给所有你要求填写（有required="required"属性）的字段加上星号（*），你可以定制``field_label``片段。

In Twig, if you're making the form customization inside the same template as your
form, modify the ``use`` tag and add the following:
在twig中，如果你要把表单定制代码放在你的表单模板中，修改一下use标签：

.. code-block:: html+jinja

    {% use 'form_div_layout.html.twig' with field_label as base_field_label %}

    {% block field_label %}
        {{ block('base_field_label') }}

        {% if required %}
            <span class="required" title="This field is required">*</span>
        {% endif %}
    {% endblock %}

In Twig, if you're making the form customization inside a separate template, use
the following:

.. code-block:: html+jinja

    {% extends 'form_div_layout.html.twig' %}

    {% block field_label %}
        {{ parent() }}

        {% if required %}
            <span class="required" title="This field is required">*</span>
        {% endif %}
    {% endblock %}

When using PHP as a templating engine you have to copy the content from the
original template:
如果你使用php作为模板引擎你可以从源模板中复制内容：

.. code-block:: html+php

    <!-- field_label.html.php -->

    <!-- original content -->
    <label for="<?php echo $view->escape($id) ?>" <?php foreach($attr as $k => $v) { printf('%s="%s" ', $view->escape($k), $view->escape($v)); } ?>><?php echo $view->escape($view['translator']->trans($label)) ?></label>

    <!-- customization -->
    <?php if ($required) : ?>
        <span class="required" title="This field is required">*</span>
    <?php endif ?>

.. tip::
    See :ref:`cookbook-form-theming-methods` for how to apply this customization.

Adding "help" messages
添加help信息
~~~~~~~~~~~~~~~~~~~~~~

You can also customize your form widgets to have an optional "help" message.
你还可以通过可选的help信息来定制表单widget。

In Twig, If you're making the form customization inside the same template as your
form, modify the ``use`` tag and add the following:
在twig中，如果你在表单模板内使用表单定制代码，可以修改use标签：

.. code-block:: html+jinja

    {% use 'form_div_layout.html.twig' with field_widget as base_field_widget %}

    {% block field_widget %}
        {{ block('base_field_widget') }}

        {% if help is defined %}
            <span class="help">{{ help }}</span>
        {% endif %}
    {% endblock %}

In twig, If you're making the form customization inside a separate template, use
the following:
如果在另一个文件写定制代码，可以：

.. code-block:: html+jinja

    {% extends 'form_div_layout.html.twig' %}

    {% block field_widget %}
        {{ parent() }}

        {% if help is defined %}
            <span class="help">{{ help }}</span>
        {% endif %}
    {% endblock %}

When using PHP as a templating engine you have to copy the content from the
original template:
当使用php时，你必须从源模板中复制代码：

.. code-block:: html+php

    <!-- field_widget.html.php -->

    <!-- Original content -->
    <input
        type="<?php echo isset($type) ? $view->escape($type) : "text" ?>"
        value="<?php echo $view->escape($value) ?>"
        <?php echo $view['form']->renderBlock('attributes') ?>
    />

    <!-- Customization -->
    <?php if (isset($help)) : ?>
        <span class="help"><?php echo $view->escape($help) ?></span>
    <?php endif ?>

To render a help message below a field, pass in a ``help`` variable:
要在一个字段下方提交help信息，可以 传递help参数：

.. configuration-block::

    .. code-block:: jinja

        {{ form_widget(form.title, { 'help': 'foobar' }) }}

    .. code-block:: php

        <?php echo $view['form']->widget($form['title'], array('help' => 'foobar')) ?>

.. tip::
    See :ref:`cookbook-form-theming-methods` for how to apply this customization.

.. _`form_div_layout.html.twig`: https://github.com/symfony/symfony/blob/master/src/Symfony/Bridge/Twig/Resources/views/Form/form_div_layout.html.twig