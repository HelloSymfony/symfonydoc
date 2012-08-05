.. index::
   single: Form; Events

How to Dynamically Generate Forms Using Form Events
如何使用表单事件来动态创建表单
===================================================

Before jumping right into dynamic form generation, let's have a quick review 
of what a bare form class looks like::
首先让我们看一下这个表单类::

    //src/Acme/DemoBundle/Form/ProductType.php
    namespace Acme\DemoBundle\Form;

    use Symfony\Component\Form\AbstractType;
    use Symfony\Component\Form\FormBuilder;
    
    class ProductType extends AbstractType
    {
        public function buildForm(FormBuilder $builder, array $options)
        {
            $builder->add('name');
            $builder->add('price');
        }

        public function getName()
        {
            return 'product';
        }
    }

.. note::

    If this particular section of code isn't already familiar to you, you 
    probably need to take a step back and first review the :doc:`Forms chapter </book/forms>` 
    before proceeding.
    如果你对这段代码不熟悉，你需要回顾一下:doc:`Forms chapter </book/forms>`。

Let's assume for a moment that this form utilizes an imaginary "Product" class
that has only two relevant properties ("name" and "price"). The form generated 
from this class will look the exact same regardless of a new Product is being created
or if an existing product is being edited (e.g. a product fetched from the database).
我们假定这个表单使用一个具有两个属性（name和price）的Product类。不管一个新的Product要被创建
或者一个已经存在的Product要被修改，通过这个类集成的表单都会是一个样子。

Suppose now, that you don't want the user to be able to change the `name` value 
once the object has been created. To do this, you can rely on Symfony's :ref:`Event Dispatcher <book-internals-event-dispatcher>` 
system to analyze the data on the object and modify the form based on the 
Product object's data. In this entry, you'll learn how to add this level of 
flexibility to your forms.
假设你不想在修改一个已经创建的对象时让用户能够修改它的name属性。要达到这个目的，你可以使用symfony
的:ref:`Event Dispatcher <book-internals-event-dispatcher>`系统来解析对象上的数据并根据这个数据来
输出表单。

.. _`cookbook-forms-event-subscriber`:

Adding An Event Subscriber To A Form Class
向表单类添加一个事件subscriber
------------------------------------------

So, instead of directly adding that "name" widget via our ProductType form 
class, let's delegate the responsibility of creating that particular field 
to an Event Subscriber::
我们可以让一个事件subscriber来代理创建那个name字段，而不是通过ProductType来直接添加name字段::

    //src/Acme/DemoBundle/Form/ProductType.php
    namespace Acme\DemoBundle\Form

    use Symfony\Component\Form\AbstractType
    use Symfony\Component\Form\FormBuilder;
    use Acme\DemoBundle\Form\EventListener\AddNameFieldSubscriber;

    class ProductType extends AbstractType
    {
        public function buildForm(FormBuilder $builder, array $options)
        {
            $subscriber = new AddNameFieldSubscriber($builder->getFormFactory());
            $builder->addEventSubscriber($subscriber);
            $builder->add('price');
        }

        public function getName()
        {
            return 'product';
        }
    }

The event subscriber is passed the FormFactory object in its constructor so 
that our new subscriber is capable of creating the form widget once it is 
notified of the dispatched event during form creation.
这个事件subscriber将FormFactory传递到它的constructor中，这样我们的subscriber在表单创建的过程中就可以根据
分配的事件来创建表单widget了。

.. _`cookbook-forms-inside-subscriber-class`:

Inside the Event Subscriber Class
事件subscriber的内部
---------------------------------

The goal is to create a "name" field *only* if the underlying Product object
is new (e.g. hasn't been persisted to the database). Based on that, the subscriber
might look like the following::
我们的目的是只在Product对象还没有创建的时候创建一个name字段。subscriber可能会像这样::

    // src/Acme/DemoBundle/Form/EventListener/AddNameFieldSubscriber.php
    namespace Acme\DemoBundle\Form\EventListener;

    use Symfony\Component\Form\Event\DataEvent;
    use Symfony\Component\Form\FormFactoryInterface;
    use Symfony\Component\EventDispatcher\EventSubscriberInterface;
    use Symfony\Component\Form\FormEvents;

    class AddNameFieldSubscriber implements EventSubscriberInterface
    {
        private $factory;
        
        public function __construct(FormFactoryInterface $factory)
        {
            $this->factory = $factory;
        }
        
        public static function getSubscribedEvents()
        {
            // Tells the dispatcher that we want to listen on the form.pre_set_data
            // event and that the preSetData method should be called.
            return array(FormEvents::PRE_SET_DATA => 'preSetData');
        }

        public function preSetData(DataEvent $event)
        {
            $data = $event->getData();
            $form = $event->getForm();
            
            // During form creation setData() is called with null as an argument 
            // by the FormBuilder constructor. We're only concerned with when 
            // setData is called with an actual Entity object in it (whether new,
            // or fetched with Doctrine). This if statement let's us skip right 
            // over the null condition.
            if (null === $data) {
                return;
            }

            // check if the product object is "new"
            if (!$data->getId()) {
                $form->add($this->factory->createNamed('text', 'name'));
            }
        }
    }

.. caution::

    It is easy to misunderstand the purpose of the ``if (null === $data)`` segment 
    of this event subscriber. To fully understand its role, you might consider 
    also taking a look at the `Form class`_ and paying special attention to 
    where setData() is called at the end of the constructor, as well as the 
    setData() method itself.
    你很容易误解``if (null === $data)``的目的。要完全了解它的角色，你可以参阅`Form class`_并
    注意setData()是在constructor中的什么地方被执行的，还要参见setMethod()方法本身。

The ``FormEvents::PRE_SET_DATA`` line actually resolves to the string ``form.pre_set_data``. 
The `FormEvents class`_ serves an organizational purpose. It is a centralized  location
in which you can find all of the various form events available.
``FormEvents::PRE_SET_DATA``实际上被解析为``form.pre_set_data``。`FormEvents class`_则是用于组织的
目的，你可以在它的目录中找到所有可用的表单事件。

While this example could have used the ``form.set_data`` event or even the ``form.post_set_data`` 
events just as effectively, by using ``form.pre_set_data`` we guarantee that 
the data being retrieved from the ``Event`` object has in no way been modified 
by any other subscribers or listeners. This is because ``form.pre_set_data`` 
passes a `DataEvent`_ object instead of the `FilterDataEvent`_ object passed 
by the ``form.set_data`` event. `DataEvent`_, unlike its child `FilterDataEvent`_, 
lacks a setData() method.
这个例子虽然还可以使用``form.set_data``或者``form.post_set_data`` ，但使用``form.pre_set_data``
能保证从Event对象中获取的数据不会被其他subscriber或listener修改。这是因为``form.pre_set_data``传递
了一个`DataEvent`_对象，而不是通过``form.set_data``传递的`FilterDataEvent`_对象。`FilterDataEvent`_
的父类，`DataEvent`_，没有setData()方法。

.. note::

    You may view the full list of form events via the `FormEvents class`_, 
    found in the form bundle.

.. _`DataEvent`: https://github.com/symfony/symfony/blob/master/src/Symfony/Component/Form/Event/DataEvent.php
.. _`FormEvents class`: https://github.com/symfony/Form/blob/master/FormEvents.php
.. _`Form class`: https://github.com/symfony/symfony/blob/master/src/Symfony/Component/Form/Form.php
.. _`FilterDataEvent`: https://github.com/symfony/symfony/blob/master/src/Symfony/Component/Form/Event/FilterDataEvent.php
