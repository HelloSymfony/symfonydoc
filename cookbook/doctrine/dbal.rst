.. index::
   pair: Doctrine; DBAL

How to use Doctrine's DBAL Layer
如何使用doctrine的DBAL层
================================

.. note::

    This article is about Doctrine DBAL's layer. Typically, you'll work with
    the higher level Doctrine ORM layer, which simply uses the DBAL behind
    the scenes to actually communicate with the database. To read more about
    the Doctrine ORM, see ":doc:`/book/doctrine`".
    本文是关于如何使用doctrine的DBAL层。一般的，你是在doctrine ORM层上工作，这是高一级的层，
    它使用DBAL层来与数据库通信。更多关于doctrine ORM的信息请参见":doc:`/book/doctrine`"。

The `Doctrine`_ Database Abstraction Layer (DBAL) is an abstraction layer that
sits on top of `PDO`_ and offers an intuitive and flexible API for communicating
with the most popular relational databases. In other words, the DBAL library
makes it easy to execute queries and perform other database actions.
`Doctrine`_数据库抽象层（DBAL）是一个位于`PDO`_顶端的层，它通过一个直观又灵活的API来与
最常用的关系型数据库通信。换句话说，DBAL库使得执行请求和其他数据库行为更方便了。

.. tip::

    Read the official Doctrine `DBAL Documentation`_ to learn all the details
    and capabilities of Doctrine's DBAL library.
    详情请见官方文档`DBAL Documentation`_。

To get started, configure the database connection parameters:
首先，配置数据库连接参数:

.. configuration-block::

    .. code-block:: yaml

        # app/config/config.yml
        doctrine:
            dbal:
                driver:   pdo_mysql
                dbname:   Symfony2
                user:     root
                password: null
                charset:  UTF8

    .. code-block:: xml

        // app/config/config.xml
        <doctrine:config>
            <doctrine:dbal
                name="default"
                dbname="Symfony2"
                user="root"
                password="null"
                driver="pdo_mysql"
            />
        </doctrine:config>

    .. code-block:: php

        // app/config/config.php
        $container->loadFromExtension('doctrine', array(
            'dbal' => array(
                'driver'    => 'pdo_mysql',
                'dbname'    => 'Symfony2',
                'user'      => 'root',
                'password'  => null,
            ),
        ));

For full DBAL configuration options, see :ref:`reference-dbal-configuration`.
更多关于DBAL配置的选项请参见:ref:`reference-dbal-configuration`。

You can then access the Doctrine DBAL connection by accessing the
``database_connection`` service:
你还可以通过访问``database_connection``服务来访问doctrine DBAL连接：

.. code-block:: php

    class UserController extends Controller
    {
        public function indexAction()
        {
            $conn = $this->get('database_connection');
            $users = $conn->fetchAll('SELECT * FROM users');

            // ...
        }
    }

Registering Custom Mapping Types
注册定制的映射类型
--------------------------------

You can register custom mapping types through Symfony's configuration. They
will be added to all configured connections. For more information on custom
mapping types, read Doctrine's `Custom Mapping Types`_ section of their documentation.
你可以通过symfony的配置来注册定制的映射类型。它会被添加到所有被配置的连接上。
更多关于定制映射类型的信息请参见`Custom Mapping Types`_。

.. configuration-block::

    .. code-block:: yaml

        # app/config/config.yml
        doctrine:
            dbal:
                types:
                    custom_first: Acme\HelloBundle\Type\CustomFirst
                    custom_second: Acme\HelloBundle\Type\CustomSecond

    .. code-block:: xml

        <!-- app/config/config.xml -->
        <container xmlns="http://symfony.com/schema/dic/services"
            xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
            xmlns:doctrine="http://symfony.com/schema/dic/doctrine"
            xsi:schemaLocation="http://symfony.com/schema/dic/services http://symfony.com/schema/dic/services/services-1.0.xsd
                                http://symfony.com/schema/dic/doctrine http://symfony.com/schema/dic/doctrine/doctrine-1.0.xsd">

            <doctrine:config>
                <doctrine:dbal>
                <doctrine:dbal default-connection="default">
                    <doctrine:connection>
                        <doctrine:mapping-type name="enum">string</doctrine:mapping-type>
                    </doctrine:connection>
                </doctrine:dbal>
            </doctrine:config>
        </container>

    .. code-block:: php

        // app/config/config.php
        $container->loadFromExtension('doctrine', array(
            'dbal' => array(
                'connections' => array(
                    'default' => array(
                        'mapping_types' => array(
                            'enum'  => 'string',
                        ),
                    ),
                ),
            ),
        ));

Registering Custom Mapping Types in the SchemaTool
在SchemaTool中注册定制的映射类型
--------------------------------------------------

The SchemaTool is used to inspect the database to compare the schema. To
achieve this task, it needs to know which mapping type needs to be used
for each database types. Registering new ones can be done through the configuration.
SchemaTool用于监测数据库并比较其结构（schema）。要达到这个目的，它需要知道对于每个数据库类型
要使用哪个映射类型。通过配置可以注册新的类型。

Let's map the ENUM type (not suppoorted by DBAL by default) to a the ``string``
mapping type:
现在我们来将ENUM类型（它现在还不被DBAL支持）映射到string类上：

.. configuration-block::

    .. code-block:: yaml

        # app/config/config.yml
        doctrine:
            dbal:
                connections:
                    default:
                        // Other connections parameters
                        mapping_types:
                            enum: string

    .. code-block:: xml

        <!-- app/config/config.xml -->
        <container xmlns="http://symfony.com/schema/dic/services"
            xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
            xmlns:doctrine="http://symfony.com/schema/dic/doctrine"
            xsi:schemaLocation="http://symfony.com/schema/dic/services http://symfony.com/schema/dic/services/services-1.0.xsd
                                http://symfony.com/schema/dic/doctrine http://symfony.com/schema/dic/doctrine/doctrine-1.0.xsd">

            <doctrine:config>
                <doctrine:dbal>
                    <doctrine:type name="custom_first" class="Acme\HelloBundle\Type\CustomFirst" />
                    <doctrine:type name="custom_second" class="Acme\HelloBundle\Type\CustomSecond" />
                </doctrine:dbal>
            </doctrine:config>
        </container>

    .. code-block:: php

        // app/config/config.php
        $container->loadFromExtension('doctrine', array(
            'dbal' => array(
                'types' => array(
                    'custom_first'  => 'Acme\HelloBundle\Type\CustomFirst',
                    'custom_second' => 'Acme\HelloBundle\Type\CustomSecond',
                ),
            ),
        ));

.. _`PDO`:           http://www.php.net/pdo
.. _`Doctrine`:      http://www.doctrine-project.org
.. _`DBAL Documentation`: http://docs.doctrine-project.org/projects/doctrine-dbal/en/latest/index.html
.. _`Custom Mapping Types`: http://docs.doctrine-project.org/projects/doctrine-dbal/en/latest/reference/types.html#custom-mapping-types