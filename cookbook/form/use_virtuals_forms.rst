.. index::
   single: Form; Use virtual forms

How to use the Virtual Form Field Option
如何使用虚拟表单字段选项
========================================

The ``virtual`` form field option can be very useful when you have some
duplicated fields in different entities.
如果你在不同的实体类中有重复的字段时，可以使用虚拟表单字段选项。

For example, imagine you have two entities, a ``Company`` and a ``Customer``::
比如，假设你有两个实体类，一个Company和一个Customer::

    // src/Acme/HelloBundle/Entity/Company.php
    namespace Acme\HelloBundle\Entity;

    class Company
    {
        private $name;
        private $website;

        private $address;
        private $zipcode;
        private $city;
        private $country;
    }

.. code-block:: php

    // src/Acme/HelloBundle/Entity/Company.php
    namespace Acme\HelloBundle\Entity;

    class Customer
    {
        private $firstName;
        private $lastName;

        private $address;
        private $zipcode;
        private $city;
        private $country;
    }

Like you can see, each entity shares a few of the same fields: ``address``,
``zipcode``, ``city``, ``country``.
如你所见，每个实体类都有一些相同的字段：``address``,
``zipcode``, ``city``, ``country``。

Now, you want to build two forms: one for a ``Company`` and the second for
a ``Customer``.
现在你需要创建两个表单：一个是Company的，一个是Customer的。

Start by creating a very simple ``CompanyType`` and ``CustomerType``::
首先创建一个简单的``CompanyType``和``CustomerType``::

    // src/Acme/HelloBundle/Form/Type/CompanyType.php
    namespace Acme\HelloBundle\Form\Type;

    class CompanyType extends AbstractType
    {
        public function buildForm(FormBuilder $builder, array $options)
        {
            $builder
                ->add('name', 'text')
                ->add('website', 'text')
            ;
        }
    }

.. code-block:: php

    // src/Acme/HelloBundle/Form/Type/CustomerType.php
    namespace Acme\HelloBundle\Form\Type;

    class CustomerType extends AbstractType
    {
        public function buildForm(FormBuilder $builder, array $options)
        {
            $builder
                ->add('firstName', 'text')
                ->add('lastName', 'text')
            ;
        }
    }

Now, we have to deal with the four duplicated fields. Here is a (simple)
location form type::
现在我们需要处理这四个重复的字段。以下是一个location表单类型（form type）::

    // src/Acme/HelloBundle/Form/Type/LocationType.php
    namespace Acme\HelloBundle\Form\Type;

    class LocationType extends AbstractType
    {
        public function buildForm(FormBuilder $builder, array $options)
        {
            $builder
                ->add('address', 'textarea')
                ->add('zipcode', 'string')
                ->add('city', 'string')
                ->add('country', 'text')
            ;
        }

        public function getName()
        {
            return 'location';
        }
    }

We don't *actually* have a location field in each of our entities, so we
can't directly link our ``LocationType`` to our ``CompanyType`` or ``CustomerType``.
But we absolutely want to have a dedicated form type to deal with location (remember, DRY!).
我们没有location实体类，所以我们不能直接将LocationType链接到CompanyType或CustomerType，但是我们
需要一个专用的表单类型来处理location。

The ``virtual`` form field option is the solution.
可以使用虚拟表单字段。

We can set the option ``'virtual' => true`` in the ``getDefaultOptions`` method
of ``LocationType`` and directly start using it in the two original form types.
我们可以在LocationType中的getDefaultOptions方法中设置virtual=>true选项，然后就可以在原来的
两个表单类型中使用它了。

Look at the result::
以下是结果::

    // CompanyType
    public function buildForm(FormBuilder $builder, array $options)
    {
        $builder->add('foo', new LocationType());
    }

.. code-block:: php

    // CustomerType
    public function buildForm(FormBuilder $builder, array $options)
    {
        $builder->add('bar', new LocationType());
    }

With the virtual option set to false (default behavior), the Form Component
expect each underlying object to have a ``foo`` (or ``bar``) property that
is either some object or array which contains the four location fields.
Of course, we don't have this object/array in our entities and we don't want it!
如果virtual选项被设置为false（这是默认情况），那么Form Component会期望每个表单对象都有
一个包含了以上四个location字段的foo（或bar）属性（该属性是对象或数组）。当然我们的实体类中没有这些对象/数组，我们也不想要！

With the virtual option set to true, the Form component skips the ``foo`` (or ``bar``)
property, and instead "gets" and "sets" the 4 location fields directly
on the underlying object!
当virtual选项被设置为true的时候，Form Component会跳过foo（或bar）属性，并直接在表单对象中get和set
这四个location字段。

.. note::

    Instead of setting the ``virtual`` option inside ``LocationType``, you
    can (just like with any options) also pass it in as an array option to
    the third argument of ``$builder->add()``.
    除了在LocationType中设置virtual选项，你还可以将virtual选项作为第三个参数传递给$builder->add()。
