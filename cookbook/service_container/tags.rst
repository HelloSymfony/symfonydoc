.. index::
   single: Service Container; Tags

How to make your Services use Tags
如何使你的服务使用标签
==================================

Several of Symfony2's core services depend on tags to recognize which services
should be loaded, notified of events, or handled in some other special way.
For example, Twig uses the tag  ``twig.extension`` to load extra extensions.
有几个symfony2核心服务根据标签（tag）来判断应该加载哪个服务、接收事件通知、或处理其他事项。
比如，twig使用twig.extension来加载额外的扩展。

But you can also use tags in your own bundles. For example in case your service
handles a collection of some kind, or implements a "chain", in which several alternative
strategies are tried until one of them is successful. In this article I will use the example
of a "transport chain", which is a collection of classes implementing ``\Swift_Transport``.
Using the chain, the Swift mailer may try several ways of transport, until one succeeds.
This post focuses mainly on the dependency injection part of the story.
但你可以在你自己的bundle中使用标签。比如，假设你的服务要处理某个集合、或植入一个“链”且该链中有多个
可选项目要执行，直到返回成功的值为止。本章我们将使用transport链，它是一系列植入了\Swift_Transport的集合。
通过使用这个链，Swift mailer会试验该集合的各个项，直到其中一个成功为止。本章主要关注dependency injection。

To begin with, define the ``TransportChain`` class::
首先，定义TransportChain类::

    namespace Acme\MailerBundle;
    
    class TransportChain
    {
        private $transports;
    
        public function __construct()
        {
            $this->transports = array();
        }
    
        public function addTransport(\Swift_Transport  $transport)
        {
            $this->transports[] = $transport;
        }
    }

Then, define the chain as a service:
然后将这个链定义为服务:

.. configuration-block::

    .. code-block:: yaml

        # src/Acme/MailerBundle/Resources/config/services.yml
        parameters:
            acme_mailer.transport_chain.class: Acme\MailerBundle\TransportChain
        
        services:
            acme_mailer.transport_chain:
                class: %acme_mailer.transport_chain.class%

    .. code-block:: xml

        <!-- src/Acme/MailerBundle/Resources/config/services.xml -->

        <parameters>
            <parameter key="acme_mailer.transport_chain.class">Acme\MailerBundle\TransportChain</parameter>
        </parameters>
    
        <services>
            <service id="acme_mailer.transport_chain" class="%acme_mailer.transport_chain.class%" />
        </services>
        
    .. code-block:: php
    
        // src/Acme/MailerBundle/Resources/config/services.php
        use Symfony\Component\DependencyInjection\Definition;
        
        $container->setParameter('acme_mailer.transport_chain.class', 'Acme\MailerBundle\TransportChain');
        
        $container->setDefinition('acme_mailer.transport_chain', new Definition('%acme_mailer.transport_chain.class%'));

Define Services with a Custom Tag
使用定制的标签来定义服务
---------------------------------

Now we want several of the ``\Swift_Transport`` classes to be instantiated
and added to the chain automatically using the ``addTransport()`` method.
As an example we add the following transports as services:
现在我们想初始化\Swift_Transport类并使用addTransport()这个方法来自动添加到链上。
我们添加以下transport作为服务:

.. configuration-block::

    .. code-block:: yaml

        # src/Acme/MailerBundle/Resources/config/services.yml
        services:
            acme_mailer.transport.smtp:
                class: \Swift_SmtpTransport
                arguments:
                    - %mailer_host%
                tags:
                    -  { name: acme_mailer.transport }
            acme_mailer.transport.sendmail:
                class: \Swift_SendmailTransport
                tags:
                    -  { name: acme_mailer.transport }
    
    .. code-block:: xml

        <!-- src/Acme/MailerBundle/Resources/config/services.xml -->
        <service id="acme_mailer.transport.smtp" class="\Swift_SmtpTransport">
            <argument>%mailer_host%</argument>
            <tag name="acme_mailer.transport" />
        </service>
    
        <service id="acme_mailer.transport.sendmail" class="\Swift_SendmailTransport">
            <tag name="acme_mailer.transport" />
        </service>
        
    .. code-block:: php
    
        // src/Acme/MailerBundle/Resources/config/services.php
        use Symfony\Component\DependencyInjection\Definition;
        
        $definitionSmtp = new Definition('\Swift_SmtpTransport', array('%mailer_host%'));
        $definitionSmtp->addTag('acme_mailer.transport');
        $container->setDefinition('acme_mailer.transport.smtp', $definitionSmtp);
        
        $definitionSendmail = new Definition('\Swift_SendmailTransport');
        $definitionSendmail->addTag('acme_mailer.transport');
        $container->setDefinition('acme_mailer.transport.sendmail', $definitionSendmail);

Notice the tags named "acme_mailer.transport". We want the bundle to recognize
these transports and add them to the chain all by itself. In order to achieve
this, we need to  add a ``build()`` method to the ``AcmeMailerBundle`` class::
注意到标签名为acme_mailer.transport。我们想要bundle自己将这些transport添加到链上。
要达到这个目的，我们需要对AcmeMailerBundle类添加一个build()方法::

    namespace Acme\MailerBundle;
    
    use Symfony\Component\HttpKernel\Bundle\Bundle;
    use Symfony\Component\DependencyInjection\ContainerBuilder;
    
    use Acme\MailerBundle\DependencyInjection\Compiler\TransportCompilerPass;
    
    class AcmeMailerBundle extends Bundle
    {
        public function build(ContainerBuilder $container)
        {
            parent::build($container);
    
            $container->addCompilerPass(new TransportCompilerPass());
        }
    }

Create a ``CompilerPass``
创建一个CompilerPass
-------------------------

You will have spotted a reference to the not yet existing ``TransportCompilerPass`` class.
This class will make sure that all services with a tag ``acme_mailer.transport``
will be added to the ``TransportChain`` class by calling the ``addTransport()``
method. The ``TransportCompilerPass`` should look like this::
你应该注意到了这个TransportCompilerPass类，我们还没有设置它。这个类将保证所有有标签acme_transport的
服务能通过执行addTransport()被添加到TransportChai类中。TransportCompilerPass看起来应该像这样::

    namespace Acme\MailerBundle\DependencyInjection\Compiler;
    
    use Symfony\Component\DependencyInjection\ContainerBuilder;
    use Symfony\Component\DependencyInjection\Compiler\CompilerPassInterface;
    use Symfony\Component\DependencyInjection\Reference;
    
    class TransportCompilerPass implements CompilerPassInterface
    {
        public function process(ContainerBuilder $container)
        {
            if (false === $container->hasDefinition('acme_mailer.transport_chain')) {
                return;
            }
    
            $definition = $container->getDefinition('acme_mailer.transport_chain');
    
            foreach ($container->findTaggedServiceIds('acme_mailer.transport') as $id => $attributes) {
                $definition->addMethodCall('addTransport', array(new Reference($id)));
            }
        }
    }

The ``process()`` method checks for the existence of the ``acme_mailer.transport_chain``
service, then looks for all services tagged ``acme_mailer.transport``. It adds
to the definition of the ``acme_mailer.transport_chain`` service a call to
``addTransport()`` for each "acme_mailer.transport" service it has found.
The first argument of each of these calls will be the mailer transport service
itself.
process()方法会检测acme_mailer.transport_chain服务是否存在，然后查找所有具有acme_mailer.transport标签的服务。
它在acme_mailer.transport_chain服务的定义中，对每个acme_mailer.transport服务添加了一个addTransport()方法。
这个方法的第一个参数将是这个mailer transport服务本身。

.. note::

    By convention, tag names consist of the name of the bundle (lowercase,
    underscores as separators), followed by a dot, and finally the "real"
    name, so the tag "transport" in the AcmeMailerBundle should be: ``acme_mailer.transport``.
    按照惯例，标签名称包含了bundle的名称（小写，以下划线作为分隔线），后面跟着一个点，最后是真实名称，
    所以AcmeMailerBundle中的transport标签应该是：adme_mailer.transport。
    
The Compiled Service Definition
被编译过的服务定义
-------------------------------

Adding the compiler pass will result in the automatic generation of the following
lines of code in the compiled service container. In case you are working
in the "dev" environment, open the file ``/cache/dev/appDevDebugProjectContainer.php``
and look for the method ``getTransportChainService()``. It should look like this::
添加compiler pass会自动集成以下代码。如果你在dev环境中，打开``/cache/dev/appDevDebugProjectContainer.php``
并查找getTransportChainService()方法，它看起来会是这样::

    protected function getAcmeMailer_TransportChainService()
    {
        $this->services['acme_mailer.transport_chain'] = $instance = new \Acme\MailerBundle\TransportChain();

        $instance->addTransport($this->get('acme_mailer.transport.smtp'));
        $instance->addTransport($this->get('acme_mailer.transport.sendmail'));

        return $instance;
    }
