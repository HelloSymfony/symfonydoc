.. _doctrine-event-config:

Registering Event Listeners and Subscribers
注册事件listener和subscriber
===========================================

Doctrine packages a rich event system that fires events when almost anything
happens inside the system. For you, this means that you can create arbitrary
:doc:`services</book/service_container>` and tell Doctrine to notify those
objects whenever a certain action (e.g. ``prePersist``) happens within Doctrine.
This could be useful, for example, to create an independent search index
whenever an object in your database is saved.
doctrine有非常丰富的事件系统，它会在几乎任何情况下都会激发事件。这意味着，你可以创建任意的
:doc:`services</book/service_container>`并告诉doctrine在doctrine中的任意事件（如prePersist）发生时
告知这些对象。这非常有用，比如，你可以在将一个实体类存储入数据库后创建一个独立的搜索索引。

Doctrine defines two types of objects that can listen to Doctrine events:
listeners and subscribers. Both are very similar, but listeners are a bit
more straightforward. For more, see `The Event System`_ on Doctrine's website.
doctrine定义了两种可以收听doctrine事件的对象：listener和subscriber。两者都很有用，但listener更直接。
详情请见doctrine网站的`The Event System`_。

Configuring the Listener/Subscriber
配置listener/subscriber
-----------------------------------

To register a service to act as an event listener or subscriber you just have
to :ref:`tag<book-service-container-tags>` it with the appropriate name. Depending
on your use-case, you can hook a listener into every DBAL connection and ORM
entity manager or just into one specific DBAL connection and all the entity
managers that use this connection.
要注册一个如事件listener和subscriber那样工作的服务，你只需使用合适的名称来:ref:`tag<book-service-container-tags>`它。
根据你要使用的情况，你可以将一个listener和每个DBAL连接、ORM entity manager挂钩，也可以仅与一个DBAL、所有的entity 
manager挂钩。

.. configuration-block::

    .. code-block:: yaml

        doctrine:
            dbal:
                default_connection: default
                connections:
                    default:
                        driver: pdo_sqlite
                        memory: true

        services:
            my.listener:
                class: Acme\SearchBundle\Listener\SearchIndexer
                tags:
                    - { name: doctrine.event_listener, event: postPersist }
            my.listener2:
                class: Acme\SearchBundle\Listener\SearchIndexer2
                tags:
                    - { name: doctrine.event_listener, event: postPersist, connection: default }
            my.subscriber:
                class: Acme\SearchBundle\Listener\SearchIndexerSubscriber
                tags:
                    - { name: doctrine.event_subscriber, connection: default }

    .. code-block:: xml

        <?xml version="1.0" ?>
        <container xmlns="http://symfony.com/schema/dic/services"
            xmlns:doctrine="http://symfony.com/schema/dic/doctrine">

            <doctrine:config>
                <doctrine:dbal default-connection="default">
                    <doctrine:connection driver="pdo_sqlite" memory="true" />
                </doctrine:dbal>
            </doctrine:config>

            <services>
                <service id="my.listener" class="Acme\SearchBundle\Listener\SearchIndexer">
                    <tag name="doctrine.event_listener" event="postPersist" />
                </service>
                <service id="my.listener2" class="Acme\SearchBundle\Listener\SearchIndexer2">
                    <tag name="doctrine.event_listener" event="postPersist" connection="default" />
                </service>
                <service id="my.subscriber" class="Acme\SearchBundle\Listener\SearchIndexerSubscriber">
                    <tag name="doctrine.event_subscriber" connection="default" />
                </service>
            </services>
        </container>

Creating the Listener Class
创建listener类
---------------------------

In the previous example, a service ``my.listener`` was configured as a Doctrine
listener on the event ``postPersist``. That class behind that service must have
a ``postPersist`` method, which will be called when the event is thrown::
在以上的范例中，对于postPersist配置了一个名为my.listener的服务来作为doctrine的listener。
那个服务背后的类必须有一个postPersist方法，该方法会在这个事件被抛出后执行::

    // src/Acme/SearchBundle/Listener/SearchIndexer.php
    namespace Acme\SearchBundle\Listener;
    
    use Doctrine\ORM\Event\LifecycleEventArgs;
    use Acme\StoreBundle\Entity\Product;
    
    class SearchIndexer
    {
        public function postPersist(LifecycleEventArgs $args)
        {
            $entity = $args->getEntity();
            $entityManager = $args->getManager();
            
            // perhaps you only want to act on some "Product" entity
            if ($entity instanceof Product) {
                // do something with the Product
            }
        }
    }

In each event, you have access to a ``LifecycleEventArgs`` object, which
gives you access to both the entity object of the event and the entity manager
itself.
对于每个事件，你都可以访问``LifecycleEventArgs``对象，它使你能够访问事件的实体对象以及entity manager本身。

One important thing to notice is that a listener will be listening for *all*
entities in your application. So, if you're interested in only handling a
specific type of entity (e.g. a ``Product`` entity but not a ``BlogPost``
entity), you should check for the class name of the entity in your method
(as shown above).
要注意的一点就是listener会收听你应用中的所有实体事件，所以如果你只想要处理某个特定的实体，（比如Product实体
而不是BlogPost实体），你应该在你的方法中检测该实体的类名称（如上例）。

.. _`The Event System`: http://docs.doctrine-project.org/projects/doctrine-orm/en/latest/reference/events.html