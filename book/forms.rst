.. index::
   single: Forms

Forms
表单
=====

Dealing with HTML forms is one of the most common - and challenging - tasks for
a web developer. Symfony2 integrates a Form component that makes dealing with
forms easy. In this chapter, you'll build a complex form from the ground-up,
learning the most important features of the form library along the way.
对于一个web开发者来说，最常用又最难办的事就是跟表单打交道。symfony2内置了一个Form component，它使得
跟表单打交道变得容易了。在本章中，你会学习如何创建表单，以及Form component的各种特征。

.. note::

   The Symfony form component is a standalone library that can be used outside
   of Symfony2 projects. For more information, see the `Symfony2 Form Component`_
   on Github.
   Form component是一个独立的库，它可以在symfony2应用的外部使用。详情请见`Symfony2 Form Component`_。

.. index::
   single: Forms; Create a simple form

Creating a Simple Form
创建一个简单表单
----------------------

Suppose you're building a simple todo list application that will need to
display "tasks". Because your users will need to edit and create tasks, you're
going to need to build a form. But before you begin, first focus on the generic
``Task`` class that represents and stores the data for a single task:
假设你要创建一个简单的todo list应用，这个应用会显示你要完成的任务（task）。因为你的用户
需要修改和创建任务，所以你需要一个表单。但在开始之前，先看看这个为一个任务存储和显示数据的
task类吧：

.. code-block:: php

    // src/Acme/TaskBundle/Entity/Task.php
    namespace Acme\TaskBundle\Entity;

    class Task
    {
        protected $task;

        protected $dueDate;

        public function getTask()
        {
            return $this->task;
        }
        public function setTask($task)
        {
            $this->task = $task;
        }

        public function getDueDate()
        {
            return $this->dueDate;
        }
        public function setDueDate(\DateTime $dueDate = null)
        {
            $this->dueDate = $dueDate;
        }
    }

.. note::

   If you're coding along with this example, create the ``AcmeTaskBundle``
   first by running the following command (and accepting all of the default
   options):
   如果你按照这个例子做，你必须先用命令行创建一个``AcmeTaskBundle``（而且要接受所有默认设置）：

   .. code-block:: bash

        php app/console generate:bundle --namespace=Acme/TaskBundle

This class is a "plain-old-PHP-object" because, so far, it has nothing
to do with Symfony or any other library. It's quite simply a normal PHP object
that directly solves a problem inside *your* application (i.e. the need to
represent a task in your application). Of course, by the end of this chapter,
you'll be able to submit data to a ``Task`` instance (via an HTML form), validate
its data, and persist it to the database.
这个类目前只是一个简单的php类，因为暂时它还不能和symfony以及数据库交互。它还是一个简单的类，
可以直接解决你应用中的问题（比如你想把任务在你的应用中展示的需要）。当然，在本章末尾，你将学到
如何使用HTML表单将数据提交到一个task实例，验证它的数据，并将它插入数据库。

.. index::
   single: Forms; Create a form in a controller

Building the Form
创建表单
~~~~~~~~~~~~~~~~~

Now that you've created a ``Task`` class, the next step is to create and
render the actual HTML form. In Symfony2, this is done by building a form
object and then rendering it in a template. For now, this can all be done
from inside a controller::
现在你已经创建了一个task类，下一步就是要创建并输出一个HTML表单。在symfony2中，你可以
创建一个表单的实例并将它在模板中输出。现在，在控制器中：

    // src/Acme/TaskBundle/Controller/DefaultController.php
    namespace Acme\TaskBundle\Controller;

    use Symfony\Bundle\FrameworkBundle\Controller\Controller;
    use Acme\TaskBundle\Entity\Task;
    use Symfony\Component\HttpFoundation\Request;

    class DefaultController extends Controller
    {
        public function newAction(Request $request)
        {
            // create a task and give it some dummy data for this example
            $task = new Task();
            $task->setTask('Write a blog post');
            $task->setDueDate(new \DateTime('tomorrow'));

            $form = $this->createFormBuilder($task)
                ->add('task', 'text')
                ->add('dueDate', 'date')
                ->getForm();

            return $this->render('AcmeTaskBundle:Default:new.html.twig', array(
                'form' => $form->createView(),
            ));
        }
    }

.. tip::

   This example shows you how to build your form directly in the controller.
   Later, in the ":ref:`book-form-creating-form-classes`" section, you'll learn
   how to build your form in a standalone class, which is recommended as
   your form becomes reusable.
   这个例子展示了如何直接在控制器中创建表单。在以下":ref:`book-form-creating-form-classes`"章节中，
   你将学习如何在一个单独的类中创建表单，这样你的表单就能够被重复使用了。

Creating a form requires relatively little code because Symfony2 form objects
are built with a "form builder". The form builder's purpose is to allow you
to write simple form "recipes", and have it do all the heavy-lifting of actually
building the form.
创建一个表单只要很少的代码，因为symfony2中表单对象都是通过"form builder"创建的。"form builder"的
目的就是允许你写简单的创建表单的内容，幕后工作都由它来完成。

In this example, you've added two fields to your form - ``task`` and ``dueDate`` -
corresponding to the ``task`` and ``dueDate`` properties of the ``Task`` class.
You've also assigned each a "type" (e.g. ``text``, ``date``), which, among
other things, determines which HTML form tag(s) is rendered for that field.
在这个例子中，你已经给你的表单加上了两个字段——``task``和``dueDate``，你还规定了它们的类型(e.g. ``text``, ``date``)，
这些类型规定了哪种HTML标签要在输出表单时使用。

Symfony2 comes with many built-in types that will be discussed shortly
(see :ref:`book-forms-type-reference`).
symfony2有许多内置的类型，下面将论述（see :ref:`book-forms-type-reference`）。

.. index::
  single: Forms; Basic template rendering

Rendering the Form
输出表单
~~~~~~~~~~~~~~~~~~

Now that the form has been created, the next step is to render it. This is
done by passing a special form "view" object to your template (notice the
``$form->createView()`` in the controller above) and using a set of form
helper functions:
既然表单已经被创建，下一步就是要输出它。你要将一个表单"view"对象传递到你的模板里（注意上面控制器中的
``$form->createView()``），并使用一系列表单helper方法：

.. configuration-block::

    .. code-block:: html+jinja

        {# src/Acme/TaskBundle/Resources/views/Default/new.html.twig #}

        <form action="{{ path('task_new') }}" method="post" {{ form_enctype(form) }}>
            {{ form_widget(form) }}

            <input type="submit" />
        </form>

    .. code-block:: html+php

        <!-- src/Acme/TaskBundle/Resources/views/Default/new.html.php -->

        <form action="<?php echo $view['router']->generate('task_new') ?>" method="post" <?php echo $view['form']->enctype($form) ?> >
            <?php echo $view['form']->widget($form) ?>

            <input type="submit" />
        </form>

.. image:: /images/book/form-simple.png
    :align: center

.. note::

    This example assumes that you've created a route called ``task_new``
    that points to the ``AcmeTaskBundle:Default:new`` controller that
    was created earlier.
    这个例子假设你已经创建了一个名为``task_new``的路径并指向``AcmeTaskBundle:Default:new``控制器。

That's it! By printing ``form_widget(form)``, each field in the form is
rendered, along with a label and error message (if there is one). As easy
as this is, it's not very flexible (yet). Usually, you'll want to render each
form field individually so you can control how the form looks. You'll learn how
to do that in the ":ref:`form-rendering-template`" section.
就是这样！通过输入``form_widget(form)``，表单中的每个字段都被输出了，还有label和error信息（如果有的话）。
通常情况下，你需要将表单的每个部分分别输出，这样你才能定义表单样式。你可以学习":ref:`form-rendering-template`"这节。

Before moving on, notice how the rendered ``task`` input field has the value
of the ``task`` property from the ``$task`` object (i.e. "Write a blog post").
This is the first job of a form: to take data from an object and translate
it into a format that's suitable for being rendered in an HTML form.
在动手之前，注意被输出的task input字段都有些从$task对象中来的什么信息。这就是表单的第一要务：
从一个对象中取数据并且将这些数据转换成HTML表单可以读取的格式。

.. tip::

   The form system is smart enough to access the value of the protected
   ``task`` property via the ``getTask()`` and ``setTask()`` methods on the
   ``Task`` class. Unless a property is public, it *must* have a "getter" and
   "setter" method so that the form component can get and put data onto the
   property. For a Boolean property, you can use an "isser" method (e.g.
   ``isPublished()``) instead of a getter (e.g. ``getPublished()``).
   表单系统很灵活，它知道如何通过task实体类中的``getTask()``和``setTask()``方法来读取被保护（protected）的task的值。
   对于一个boolean值，你可以使用 "isser"方法，如``isPublished()``；而不是getter方法，如``getPublished()``。

.. index::
  single: Forms; Handling form submission

Handling Form Submissions
处理表单提交
~~~~~~~~~~~~~~~~~~~~~~~~~

The second job of a form is to translate user-submitted data back to the
properties of an object. To make this happen, the submitted data from the
user must be bound to the form. Add the following functionality to your
controller::
表单的第二件工作就是将用户提交的数据传送到对象的属性中。要达到这个目的用户提交的数据必须被绑定到表单中。
在你的控制器中加入以下代码::

    // ...

    public function newAction(Request $request)
    {
        // just setup a fresh $task object (remove the dummy data)
        $task = new Task();

        $form = $this->createFormBuilder($task)
            ->add('task', 'text')
            ->add('dueDate', 'date')
            ->getForm();

        if ($request->getMethod() == 'POST') {
            $form->bindRequest($request);

            if ($form->isValid()) {
                // perform some action, such as saving the task to the database

                return $this->redirect($this->generateUrl('task_success'));
            }
        }

        // ...
    }

Now, when submitting the form, the controller binds the submitted data to the
form, which translates that data back to the ``task`` and ``dueDate`` properties
of the ``$task`` object. This all happens via the ``bindRequest()`` method.
现在，当提交表单时，控制器将提交的数据与表单绑定，这样的话可以将数据传送回$task对象的
task属性和dueDate属性。通过``bindRequest()``方法可以实现这个目的。

.. note::

    As soon as ``bindRequest()`` is called, the submitted data is transferred
    to the underlying object immediately. This happens regardless of whether
    or not the underlying data is actually valid.
    一旦``bindRequest()``被执行，提交的数据就会被传送给对象。虽然该数据还没有被验证。

This controller follows a common pattern for handling forms, and has three
possible paths:
控制器根据以下三种方法来处理表单：

#. When initially loading the page in a browser, the request method is ``GET``
   and the form is simply created and rendered;
   当最先在浏览器加载页面时，请求方法是get，表单被简单地创建和输出；

#. When the user submits the form (i.e. the method is ``POST``) with invalid
   data (validation is covered in the next section), the form is bound and
   then rendered, this time displaying all validation errors;
   当这个用户输入不能通过验证（下一节会讲验证）的数据提交表单时（这个提交方法是POST），这个表单
   被绑定然后输出，这次会显示所有验证错误；

#. When the user submits the form with valid data, the form is bound and
   you have the opportunity to perform some actions using the ``$task``
   object (e.g. persisting it to the database) before redirecting the user
   to some other page (e.g. a "thank you" or "success" page).
   当用户输入可以通过验证的数据时，表单被绑定，并且你可以通过$task对象来做一些事情（比如
   将数据插入数据库），在你将用户重定向到另一个页面之前（如一个“thank you”或“success”页面）。

.. note::

   Redirecting a user after a successful form submission prevents the user
   from being able to hit "refresh" and re-post the data.
   在成功提交表单之后重定向这个用户可以避免这个用户刷新页面并多次提交数据。

.. index::
   single: Forms; Validation

Form Validation
表单认证
---------------

In the previous section, you learned how a form can be submitted with valid
or invalid data. In Symfony2, validation is applied to the underlying object
(e.g. ``Task``). In other words, the question isn't whether the "form" is
valid, but whether or not the ``$task`` object is valid after the form has
applied the submitted data to it. Calling ``$form->isValid()`` is a shortcut
that asks the ``$task`` object whether or not it has valid data.
在前一节中，你学习了在用户输入能通过验证或不能通过验证的数据后，一个表单能如何被提交。在symfony2中，
验证被应用于这个实体对象（如task），也就是说，问题不在于表单是否能在提交后通过验证，而在于这个$task对象是否能通过验证。
执行``$form->isValid()``命令是一个验证$task中的数据的便捷方法。

Validation is done by adding a set of rules (called constraints) to a class. To
see this in action, add validation constraints so that the ``task`` field cannot
be empty and the ``dueDate`` field cannot be empty and must be a valid \DateTime
object.
通过向对象添加一系列规则（或称约束条件）可以进行验证。向task字段添加规定它不能为空的验证约束条件，
向dueDate添加规定它不能为空并且必须是\DateTime对象的约束条件。

.. configuration-block::

    .. code-block:: yaml

        # Acme/TaskBundle/Resources/config/validation.yml
        Acme\TaskBundle\Entity\Task:
            properties:
                task:
                    - NotBlank: ~
                dueDate:
                    - NotBlank: ~
                    - Type: \DateTime

    .. code-block:: php-annotations

        // Acme/TaskBundle/Entity/Task.php
        use Symfony\Component\Validator\Constraints as Assert;

        class Task
        {
            /**
             * @Assert\NotBlank()
             */
            public $task;

            /**
             * @Assert\NotBlank()
             * @Assert\Type("\DateTime")
             */
            protected $dueDate;
        }

    .. code-block:: xml

        <!-- Acme/TaskBundle/Resources/config/validation.xml -->
        <class name="Acme\TaskBundle\Entity\Task">
            <property name="task">
                <constraint name="NotBlank" />
            </property>
            <property name="dueDate">
                <constraint name="NotBlank" />
                <constraint name="Type">
                    <value>\DateTime</value>
                </constraint>
            </property>
        </class>

    .. code-block:: php

        // Acme/TaskBundle/Entity/Task.php
        use Symfony\Component\Validator\Mapping\ClassMetadata;
        use Symfony\Component\Validator\Constraints\NotBlank;
        use Symfony\Component\Validator\Constraints\Type;

        class Task
        {
            // ...

            public static function loadValidatorMetadata(ClassMetadata $metadata)
            {
                $metadata->addPropertyConstraint('task', new NotBlank());

                $metadata->addPropertyConstraint('dueDate', new NotBlank());
                $metadata->addPropertyConstraint('dueDate', new Type('\DateTime'));
            }
        }

That's it! If you re-submit the form with invalid data, you'll see the
corresponding errors printed out with the form.
这就是了！如果你输入不能通过验证的数据并提交表单，你会看见响应的错误被同表单一起输出。

.. _book-forms-html5-validation-disable:

.. sidebar:: HTML5 Validation

   As of HTML5, many browsers can natively enforce certain validation constraints
   on the client side. The most common validation is activated by rendering
   a ``required`` attribute on fields that are required. For browsers that
   support HTML5, this will result in a native browser message being displayed
   if the user tries to submit the form with that field blank.
   在HTML5中，很多浏览器都会在客户端强制弹出验证限制条件。最常用的验证就是检查该字段是否为空（在那些有属性“required”的字段中）。
   对于那些支持HTML5的浏览器来说，当用户提交空字段时，它会自动弹出验证信息。
   
   Generated forms take full advantage of this new feature by adding sensible
   HTML attributes that trigger the validation. The client-side validation,
   however, can be disabled by adding the ``novalidate`` attribute to the
   ``form`` tag or ``formnovalidate`` to the submit tag. This is especially
   useful when you want to test your server-side validation constraints,
   but are being prevented by your browser from, for example, submitting
   blank fields.
   被集成的表单利用了这一特点，通过添加HTML属性来激活这种验证。但也可以通过在form标签里添加“novalidate”属性
   或在submit标签里添加“formnovalidate”属性来禁止这种验证。
   
Validation is a very powerful feature of Symfony2 and has its own
:doc:`dedicated chapter</book/validation>`.
验证是symfony中一个十分强大的功能，详情请见:doc:`dedicated chapter</book/validation>`。

.. index::
   single: Forms; Validation Groups

.. _book-forms-validation-groups:

Validation Groups
~~~~~~~~~~~~~~~~~

.. tip::

    If you're not using :ref:`validation groups <book-validation-validation-groups>`,
    then you can skip this section.

If your object takes advantage of :ref:`validation groups <book-validation-validation-groups>`,
you'll need to specify which validation group(s) your form should use::

    $form = $this->createFormBuilder($users, array(
        'validation_groups' => array('registration'),
    ))->add(...)
    ;

If you're creating :ref:`form classes<book-form-creating-form-classes>` (a
good practice), then you'll need to add the following to the ``getDefaultOptions()``
method::

    public function getDefaultOptions()
    {
        return array(
            'validation_groups' => array('registration')
        );
    }

In both of these cases, *only* the ``registration`` validation group will
be used to validate the underlying object.

Groups based on Submitted Data
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

.. versionadded:: 2.1
   The ability to specify a callback or Closure in ``validation_groups``
   is new to version 2.1

If you need some advanced logic to determine the validation groups (e.g.
based on submitted data), you can set the ``validation_groups`` option
to an array callback, or a ``Closure``::

    public function getDefaultOptions()
    {
        return array(
            'validation_groups' => array('Acme\\AcmeBundle\\Entity\\Client', 'determineValidationGroups'),
        );
    }

This will call the static method ``determineValidationGroups()`` on the
``Client`` class after the form is bound, but before validation is executed.
The Form object is passed as an argument to that method (see next example).
You can also define whole logic inline by using a Closure::

    public function getDefaultOptions()
    {
        return array(
            'validation_groups' => function(FormInterface $form) {
                $data = $form->getData();
                if (Entity\Client::TYPE_PERSON == $data->getType()) {
                    return array('person')
                } else {
                    return array('company');
                }
            },
        );
    }

.. index::
   single: Forms; Built-in Field Types

.. _book-forms-type-reference:

Built-in Field Types
内置字段类型
--------------------

Symfony comes standard with a large group of field types that cover all of
the common form fields and data types you'll encounter:
symfony内置有许多字段类型，包含了你可能用到的全部的常用表单字段和数据类型：

.. include:: /reference/forms/types/map.rst.inc

You can also create your own custom field types. This topic is covered in
the ":doc:`/cookbook/form/create_custom_field_type`" article of the cookbook.
你也可以创建自己的字段类型。关于这点请参阅":doc:`/cookbook/form/create_custom_field_type`"。

.. index::
   single: Forms; Field type options

Field Type Options
字段类型选项
~~~~~~~~~~~~~~~~~~

Each field type has a number of options that can be used to configure it.
For example, the ``dueDate`` field is currently being rendered as 3 select
boxes. However, the :doc:`date field</reference/forms/types/date>` can be
configured to be rendered as a single text box (where the user would enter
the date as a string in the box)::
每个字段类型都有一系列的选项。比如，当前状态下dueDate字段的输出样式是三个选择框。
但是:doc:`date field</reference/forms/types/date>`也可以被设置为通过一个单独的文本框提交
（这是用户必须以字符串的形式输入日期）::

    ->add('dueDate', 'date', array('widget' => 'single_text'))

.. image:: /images/book/form-simple2.png
    :align: center

Each field type has a number of different options that can be passed to it.
Many of these are specific to the field type and details can be found in
the documentation for each type.
每个字段类型都有一系列的不同的选项可以使用。这些选项可以在每个类型的文档中找到。

.. sidebar:: The ``required`` option

    The most common option is the ``required`` option, which can be applied to
    any field. By default, the ``required`` option is set to ``true``, meaning
    that HTML5-ready browsers will apply client-side validation if the field
    is left blank. If you don't want this behavior, either set the ``required``
    option on your field to ``false`` or :ref:`disable HTML5 validation<book-forms-html5-validation-disable>`.
    最常用的选项是“required”选项，它可以被用于任何字段。默认情况下，“required”被设置为“true”表示在用户提交
    空白框时浏览器会弹出验证框。如果你不想要这个验证，你可以在你的字段上设“false”或:ref:`disable HTML5 validation<book-forms-html5-validation-disable>`。
    
    Also note that setting the ``required`` option to ``true`` will **not**
    result in server-side validation to be applied. In other words, if a
    user submits a blank value for the field (either with an old browser
    or web service, for example), it will be accepted as a valid value unless
    you use Symfony's ``NotBlank`` or ``NotNull`` validation constraint.
    还要注意的是将“required”选项设置为“true”不会使服务器端应用验证。换句话说，如果一个用户
    提交了空白字段（通过老式浏览器或web service等方式），除非你使用symfony的“NotBlank”或者“NotNull”
    验证限制条件，否则它就会被接受。
    
    In other words, the ``required`` option is "nice", but true server-side
    validation should *always* be used.
    换句话说，用“required”方法不错，但你应该使用真正的服务器端验证。

.. sidebar:: The ``label`` option

    The label for the form field can be set using the ``label`` option,
    which can be applied to any field::
    表单字段的label可以通过label选项被设置，这对任何字段都适用：

        ->add('dueDate', 'date', array(
            'widget' => 'single_text',
            'label'  => 'Due Date',
        ))

    The label for a field can also be set in the template rendering the
    form, see below.
    表单字段的label也可以在输出这个表单的模板中被设置：

.. index::
   single: Forms; Field type guessing

.. _book-forms-field-guessing:

Field Type Guessing
字段类型猜测
-------------------

Now that you've added validation metadata to the ``Task`` class, Symfony
already knows a bit about your fields. If you allow it, Symfony can "guess"
the type of your field and set it up for you. In this example, Symfony can
guess from the validation rules that both the ``task`` field is a normal
``text`` field and the ``dueDate`` field is a ``date`` field::
现在你已经向task类添加了认证metadata，symfony也已经知道了许多有关你字段的信息。如果你
允许，symfony可以“猜测”你的字段类型并自动生成。在这个例子中，symfony可以从验证规则中
猜测，task是一个普通text字段，dueDate是一个date字段::

    public function newAction()
    {
        $task = new Task();

        $form = $this->createFormBuilder($task)
            ->add('task')
            ->add('dueDate', null, array('widget' => 'single_text'))
            ->getForm();
    }

The "guessing" is activated when you omit the second argument to the ``add()``
method (or if you pass ``null`` to it). If you pass an options array as the
third argument (done for ``dueDate`` above), these options are applied to
the guessed field.
当你忽略add()的第二个参数时，猜测（guessing）就激活了。如果你将选项数组作为第三个参数
（以上dueDate例子），这些选项将应用于这些猜测的字段。

.. caution::

    If your form uses a specific validation group, the field type guesser
    will still consider *all* validation constraints when guessing your
    field types (including constraints that are not part of the validation
    group(s) being used).
    如果你使用一个特定的验证集，猜测机制则会考虑所有的验证限制条件（包括这个验证集所共有的限制条件）。

.. index::
   single: Forms; Field type guessing

Field Type Options Guessing
字段类型选项猜测
~~~~~~~~~~~~~~~~~~~~~~~~~~~

In addition to guessing the "type" for a field, Symfony can also try to guess
the correct values of a number of field options.
除了猜测字段的类型，symfony还能猜测一些字段选项的值。

.. tip::

    When these options are set, the field will be rendered with special HTML
    attributes that provide for HTML5 client-side validation. However, it
    doesn't generate the equivalent server-side constraints (e.g. ``Assert\MaxLength``).
    And though you'll need to manually add your server-side validation, these
    field type options can then be guessed from that information.
    当这些选项被设置的时候，这些字段会被添加一些能够使HTML5客户端验证激活的属性，但是它不能集成
    服务器端的属性（如``Assert\MaxLength``）。尽管你要手动添加你的服务器端的验证，但这些字段类型选项
    可以从验证信息中被猜测。

* ``required``: The ``required`` option can be guessed based off of the validation
  rules (i.e. is the field ``NotBlank`` or ``NotNull``) or the Doctrine metadata
  (i.e. is the field ``nullable``). This is very useful, as your client-side
  validation will automatically match your validation rules.
  required：required选项可以在验证规则的基础上（``NotBlank``或``NotNull``）或者doctrine metadata（“nullable”）的
  基础上被猜测。这很有用，因为你的客户端的认证会自动匹配你的验证规则。

* ``max_length``: If the field is some sort of text field, then the ``max_length``
  option can be guessed from the validation constraints (if ``MaxLength`` or ``Max``
  is used) or from the Doctrine metadata (via the field's length).
  max_length：如果字段是text字段，那么``max_length``选项可以从约束条件（``MaxLength``或者``Max``）
  中猜测，还可以从doctrine的metadata中猜测（通过字段的length选项）。
  
.. note::

  These field options are *only* guessed if you're using Symfony to guess
  the field type (i.e. omit or pass ``null`` as the second argument to ``add()``).
  仅仅当你使用symfony的时候，这些字段选项可以被猜测（当你不设add()中第二个参数的时候）。

If you'd like to change one of the guessed values, you can override it by
passing the option in the options field array::
如果你想改变猜测的值，你可以通过覆盖选项字段array中的选项达到目的::

    ->add('task', null, array('max_length' => 4))

.. index::
   single: Forms; Rendering in a Template

.. _form-rendering-template:

Rendering a Form in a Template
在模板中输出表单
------------------------------

So far, you've seen how an entire form can be rendered with just one line
of code. Of course, you'll usually need much more flexibility when rendering:
现在，你已经知道了如只用一句代码来输出整个表单。当然，你还可以更灵活：

.. configuration-block::

    .. code-block:: html+jinja

        {# src/Acme/TaskBundle/Resources/views/Default/new.html.twig #}

        <form action="{{ path('task_new') }}" method="post" {{ form_enctype(form) }}>
            {{ form_errors(form) }}

            {{ form_row(form.task) }}
            {{ form_row(form.dueDate) }}

            {{ form_rest(form) }}

            <input type="submit" />
        </form>

    .. code-block:: html+php

        <!-- // src/Acme/TaskBundle/Resources/views/Default/newAction.html.php -->

        <form action="<?php echo $view['router']->generate('task_new') ?>" method="post" <?php echo $view['form']->enctype($form) ?>>
            <?php echo $view['form']->errors($form) ?>

            <?php echo $view['form']->row($form['task']) ?>
            <?php echo $view['form']->row($form['dueDate']) ?>

            <?php echo $view['form']->rest($form) ?>

            <input type="submit" />
        </form>

Let's take a look at each part:
让我们来看看这段代码：

* ``form_enctype(form)`` - If at least one field is a file upload field, this
  renders the obligatory ``enctype="multipart/form-data"``;
* ``form_enctype(form)``——如果至少有一个字段是一个file upload字段，这个语句会强制输出  
  ``enctype="multipart/form-data"``；

* ``form_errors(form)`` - Renders any errors global to the whole form
  (field-specific errors are displayed next to each field);
* ``form_errors(form)``——输出整个表单的全局性错误（每个字段的特定错误会在字段后方显示）；

* ``form_row(form.dueDate)`` - Renders the label, any errors, and the HTML
  form widget for the given field (e.g. ``dueDate``) inside, by default, a
  ``div`` element;
* ``form_row(form.dueDate)``——输出给定字段(e.g. ``dueDate``)的label，errors和HTML表单widget，
  默认情况下，有一个div标签；

* ``form_rest(form)`` - Renders any fields that have not yet been rendered.
  It's usually a good idea to place a call to this helper at the bottom of
  each form (in case you forgot to output a field or don't want to bother
  manually rendering hidden fields). This helper is also useful for taking
  advantage of the automatic :ref:`CSRF Protection<forms-csrf>`.
* ``form_rest(form)``——输出所有你没有输出的字段。最好是把这个helper方法放在表单的末尾，以防
  忘了要输出某个字段或者不想手动输出隐藏的字段。这个helper同时也能利用自动的:ref:`CSRF Protection<forms-csrf>`。  

The majority of the work is done by the ``form_row`` helper, which renders
the label, errors and HTML form widget of each field inside a ``div`` tag
by default. In the :ref:`form-theming` section, you'll learn how the ``form_row``
output can be customized on many different levels.
主要工作是靠``form_row``这个helper方法来完成的，默认情况下，它输出每个字段的label，errors和HTML widget
并将它们包围在一个div标签内。在:ref:`form-theming`这节，你将学习这个``form_row``能够如何被定制。

.. tip::

    You can access the current data of your form via ``form.vars.value``:
    你可以访问你的表单的当前数据，通过``form.vars.value``：

    .. configuration-block::

        .. code-block:: jinja

            {{ form.vars.value.task }}

        .. code-block:: html+php

            <?php echo $view['form']->get('value')->getTask() ?>

.. index::
   single: Forms; Rendering each field by hand

Rendering each Field by Hand
手动输出每个字段
~~~~~~~~~~~~~~~~~~~~~~~~~~~~

The ``form_row`` helper is great because you can very quickly render each
field of your form (and the markup used for the "row" can be customized as
well). But since life isn't always so simple, you can also render each field
entirely by hand. The end-product of the following is the same as when you
used the ``form_row`` helper:
``form_row``方法很强大，因为你可以通过它迅速输出你的表单的每个字段（而且每个row都是可以被定制的）。
但实际上还往往不能那么简单，你还可以手动输出每个字段。以下代码和你使用``form_row``时的结果是一样的：

.. configuration-block::

    .. code-block:: html+jinja

        {{ form_errors(form) }}

        <div>
            {{ form_label(form.task) }}
            {{ form_errors(form.task) }}
            {{ form_widget(form.task) }}
        </div>

        <div>
            {{ form_label(form.dueDate) }}
            {{ form_errors(form.dueDate) }}
            {{ form_widget(form.dueDate) }}
        </div>

        {{ form_rest(form) }}

    .. code-block:: html+php

        <?php echo $view['form']->errors($form) ?>

        <div>
            <?php echo $view['form']->label($form['task']) ?>
            <?php echo $view['form']->errors($form['task']) ?>
            <?php echo $view['form']->widget($form['task']) ?>
        </div>

        <div>
            <?php echo $view['form']->label($form['dueDate']) ?>
            <?php echo $view['form']->errors($form['dueDate']) ?>
            <?php echo $view['form']->widget($form['dueDate']) ?>
        </div>

        <?php echo $view['form']->rest($form) ?>

If the auto-generated label for a field isn't quite right, you can explicitly
specify it:
如果字段自动生成的label不是你想要的，你可以显性地定义它：

.. configuration-block::

    .. code-block:: html+jinja

        {{ form_label(form.task, 'Task Description') }}

    .. code-block:: html+php

        <?php echo $view['form']->label($form['task'], 'Task Description') ?>

Some field types have additional rendering options that can be passed
to the widget. These options are documented with each type, but one common
options is ``attr``, which allows you to modify attributes on the form element.
The following would add the ``task_field`` class to the rendered input text
field:
有些字段类型有着额外的能够被传递到widget的选项。这些选项在每个类型的文档中都有，但最常用的一个就是
attr，它允许你修改表单元素的属性。以下这段代码会为被输出的input text字段添加``task_field`` class：

.. configuration-block::

    .. code-block:: html+jinja

        {{ form_widget(form.task, { 'attr': {'class': 'task_field'} }) }}

    .. code-block:: html+php

        <?php echo $view['form']->widget($form['task'], array(
            'attr' => array('class' => 'task_field'),
        )) ?>

If you need to render form fields "by hand" then you can access individual
values for fields such as the ``id``, ``name`` and ``label``. For example
to get the ``id``:
如果你要手动输出表单，那么你就可以访问一些值，比如说``id``, ``name``和``label``。例如访问id：

.. configuration-block::

    .. code-block:: html+jinja

        {{ form.task.vars.id }}

    .. code-block:: html+php

        <?php echo $form['task']->get('id') ?>

To get the value used for the form field's name attribute you need to use
the ``full_name`` value:
要访问表单字段的name属性，你需要使用``full_name``值：

.. configuration-block::

    .. code-block:: html+jinja

        {{ form.task.vars.full_name }}

    .. code-block:: html+php

        <?php echo $form['task']->get('full_name') ?>

Twig Template Function Reference
twig模板方法索引
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

If you're using Twig, a full reference of the form rendering functions is
available in the :doc:`reference manual</reference/forms/twig_reference>`.
Read this to know everything about the helpers available and the options
that can be used with each.
如果你使用twig，在:doc:`reference manual</reference/forms/twig_reference>`文档
里有表单输出方法的全部参考。读完这个参考你就可以了解所有的helper方法及其选项了。

.. index::
   single: Forms; Creating form classes

.. _book-form-creating-form-classes:

Creating Form Classes
创建表单类
---------------------

As you've seen, a form can be created and used directly in a controller.
However, a better practice is to build the form in a separate, standalone PHP
class, which can then be reused anywhere in your application. Create a new class
that will house the logic for building the task form:
如你所见，一个表单可以在一个控制器里直接被创建和使用。但是，一个更好的方法是将这个表单
在一个独立的php类中创建，这样你就可以在你的应用中重复使用了。为task表单创建一个新类：

.. code-block:: php

    // src/Acme/TaskBundle/Form/Type/TaskType.php

    namespace Acme\TaskBundle\Form\Type;

    use Symfony\Component\Form\AbstractType;
    use Symfony\Component\Form\FormBuilder;

    class TaskType extends AbstractType
    {
        public function buildForm(FormBuilder $builder, array $options)
        {
            $builder->add('task');
            $builder->add('dueDate', null, array('widget' => 'single_text'));
        }

        public function getName()
        {
            return 'task';
        }
    }

This new class contains all the directions needed to create the task form
(note that the ``getName()`` method should return a unique identifier for this
form "type"). It can be used to quickly build a form object in the controller:
这个新类包含了所有的task表单所需的元素(注意``getName()``方法应该为这个表单“type”返回一个唯一的标识符)。
它可以被用来在控制器内迅速地创建一个表单对象：

.. code-block:: php

    // src/Acme/TaskBundle/Controller/DefaultController.php

    // add this new use statement at the top of the class
    use Acme\TaskBundle\Form\Type\TaskType;

    public function newAction()
    {
        $task = // ...
        $form = $this->createForm(new TaskType(), $task);

        // ...
    }

Placing the form logic into its own class means that the form can be easily
reused elsewhere in your project. This is the best way to create forms, but
the choice is ultimately up to you.
将表单放在单独的文件中意味着这个表单可以方便地在你的程序的任何地方使用。这是创建表单的最好
方法，但你也可以选择用或不用。

.. _book-forms-data-class:

.. sidebar:: Setting the ``data_class``

    Every form needs to know the name of the class that holds the underlying
    data (e.g. ``Acme\TaskBundle\Entity\Task``). Usually, this is just guessed
    based off of the object passed to the second argument to ``createForm``
    (i.e. ``$task``). Later, when you begin embedding forms, this will no
    longer be sufficient. So, while not always necessary, it's generally a
    good idea to explicitly specify the ``data_class`` option by adding the
    following to your form type class::
    每个表单都需要知道要存储提交的数据的那个类的名字（e.g. ``Acme\TaskBundle\Entity\Task``）。
    通常，symfony都是通过传递给``createForm``的第二个参数(i.e. ``$task``)来猜测的。不过，等你要
    嵌入表单的时候，这样做还是不够的。虽然并非总是需要，但你最好显性地定义``data_class``选项，通过在
    你的表单的类中添加以下代码：

        public function getDefaultOptions()
        {
            return array(
                'data_class' => 'Acme\TaskBundle\Entity\Task',
            );
        }

.. tip::

    When mapping forms to objects, all fields are mapped. Any fields on the
    form that do not exist on the mapped object will cause an exception to
    be thrown.
    当将表单映射到对象时，所有字段都被映射了。所有在表单上有但在对象中没有的字段都
    会抛出错误。

    In cases where you need extra fields in the form (for example: a "do you
    agree with these terms" checkbox) that will not be mapped to the underlying
    object, you need to set the property_path option to ``false``::
    如果你需要在表单中添加额外的字段（比如一个“你是否同意以上条款”的复选框），而不想将这些字段
    映射到对象中，你需要将property_path选项设置为``false``::

        public function buildForm(FormBuilder $builder, array $options)
        {
            $builder->add('task');
            $builder->add('dueDate', null, array('property_path' => false));
        }

    Additionally, if there are any fields on the form that aren't included in
    the submitted data, those fields will be explicitly set to ``null``.
    另外，如果表单中有字段没有包含在被提交的数据中，这些字段会被设置为null。

.. index::
   pair: Forms; Doctrine

Forms and Doctrine
表单和doctrine
------------------

The goal of a form is to translate data from an object (e.g. ``Task``) to an
HTML form and then translate user-submitted data back to the original object. As
such, the topic of persisting the ``Task`` object to the database is entirely
unrelated to the topic of forms. But, if you've configured the ``Task`` class
to be persisted via Doctrine (i.e. you've added
:ref:`mapping metadata<book-doctrine-adding-mapping>` for it), then persisting
it after a form submission can be done when the form is valid::
表单的任务就是将对象(e.g. ``Task``)中的数据转化为一个HTML表单并将用户提交的数据提交到源对象。
所以，关于task对象如何将数据提交到数据库和表单的话题无关。但是如果你指定通过doctrine来提交
``Task``类的数据（或者说你已经为它添加了:ref:`mapping metadata<book-doctrine-adding-mapping>`），
那么当表单的数据可以通过验证的时候，就能在表单提交后将数据提交到数据库了::

    if ($form->isValid()) {
        $em = $this->getDoctrine()->getManager();
        $em->persist($task);
        $em->flush();

        return $this->redirect($this->generateUrl('task_success'));
    }

If, for some reason, you don't have access to your original ``$task`` object,
you can fetch it from the form::
如果，因为某些原因，你无法访问这个``$task``对象，你可以通过表单获取它::

    $task = $form->getData();

For more information, see the :doc:`Doctrine ORM chapter</book/doctrine>`.
更多信息请参见:doc:`Doctrine ORM chapter</book/doctrine>`。

The key thing to understand is that when the form is bound, the submitted
data is transferred to the underlying object immediately. If you want to
persist that data, you simply need to persist the object itself (which already
contains the submitted data).
关键是，当表单被绑定之后，被提交的数据会马上被传递到对象中。如果你想将数据插入数据库，你
只要提交这个对象就行了（它已经包含了被提交的数据了）。

.. index::
   single: Forms; Embedded forms

Embedded Forms
嵌入的表单
--------------

Often, you'll want to build a form that will include fields from many different
objects. For example, a registration form may contain data belonging to
a ``User`` object as well as many ``Address`` objects. Fortunately, this
is easy and natural with the form component.
往往你需要创建一个包含不同对象中的字段的表单。比如，一个注册表单可能包含User对象的字段，也
包含Address对象的字段。这在表单系统中很容易实现。

Embedding a Single Object
嵌入一个单独的对象
~~~~~~~~~~~~~~~~~~~~~~~~~

Suppose that each ``Task`` belongs to a simple ``Category`` object. Start,
of course, by creating the ``Category`` object::
假设每个task都隶属于一个category类。首先创建一个category类::

    // src/Acme/TaskBundle/Entity/Category.php
    namespace Acme\TaskBundle\Entity;

    use Symfony\Component\Validator\Constraints as Assert;

    class Category
    {
        /**
         * @Assert\NotBlank()
         */
        public $name;
    }

Next, add a new ``category`` property to the ``Task`` class::
接下来，将category属性添加到task类中::

    // ...

    class Task
    {
        // ...

        /**
         * @Assert\Type(type="Acme\TaskBundle\Entity\Category")
         */
        protected $category;

        // ...

        public function getCategory()
        {
            return $this->category;
        }

        public function setCategory(Category $category = null)
        {
            $this->category = $category;
        }
    }

Now that your application has been updated to reflect the new requirements,
create a form class so that a ``Category`` object can be modified by the user::
现在你的应用已经被更新，创建一个表单类，这样category类就能被用户修改了::

    // src/Acme/TaskBundle/Form/Type/CategoryType.php
    namespace Acme\TaskBundle\Form\Type;

    use Symfony\Component\Form\AbstractType;
    use Symfony\Component\Form\FormBuilder;

    class CategoryType extends AbstractType
    {
        public function buildForm(FormBuilder $builder, array $options)
        {
            $builder->add('name');
        }

        public function getDefaultOptions()
        {
            return array(
                'data_class' => 'Acme\TaskBundle\Entity\Category',
            );
        }

        public function getName()
        {
            return 'category';
        }
    }

The end goal is to allow the ``Category`` of a ``Task`` to be modified right
inside the task form itself. To accomplish this, add a ``category`` field
to the ``TaskType`` object whose type is an instance of the new ``CategoryType``
class:
最终目的是要允许task的category能够在task表单中直接被修改。要达到这个目的，在``TaskType``
对象中添加一个类型为``CategoryType``类的实例的category字段：

.. code-block:: php

    public function buildForm(FormBuilder $builder, array $options)
    {
        // ...

        $builder->add('category', new CategoryType());
    }

The fields from ``CategoryType`` can now be rendered alongside those from
the ``TaskType`` class. Render the ``Category`` fields in the same way
as the original ``Task`` fields:
现在这个``CategoryType``的字段可以和``TaskType``类的字段一起被输出了。像task表单的其他
字段一样提交category字段：

.. configuration-block::

    .. code-block:: html+jinja

        {# ... #}

        <h3>Category</h3>
        <div class="category">
            {{ form_row(form.category.name) }}
        </div>

        {{ form_rest(form) }}
        {# ... #}

    .. code-block:: html+php

        <!-- ... -->

        <h3>Category</h3>
        <div class="category">
            <?php echo $view['form']->row($form['category']['name']) ?>
        </div>

        <?php echo $view['form']->rest($form) ?>
        <!-- ... -->

When the user submits the form, the submitted data for the ``Category`` fields
are used to construct an instance of ``Category``, which is then set on the
``category`` field of the ``Task`` instance.
当用户提交表单时，category字段的数据被用于创建一个category对象的实例，这个实例被设置在task
实例的category字段上。

The ``Category`` instance is accessible naturally via ``$task->getCategory()``
and can be persisted to the database or used however you need.
category实例可以通过``$task->getCategory()``访问，你可以将它插入数据库，或做其他工作。

Embedding a Collection of Forms
嵌入一个表单集合
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

You can also embed a collection of forms into one form (imagine a ``Category``
form with many ``Product`` sub-forms). This is done by using the ``collection``
field type.
你也可以在一个表单里嵌入一个表单集合（比如将许多product表单嵌入category表单中）。
这是通过设置collection字段类型做到的。

For more information see the ":doc:`/cookbook/form/form_collections`" cookbook
entry and  the :doc:`collection</reference/forms/types/collection>` field type reference.
要获取更多信息请参阅":doc:`/cookbook/form/form_collections`"和字段类型索引:doc:`collection</reference/forms/types/collection>`。

.. index::
   single: Forms; Theming
   single: Forms; Customizing fields

.. _form-theming:

Form Theming
表单主题
------------

Every part of how a form is rendered can be customized. You're free to change
how each form "row" renders, change the markup used to render errors, or
even customize how a ``textarea`` tag should be rendered. Nothing is off-limits,
and different customizations can be used in different places.
表单的每个部分的输出都可以被定制。你可以选择每行如何被输出，改变错误信息的样式，或定义一个textarea
标签如何被输出。你可以做很多事情，并且不同的定制可以被用于不同的地方。

Symfony uses templates to render each and every part of a form, such as
``label`` tags, ``input`` tags, error messages and everything else.
symfony使用模板来输出表单及其标签，如label，input，errors等等。

In Twig, each form "fragment" is represented by a Twig block. To customize
any part of how a form renders, you just need to override the appropriate block.
在twig中，每个表单“片段”都被包围在一个twig block中。要定制表单的任意一个部分，你
只需要覆盖一个合适的block就行了。

In PHP, each form "fragment" is rendered via an individual template file.
To customize any part of how a form renders, you just need to override the
existing template by creating a new one.
在php中，每个片段都是通过一个模板文件来输出的。要定制表单每个部分的输出，你只要
修改合适的文件就行了。

To understand how this works, let's customize the ``form_row`` fragment and
add a class attribute to the ``div`` element that surrounds each row. To
do this, create a new template file that will store the new markup:
要了解这是如何工作的，让我们定制一个``form_row``片段并将一个class属性加到它的div标签里面。
要达到这个目的，需要创建一个可以存储这些定制的模板：

.. configuration-block::

    .. code-block:: html+jinja

        {# src/Acme/TaskBundle/Resources/views/Form/fields.html.twig #}

        {% block field_row %}
        {% spaceless %}
            <div class="form_row">
                {{ form_label(form) }}
                {{ form_errors(form) }}
                {{ form_widget(form) }}
            </div>
        {% endspaceless %}
        {% endblock field_row %}

    .. code-block:: html+php

        <!-- src/Acme/TaskBundle/Resources/views/Form/field_row.html.php -->

        <div class="form_row">
            <?php echo $view['form']->label($form, $label) ?>
            <?php echo $view['form']->errors($form) ?>
            <?php echo $view['form']->widget($form, $parameters) ?>
        </div>

The ``field_row`` form fragment is used when rendering most fields via the
``form_row`` function. To tell the form component to use your new ``field_row``
fragment defined above, add the following to the top of the template that
renders the form:
当通过``field_row``方法来输出字段时，``field_row``表单片段被用到。要告诉你的表单系统使用你创建
的``field_row``片段时，将以下代码加入到你要输出表单的模板中：

.. configuration-block:: php

    .. code-block:: html+jinja

        {# src/Acme/TaskBundle/Resources/views/Default/new.html.twig #}

        {% form_theme form 'AcmeTaskBundle:Form:fields.html.twig' %}

        {% form_theme form 'AcmeTaskBundle:Form:fields.html.twig' 'AcmeTaskBundle:Form:fields2.html.twig' %}

        <form ...>

    .. code-block:: html+php

        <!-- src/Acme/TaskBundle/Resources/views/Default/new.html.php -->

        <?php $view['form']->setTheme($form, array('AcmeTaskBundle:Form')) ?>

        <?php $view['form']->setTheme($form, array('AcmeTaskBundle:Form', 'AcmeTaskBundle:Form')) ?>

        <form ...>

The ``form_theme`` tag (in Twig) "imports" the fragments defined in the given
template and uses them when rendering the form. In other words, when the
``form_row`` function is called later in this template, it will use the ``field_row``
block from your custom theme (instead of the default ``field_row`` block
that ships with Symfony).
``form_theme``标签（twig中）导入以上模板中定制的片段并且在输出表单的时候使用它们。换句话说，
当在这个模板中执行``form_row``方法时，它会使用你定制的片段中的``field_row`` block（而不是默认的
symfony中的``field_row``）。

Your custom theme does not have to override all the blocks. When rendering a block
which is not overridden in your custom theme, the theming engine will fall back
to the global theme (defined at the bundle level).
你所定制的主题不需要覆盖所有的block。当输出一个没有被覆盖的block的时候，symfony的主题引擎会使用
源主题。

If several custom themes are provided they will be searched in the listed order
before falling back to the global theme.
如果数个定制主题被提供，则symfony会搜索所有的定制主题，然后才使用源主题。

To customize any portion of a form, you just need to override the appropriate
fragment. Knowing exactly which block or file to override is the subject of
the next section.
要定制一个表单中的任何部分，你只需要覆盖合适的片段。下一节将学习如何知道哪个片段要被覆盖。

.. versionadded:: 2.1
   An alternate Twig syntax for ``form_theme`` has been introduced in 2.1. It accepts
   any valid Twig expression (the most noticeable difference is using an array when
   using multiple themes).
   一个可选的twig语法在2.1中被引进。它接受任何被验证的twig表达式（在使用多个主题时，最大的不同就是
   它会使用数组）。

   .. code-block:: html+jinja

       {# src/Acme/TaskBundle/Resources/views/Default/new.html.twig #}

       {% form_theme form with 'AcmeTaskBundle:Form:fields.html.twig' %}

       {% form_theme form with ['AcmeTaskBundle:Form:fields.html.twig', 'AcmeTaskBundle:Form:fields2.html.twig'] %}

For a more extensive discussion, see :doc:`/cookbook/form/form_customization`.
更多信息请参见:doc:`/cookbook/form/form_customization`。

.. index::
   single: Forms; Template fragment naming

.. _form-template-blocks:

Form Fragment Naming
表单片段命名
~~~~~~~~~~~~~~~~~~~~

In Symfony, every part of a form that is rendered - HTML form elements, errors,
labels, etc - is defined in a base theme, which is a collection of blocks
in Twig and a collection of template files in PHP.
在symfony中，被输出的表单的每个部分——HTML元素，errors，label，等等，都在基本主题中被定义了。
基本主题是twig中的一系列block集合以及php中的一系列模板文件集合。

In Twig, every block needed is defined in a single template file (`form_div_layout.html.twig`_)
that lives inside the `Twig Bridge`_. Inside this file, you can see every block
needed to render a form and every default field type.
在twig中，每个需要的block都在一个`Twig Bridge`_中的模板文件中被定义（`form_div_layout.html.twig`_）。
在这个文件中，你将见到输出表单时所需的每一个部分以及它们默认的字段类型。

In PHP, the fragments are individual template files. By default they are located in
the `Resources/views/Form` directory of the framework bundle (`view on GitHub`_).
php中，片段是单独的模板文件。默认情况下，它们被至于framework bundle(`view on GitHub`_)中的
`Resources/views/Form`目录下。

Each fragment name follows the same basic pattern and is broken up into two pieces,
separated by a single underscore character (``_``). A few examples are:
每个片段名都遵循着同一个基本模式，并且分两个部分，由一个下划线隔开：

* ``field_row`` - used by ``form_row`` to render most fields;
* ``field_row``——由``form_row``使用来输出大多数字段；
* ``textarea_widget`` - used by ``form_widget`` to render a ``textarea`` field
  type;
* ``textarea_widget``——由``form_widget``使用，用于输出一个textarea字段类型；  
* ``field_errors`` - used by ``form_errors`` to render errors for a field;
* ``field_errors``——由``form_errors``使用，用于输出字段的errors；

Each fragment follows the same basic pattern: ``type_part``. The ``type`` portion
corresponds to the field *type* being rendered (e.g. ``textarea``, ``checkbox``,
``date``, etc) whereas the ``part`` portion corresponds to *what* is being
rendered (e.g. ``label``, ``widget``, ``errors``, etc). By default, there
are 4 possible *parts* of a form that can be rendered:
每个字段都遵循这相同的模式：``type_part``。type部分是指要输出的字段类型（e.g. ``textarea``, ``checkbox``,
``date``, etc），而part部分则指要输出字段的哪一部分（e.g. ``label``, ``widget``, ``errors``, etc）。
默认情况下，表单有四个部分要被输出：

+-------------+--------------------------+---------------------------------------------------------+
| ``label``   | (e.g. ``field_label``)   | renders the field's label                               |
+-------------+--------------------------+---------------------------------------------------------+
| ``widget``  | (e.g. ``field_widget``)  | renders the field's HTML representation                 |
+-------------+--------------------------+---------------------------------------------------------+
| ``errors``  | (e.g. ``field_errors``)  | renders the field's errors                              |
+-------------+--------------------------+---------------------------------------------------------+
| ``row``     | (e.g. ``field_row``)     | renders the field's entire row (label, widget & errors) |
+-------------+--------------------------+---------------------------------------------------------+

.. note::

    There are actually 3 other *parts*  - ``rows``, ``rest``, and ``enctype`` -
    but you should rarely if ever need to worry about overriding them.
    事实上还有三个其他部分——``rows``, ``rest``, 还有``enctype``。但你几乎不需要覆盖它们。

By knowing the field type (e.g. ``textarea``) and which part you want to
customize (e.g. ``widget``), you can construct the fragment name that needs
to be overridden (e.g. ``textarea_widget``).
知道了字段类型以及哪一部分字段你要定制（e.g. ``widget``），你就可以构建需要被覆盖的片段名称了（e.g. ``textarea_widget``）。

.. index::
   single: Forms; Template Fragment Inheritance

Template Fragment Inheritance
模板片段继承
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

In some cases, the fragment you want to customize will appear to be missing.
For example, there is no ``textarea_errors`` fragment in the default themes
provided with Symfony. So how are the errors for a textarea field rendered?
某些情况下，你想要定制的片段可能找不到，比如，在symfony提供的默认主题中没有``textarea_errors``
这么一个片段。那么如何定制一个textarea字段的errors呢？

The answer is: via the ``field_errors`` fragment. When Symfony renders the errors
for a textarea type, it looks first for a ``textarea_errors`` fragment before
falling back to the ``field_errors`` fragment. Each field type has a *parent*
type (the parent type of ``textarea`` is ``field``), and Symfony uses the
fragment for the parent type if the base fragment doesn't exist.
答案是：通过``field_errors``片段。当symfony要输出一个textarea类型的errors时，它先是查找一个
``textarea_errors``片段，找不到的话再使用``field_errors``片段。每个字段类型都有一个父类型（textarea的父类型是field），
如果找不到基本片段，symfony会使用父类型的片段。

So, to override the errors for *only* ``textarea`` fields, copy the
``field_errors`` fragment, rename it to ``textarea_errors`` and customize it. To
override the default error rendering for *all* fields, copy and customize the
``field_errors`` fragment directly.
所以，如果只想覆盖textarea字段的errors，只要复制``field_errors``片段，重命名为``textarea_errors``
并定制就可以了。要覆盖对所有字段输出的错误，就直接复制并定制``field_errors``片段。

.. tip::

    The "parent" type of each field type is available in the
    :doc:`form type reference</reference/forms/types>` for each field type.
    每个类型的父类型在:doc:`form type reference</reference/forms/types>`中有参考。

.. index::
   single: Forms; Global Theming

Global Form Theming
全局表单主题
~~~~~~~~~~~~~~~~~~~

In the above example, you used the ``form_theme`` helper (in Twig) to "import"
the custom form fragments into *just* that form. You can also tell Symfony
to import form customizations across your entire project.
在上面的例子中，你使用了``form_theme``helper（twig中的）来将定制的表单片段导入那个表单。
你还可以告诉symfony将表单定制导入你的代码任意处。

Twig
....

To automatically include the customized blocks from the ``fields.html.twig``
template created earlier in *all* templates, modify your application configuration
file:
要自动将``fields.html.twig``中定制的block包含入所有的模板中，修改你的应用配置：

.. configuration-block::

    .. code-block:: yaml

        # app/config/config.yml

        twig:
            form:
                resources:
                    - 'AcmeTaskBundle:Form:fields.html.twig'
            # ...

    .. code-block:: xml

        <!-- app/config/config.xml -->

        <twig:config ...>
                <twig:form>
                    <resource>AcmeTaskBundle:Form:fields.html.twig</resource>
                </twig:form>
                <!-- ... -->
        </twig:config>

    .. code-block:: php

        // app/config/config.php

        $container->loadFromExtension('twig', array(
            'form' => array('resources' => array(
                'AcmeTaskBundle:Form:fields.html.twig',
             ))
            // ...
        ));

Any blocks inside the ``fields.html.twig`` template are now used globally
to define form output.
现在所有``fields.html.twig``模板中的block都可以在全局应用了。

.. sidebar::  Customizing Form Output all in a Single File with Twig

    In Twig, you can also customize a form block right inside the template
    where that customization is needed:
    在twig中，你还可以在模板内部直接定制表单block：

    .. code-block:: html+jinja

        {% extends '::base.html.twig' %}

        {# import "_self" as the form theme #}
        {% form_theme form _self %}

        {# make the form fragment customization #}
        {% block field_row %}
            {# custom field row output #}
        {% endblock field_row %}

        {% block content %}
            {# ... #}

            {{ form_row(form.task) }}
        {% endblock %}

    The ``{% form_theme form _self %}`` tag allows form blocks to be customized
    directly inside the template that will use those customizations. Use
    this method to quickly make form output customizations that will only
    ever be needed in a single template.
    ``{% form_theme form _self %}``标签允许在模板中直接定制表单block。这个方法可以迅速地
    运用只在一个单独模板里适用的定制。

PHP
...

To automatically include the customized templates from the ``Acme/TaskBundle/Resources/views/Form``
directory created earlier in *all* templates, modify your application configuration
file:
要在所有模板中自动包含所有``Acme/TaskBundle/Resources/views/Form``目录下的定制，修改你的配置文件：

.. configuration-block::

    .. code-block:: yaml

        # app/config/config.yml

        framework:
            templating:
                form:
                    resources:
                        - 'AcmeTaskBundle:Form'
        # ...


    .. code-block:: xml

        <!-- app/config/config.xml -->

        <framework:config ...>
            <framework:templating>
                <framework:form>
                    <resource>AcmeTaskBundle:Form</resource>
                </framework:form>
            </framework:templating>
            <!-- ... -->
        </framework:config>

    .. code-block:: php

        // app/config/config.php

        $container->loadFromExtension('framework', array(
            'templating' => array('form' =>
                array('resources' => array(
                    'AcmeTaskBundle:Form',
             )))
            // ...
        ));

Any fragments inside the ``Acme/TaskBundle/Resources/views/Form`` directory
are now used globally to define form output.
在``Acme/TaskBundle/Resources/views/Form``目录下的片段现在可以在全局运用了。

.. index::
   single: Forms; CSRF Protection

.. _forms-csrf:

CSRF Protection
CSRF保护
---------------

CSRF - or `Cross-site request forgery`_ - is a method by which a malicious
user attempts to make your legitimate users unknowingly submit data that
they don't intend to submit. Fortunately, CSRF attacks can be prevented by
using a CSRF token inside your forms.
CSRF——或称`Cross-site request forgery`_——是一种攻击方法，通过它，恶意用户能使
合法用户不知不觉中提交他们所不想提交的数据。CSRF可以通过你表单的一个CSRF token
来避免。

The good news is that, by default, Symfony embeds and validates CSRF tokens
automatically for you. This means that you can take advantage of the CSRF
protection without doing anything. In fact, every form in this chapter has
taken advantage of the CSRF protection!
symfony自动为你嵌入了CSRF token。这表示你不需要做任何事情就可以利用CSRF保护了。
事实上，本章中所有的表单都受到了CSRF保护！

CSRF protection works by adding a hidden field to your form - called ``_token``
by default - that contains a value that only you and your user knows. This
ensures that the user - not some other entity - is submitting the given data.
Symfony automatically validates the presence and accuracy of this token.
CSRF保护是通过在你的表单中添加一个隐藏字段来实现的，这个字段默认名叫``_token``，它包含了
一个值，这个值只有你和你的用户知道。这保证了用户——而不是其他实体——提交给定数据。symfony
自动验证这个token。

The ``_token`` field is a hidden field and will be automatically rendered
if you include the ``form_rest()`` function in your template, which ensures
that all un-rendered fields are output.
``_token``字段是一个隐藏字段，如果你执行``form_rest()``方法的话，它就会被提交。

The CSRF token can be customized on a form-by-form basis. For example::
CSRF token可以在一个表单对表单基础上定制，比如：

    class TaskType extends AbstractType
    {
        // ...

        public function getDefaultOptions()
        {
            return array(
                'data_class'      => 'Acme\TaskBundle\Entity\Task',
                'csrf_protection' => true,
                'csrf_field_name' => '_token',
                // a unique key to help generate the secret token
                'intention'       => 'task_item',
            );
        }

        // ...
    }

To disable CSRF protection, set the ``csrf_protection`` option to false.
Customizations can also be made globally in your project. For more information,
see the :ref:`form configuration reference <reference-framework-form>`
section.
要取消CSRF保护，只要将``csrf_protection``选项设置为false。这个定制亦可以被设为全局性的，见:ref:`form configuration reference <reference-framework-form>`。

.. note::

    The ``intention`` option is optional but greatly enhances the security of
    the generated token by making it different for each form.
    ``intention``选项是可选的，但是如果让它在每个表单中都不同，它可以极大地提升token的安全
    性能。

.. index:
   single: Forms; With no class

Using a Form without a Class
使用一个不针对类的表单
----------------------------

In most cases, a form is tied to an object, and the fields of the form get
and store their data on the properties of that object. This is exactly what
you've seen so far in this chapter with the `Task` class.
多数情况下，一个表单与一个对象绑定，并且表单的字段还要将它们的数据传给这个对象。

But sometimes, you may just want to use a form without a class, and get back
an array of the submitted data. This is actually really easy::
但有些时候，你只想使用一个不针对类的表单，并且取回提交的数据。这很容易::

    // make sure you've imported the Request namespace above the class
    use Symfony\Component\HttpFoundation\Request
    // ...

    public function contactAction(Request $request)
    {
        $defaultData = array('message' => 'Type your message here');
        $form = $this->createFormBuilder($defaultData)
            ->add('name', 'text')
            ->add('email', 'email')
            ->add('message', 'textarea')
            ->getForm();

            if ($request->getMethod() == 'POST') {
                $form->bindRequest($request);

                // data is an array with "name", "email", and "message" keys
                $data = $form->getData();
            }

        // ... render the form
    }

By default, a form actually assumes that you want to work with arrays of
data, instead of an object. There are exactly two ways that you can change
this behavior and tie the form to an object instead:
默认情况下，一个表单假定你想取得数组而不是对象。有两种方法可以改变这种行为，并且将表单
与一个对象联系起来：

1. Pass an object when creating the form (as the first argument to ``createFormBuilder``
   or the second argument to ``createForm``);
1. 创建表单的时候传递一个对象（作为第一个参数传递到``createFormBuilder``或者作为第二个参数传递到``createForm``）；   

2. Declare the ``data_class`` option on your form.
2. 在你的表单中声明``data_class``选项。

If you *don't* do either of these, then the form will return the data as
an array. In this example, since ``$defaultData`` is not an object (and
no ``data_class`` option is set), ``$form->getData()`` ultimately returns
an array.
如果你两个都不做，那么这个表单会将数据作为数组返回。在这个例子中，由于``$defaultData``
不是一个对象（而且``data_class``选项也没有设置），``$form->getData()``最终会返回一个数组。

.. tip::

    You can also access POST values (in this case "name") directly through
    the request object, like so:
    你还可以直接通过request对象访问POST值（这个例子中是“name”），如:

    .. code-block:: php

        $this->get('request')->request->get('name');

    Be advised, however, that in most cases using the getData() method is
    a better choice, since it returns the data (usually an object) after
    it's been transformed by the form framework.
    但是，在大多数情况下，最好使用getData()方法，因为它在表单系统把数据编译了
    之后返回数据（通常是对象）。

Adding Validation
添加认证
~~~~~~~~~~~~~~~~~

The only missing piece is validation. Usually, when you call ``$form->isValid()``,
the object is validated by reading the constraints that you applied to that
class. But without a class, how can you add constraints to the data of your
form?
唯一没讲的就是认证了。通常，当你执行``$form->isValid()``时，对象已经通过读取你传递到那个类的
限制条件来进行认证了。但是如果没有类，你怎样将限制条件加到你的表单数据中呢？

The answer is to setup the constraints yourself, and pass them into your
form. The overall approach is covered a bit more in the :ref:`validation chapter<book-validation-raw-values>`,
but here's a short example::
答案是你自己设置限制条件，并将他们传递到你的表单中。更多请参见:ref:`validation chapter<book-validation-raw-values>`。
以下是一个简单例子：

    // import the namespaces above your controller class
    use Symfony\Component\Validator\Constraints\Email;
    use Symfony\Component\Validator\Constraints\MinLength;
    use Symfony\Component\Validator\Constraints\Collection;

    $collectionConstraint = new Collection(array(
        'name' => new MinLength(5),
        'email' => new Email(array('message' => 'Invalid email address')),
    ));

    // create a form, no default values, pass in the constraint option
    $form = $this->createFormBuilder(null, array(
        'validation_constraint' => $collectionConstraint,
    ))->add('email', 'email')
        // ...
    ;

Now, when you call `$form->bindRequest($request)`, the constraints setup here are run
against your form's data. If you're using a form class, override the ``getDefaultOptions``
method to specify the option::
现在，当你执行`$form->bindRequest($request)`时，这里设置的限制条件被运行。如果你使用一个表单类，
覆盖``getDefaultOptions``方法::

    namespace Acme\TaskBundle\Form\Type;

    use Symfony\Component\Form\AbstractType;
    use Symfony\Component\Form\FormBuilder;
    use Symfony\Component\Validator\Constraints\Email;
    use Symfony\Component\Validator\Constraints\MinLength;
    use Symfony\Component\Validator\Constraints\Collection;

    class ContactType extends AbstractType
    {
        // ...

        public function getDefaultOptions()
        {
            $collectionConstraint = new Collection(array(
                'name' => new MinLength(5),
                'email' => new Email(array('message' => 'Invalid email address')),
            ));

            return array('validation_constraint' => $collectionConstraint);
        }
    }

Now, you have the flexibility to create forms - with validation - that return
an array of data, instead of an object. In most cases, it's better - and
certainly more robust - to bind your form to an object. But for simple forms,
this is a great approach.
现在，你就能够使用验证灵活创建表单了——它返回一个数组，而不是对象。大多数情况下，最好能将表单
绑定一个对象，这样的代码更健壮。但对于简单的例子，这就够了。

Final Thoughts
总结                                                                                            
--------------

You now know all of the building blocks necessary to build complex and
functional forms for your application. When building forms, keep in mind that
the first goal of a form is to translate data from an object (``Task``) to an
HTML form so that the user can modify that data. The second goal of a form is to
take the data submitted by the user and to re-apply it to the object.
你现在已经知道了如何创建复杂和功能性强的表单。当创建表单时，要牢记表单的第一要务是将
对象(``Task``)中的数据编译到HTML表单中，让用户修改那个数据。表单的第二个工作是将用户提交到 数据返还到
对象中。

There's still much more to learn about the powerful world of forms, such as
how to handle :doc:`file uploads with Doctrine
</cookbook/doctrine/file_uploads>` or how to create a form where a dynamic
number of sub-forms can be added (e.g. a todo list where you can keep adding
more fields via Javascript before submitting). See the cookbook for these
topics. Also, be sure to lean on the
:doc:`field type reference documentation</reference/forms/types>`, which
includes examples of how to use each field type and its options.
还有很多关于表单的东西要学，如如何处理:doc:`file uploads with Doctrine
</cookbook/doctrine/file_uploads>`，或如何创建一个可以添加动态数量子表单的表单（如一个你可以
通过javascript添加字段的todo list）。请参阅cookbook。并学习:doc:`field type reference documentation</reference/forms/types>`，
它有许多关于如何使用不同类型字段的内容。

Learn more from the Cookbook
----------------------------

* :doc:`/cookbook/doctrine/file_uploads`
* :doc:`File Field Reference </reference/forms/types/file>`
* :doc:`Creating Custom Field Types </cookbook/form/create_custom_field_type>`
* :doc:`/cookbook/form/form_customization`
* :doc:`/cookbook/form/dynamic_form_generation`
* :doc:`/cookbook/form/data_transformers`

.. _`Symfony2 Form Component`: https://github.com/symfony/Form
.. _`DateTime`: http://php.net/manual/en/class.datetime.php
.. _`Twig Bridge`: https://github.com/symfony/symfony/tree/master/src/Symfony/Bridge/Twig
.. _`form_div_layout.html.twig`: https://github.com/symfony/symfony/blob/master/src/Symfony/Bridge/Twig/Resources/views/Form/form_div_layout.html.twig
.. _`Cross-site request forgery`: http://en.wikipedia.org/wiki/Cross-site_request_forgery
.. _`view on GitHub`: https://github.com/symfony/symfony/tree/master/src/Symfony/Bundle/FrameworkBundle/Resources/views/Form
