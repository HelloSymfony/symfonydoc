How to Use a Factory to Create Services
如何使用Factory来创建服务
=======================================

Symfony2's Service Container provides a powerful way of controlling the 
creation of objects, allowing you to specify arguments passed to the constructor
as well as calling methods and setting parameters. Sometimes, however, this
will not provide you with everything you need to construct your objects.
For this situation, you can use a factory to create the object and tell the
service container to call a method on the factory rather than directly instantiating
the object.
symfony2的服务容器（service container）提供了一个控制对象创建的强大方法，它允许你指定要
传递到constructor方法中的参数、执行函数和设置参数。但有时它也不能提供给你所有创建你的对象所需要的东西。
在这种情况下，你可以使用一个factory来创建对象并告诉服务容器执行factory中的方法，而不是直接初始化
这个对象。

Suppose you have a factory that configures and returns a new NewsletterManager
object::
假设你有一个factory，它配置并返回一个新的NewsletterManager对象::

    namespace Acme\HelloBundle\Newsletter;

    class NewsletterFactory
    {
        public function get()
        {
            $newsletterManager = new NewsletterManager();
            
            // ...
            
            return $newsletterManager;
        }
    }

To make the ``NewsletterManager`` object available as a service, you can
configure the service container to use the ``NewsletterFactory`` factory
class:
要使得NewsletterManager对象成为服务，你可以使用NewsletterFactory这个factory类来配置
服务容器:

.. configuration-block::

    .. code-block:: yaml

        # src/Acme/HelloBundle/Resources/config/services.yml
        parameters:
            # ...
            newsletter_manager.class: Acme\HelloBundle\Newsletter\NewsletterManager
            newsletter_factory.class: Acme\HelloBundle\Newsletter\NewsletterFactory
        services:
            newsletter_manager:
                class:          %newsletter_manager.class%
                factory_class:  %newsletter_factory.class%
                factory_method: get 

    .. code-block:: xml

        <!-- src/Acme/HelloBundle/Resources/config/services.xml -->
        <parameters>
            <!-- ... -->
            <parameter key="newsletter_manager.class">Acme\HelloBundle\Newsletter\NewsletterManager</parameter>
            <parameter key="newsletter_factory.class">Acme\HelloBundle\Newsletter\NewsletterFactory</parameter>
        </parameters>

        <services>
            <service id="newsletter_manager" 
                     class="%newsletter_manager.class%"
                     factory-class="%newsletter_factory.class%"
                     factory-method="get"
            />
        </services>

    .. code-block:: php

        // src/Acme/HelloBundle/Resources/config/services.php
        use Symfony\Component\DependencyInjection\Definition;

        // ...
        $container->setParameter('newsletter_manager.class', 'Acme\HelloBundle\Newsletter\NewsletterManager');
        $container->setParameter('newsletter_factory.class', 'Acme\HelloBundle\Newsletter\NewsletterFactory');

        $container->setDefinition('newsletter_manager', new Definition(
            '%newsletter_manager.class%'
        ))->setFactoryClass(
            '%newsletter_factory.class%'
        )->setFactoryMethod(
            'get'
        );

When you specify the class to use for the factory (via ``factory_class``)
the method will be called statically. If the factory itself should be instantiated
and the resulting object's method called (as in this example), configure the
factory itself as a service:
当你指定（通过factory_class）这个factory要使用的类时，这个方法会被静态执行。如果factory本身应该被
初始化并且执行这个方法（如本例），你要将factory本身配置为服务:

.. configuration-block::

    .. code-block:: yaml

        # src/Acme/HelloBundle/Resources/config/services.yml
        parameters:
            # ...
            newsletter_manager.class: Acme\HelloBundle\Newsletter\NewsletterManager
            newsletter_factory.class: Acme\HelloBundle\Newsletter\NewsletterFactory
        services:
            newsletter_factory:
                class:            %newsletter_factory.class%
            newsletter_manager:
                class:            %newsletter_manager.class%
                factory_service:  newsletter_factory
                factory_method:   get 

    .. code-block:: xml

        <!-- src/Acme/HelloBundle/Resources/config/services.xml -->
        <parameters>
            <!-- ... -->
            <parameter key="newsletter_manager.class">Acme\HelloBundle\Newsletter\NewsletterManager</parameter>
            <parameter key="newsletter_factory.class">Acme\HelloBundle\Newsletter\NewsletterFactory</parameter>
        </parameters>

        <services>
            <service id="newsletter_factory" class="%newsletter_factory.class%"/>
            <service id="newsletter_manager" 
                     class="%newsletter_manager.class%"
                     factory-service="newsletter_factory"
                     factory-method="get"
            />
        </services>

    .. code-block:: php

        // src/Acme/HelloBundle/Resources/config/services.php
        use Symfony\Component\DependencyInjection\Definition;

        // ...
        $container->setParameter('newsletter_manager.class', 'Acme\HelloBundle\Newsletter\NewsletterManager');
        $container->setParameter('newsletter_factory.class', 'Acme\HelloBundle\Newsletter\NewsletterFactory');

        $container->setDefinition('newsletter_factory', new Definition(
            '%newsletter_factory.class%'
        ))
        $container->setDefinition('newsletter_manager', new Definition(
            '%newsletter_manager.class%'
        ))->setFactoryService(
            'newsletter_factory'
        )->setFactoryMethod(
            'get'
        );

.. note::

   The factory service is specified by its id name and not a reference to 
   the service itself. So, you do not need to use the @ syntax.
   这个factory服务被通过它的id名称指定了，它不是对它这个服务本身的引用，所以你不必使用@语句。

Passing Arguments to the Factory Method
向factory的方法中传递参数
---------------------------------------

If you need to pass arguments to the factory method, you can use the ``arguments``
options inside the service container. For example, suppose the ``get`` method
in the previous example takes the ``templating`` service as an argument:
如果你需要向factory方法中传递参数，你可以使用服务容器中的arguments选项。比如，假设get方法
使用templating服务作为参数:

.. configuration-block::

    .. code-block:: yaml

        # src/Acme/HelloBundle/Resources/config/services.yml
        parameters:
            # ...
            newsletter_manager.class: Acme\HelloBundle\Newsletter\NewsletterManager
            newsletter_factory.class: Acme\HelloBundle\Newsletter\NewsletterFactory
        services:
            newsletter_factory:
                class:            %newsletter_factory.class%
            newsletter_manager:
                class:            %newsletter_manager.class%
                factory_service:  newsletter_factory
                factory_method:   get
                arguments:
                    -             @templating

    .. code-block:: xml

        <!-- src/Acme/HelloBundle/Resources/config/services.xml -->
        <parameters>
            <!-- ... -->
            <parameter key="newsletter_manager.class">Acme\HelloBundle\Newsletter\NewsletterManager</parameter>
            <parameter key="newsletter_factory.class">Acme\HelloBundle\Newsletter\NewsletterFactory</parameter>
        </parameters>

        <services>
            <service id="newsletter_factory" class="%newsletter_factory.class%"/>
            <service id="newsletter_manager" 
                     class="%newsletter_manager.class%"
                     factory-service="newsletter_factory"
                     factory-method="get"
            >
                <argument type="service" id="templating" />
            </service>
        </services>

    .. code-block:: php

        // src/Acme/HelloBundle/Resources/config/services.php
        use Symfony\Component\DependencyInjection\Definition;

        // ...
        $container->setParameter('newsletter_manager.class', 'Acme\HelloBundle\Newsletter\NewsletterManager');
        $container->setParameter('newsletter_factory.class', 'Acme\HelloBundle\Newsletter\NewsletterFactory');

        $container->setDefinition('newsletter_factory', new Definition(
            '%newsletter_factory.class%'
        ))
        $container->setDefinition('newsletter_manager', new Definition(
            '%newsletter_manager.class%',
            array(new Reference('templating'))
        ))->setFactoryService(
            'newsletter_factory'
        )->setFactoryMethod(
            'get'
        );