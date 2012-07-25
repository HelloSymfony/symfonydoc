.. index::
   single: Validation

Validation
验证
==========

Validation is a very common task in web applications. Data entered in forms
needs to be validated. Data also needs to be validated before it is written
into a database or passed to a web service.
在web应用中，验证是一个常用任务。输入表单的数据都需要验证。在将数据载入数据库之前或传递
给某个web服务时也要被验证。

Symfony2 ships with a `Validator`_ component that makes this task easy and transparent.
This component is based on the `JSR303 Bean Validation specification`_. What?
A Java specification in PHP? You heard right, but it's not as bad as it sounds.
Let's look at how it can be used in PHP.
symfony2内置有`Validator`_ component，它可以使这项工作变得简单。这个component是基于
`JSR303 Bean Validation specification`_的。什么？一个php中的java应用？你没有听错。以下将
讲述如何在php中使用它。

.. index:
   single: Validation; The basics

The Basics of Validation
验证基础
------------------------

The best way to understand validation is to see it in action. To start, suppose
you've created a plain-old-PHP object that you need to use somewhere in
your application:
假设你创建了一个普通php对象：

.. code-block:: php

    // src/Acme/BlogBundle/Entity/Author.php
    namespace Acme\BlogBundle\Entity;

    class Author
    {
        public $name;
    }

So far, this is just an ordinary class that serves some purpose inside your
application. The goal of validation is to tell you whether or not the data
of an object is valid. For this to work, you'll configure a list of rules
(called :ref:`constraints<validation-constraints>`) that the object must
follow in order to be valid. These rules can be specified via a number of
different formats (YAML, XML, annotations, or PHP).
目前为止这只是一个普通的类，使用它可以在你的应用中达到某些目的。验证的目的是要告诉你
对象中的数据是否可用。要使它能工作，你需要配置一系列该对象必须遵的规则（或称:ref:`constraints<validation-constraints>`）。
这个规则可以通过不同格式来配置（YAML,XML,annotation，或php）。

For example, to guarantee that the ``$name`` property is not empty, add the
following:
比如，要确保$name属性不是空的，将以下代码添加到文件中：

.. configuration-block::

    .. code-block:: yaml

        # src/Acme/BlogBundle/Resources/config/validation.yml
        Acme\BlogBundle\Entity\Author:
            properties:
                name:
                    - NotBlank: ~

    .. code-block:: php-annotations

        // src/Acme/BlogBundle/Entity/Author.php
        use Symfony\Component\Validator\Constraints as Assert;

        class Author
        {
            /**
             * @Assert\NotBlank()
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
                    <constraint name="NotBlank" />
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
                $metadata->addPropertyConstraint('name', new NotBlank());
            }
        }

.. tip::

    Protected and private properties can also be validated, as well as "getter"
    methods (see `validator-constraint-targets`).

.. index::
   single: Validation; Using the validator

Using the ``validator`` Service
使用validator服务
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Next, to actually validate an ``Author`` object, use the ``validate`` method
on the ``validator`` service (class :class:`Symfony\\Component\\Validator\\Validator`).
The job of the ``validator`` is easy: to read the constraints (i.e. rules)
of a class and verify whether or not the data on the object satisfies those
constraints. If validation fails, an array of errors is returned. Take this
simple example from inside a controller:
然后，要验证一个Author对象，可以在validator服务上使用validate方法（:class:`Symfony\\Component\\Validator\\Validator`）。
validator的工作很简单：读取类的规则（rule）并确定这个对象的数据是否符合这些规则。
如果验证失败，一个error数组会返回。以下是一个例子：

.. code-block:: php

    use Symfony\Component\HttpFoundation\Response;
    use Acme\BlogBundle\Entity\Author;
    // ...

    public function indexAction()
    {
        $author = new Author();
        // ... do something to the $author object

        $validator = $this->get('validator');
        $errors = $validator->validate($author);

        if (count($errors) > 0) {
            return new Response(print_r($errors, true));
        } else {
            return new Response('The author is valid! Yes!');
        }
    }

If the ``$name`` property is empty, you will see the following error
message:
如果$name属性是空的，你将看到以下的错误信息：

.. code-block:: text

    Acme\BlogBundle\Author.name:
        This value should not be blank

If you insert a value into the ``name`` property, the happy success message
will appear.
如果不是空的，成功信息会显示。

.. tip::

    Most of the time, you won't interact directly with the ``validator``
    service or need to worry about printing out the errors. Most of the time,
    you'll use validation indirectly when handling submitted form data. For
    more information, see the :ref:`book-validation-forms`.
    通常你不必使用validator服务或者担心错误输出。一般的，当处理表单数据时，你会间接使用
    验证。更多信息请参见:ref:`book-validation-forms`。

You could also pass the collection of errors into a template.
你还可以将错误信息传递到模板中。

.. code-block:: php

    if (count($errors) > 0) {
        return $this->render('AcmeBlogBundle:Author:validate.html.twig', array(
            'errors' => $errors,
        ));
    } else {
        // ...
    }

Inside the template, you can output the list of errors exactly as needed:
在模板中，你可以像这样输出一系列错误：

.. configuration-block::

    .. code-block:: html+jinja

        {# src/Acme/BlogBundle/Resources/views/Author/validate.html.twig #}

        <h3>The author has the following errors</h3>
        <ul>
        {% for error in errors %}
            <li>{{ error.message }}</li>
        {% endfor %}
        </ul>

    .. code-block:: html+php

        <!-- src/Acme/BlogBundle/Resources/views/Author/validate.html.php -->

        <h3>The author has the following errors</h3>
        <ul>
        <?php foreach ($errors as $error): ?>
            <li><?php echo $error->getMessage() ?></li>
        <?php endforeach; ?>
        </ul>

.. note::

    Each validation error (called a "constraint violation"), is represented by
    a :class:`Symfony\\Component\\Validator\\ConstraintViolation` object.

.. index::
   single: Validation; Validation with forms

.. _book-validation-forms:

Validation and Forms
验证和表单
~~~~~~~~~~~~~~~~~~~~

The ``validator`` service can be used at any time to validate any object.
In reality, however, you'll usually work with the ``validator`` indirectly
when working with forms. Symfony's form library uses the ``validator`` service
internally to validate the underlying object after values have been submitted
and bound. The constraint violations on the object are converted into ``FieldError``
objects that can easily be displayed with your form. The typical form submission
workflow looks like the following from inside a controller::
validator服务可以用于在任何时候验证任何对象。但实际上你往往会在表单中间接使用validator。
在表单数据提交并绑定之后，symfony的表单库会在内部使用validator服务来验证实体类中的数据。
违反规则的错误信息会被转化为``FieldError``对象，它可以方便地和你的表单一起输出。
通常情况下表单提交的流程会是这样的：

    use Acme\BlogBundle\Entity\Author;
    use Acme\BlogBundle\Form\AuthorType;
    use Symfony\Component\HttpFoundation\Request;
    // ...

    public function updateAction(Request $request)
    {
        $author = new Acme\BlogBundle\Entity\Author();
        $form = $this->createForm(new AuthorType(), $author);

        if ($request->getMethod() == 'POST') {
            $form->bindRequest($request);

            if ($form->isValid()) {
                // the validation passed, do something with the $author object

                return $this->redirect($this->generateUrl('...'));
            }
        }

        return $this->render('BlogBundle:Author:form.html.twig', array(
            'form' => $form->createView(),
        ));
    }

.. note::

    This example uses an ``AuthorType`` form class, which is not shown here.
    这个例子使用了一个``AuthorType``类，这里没有指明。

For more information, see the :doc:`Forms</book/forms>` chapter.
请参阅:doc:`Forms</book/forms>`。

.. index::
   pair: Validation; Configuration

.. _book-validation-configuration:

Configuration
配置
-------------

The Symfony2 validator is enabled by default, but you must explicitly enable
annotations if you're using the annotation method to specify your constraints:
symfony2的验证默认情况下是激活的，但是如果你要使用annotation方法的话就必须显性地激活它：

.. configuration-block::

    .. code-block:: yaml

        # app/config/config.yml
        framework:
            validation: { enable_annotations: true }

    .. code-block:: xml

        <!-- app/config/config.xml -->
        <framework:config>
            <framework:validation enable_annotations="true" />
        </framework:config>

    .. code-block:: php

        // app/config/config.php
        $container->loadFromExtension('framework', array('validation' => array(
            'enable_annotations' => true,
        )));

.. index::
   single: Validation; Constraints

.. _validation-constraints:

Constraints
规则
-----------

The ``validator`` is designed to validate objects against *constraints* (i.e.
rules). In order to validate an object, simply map one or more constraints
to its class and then pass it to the ``validator`` service.
validator根据规则（rule）来验证对象。要验证一个对象，只要将一个或多个规则映射到它的类
并将它传递到validator服务中。

Behind the scenes, a constraint is simply a PHP object that makes an assertive
statement. In real life, a constraint could be: "The cake must not be burned".
In Symfony2, constraints are similar: they are assertions that a condition
is true. Given a value, a constraint will tell you whether or not that value
adheres to the rules of the constraint.
背后的原理是，一个规则仅仅是一个创建判断语句的php对象。在symfony2中，规则也一样：
它们都是对一个条件是否为“真”的判断。对于一个给定的值，规则会告诉你这个值是否
与规则规定的相匹配。

Supported Constraints
支持的规则
~~~~~~~~~~~~~~~~~~~~~

Symfony2 packages a large number of the most commonly-needed constraints:
symfony2内置大量常用的规则：

.. include:: /reference/constraints/map.rst.inc

You can also create your own custom constraints. This topic is covered in
the ":doc:`/cookbook/validation/custom_constraint`" article of the cookbook.
你还可以创建你的自定义规则。参见":doc:`/cookbook/validation/custom_constraint`"。

.. index::
   single: Validation; Constraints configuration

.. _book-validation-constraint-configuration:

Constraint Configuration
规则配置
~~~~~~~~~~~~~~~~~~~~~~~~

Some constraints, like :doc:`NotBlank</reference/constraints/NotBlank>`,
are simple whereas others, like the :doc:`Choice</reference/constraints/Choice>`
constraint, have several configuration options available. Suppose that the
``Author`` class has another property, ``gender`` that can be set to either
"male" or "female":
有些规则，像:doc:`NotBlank</reference/constraints/NotBlank>`很简单，但有些很复杂，比如
:doc:`Choice</reference/constraints/Choice>`规则，它有好几个配置选项。假设Author类有另外一个属性，
gender，它可以被设置为male或famale：

.. configuration-block::

    .. code-block:: yaml

        # src/Acme/BlogBundle/Resources/config/validation.yml
        Acme\BlogBundle\Entity\Author:
            properties:
                gender:
                    - Choice: { choices: [male, female], message: Choose a valid gender. }

    .. code-block:: php-annotations

        // src/Acme/BlogBundle/Entity/Author.php
        use Symfony\Component\Validator\Constraints as Assert;

        class Author
        {
            /**
             * @Assert\Choice(
             *     choices = { "male", "female" },
             *     message = "Choose a valid gender."
             * )
             */
            public $gender;
        }

    .. code-block:: xml

        <!-- src/Acme/BlogBundle/Resources/config/validation.xml -->
        <?xml version="1.0" encoding="UTF-8" ?>
        <constraint-mapping xmlns="http://symfony.com/schema/dic/constraint-mapping"
            xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
            xsi:schemaLocation="http://symfony.com/schema/dic/constraint-mapping http://symfony.com/schema/dic/constraint-mapping/constraint-mapping-1.0.xsd">

            <class name="Acme\BlogBundle\Entity\Author">
                <property name="gender">
                    <constraint name="Choice">
                        <option name="choices">
                            <value>male</value>
                            <value>female</value>
                        </option>
                        <option name="message">Choose a valid gender.</option>
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
            public $gender;

            public static function loadValidatorMetadata(ClassMetadata $metadata)
            {
                $metadata->addPropertyConstraint('gender', new Choice(array(
                    'choices' => array('male', 'female'),
                    'message' => 'Choose a valid gender.',
                )));
            }
        }

.. _validation-default-option:

The options of a constraint can always be passed in as an array. Some constraints,
however, also allow you to pass the value of one, "*default*", option in place
of the array. In the case of the ``Choice`` constraint, the ``choices``
options can be specified in this way.
一个规则的选项总是可以作为数组传递进来。有些规则还允许你在这个数组的位置上传递一个默认的选项的值，default。
比如``Choice``规则中，choices选项可以用这种方法来指定：

.. configuration-block::

    .. code-block:: yaml

        # src/Acme/BlogBundle/Resources/config/validation.yml
        Acme\BlogBundle\Entity\Author:
            properties:
                gender:
                    - Choice: [male, female]

    .. code-block:: php-annotations

        // src/Acme/BlogBundle/Entity/Author.php
        use Symfony\Component\Validator\Constraints as Assert;

        class Author
        {
            /**
             * @Assert\Choice({"male", "female"})
             */
            protected $gender;
        }

    .. code-block:: xml

        <!-- src/Acme/BlogBundle/Resources/config/validation.xml -->
        <?xml version="1.0" encoding="UTF-8" ?>
        <constraint-mapping xmlns="http://symfony.com/schema/dic/constraint-mapping"
            xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
            xsi:schemaLocation="http://symfony.com/schema/dic/constraint-mapping http://symfony.com/schema/dic/constraint-mapping/constraint-mapping-1.0.xsd">

            <class name="Acme\BlogBundle\Entity\Author">
                <property name="gender">
                    <constraint name="Choice">
                        <value>male</value>
                        <value>female</value>
                    </constraint>
                </property>
            </class>
        </constraint-mapping>

    .. code-block:: php

        // src/Acme/BlogBundle/Entity/Author.php
        use Symfony\Component\Validator\Mapping\ClassMetadata;
        use Symfony\Component\Validator\Constraints\Choice;

        class Author
        {
            protected $gender;

            public static function loadValidatorMetadata(ClassMetadata $metadata)
            {
                $metadata->addPropertyConstraint('gender', new Choice(array('male', 'female')));
            }
        }

This is purely meant to make the configuration of the most common option of
a constraint shorter and quicker.
这只是让常用的规则的选项配置变得更简洁。

If you're ever unsure of how to specify an option, either check the API documentation
for the constraint or play it safe by always passing in an array of options
(the first method shown above).
如果你不确定如何配置一个选项，请查看API文档或保险起见，使用第一个方法，以数组来传递选项。

Translation Constraint Messages
translation规则信息
-------------------------------

For information on translating the constraint messages, see
:ref:`book-translation-constraint-messages`.
要了解关于翻译规则信息，请参阅:ref:`book-translation-constraint-messages`。

.. index::
   single: Validation; Constraint targets

.. _validator-constraint-targets:

Constraint Targets
规则的对象
------------------

Constraints can be applied to a class property (e.g. ``name``) or a public
getter method (e.g. ``getFullName``). The first is the most common and easy
to use, but the second allows you to specify more complex validation rules.
规则可以被应用于一个类的属性（name）或一个public getter方法（getName）。第一个是
最常用的，但是第二个允许你指定更复杂的验证规则。

.. index::
   single: Validation; Property constraints

.. _validation-property-target:

Properties
属性
~~~~~~~~~~

Validating class properties is the most basic validation technique. Symfony2
allows you to validate private, protected or public properties. The next
listing shows you how to configure the ``$firstName`` property of an ``Author``
class to have at least 3 characters.
symfony2允许你验证private，protected或public属性。下面的例子展示了如何配置Author类中的$firstName
属性，从而允许它有最少3个字母：

.. configuration-block::

    .. code-block:: yaml

        # src/Acme/BlogBundle/Resources/config/validation.yml
        Acme\BlogBundle\Entity\Author:
            properties:
                firstName:
                    - NotBlank: ~
                    - MinLength: 3

    .. code-block:: php-annotations

        // Acme/BlogBundle/Entity/Author.php
        use Symfony\Component\Validator\Constraints as Assert;

        class Author
        {
            /**
             * @Assert\NotBlank()
             * @Assert\MinLength(3)
             */
            private $firstName;
        }

    .. code-block:: xml

        <!-- src/Acme/BlogBundle/Resources/config/validation.xml -->
        <class name="Acme\BlogBundle\Entity\Author">
            <property name="firstName">
                <constraint name="NotBlank" />
                <constraint name="MinLength">3</constraint>
            </property>
        </class>

    .. code-block:: php

        // src/Acme/BlogBundle/Entity/Author.php
        use Symfony\Component\Validator\Mapping\ClassMetadata;
        use Symfony\Component\Validator\Constraints\NotBlank;
        use Symfony\Component\Validator\Constraints\MinLength;

        class Author
        {
            private $firstName;

            public static function loadValidatorMetadata(ClassMetadata $metadata)
            {
                $metadata->addPropertyConstraint('firstName', new NotBlank());
                $metadata->addPropertyConstraint('firstName', new MinLength(3));
            }
        }

.. index::
   single: Validation; Getter constraints

Getters
~~~~~~~

Constraints can also be applied to the return value of a method. Symfony2
allows you to add a constraint to any public method whose name starts with
"get" or "is". In this guide, both of these types of methods are referred
to as "getters".
规则还可以用于一个方法的返回值。symfony2允许你给任何以get或is开头的public方法添加规则。
这两种都被称作getter。

The benefit of this technique is that it allows you to validate your object
dynamically. For example, suppose you want to make sure that a password field
doesn't match the first name of the user (for security reasons). You can
do this by creating an ``isPasswordLegal`` method, and then asserting that
this method must return ``true``:
这个功能的好处就是它允许你动态验证你的对象。比如，假设你需要确认一个password字段不匹配这个用户的
first name（安全原因），你可以创建一个``isPasswordLegal``方法，然后设置这个方法必须返回true：

.. configuration-block::

    .. code-block:: yaml

        # src/Acme/BlogBundle/Resources/config/validation.yml
        Acme\BlogBundle\Entity\Author:
            getters:
                passwordLegal:
                    - "True": { message: "The password cannot match your first name" }

    .. code-block:: php-annotations

        // src/Acme/BlogBundle/Entity/Author.php
        use Symfony\Component\Validator\Constraints as Assert;

        class Author
        {
            /**
             * @Assert\True(message = "The password cannot match your first name")
             */
            public function isPasswordLegal()
            {
                // return true or false
            }
        }

    .. code-block:: xml

        <!-- src/Acme/BlogBundle/Resources/config/validation.xml -->
        <class name="Acme\BlogBundle\Entity\Author">
            <getter property="passwordLegal">
                <constraint name="True">
                    <option name="message">The password cannot match your first name</option>
                </constraint>
            </getter>
        </class>

    .. code-block:: php

        // src/Acme/BlogBundle/Entity/Author.php
        use Symfony\Component\Validator\Mapping\ClassMetadata;
        use Symfony\Component\Validator\Constraints\True;

        class Author
        {
            public static function loadValidatorMetadata(ClassMetadata $metadata)
            {
                $metadata->addGetterConstraint('passwordLegal', new True(array(
                    'message' => 'The password cannot match your first name',
                )));
            }
        }

Now, create the ``isPasswordLegal()`` method, and include the logic you need::

    public function isPasswordLegal()
    {
        return ($this->firstName != $this->password);
    }

.. note::

    The keen-eyed among you will have noticed that the prefix of the getter
    ("get" or "is") is omitted in the mapping. This allows you to move the
    constraint to a property with the same name later (or vice versa) without
    changing your validation logic.
    你可能会发现getter（get或is）的前缀在映射中被忽略了。这允许你将这个规则移至
    有同样名称的属性，而不必修改你的验证逻辑。

.. _validation-class-target:

Classes
~~~~~~~

Some constraints apply to the entire class being validated. For example,
the :doc:`Callback</reference/constraints/Callback>` constraint is a generic
constraint that's applied to the class itself. When that class is validated,
methods specified by that constraint are simply executed so that each can
provide more custom validation.
有些验证应用于整个类。比如，:doc:`Callback</reference/constraints/Callback>`规则就是一个
整体的规则，并直接应用于这个类。当那个类被验证后，这个规则的方法会被执行。

.. _book-validation-validation-groups:

Validation Groups
验证集合
-----------------

So far, you've been able to add constraints to a class and ask whether or
not that class passes all of the defined constraints. In some cases, however,
you'll need to validate an object against only *some* of the constraints
on that class. To do this, you can organize each constraint into one or more
"validation groups", and then apply validation against just one group of
constraints.
现在，你已经可以向一个类添加规则并确定那个类的数据是否符合规则。但是在某些情况下，你
只需要根据某些规则来验证一个对象。要达到这个目的，你可以将规则组织在一个或多个“验证集合”中，
然后根据一个集合来应用验证。

For example, suppose you have a ``User`` class, which is used both when a
user registers and when a user updates his/her contact information later:
比如，假设你有一个User类，在用户注册或用户更新信息时都要用到它：

.. configuration-block::

    .. code-block:: yaml

        # src/Acme/BlogBundle/Resources/config/validation.yml
        Acme\BlogBundle\Entity\User:
            properties:
                email:
                    - Email: { groups: [registration] }
                password:
                    - NotBlank: { groups: [registration] }
                    - MinLength: { limit: 7, groups: [registration] }
                city:
                    - MinLength: 2

    .. code-block:: php-annotations

        // src/Acme/BlogBundle/Entity/User.php
        namespace Acme\BlogBundle\Entity;

        use Symfony\Component\Security\Core\User\UserInterface;
        use Symfony\Component\Validator\Constraints as Assert;

        class User implements UserInterface
        {
            /**
            * @Assert\Email(groups={"registration"})
            */
            private $email;

            /**
            * @Assert\NotBlank(groups={"registration"})
            * @Assert\MinLength(limit=7, groups={"registration"})
            */
            private $password;

            /**
            * @Assert\MinLength(2)
            */
            private $city;
        }

    .. code-block:: xml

        <!-- src/Acme/BlogBundle/Resources/config/validation.xml -->
        <class name="Acme\BlogBundle\Entity\User">
            <property name="email">
                <constraint name="Email">
                    <option name="groups">
                        <value>registration</value>
                    </option>
                </constraint>
            </property>
            <property name="password">
                <constraint name="NotBlank">
                    <option name="groups">
                        <value>registration</value>
                    </option>
                </constraint>
                <constraint name="MinLength">
                    <option name="limit">7</option>
                    <option name="groups">
                        <value>registration</value>
                    </option>
                </constraint>
            </property>
            <property name="city">
                <constraint name="MinLength">7</constraint>
            </property>
        </class>

    .. code-block:: php

        // src/Acme/BlogBundle/Entity/User.php
        namespace Acme\BlogBundle\Entity;

        use Symfony\Component\Validator\Mapping\ClassMetadata;
        use Symfony\Component\Validator\Constraints\Email;
        use Symfony\Component\Validator\Constraints\NotBlank;
        use Symfony\Component\Validator\Constraints\MinLength;

        class User
        {
            public static function loadValidatorMetadata(ClassMetadata $metadata)
            {
                $metadata->addPropertyConstraint('email', new Email(array(
                    'groups' => array('registration')
                )));

                $metadata->addPropertyConstraint('password', new NotBlank(array(
                    'groups' => array('registration')
                )));
                $metadata->addPropertyConstraint('password', new MinLength(array(
                    'limit'  => 7,
                    'groups' => array('registration')
                )));

                $metadata->addPropertyConstraint('city', new MinLength(3));
            }
        }

With this configuration, there are two validation groups:
根据这个配置，这里有两个验证集合：

* ``Default`` - contains the constraints not assigned to any other group;
* ``Default``——包含没有被指派给其他集合的规则；

* ``registration`` - contains the constraints on the ``email`` and ``password``
  fields only.
* ``registration``——包含应用于``email``和``password``的规则。

To tell the validator to use a specific group, pass one or more group names
as the second argument to the ``validate()`` method::
要告诉validator使用一个特定的集合，你可以将一个或多个集合的名称作为第二个参数传递给validate()方法::

    $errors = $validator->validate($author, array('registration'));

Of course, you'll usually work with validation indirectly through the form
library. For information on how to use validation groups inside forms, see
:ref:`book-forms-validation-groups`.
你往往需要通过form库间接应用验证。更多关于如何在表单中使用验证集合的信息请参阅:ref:`book-forms-validation-groups`。

.. index::
   single: Validation; Validating raw values

.. _book-validation-raw-values:

Validating Values and Arrays
验证值和数组
----------------------------

So far, you've seen how you can validate entire objects. But sometimes, you
just want to validate a simple value - like to verify that a string is a valid
email address. This is actually pretty easy to do. From inside a controller,
it looks like this::
现在，你已经知道了如何验证整个对象。但是有些时候你只想验证一个简单的值——比如验证一个
字符串是否是一个可用的email地址。你可以在控制器中：

    // add this to the top of your class
    use Symfony\Component\Validator\Constraints\Email;
    
    public function addEmailAction($email)
    {
        $emailConstraint = new Email();
        // all constraint "options" can be set this way
        $emailConstraint->message = 'Invalid email address';

        // use the validator to validate the value
        $errorList = $this->get('validator')->validateValue($email, $emailConstraint);

        if (count($errorList) == 0) {
            // this IS a valid email address, do something
        } else {
            // this is *not* a valid email address
            $errorMessage = $errorList[0]->getMessage()
            
            // do something with the error
        }
        
        // ...
    }

By calling ``validateValue`` on the validator, you can pass in a raw value and
the constraint object that you want to validate that value against. A full
list of the available constraints - as well as the full class name for each
constraint - is available in the :doc:`constraints reference</reference/constraints>`
section .
通过执行``validateValue``，你可以传递一个原始值和一个规则对象。在:doc:`constraints reference</reference/constraints>`
中你可以查看全部可用的规则以及各个规则的类名。

The ``validateValue`` method returns a :class:`Symfony\\Component\\Validator\\ConstraintViolationList`
object, which acts just like an array of errors. Each error in the collection
is a :class:`Symfony\\Component\\Validator\\ConstraintViolation` object,
which holds the error message on its `getMessage` method.
``validateValue``方法返回一个:class:`Symfony\\Component\\Validator\\ConstraintViolationList`对象，
这个对象包含了一系列错误组成的数组。每个错误都是一个:class:`Symfony\\Component\\Validator\\ConstraintViolation`
对象，这些对象在它们的getMessage方法中包含了错误的信息。

Final Thoughts
总结
--------------

The Symfony2 ``validator`` is a powerful tool that can be leveraged to
guarantee that the data of any object is "valid". The power behind validation
lies in "constraints", which are rules that you can apply to properties or
getter methods of your object. And while you'll most commonly use the validation
framework indirectly when using forms, remember that it can be used anywhere
to validate any object.
symfony2的validator是一个强大的工具，它可以被用来确认任何对象的数据是可用的。
验证的强大之处主要在于“规则”，它是你可以应用于属性或getter方法的rule。虽然你大部分情况下在验证表单
时使用验证系统，但要记住它可以被用于任何地方来验证任何类。


Learn more from the Cookbook
----------------------------

* :doc:`/cookbook/validation/custom_constraint`

.. _Validator: https://github.com/symfony/Validator
.. _JSR303 Bean Validation specification: http://jcp.org/en/jsr/detail?id=303
