.. index::
   single: Events; Create Listener

How to create an Event Listener
如何创建一个事件收听器
===============================

Symfony has various events and hooks that can be used to trigger custom
behavior in your application. Those events are thrown by the HttpKernel 
component and can be viewed in the :class:`Symfony\\Component\\HttpKernel\\KernelEvents` class. 
symfony有许多可以被用于在你的应用中激发定制的行为的事件。这些事件被HttpKernel component抛出，并且
可以在:class:`Symfony\\Component\\HttpKernel\\KernelEvents`类中查看。

To hook into an event and add your own custom logic, you have to  create
a service that will act as an event listener on that event. In this entry,
we will create a service that will act as an Exception Listener, allowing
us to modify how exceptions are shown by  our application. The ``KernelEvents::EXCEPTION``
event is just one of the core kernel events::
要与一个事件挂钩并添加你自己定制的逻辑，你可以创建一个像事件收听器（event listener）那样工作的服务（service）。
在本例中我们将创建一个错误收听器，它能够修改错误的显示方式。``KernelEvents::EXCEPTION``事件仅仅是核心kernel事件中的一个::

    // src/Acme/DemoBundle/Listener/AcmeExceptionListener.php
    namespace Acme\DemoBundle\Listener;

    use Symfony\Component\HttpKernel\Event\GetResponseForExceptionEvent;

    class AcmeExceptionListener
    {
        public function onKernelException(GetResponseForExceptionEvent $event)
        {
            // We get the exception object from the received event
            $exception = $event->getException();
            $message = 'My Error says: ' . $exception->getMessage();
            
            // Customize our response object to display our exception details
            $response->setContent($message);
            $response->setStatusCode($exception->getStatusCode());
            
            // Send our modified response object to the event
            $event->setResponse($response);
        }
    }

.. tip::

    Each event receives a slightly different type of ``$event`` object. For
    the ``kernel.exception`` event, it is :class:`Symfony\\Component\\HttpKernel\\Event\\GetResponseForExceptionEvent`.
    To see what type of object each event listener receives, see :class:`Symfony\\Component\\HttpKernel\\KernelEvents`,
    每个事件接收的$event对象的类型都有所不同。对于kernel.exception事件，它是:class:`Symfony\\Component\\HttpKernel\\Event\\GetResponseForExceptionEvent`。
    要查看每个事件收听器接收的对象类型，请参阅:class:`Symfony\\Component\\HttpKernel\\KernelEvents`。

Now that the class is created, we just need to register it as a service and
and notify Symfony that it is a "listener" on the ``kernel.exception`` event
by using a special "tag":
现在这个类已经被创建，我们只需要将它注册为服务，并
通过使用一个特定的tag告知symfony它是一个kernel.exception事件的listener就可以了。

.. configuration-block::

    .. code-block:: yaml

        services:
            kernel.listener.your_listener_name:
                class: Acme\DemoBundle\Listener\AcmeExceptionListener
                tags:
                    - { name: kernel.event_listener, event: kernel.exception, method: onKernelException }

    .. code-block:: xml

        <service id="kernel.listener.your_listener_name" class="Acme\DemoBundle\Listener\AcmeExceptionListener">
            <tag name="kernel.event_listener" event="kernel.exception" method="onKernelException" />
        </service>

    .. code-block:: php

        $container
            ->register('kernel.listener.your_listener_name', 'Acme\DemoBundle\Listener\AcmeExceptionListener')
            ->addTag('kernel.event_listener', array('event' => 'kernel.exception', 'method' => 'onKernelException'))
        ;
        
.. note::

    There is an additional tag option ``priority`` that is optional and defaults
    to 0. This value can be from -255 to 255, and the listeners will be executed
    in the order of their priority. This is useful when you need to guarantee
    that one listener is executed before another.
    还有一个tag选项priority（可选），它的默认值为0。它的值可以是-255到255，listener会根据它们的priority
    来被执行。当你需要保证一个listener在另一个listener之前被执行时就需要用到它了。
