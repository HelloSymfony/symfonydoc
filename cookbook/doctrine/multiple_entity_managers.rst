How to work with Multiple Entity Managers
如何使用多个Entity Manager
=========================================

You can use multiple entity managers in a Symfony2 application. This is
necessary if you are using different databases or even vendors with entirely
different sets of entities. In other words, one entity manager that connects
to one database will handle some entities while another entity manager that
connects to another database might handle the rest.
你可以在symfony2中使用多个entity manager。如果你使用的是不同的数据库或有完全不同实体类设置的
vendor，这一点是必须的。换句话说，联系到某个数据库的entity manager会处理某些实体类，而联系到
另一个数据库的entity manager会处理余下的。

.. note::

    Using multiple entity managers is pretty easy, but more advanced and not
    usually required. Be sure you actually need multiple entity managers before
    adding in this layer of complexity.
    使用多个entity manager很容易，但不常用。在添加之前请确保你需要多个entity manager。

The following configuration code shows how you can configure two entity managers:
以下的代码显示了你如何配置两个entity manager：

.. configuration-block::

    .. code-block:: yaml

        doctrine:
            orm:
                default_entity_manager:   default
                entity_managers:
                    default:
                        connection:       default
                        mappings:
                            AcmeDemoBundle: ~
                            AcmeStoreBundle: ~
                    customer:
                        connection:       customer
                        mappings:
                            AcmeCustomerBundle: ~

In this case, you've defined two entity managers and called them ``default``
and ``customer``. The ``default`` entity manager manages entities in the
``AcmeDemoBundle`` and ``AcmeStoreBundle``, while the ``customer`` entity
manager manages entities in the ``AcmeCustomerBundle``.
在本例中，你定义了两个entity manager并称其为default和customer。default entity manager管理
AcmeDemoBundle和AcmeStoreBundle中的实体类，customer entity manager管理AcmeCustomBundle中的实体类。

When working with multiple entity managers, you should be explicit about which
entity manager you want. If you *do* omit the entity manager's name when
asking for it, the default entity manager (i.e. ``default``) is returned::
在使用多个entity manager时，你必须显性指定你需要哪个entity manager。如果你忽略了entity manager的名称，
默认的那个（default）会被返回::

    class UserController extends Controller
    {
        public function indexAction()
        {
            // both return the "default" em
            $em = $this->get('doctrine')->getManager();
            $em = $this->get('doctrine')->getManager('default');
            
            $customerEm =  $this->get('doctrine')->getManager('customer');
        }
    }

You can now use Doctrine just as you did before - using the ``default`` entity
manager to persist and fetch entities that it manages and the ``customer``
entity manager to persist and fetch its entities.
你可以像以前一样使用doctrine——使用default entity manager来persist和获取它所管理的实体类并使用
customer entity manager来persist和获取它管理的实体类。
