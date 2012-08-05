.. index::
   single: Form; Custom field type

How to Create a Custom Form Field Type
如何创建一个定制的表单字段类型
======================================

Symfony comes with a bunch of core field types available for building forms.
However there are situations where we want to create a custom form field
type for a specific purpose. This recipe assumes we need a field definition
that holds a person's gender, based on the existing choice field. This section
explains how the field is defined, how we can customize its layout and finally,
how we can register it for use in our application.
symfony有一套创建表单要用到的核心字段类型。但是有时候我们也需要自己创建一个定制的表单
类型。本章假设我们需要一个在choice字段基础上，但可以选择人物性别的字段定义。本章介绍如何定义字段，
如何定制该字段的样式，最后介绍如何注册它从而使它能够被应用。

Defining the Field Type
定义一个字段类型
-----------------------

In order to create the custom field type, first we have to create the class
representing the field. In our situation the class holding the field type
will be called `GenderType` and the file will be stored in the default location
for form fields, which is ``<BundleName>\Form\Type``. Make sure the field extends
:class:`Symfony\\Component\\Form\\AbstractType`::
要创建一个字段类型，首先我们必须创建一个代表这个字段的类。在本章中这个存储该字段的类将被称作`GenderType`，
并存储在默认位置下，即``<BundleName>\Form\Type``。要确定这个字段扩展了:class:`Symfony\\Component\\Form\\AbstractType`::

    # src/Acme/DemoBundle/Form/Type/GenderType.php
    namespace Acme\DemoBundle\Form\Type;

    use Symfony\Component\Form\AbstractType;
    use Symfony\Component\Form\FormBuilder;

    class GenderType extends AbstractType
    {
        public function getDefaultOptions()
        {
            return array(
                'choices' => array(
                    'm' => 'Male',
                    'f' => 'Female',
                )
            );
        }

        public function getParent(array $options)
        {
            return 'choice';
        }

        public function getName()
        {
            return 'gender';
        }
    }

.. tip::

    The location of this file is not important - the ``Form\Type`` directory
    is just a convention.
    文件的位置并不重要——``Form\Type``目录仅仅是一个惯例。

Here, the return value of the ``getParent`` function indicates that we're
extending the ``choice`` field type. This means that, by default, we inherit
all of the logic and rendering of that field type. To see some of the logic,
check out the `ChoiceType`_ class. There are three methods that are particularly
important:
在这里，被返回的``getParent``方法的值表示我们要扩展choice这个字段类型。这意味着我们会继承
那个字段类型的所有逻辑和输出样式。要了解它的逻辑，请参阅`ChoiceType`_类。有三种方法非常重要：

* ``buildForm()`` - Each field type has a ``buildForm`` method, which is where
  you configure and build any field(s). Notice that this is the same method
  you use to setup *your* forms, and it works the same here.
* ``buildForm()``——每个字段类型都有一个``buildForm``方法，它能配置并创建所有字段。注意这个方法
  和你所用来设置表单的方法是同一个，工作原理也一样。

* ``buildView()`` - This method is used to set any extra variables you'll
  need when rendering your field in a template. For example, in `ChoiceType`_,
  a ``multiple`` variable is set and used in the template to set (or not
  set) the ``multiple`` attribute on the ``select`` field. See `Creating a Template for the Field`_
  for more details.
* ``buildView()``——这个方法用来设置任何你输出表单时所要用到的额外变量。比如，在`ChoiceType`_中，
  一个multiple变量被设置并且在模板中被使用，从而在select字段上添加mutiple属性。参见`Creating a Template for the Field`_。

* ``getDefaultOptions()`` - This defines options for your form type that
  can be used in ``buildForm()`` and ``buildView()``. There are a lot of
  options common to all fields (see `FieldType`_), but you can create any
  others that you need here.
* ``getDefaultOptions()``——它定义了可以在``buildForm()``和``buildView()``中使用的选项。
  所有的字段都有许多常用选项（参见`FieldType`_），但你也可以创建你自己的选项。

.. tip::

    If you're creating a field that consists of many fields, then be sure
    to set your "parent" type as ``form`` or something that extends ``form``.
    Also, if you need to modify the "view" of any of your child types from
    your parent type, use the ``buildViewBottomUp()`` method.
    如果你要创建一个包含许多字段的字段，那么要确保将parent类型设置为form或扩展form的类型。
    并且，如果你需要从parent类型中修改子类型的view，要使用``buildViewBottomUp()``方法。

The ``getName()`` method returns an identifier which should be unique in
your application. This is used in various places, such as when customizing
how your form type will be rendered.
``getName()``方法返回的是一个标识符，而且在你的应用中，应该是唯一的。它可以被用于许多地方，比如说当
定制你的表单如何输出时。

The goal of our field was to extend the choice type to enable selection of
a gender. This is achieved by fixing the ``choices`` to a list of possible
genders.
我们定制字段的目的是要扩展choice字段并使它创建性别选项。通过将choices选项固定到
一系列的性别选项就可以达到这个目的。

Creating a Template for the Field
为字段创建一个模板
---------------------------------

Each field type is rendered by a template fragment, which is determined in
part by the value of your ``getName()`` method. For more information, see
:ref:`cookbook-form-customization-form-themes`.
每个字段类型都有一个对应的模板片段，该片段的名称有一部分则由这个``getName()``方法确定。
参见:ref:`cookbook-form-customization-form-themes`。

In this case, since our parent field is ``choice``, we don't *need* to do
any work as our custom field type will automatically be rendered like a ``choice``
type. But for the sake of this example, let's suppose that when our field
is "expanded" (i.e. radio buttons or checkboxes, instead of a select field),
we want to always render it in a ``ul`` element. In your form theme template
(see above link for details), create a ``gender_widget`` block to handle this:
在这个例子中，由于我们的parent字段是choice，不需要任何工作，我们定制的字段类型就会作为choice类型输出。
但是假设如果我们的字段有expanded选项（被设置为单选或复选框），而且我们想始终使用ul标签来输出它。在你
的表单主题模板中（参见:ref:`cookbook-form-customization-form-themes`），创建一个``gender_widget`` block：

.. code-block:: html+jinja

    {# src/Acme/DemoBundle/Resources/views/Form/fields.html.twig #}

    {% block gender_widget %}
    {% spaceless %}
        {% if expanded %}
            <ul {{ block('widget_container_attributes') }}>
            {% for child in form %}
                <li>
                    {{ form_widget(child) }}
                    {{ form_label(child) }}
                </li>
            {% endfor %}
            </ul>
        {% else %}
            {# just let the choice widget render the select tag #}
            {{ block('choice_widget') }}
        {% endif %}
    {% endspaceless %}
    {% endblock %}

.. note::

    Make sure the correct widget prefix is used. In this example the name should
    be ``gender_widget``, according to the value returned by ``getName``.
    Further, the main config file should point to the custom form template
    so that it's used when rendering all forms.
    确保widget的前缀是正确的。在这个例子中，这个名称应该为``gender_widget``，这是根据``getName``
    方法的返回值决定的。并且，在主要配置文件中应该指向定制的表单模板的位置，这样在输出表单的
    时候就可以使用这个模板了：

    .. code-block:: yaml

        # app/config/config.yml

        twig:
            form:
                resources:
                    - 'AcmeDemoBundle:Form:fields.html.twig'

Using the Field Type
使用字段类型
--------------------

You can now use your custom field type immediately, simply by creating a
new instance of the type in one of your forms::
你现在可以马上使用你所定制的字段类型，只要在你的表单中创建一个该类型的实例：

    // src/Acme/DemoBundle/Form/Type/AuthorType.php
    namespace Acme\DemoBundle\Form\Type;

    use Symfony\Component\Form\AbstractType;
    use Symfony\Component\Form\FormBuilder;
    
    class AuthorType extends AbstractType
    {
        public function buildForm(FormBuilder $builder, array $options)
        {
            $builder->add('gender_code', new GenderType(), array(
                'empty_value' => 'Choose a gender',
            ));
        }
    }

But this only works because the ``GenderType()`` is very simple. What if
the gender codes were stored in configuration or in a database? The next
section explains how more complex field types solve this problem.
但是本例中``GenderType()``十分简单，如果这个性别的代码再复杂些，比如说存储在配置或数据库中呢？
下一节将解释如何使用复杂的字段类型。

Creating your Field Type as a Service
通过服务来创建你的字段类型
-------------------------------------

So far, this entry has assumed that you have a very simple custom field type.
But if you need access to configuration, a database connection, or some other
service, then you'll want to register your custom type as a service. For
example, suppose that we're storing the gender parameters in configuration:
现在，已经假设了你有一个非常简单的自定义字段类型。但如果你要想使用配置、数据库或服务（service）
中的数据来定义，那么你就必须将你的字段类型设置为服务（service）。比如，假设我们在配置中
设置性别参数：

.. configuration-block::

    .. code-block:: yaml
    
        # app/config/config.yml
        parameters:
            genders:
                m: Male
                f: Female

    .. code-block:: xml

        <!-- app/config/config.xml -->
        <parameters>
            <parameter key="genders" type="collection">
                <parameter key="m">Male</parameter>
                <parameter key="f">Female</parameter>
            </parameter>
        </parameters>

To use the parameter, we'll define our custom field type as a service, injecting
the ``genders`` parameter value as the first argument to its to-be-created
``__construct`` function:
要使用这个参数，我们需要将定制的字段类型定义为服务，将genders参数作为第一个参数传给即将创建的
``__construct``方法：

.. configuration-block::

    .. code-block:: yaml

        # src/Acme/DemoBundle/Resources/config/services.yml
        services:
            form.type.gender:
                class: Acme\DemoBundle\Form\Type\GenderType
                arguments:
                    - "%genders%"
                tags:
                    - { name: form.type, alias: gender }

    .. code-block:: xml

        <!-- src/Acme/DemoBundle/Resources/config/services.xml -->
        <service id="form.type.gender" class="Acme\DemoBundle\Form\Type\GenderType">
            <argument>%genders%</argument>
            <tag name="form.type" alias="gender" />
        </service>

.. tip::

    Make sure the services file is being imported. See :ref:`service-container-imports-directive`
    for details.
    要确保这个服务文件被导入了。参见:ref:`service-container-imports-directive`。

Be sure that the ``alias`` attribute of the tag corresponds with the value
returned by the ``getName`` method defined earlier. We'll see the importance
of this in a moment when we use the custom field type. But first, add a ``__construct``
argument to ``GenderType``, which receives the gender configuration::
要确保这个alias属性的值是相对应于getName方法返回值的。当我们待会使用定制字段类型的时候就会了解它的重要性了。
但首先，还是要给``GenderType``添加一个``__construct``方法，它能够接收gender的配置信息::

    # src/Acme/DemoBundle/Form/Type/GenderType.php
    namespace Acme\DemoBundle\Form\Type;
    // ...

    class GenderType extends AbstractType
    {
        private $genderChoices;
        
        public function __construct(array $genderChoices)
        {
            $this->genderChoices = $genderChoices;
        }
    
        public function getDefaultOptions()
        {
            return array(
                'choices' => $this->genderChoices,
            );
        }
        
        // ...
    }

Great! The ``GenderType`` is now fueled by the configuration parameters and
registered as a service. And because we used the ``form.type`` alias in its
configuration, using the field is now much easier::
现在通过配置参数，``GenderType``已经被激活并且注册为服务了。并且由于我们在配置中使用了form.type  
 alias，现在使用这个字段要更容易了::

    // src/Acme/DemoBundle/Form/Type/AuthorType.php
    namespace Acme\DemoBundle\Form\Type;
    // ...

    class AuthorType extends AbstractType
    {
        public function buildForm(FormBuilder $builder, array $options)
        {
            $builder->add('gender_code', 'gender', array(
                'empty_value' => 'Choose a gender',
            ));
        }
    }

Notice that instead of instantiating a new instance, we can just refer to
it by the alias used in our service configuration, ``gender``. Have fun!
注意我们可以通过服务配置中的alias来访问原先初始化的类的实例，gender。

.. _`ChoiceType`: https://github.com/symfony/symfony/blob/master/src/Symfony/Component/Form/Extension/Core/Type/ChoiceType.php
.. _`FieldType`: https://github.com/symfony/symfony/blob/master/src/Symfony/Component/Form/Extension/Core/Type/FieldType.php