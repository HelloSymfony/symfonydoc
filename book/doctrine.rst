.. index::
   single: Doctrine

Databases and Doctrine
数据库和doctrine
======================

Let's face it, one of the most common and challenging tasks for any application
involves persisting and reading information to and from a database. Fortunately,
Symfony comes integrated with `Doctrine`_, a library whose sole goal is to
give you powerful tools to make this easy. In this chapter, you'll learn the
basic philosophy behind Doctrine and see how easy working with a database can
be.
控制器的几个最主要的功能之一就是有关如何将信息载入数据库和从数据库获取它。symfony和`doctrine`
融合在一起，doctrine充当了一个使这些工作变得简单的工具。在这一章中，你将学习doctrine的基本
原理。你会发现doctrine使得对数据库的操作变得相当简单。

.. note::

    Doctrine is totally decoupled from Symfony and using it is optional.
    This chapter is all about the Doctrine ORM, which aims to let you map
    objects to a relational database (such as *MySQL*, *PostgreSQL* or *Microsoft SQL*).
    If you prefer to use raw database queries, this is easy, and explained
    in the ":doc:`/cookbook/doctrine/dbal`" cookbook entry.
    doctrine和symfony是脱离的，你可以选择是否使用它。本章都是关于doctrine ORM的，doctrine ORM是
    一种将对象映射到数据库中的方法（比如*MySQL*, *PostgreSQL* 或者 *Microsoft SQL*）。
    如果你更想用原始的数据库请求，也很容易，这些在":doc:`/cookbook/doctrine/dbal`"中有描述。

    You can also persist data to `MongoDB`_ using Doctrine ODM library. For
    more information, read the ":doc:`/bundles/DoctrineMongoDBBundle/index`"
    documentation.
    你也可以使用doctrine ORM库将数据载入数据库。详情请见 ":doc:`/bundles/DoctrineMongoDBBundle/index`"。

A Simple Example: A Product
一个简单的例子:A Product
---------------------------

The easiest way to understand how Doctrine works is to see it in action.
In this section, you'll configure your database, create a ``Product`` object,
persist it to the database and fetch it back out.
要学习它最好的办法就是看它如何工作。在这一节中，你会配置你的数据库，创建一个``Product``
对象，将它载入数据库并取出它。

.. sidebar:: Code along with the example

    If you want to follow along with the example in this chapter, create
    an ``AcmeStoreBundle`` via:
    如果你要按照本节所讲述的做，请创建一个``AcmeStoreBundle``:
    
    .. code-block:: bash
    
        php app/console generate:bundle --namespace=Acme/StoreBundle

Configuring the Database
配置数据库
~~~~~~~~~~~~~~~~~~~~~~~~

Before you really begin, you'll need to configure your database connection
information. By convention, this information is usually configured in an
``app/config/parameters.yml`` file:
在开始之前，你必须配置你的数据库连接信息。按照惯例，这个信息一般都在``app/config/parameters.yml``
中配置:

.. code-block:: yaml

    # app/config/parameters.yml
    parameters:
        database_driver:   pdo_mysql
        database_host:     localhost
        database_name:     test_project
        database_user:     root
        database_password: password

.. note::

    Defining the configuration via ``parameters.yml`` is just a convention.
    The parameters defined in that file are referenced by the main configuration
    file when setting up Doctrine:
    在``parameters.yml``中定义配置只是一个惯例。当建立doctrine时，在该文件中定义的参数是由主要配置文件进行
    索引的。
    
    .. code-block:: yaml
    
        doctrine:
            dbal:
                driver:   %database_driver%
                host:     %database_host%
                dbname:   %database_name%
                user:     %database_user%
                password: %database_password%
    
    By separating the database information into a separate file, you can
    easily keep different versions of the file on each server. You can also
    easily store database configuration (or any sensitive information) outside
    of your project, like inside your Apache configuration, for example. For
    more information, see :doc:`/cookbook/configuration/external_parameters`.
    通过将数据库配置信息放在一个单独的文件中，你可以保证每个服务器有不同版本的文件。
    你还可以将数据库配置信息（或其他任何敏感信息）放在你的代码外部，比如放在你的Apache配置
    中。详情请见:doc:`/cookbook/configuration/external_parameters`。

Now that Doctrine knows about your database, you can have it create the database
for you:
现在doctrine已经知道了你的数据库配置，你可以让它创建数据库了:

.. code-block:: bash

    php app/console doctrine:database:create

Creating an Entity Class
创建实体类
~~~~~~~~~~~~~~~~~~~~~~~~

Suppose you're building an application where products need to be displayed.
Without even thinking about Doctrine or databases, you already know that
you need a ``Product`` object to represent those products. Create this class
inside the ``Entity`` directory of your ``AcmeStoreBundle``::
假设你要建立一个展示各个产品的应用。不需要懂doctrine或数据库知识，你就已经知道你
需要一个``Product``对象来展示这些products。在你的``AcmeStoreBundle``的``Entity``
目录中创建这个类::

    // src/Acme/StoreBundle/Entity/Product.php    
    namespace Acme\StoreBundle\Entity;

    class Product
    {
        protected $name;

        protected $price;

        protected $description;
    }

The class - often called an "entity", meaning *a basic class that holds data* -
is simple and helps fulfill the business requirement of needing products
in your application. This class can't be persisted to a database yet - it's
just a simple PHP class.
这个类——通常被称作"实体（entity）"，表示*一个存储这些数据的基本类*——它非常简单并且帮助你
载入数据。暂时它还不能被载入数据库——现在它仅仅是一个简单的PHP类。

.. tip::

    Once you learn the concepts behind Doctrine, you can have Doctrine create
    this entity class for you:
    
    .. code-block:: bash
        
        php app/console doctrine:generate:entity --entity="AcmeStoreBundle:Product" --fields="name:string(255) price:float description:text"

.. index::
    single: Doctrine; Adding mapping metadata

.. _book-doctrine-adding-mapping:

Add Mapping Information
加入映射信息
~~~~~~~~~~~~~~~~~~~~~~~

Doctrine allows you to work with databases in a much more interesting way
than just fetching rows of a column-based table into an array. Instead, Doctrine
allows you to persist entire *objects* to the database and fetch entire objects
out of the database. This works by mapping a PHP class to a database table,
and the properties of that PHP class to columns on the table:
doctrine允许你用非常有趣的方式操作数据库，而不是以数组的形式从数据库取数据。doctrine允许你
将整个*类*载入数据库并将整个类从数据库取出来。通过将一个PHP类映射到数据库表中（这个PHP类的property
作为表的列），你就能达到这个目的。

.. image:: /images/book/doctrine_image_1.png
   :align: center

For Doctrine to be able to do this, you just have to create "metadata", or
configuration that tells Doctrine exactly how the ``Product`` class and its
properties should be *mapped* to the database. This metadata can be specified
in a number of different formats including YAML, XML or directly inside the
``Product`` class via annotations:
你只需要创建"metadata",或者一些配置文件告诉doctrine如何将*Product*类和它的property
映射到数据库。这些metadata可以是YAML,XML,或者是``Product``类中的注释:

.. note::

    A bundle can accept only one metadata definition format. For example, it's
    not possible to mix YAML metadata definitions with annotated PHP entity
    class definitions.
    一个bundle只能接受一种metadata定义。比如，你不能同时使用YAML metadata和PHP实体类（entity class）注释。 

.. configuration-block::

    .. code-block:: php-annotations

        // src/Acme/StoreBundle/Entity/Product.php
        namespace Acme\StoreBundle\Entity;

        use Doctrine\ORM\Mapping as ORM;

        /**
         * @ORM\Entity
         * @ORM\Table(name="product")
         */
        class Product
        {
            /**
             * @ORM\Id
             * @ORM\Column(type="integer")
             * @ORM\GeneratedValue(strategy="AUTO")
             */
            protected $id;

            /**
             * @ORM\Column(type="string", length=100)
             */
            protected $name;

            /**
             * @ORM\Column(type="decimal", scale=2)
             */
            protected $price;

            /**
             * @ORM\Column(type="text")
             */
            protected $description;
        }

    .. code-block:: yaml

        # src/Acme/StoreBundle/Resources/config/doctrine/Product.orm.yml
        Acme\StoreBundle\Entity\Product:
            type: entity
            table: product
            id:
                id:
                    type: integer
                    generator: { strategy: AUTO }
            fields:
                name:
                    type: string
                    length: 100
                price:
                    type: decimal
                    scale: 2
                description:
                    type: text

    .. code-block:: xml

        <!-- src/Acme/StoreBundle/Resources/config/doctrine/Product.orm.xml -->
        <doctrine-mapping xmlns="http://doctrine-project.org/schemas/orm/doctrine-mapping"
              xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
              xsi:schemaLocation="http://doctrine-project.org/schemas/orm/doctrine-mapping
                            http://doctrine-project.org/schemas/orm/doctrine-mapping.xsd">

            <entity name="Acme\StoreBundle\Entity\Product" table="product">
                <id name="id" type="integer" column="id">
                    <generator strategy="AUTO" />
                </id>
                <field name="name" column="name" type="string" length="100" />
                <field name="price" column="price" type="decimal" scale="2" />
                <field name="description" column="description" type="text" />
            </entity>
        </doctrine-mapping>

.. tip::

    The table name is optional and if omitted, will be determined automatically
    based on the name of the entity class.
    是否要填写表的名字是可选的，如果不填，系统会自动根据实体类来决定表名。

Doctrine allows you to choose from a wide variety of different field types,
each with their own options. For information on the available field types,
see the :ref:`book-doctrine-field-types` section.
doctrine允许你选择多种字段类型，每个类型都有它自己的选项。详情请见:ref:`book-doctrine-field-types`。

.. seealso::

    You can also check out Doctrine's `Basic Mapping Documentation`_ for
    all details about mapping information. If you use annotations, you'll
    need to prepend all annotations with ``ORM\`` (e.g. ``ORM\Column(..)``),
    which is not shown in Doctrine's documentation. You'll also need to include
    the ``use Doctrine\ORM\Mapping as ORM;`` statement, which *imports* the
    ``ORM`` annotations prefix.
    你还可以查看doctrine的`Basic Mapping Documentation`_ 以获得更详细的信息。如果你
    使用注释，你必须在所有注释的前面加上``ORM\`` (e.g. ``ORM\Column(..)``)，这个在doctrine
    自己的文档里是没有的。你还必须加上``use Doctrine\ORM\Mapping as ORM;``语句，这条语句
    会*导入*``ORM``注释前缀。

.. caution::

    Be careful that your class name and properties aren't mapped to a protected
    SQL keyword (such as ``group`` or ``user``). For example, if your entity
    class name is ``Group``, then, by default, your table name will be ``group``,
    which will cause an SQL error in some engines. See Doctrine's
    `Reserved SQL keywords documentation`_ on how to properly escape these
    names.
    注意，你的类名和类的属性（property）不能映射到SQL关键字。比如，如果你的实体类名称是
    ``Group``，默认的情况下，你的表名会是``group``，这会导致一个SQL错误。关于如何避免这些
    命名，详情请见`Reserved SQL keywords documentation`_。

.. note::

    When using another library or program (ie. Doxygen) that uses annotations,
    you should place the ``@IgnoreAnnotation`` annotation on the class to
    indicate which annotations Symfony should ignore.
    如果你要使用其他要用到注释的库或program（ie. Doxygen），你必须将注释``@IgnoreAnnotation``
    放到类中，以表明哪些注释symfony应当忽略。

    For example, to prevent the ``@fn`` annotation from throwing an exception,
    add the following::
    比如，要避免注释``@fn``抛出一个错误，加上下面的代码::

        /**
         * @IgnoreAnnotation("fn")
         */
        class Product

Generating Getters and Setters
集成Getter和Setter
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Even though Doctrine now knows how to persist a ``Product`` object to the
database, the class itself isn't really useful yet. Since ``Product`` is just
a regular PHP class, you need to create getter and setter methods (e.g. ``getName()``,
``setName()``) in order to access its properties (since the properties are
``protected``). Fortunately, Doctrine can do this for you by running:
尽管现在doctrine已经知道如何将``Product``类载入数据库，这个类现在还没有用。它还只是一个
简单的PHP类，你要给它加上getter和setter方法(e.g. ``getName()``,``setName()``)，这样才能使用它的
属性（property）（由于这些属性是被``protected``的）。要达到这个目的，可以这样::

.. code-block:: bash

    php app/console doctrine:generate:entities Acme/StoreBundle/Entity/Product

This command makes sure that all of the getters and setters are generated
for the ``Product`` class. This is a safe command - you can run it over and
over again: it only generates getters and setters that don't exist (i.e. it
doesn't replace your existing methods).
该命令行保证所有的getter和setter都在``Product``类中被集成。这个命令你可以运行多次，它只会集成
没有被集成的getter和setter（不会替换你已经集成的方法）。

.. sidebar:: More about ``doctrine:generate:entities``

    With the ``doctrine:generate:entities`` command you can:
    通过``doctrine:generate:entities``命令你可以:

        * generate getters and setters,
        * 集成getter和setter

        * generate repository classes configured with the
            ``@ORM\Entity(repositoryClass="...")`` annotation,
        * 集成由注释``@ORM\Entity(repositoryClass="...")``配置的repository类

        * generate the appropriate constructor for 1:n and n:m relations.
        * 集成一对多和多对多关系

    The ``doctrine:generate:entities`` command saves a backup of the original
    ``Product.php`` named ``Product.php~``. In some cases, the presence of
    this file can cause a "Cannot redeclare class" error. It can be safely
    removed.
    ``doctrine:generate:entities``命令会保存一个原文件``Product.php``的备份，这个备份
    文件名为``Product.php~``。在某些情况下，这个备份文件可能会产生一个"Cannot redeclare class"
    错误。这时你可以移除它。

    Note that you don't *need* to use this command. Doctrine doesn't rely
    on code generation. Like with normal PHP classes, you just need to make
    sure that your protected/private properties have getter and setter methods.
    Since this is a common thing to do when using Doctrine, this command
    was created.
    注意你不是必须要用到这个命令。doctrine并不依赖于代码集成。像普通PHP类一样，你只要保证
    你的 protected/private 属性有getter和setter方法就可以了。但一般都会使用这个命令。

You can also generate all known entities (i.e. any PHP class with Doctrine
mapping information) of a bundle or an entire namespace:
你还可以集成所有bundle中或者整个命名空间中已知的实体类（或者说任何有着doctrine映射信息的PHP类）:

.. code-block:: bash

    php app/console doctrine:generate:entities AcmeStoreBundle
    php app/console doctrine:generate:entities Acme

.. note::

    Doctrine doesn't care whether your properties are ``protected`` or ``private``,
    or whether or not you have a getter or setter function for a property.
    The getters and setters are generated here only because you'll need them
    to interact with your PHP object.
    doctrine并不关心你的类的属性是否是``protected``或者是``private``，或者你有没有getter和setter
    方法。getter和setter方法只是用来与你的PHP类进行交互的。

Creating the Database Tables/Schema
创建数据库表
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

You now have a usable ``Product`` class with mapping information so that
Doctrine knows exactly how to persist it. Of course, you don't yet have the
corresponding ``product`` table in your database. Fortunately, Doctrine can
automatically create all the database tables needed for every known entity
in your application. To do this, run:
你现在有了一个可用的``Product``类，这个类有映射信息。当然，现在你的数据库中还没有
一个对应的``product``表。doctrine可以自动对任何实体类生成数据库表。执行以下命令:

.. code-block:: bash

    php app/console doctrine:schema:update --force

.. tip::

    Actually, this command is incredibly powerful. It compares what
    your database *should* look like (based on the mapping information of
    your entities) with how it *actually* looks, and generates the SQL statements
    needed to *update* the database to where it should be. In other words, if you add
    a new property with mapping metadata to ``Product`` and run this task
    again, it will generate the "alter table" statement needed to add that
    new column to the existing ``product`` table.
    事实上，这个命令很强大。它比较你的数据库表*应该*是什么样子（在映射信息的基础上）
    以及*实际*是什么样子。换句话说，如果你在你的实体类的metadata中加上了一个新的属性并执行以上命令，
    那么doctrine会执行"alter table"语句，从而在数据库表中添加这个列。

    An even better way to take advantage of this functionality is via
    :doc:`migrations</bundles/DoctrineMigrationsBundle/index>`, which allow you to
    generate these SQL statements and store them in migration classes that
    can be run systematically on your production server in order to track
    and migrate your database schema safely and reliably.
    一个更好的利用这个功能的方法是通过:doc:`migrations</bundles/DoctrineMigrationsBundle/index>`,
    它可以允许你在migration类中集成并存储SQL语句，这些语句可以在你的服务器上被系统的执行，
    从而安全地监测和转移数据库的内容。

Your database now has a fully-functional ``product`` table with columns that
match the metadata you've specified.
你的数据库现在有了一个有着完整功能的``product``表，这个表具有匹配你配置的metadata信息的列。

Persisting Objects to the Database
将对象导入数据库
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Now that you have a mapped ``Product`` entity and corresponding ``product``
table, you're ready to persist data to the database. From inside a controller,
this is pretty easy. Add the following method to the ``DefaultController``
of the bundle:
现在你有了一个``Product``实体类和一个对应的``product``数据库表，你就可以将数据导入
数据库了。在控制器中实现这个功能很容易，只要在``DefaultController``中加上以下的代码:

.. code-block:: php
    :linenos:

    // src/Acme/StoreBundle/Controller/DefaultController.php
    use Acme\StoreBundle\Entity\Product;
    use Symfony\Component\HttpFoundation\Response;
    // ...
    
    public function createAction()
    {
        $product = new Product();
        $product->setName('A Foo Bar');
        $product->setPrice('19.99');
        $product->setDescription('Lorem ipsum dolor');

        $em = $this->getDoctrine()->getManager();
        $em->persist($product);
        $em->flush();

        return new Response('Created product id '.$product->getId());
    }

.. note::

    If you're following along with this example, you'll need to create a
    route that points to this action to see it work.
    如果你按照这个例子做，你必须创建一个指向控制器的路径。

Let's walk through this example:
下面阐述以下这个例子:

* **lines 8-11** In this section, you instantiate and work with the ``$product``
  object like any other, normal PHP object;
* **第8-11行** 将``$product``对象实例化，就如同其他PHP对象一样;

* **line 13** This line fetches Doctrine's *entity manager* object, which is
  responsible for handling the process of persisting and fetching objects
  to and from the database;
* **第13行** 这一行获取doctrine的*entity manager*对象，这个对象可以帮助你从数据库插入和
  获取数据;
* **line 14** The ``persist()`` method tells Doctrine to "manage" the ``$product``
  object. This does not actually cause a query to be made to the database (yet).
* **第14行** ``persist()``方法告诉doctrine"处理"这个``$product``对象。这暂时还不会让数据库执行
  一个query。  

* **line 15** When the ``flush()`` method is called, Doctrine looks through
  all of the objects that it's managing to see if they need to be persisted
  to the database. In this example, the ``$product`` object has not been
  persisted yet, so the entity manager executes an ``INSERT`` query and a
  row is created in the ``product`` table.
* **第15行** 当执行``flush()``方法时，doctrine查看它正在处理的所有的对象，看它们是否
  需要被导入数据库。在这个例子中，``$product``对象还未被导入数据库，所以这个entity manager
  执行了一个``INSERT``命令，数据库表``product``中新的一行被添加。

.. note::

  In fact, since Doctrine is aware of all your managed entities, when you
  call the ``flush()`` method, it calculates an overall changeset and executes
  the most efficient query/queries possible. For example, if you persist a
  total of 100 ``Product`` objects and then subsequently call ``flush()``, 
  Doctrine will create a *single* prepared statement and re-use it for each 
  insert. This pattern is called *Unit of Work*, and it's used because it's 
  fast and efficient.
  事实上，因为dictrine知道所有你处理的实体类，当你执行``flush()``命令的时候，它计算
  出所有的改变值并执行最有效率的命令。比如，如果你导入100个``Product``对象然后执行``flush()``，
  doctrine会创建一个*单独的*语句并对每个对象重复运用。这个工作方式被称作*Unit of Work*，并且
  十分迅速高效。
  

When creating or updating objects, the workflow is always the same. In the
next section, you'll see how Doctrine is smart enough to automatically issue
an ``UPDATE`` query if the record already exists in the database.
当创建和更新对象时，工作流程是一样的。下一节你将看到当某个记录已经存在时，
doctrine是如何灵活地创建``UPDATE``命令的。

.. tip::

    Doctrine provides a library that allows you to programmatically load testing
    data into your project (i.e. "fixture data"). For information, see
    :doc:`/bundles/DoctrineFixturesBundle/index`.
    doctrine提供一个库，这个库可以帮助你在测试时将数据自动载入数据库（或者说，"fixture data"）。详情请见
    :doc:`/bundles/DoctrineFixturesBundle/index`。

Fetching Objects from the Database
从数据库取数据
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Fetching an object back out of the database is even easier. For example,
suppose you've configured a route to display a specific ``Product`` based
on its ``id`` value::
从数据库中取出对象更简单。比如，假设你已经配置了一个路径来显示一个特定的基于``id``值的``Product``::

    public function showAction($id)
    {
        $product = $this->getDoctrine()
            ->getRepository('AcmeStoreBundle:Product')
            ->find($id);
        
        if (!$product) {
            throw $this->createNotFoundException('No product found for id '.$id);
        }

        // do something, like pass the $product object into a template
    }

When you query for a particular type of object, you always use what's known
as its "repository". You can think of a repository as a PHP class whose only
job is to help you fetch entities of a certain class. You can access the
repository object for an entity class via::
当你请求一个特定类型的对象时，你可以使用它的"存储库（repository）"。你可以把存储库想象成一个PHP类，它的
唯一工作就是帮助你获取某个类的实体。你可以通过以下方法获取实体类的存储库类::

    $repository = $this->getDoctrine()
        ->getRepository('AcmeStoreBundle:Product');

.. note::

    The ``AcmeStoreBundle:Product`` string is a shortcut you can use anywhere
    in Doctrine instead of the full class name of the entity (i.e. ``Acme\StoreBundle\Entity\Product``).
    As long as your entity lives under the ``Entity`` namespace of your bundle,
    this will work.
    ``AcmeStoreBundle:Product``语句是一个便捷方式，你可以在doctrine的任何地方运用它，而
    不需要写整个路径名（如``Acme\StoreBundle\Entity\Product``）。只要你的实体在你的bundle的``Entity``
    命名空间中就行。

Once you have your repository, you have access to all sorts of helpful methods::
一旦你有了自己的存储库，你就可以运用许多有用的方法了::

    // query by the primary key (usually "id")
    $product = $repository->find($id);

    // dynamic method names to find based on a column value
    $product = $repository->findOneById($id);
    $product = $repository->findOneByName('foo');

    // find *all* products
    $products = $repository->findAll();

    // find a group of products based on an arbitrary column value
    $products = $repository->findByPrice(19.99);

.. note::

    Of course, you can also issue complex queries, which you'll learn more
    about in the :ref:`book-doctrine-queries` section.
    当然你还可以用更复杂的请求，学习更多相关知识请参阅:ref:`book-doctrine-queries`。

You can also take advantage of the useful ``findBy`` and ``findOneBy`` methods
to easily fetch objects based on multiple conditions::
你还可以利用``findBy`` 和 ``findOneBy``方法来方便地根据不同的条件获取对象::

    // query for one product matching be name and price
    $product = $repository->findOneBy(array('name' => 'foo', 'price' => 19.99));

    // query for all products matching the name, ordered by price
    $product = $repository->findBy(
        array('name' => 'foo'),
        array('price' => 'ASC')
    );

.. tip::

    When you render any page, you can see how many queries were made in the
    bottom right corner of the web debug toolbar.
    当你提交页面时，你可以通过web debug工具条右下角查看有多少请求被创建。

    .. image:: /images/book/doctrine_web_debug_toolbar.png
       :align: center
       :scale: 50
       :width: 350

    If you click the icon, the profiler will open, showing you the exact
    queries that were made.
    如果你单击这个图标，debug页面会打开，展示被创建的请求。

Updating an Object
更新一个对象
~~~~~~~~~~~~~~~~~~

Once you've fetched an object from Doctrine, updating it is easy. Suppose
you have a route that maps a product id to an update action in a controller::
一旦你已经从doctrine中获取了对象，你就能很方便地更新它。假设你已经有了一个将product的id
映射到控制器updateAction的路径::

    public function updateAction($id)
    {
        $em = $this->getDoctrine()->getManager();
        $product = $em->getRepository('AcmeStoreBundle:Product')->find($id);

        if (!$product) {
            throw $this->createNotFoundException('No product found for id '.$id);
        }

        $product->setName('New product name!');
        $em->flush();

        return $this->redirect($this->generateUrl('homepage'));
    }

Updating an object involves just three steps:
更新一个对象涉及到三个步骤:

1. fetching the object from Doctrine;
2. modifying the object;
3. calling ``flush()`` on the entity manager
1. 从doctrine中获取对象;
2. 更新这个对象;
3. 在entity manager中执行``flush()`` 

Notice that calling ``$em->persist($product)`` isn't necessary. Recall that
this method simply tells Doctrine to manage or "watch" the ``$product`` object.
In this case, since you fetched the ``$product`` object from Doctrine, it's
already managed.
注意执行``$em->persist($product)``并不是必须的。这个方法仅仅是告诉doctrine处理或者
监测这个``$product``对象。在这个例子中，因为你已经从doctrine中取出了``$product``
对象，它已经被处理了。

Deleting an Object
删除一个对象
~~~~~~~~~~~~~~~~~~

Deleting an object is very similar, but requires a call to the ``remove()``
method of the entity manager::
删除一个对象也很相似，但是需要执行entity manager中的``remove()``方法::

    $em->remove($product);
    $em->flush();

As you might expect, the ``remove()`` method notifies Doctrine that you'd
like to remove the given entity from the database. The actual ``DELETE`` query,
however, isn't actually executed until the ``flush()`` method is called.
``remove()``方法告诉doctrine你想要从数据库中删除这个实体。但是，在执行``flush()``
方法之前，这个实际的数据库请求``DELETE``还没有被执行。

.. _`book-doctrine-queries`:

Querying for Objects
对象请求
--------------------

You've already seen how the repository object allows you to run basic queries
without any work::
你已经知道了存储库（repository）对象允许你执行基本命令::

    $repository->find($id);
    
    $repository->findOneByName('Foo');

Of course, Doctrine also allows you to write more complex queries using the
Doctrine Query Language (DQL). DQL is similar to SQL except that you should
imagine that you're querying for one or more objects of an entity class (e.g. ``Product``)
instead of querying for rows on a table (e.g. ``product``).
当然，doctrine也允许你使用doctrine请求语言（Doctrine Query Language (DQL)）编写更复杂的
请求。DQL与SQL很相似，但DQL是向一个实体类（e.g. ``Product``）请求一个或更多个对象，而SQL
是向数据库请求，并返回表中的行。

When querying in Doctrine, you have two options: writing pure Doctrine queries
or using Doctrine's Query Builder.
当在doctrine中发送请求的时候，你有两个选择:编写纯粹的doctrine请求或者使用doctrine的Query Builder。

Querying for Objects with DQL
使用DQL请求对象
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Imagine that you want to query for products, but only return products that
cost more than ``19.99``, ordered from cheapest to most expensive. From inside
a controller, do the following::
比如，你想要请求product，但只想返回价钱比``19.99``更多的product，并将它们从最便宜到最贵排序。
在控制器里，可以这样::

    $em = $this->getDoctrine()->getManager();
    $query = $em->createQuery(
        'SELECT p FROM AcmeStoreBundle:Product p WHERE p.price > :price ORDER BY p.price ASC'
    )->setParameter('price', '19.99');
    
    $products = $query->getResult();

If you're comfortable with SQL, then DQL should feel very natural. The biggest
difference is that you need to think in terms of "objects" instead of rows
in a database. For this reason, you select *from* ``AcmeStoreBundle:Product``
and then alias it as ``p``.
如果你已经习惯用SQL，那么DQL会让你觉得非常自然。它们之间最大的不同就是你必须将数据库
表中的行想象成对象。所以，你从``AcmeStoreBundle:Product``中选择product并设置它的别名为``p``。

The ``getResult()`` method returns an array of results. If you're querying
for just one object, you can use the ``getSingleResult()`` method instead::
``getResult()``方法返回的是一个结果的数组。如果你只想取一个对象，你可以使用``getSingleResult()``方法代替::

    $product = $query->getSingleResult();

.. caution::

    The ``getSingleResult()`` method throws a ``Doctrine\ORM\NoResultException``
    exception if no results are returned and a ``Doctrine\ORM\NonUniqueResultException``
    if *more* than one result is returned. If you use this method, you may
    need to wrap it in a try-catch block and ensure that only one result is
    returned (if you're querying on something that could feasibly return
    more than one result)::
    如果没有结果返回，``getSingleResult()``方法抛出一个``Doctrine\ORM\NoResultException``错误;
    如果有多个结果返回，它会抛出``Doctrine\ORM\NonUniqueResultException``。如果你要用这个方法，你可能需要
    将它打包在try-catch代码块中（如果你要请求一个可能返回的结果有多种的）::
    
        $query = $em->createQuery('SELECT ....')
            ->setMaxResults(1);
        
        try {
            $product = $query->getSingleResult();
        } catch (\Doctrine\Orm\NoResultException $e) {
            $product = null;
        }
        // ...

The DQL syntax is incredibly powerful, allowing you to easily join between
entities (the topic of :ref:`relations<book-doctrine-relations>` will be
covered later), group, etc. For more information, see the official Doctrine
`Doctrine Query Language`_ documentation.
DQL语法很强大，它允许你将多个实体和组结合起来（:ref:`relations<book-doctrine-relations>`这个话题下面会提到）。
要了解更多，请参阅doctrine官方文档`Doctrine Query Language`_。

.. sidebar:: Setting Parameters

    Take note of the ``setParameter()`` method. When working with Doctrine,
    it's always a good idea to set any external values as "placeholders",
    which was done in the above query:
    注意``setParameter()``方法。当使用doctrine是，最好使用外部值作为"placeholders"，
    像上例做的那样:
    
    
    .. code-block:: text

        ... WHERE p.price > :price ...

    You can then set the value of the ``price`` placeholder by calling the
    ``setParameter()`` method::
    你可以通过``setParameter()``来设置``price``placeholder的值::

        ->setParameter('price', '19.99')

    Using parameters instead of placing values directly in the query string
    is done to prevent SQL injection attacks and should *always* be done.
    If you're using multiple parameters, you can set their values at once
    using the ``setParameters()`` method::
    使用参数而不是直接将值放在请求语句中是为了防范SQL入侵，你必须这样做。
    如果你要使用多个参数，你可以用``setParameters()``设置它们的值::

        ->setParameters(array(
            'price' => '19.99',
            'name'  => 'Foo',
        ))

Using Doctrine's Query Builder
使用doctrine的Query Builder
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Instead of writing the queries directly, you can alternatively use Doctrine's
``QueryBuilder`` to do the same job using a nice, object-oriented interface.
If you use an IDE, you can also take advantage of auto-completion as you
type the method names. From inside a controller::
你也可以使用doctrine的``QueryBuilder``来编写请求。这样的编写方式更加的面向对象化。
如果你使用IDE，你还可以在输入方法名称的时候自动完成。在控制器中::

    $repository = $this->getDoctrine()
        ->getRepository('AcmeStoreBundle:Product');

    $query = $repository->createQueryBuilder('p')
        ->where('p.price > :price')
        ->setParameter('price', '19.99')
        ->orderBy('p.price', 'ASC')
        ->getQuery();
    
    $products = $query->getResult();

The ``QueryBuilder`` object contains every method necessary to build your
query. By calling the ``getQuery()`` method, the query builder returns a
normal ``Query`` object, which is the same object you built directly in the
previous section.
``QueryBuilder``对象中包含了所有编写请求所需要的方法。通过执行``getQuery()``，
query builder返回一个``Query``对象，这个对象与你在上一节中直接建立的对象是一个对象。

For more information on Doctrine's Query Builder, consult Doctrine's
`Query Builder`_ documentation.
要了解更多doctrine的Query Builder，请参阅`Query Builder`_。

Custom Repository Classes
定制存储类
~~~~~~~~~~~~~~~~~~~~~~~~~

In the previous sections, you began constructing and using more complex queries
from inside a controller. In order to isolate, test and reuse these queries,
it's a good idea to create a custom repository class for your entity and
add methods with your query logic there.
在前面章节中，你已经开始在控制器中使用更为复杂的请求了。但如果要分离、测试或重复运用这些
请求，最好为你的实体类创建一个定制的存储类（repository）并加入你的请求逻辑。

To do this, add the name of the repository class to your mapping definition.
要达到这个目的，在你的映射定义中加入存储类的名字。

.. configuration-block::

    .. code-block:: php-annotations

        // src/Acme/StoreBundle/Entity/Product.php
        namespace Acme\StoreBundle\Entity;

        use Doctrine\ORM\Mapping as ORM;

        /**
         * @ORM\Entity(repositoryClass="Acme\StoreBundle\Repository\ProductRepository")
         */
        class Product
        {
            //...
        }

    .. code-block:: yaml

        # src/Acme/StoreBundle/Resources/config/doctrine/Product.orm.yml
        Acme\StoreBundle\Entity\Product:
            type: entity
            repositoryClass: Acme\StoreBundle\Repository\ProductRepository
            # ...

    .. code-block:: xml

        <!-- src/Acme/StoreBundle/Resources/config/doctrine/Product.orm.xml -->
        <!-- ... -->
        <doctrine-mapping>

            <entity name="Acme\StoreBundle\Entity\Product"
                    repository-class="Acme\StoreBundle\Repository\ProductRepository">
                    <!-- ... -->
            </entity>
        </doctrine-mapping>

Doctrine can generate the repository class for you by running the same command
used earlier to generate the missing getter and setter methods:
doctrine能够通过命令行集成存储类:

.. code-block:: bash

    php app/console doctrine:generate:entities Acme

Next, add a new method - ``findAllOrderedByName()`` - to the newly generated
repository class. This method will query for all of the ``Product`` entities,
ordered alphabetically.
然后，在新集成的存储类中加一个新方法``findAllOrderedByName()``。这个方法会请求所有``Product``
实体类并根据字母排序。

.. code-block:: php

    // src/Acme/StoreBundle/Repository/ProductRepository.php
    namespace Acme\StoreBundle\Repository;

    use Doctrine\ORM\EntityRepository;

    class ProductRepository extends EntityRepository
    {
        public function findAllOrderedByName()
        {
            return $this->getManager()
                ->createQuery('SELECT p FROM AcmeStoreBundle:Product p ORDER BY p.name ASC')
                ->getResult();
        }
    }

.. tip::

    The entity manager can be accessed via ``$this->getManager()``
    from inside the repository.
    这个entity manager可以从存储类中通过``$this->getManager()``获得。

You can use this new method just like the default finder methods of the repository::
你可以像存储类默认的finder方法一样使用这个新方法:

    $em = $this->getDoctrine()->getManager();
    $products = $em->getRepository('AcmeStoreBundle:Product')
                ->findAllOrderedByName();

.. note::

    When using a custom repository class, you still have access to the default
    finder methods such as ``find()`` and ``findAll()``.
    当使用定制的存储类时，你依然可以使用默认的finder方法，如``find()``和``findAll()``。

.. _`book-doctrine-relations`:

Entity Relationships/Associations
实体关系
---------------------------------

Suppose that the products in your application all belong to exactly one "category".
In this case, you'll need a ``Category`` object and a way to relate a ``Product``
object to a ``Category`` object. Start by creating the ``Category`` entity.
Since you know that you'll eventually need to persist the class through Doctrine,
you can let Doctrine create the class for you.
假设你应用中的product都属于一个"category"，在这种情况下，你需要一个``Category``类以及一个方法将
``Product``类和``Category``类连接起来。首先要创建一个``Category``实体类。你可以让doctrine
为你创建这个类，因为你知道你最终会需要通过doctrine导入这个类。

.. code-block:: bash

    php app/console doctrine:generate:entity --entity="AcmeStoreBundle:Category" --fields="name:string(255)"

This task generates the ``Category`` entity for you, with an ``id`` field,
a ``name`` field and the associated getter and setter functions.
这个命令集成了``Category``实体类，它有一个``id``字段，一个``name``字段，还有关系型
getter和setter函数。

Relationship Mapping Metadata
关系映射metadata
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

To relate the ``Category`` and ``Product`` entities, start by creating a
``products`` property on the ``Category`` class:
要将``Category``和``Product``实体类联系起来，首先要在``Category``类中创建
``Product``属性:

.. configuration-block::

    .. code-block:: php-annotations

        // src/Acme/StoreBundle/Entity/Category.php
        // ...
        use Doctrine\Common\Collections\ArrayCollection;
        
        class Category
        {
            // ...
            
            /**
             * @ORM\OneToMany(targetEntity="Product", mappedBy="category")
             */
            protected $products;
    
            public function __construct()
            {
                $this->products = new ArrayCollection();
            }
        }

    .. code-block:: yaml

        # src/Acme/StoreBundle/Resources/config/doctrine/Category.orm.yml
        Acme\StoreBundle\Entity\Category:
            type: entity
            # ...
            oneToMany:
                products:
                    targetEntity: Product
                    mappedBy: category
            # don't forget to init the collection in entity __construct() method


First, since a ``Category`` object will relate to many ``Product`` objects,
a ``products`` array property is added to hold those ``Product`` objects.
Again, this isn't done because Doctrine needs it, but instead because it
makes sense in the application for each ``Category`` to hold an array of
``Product`` objects.
首先，由于一个``Category``类能够联系到很多``Product``类，一个``Product``数组属性被添加到``Category``
中来存储这些``Product``类。这样做不是因为doctrine需要它，而是因为``Category``类需要这样一个
数组来存储一系列的``Product``类。

.. note::

    The code in the ``__construct()`` method is important because Doctrine
    requires the ``$products`` property to be an ``ArrayCollection`` object.
    This object looks and acts almost *exactly* like an array, but has some
    added flexibility. If this makes you uncomfortable, don't worry. Just
    imagine that it's an ``array`` and you'll be in good shape.
    ``__construct()``中的代码很重要，因为doctrine需要``$products``属性作为一个``ArrayCollection``
    对象。这个对象的表现跟数组差不多，但是多了些灵活性。如果你不习惯，可以把它想象成一个数组就行了。

.. tip::

   The targetEntity value in the decorator used above can reference any entity
   with a valid namespace, not just entities defined in the same class. To 
   relate to an entity defined in a different class or bundle, enter a full
   namespace as the targetEntity.
   上面的targetEntity的值可以访问任何有着自己命名空间的实体类，而不仅仅是在相同的类中定义的
   实体。要关联到另一个类或者bundle定义的实体类，只要输入一个完整的命名空间就可以了。

Next, since each ``Product`` class can relate to exactly one ``Category``
object, you'll want to add a ``$category`` property to the ``Product`` class:
因为``Product``类可以关联到一个``Category``对象，应该向 ``Product``类添加
一个``Category``属性:

.. configuration-block::

    .. code-block:: php-annotations

        // src/Acme/StoreBundle/Entity/Product.php
        // ...
    
        class Product
        {
            // ...
        
            /**
             * @ORM\ManyToOne(targetEntity="Category", inversedBy="products")
             * @ORM\JoinColumn(name="category_id", referencedColumnName="id")
             */
            protected $category;
        }

    .. code-block:: yaml

        # src/Acme/StoreBundle/Resources/config/doctrine/Product.orm.yml
        Acme\StoreBundle\Entity\Product:
            type: entity
            # ...
            manyToOne:
                category:
                    targetEntity: Category
                    inversedBy: products
                    joinColumn:
                        name: category_id
                        referencedColumnName: id

Finally, now that you've added a new property to both the ``Category`` and
``Product`` classes, tell Doctrine to generate the missing getter and setter
methods for you:
现在你已经向``Category``类和``Product``类都添加了新的属性，那么就告诉doctrine集成getter
和setter方法:

.. code-block:: bash

    php app/console doctrine:generate:entities Acme

Ignore the Doctrine metadata for a moment. You now have two classes - ``Category``
and ``Product`` with a natural one-to-many relationship. The ``Category``
class holds an array of ``Product`` objects and the ``Product`` object can
hold one ``Category`` object. In other words - you've built your classes
in a way that makes sense for your needs. The fact that the data needs to
be persisted to a database is always secondary.
暂时忽略掉doctrine的metadata。现在你有两个类——``Category``和 ``Product``，这两个类是一对多的
关系。``Category``类具有包含了``Product``对象的数组，并且这个``Product``对象具有一个``Category``对象。
换句话说，你根据你自己的需要创建了一些类。至于要不要将数据载入数据库则是第二位的。

Now, look at the metadata above the ``$category`` property on the ``Product``
class. The information here tells doctrine that the related class is ``Category``
and that it should store the ``id`` of the category record on a ``category_id``
field that lives on the ``product`` table. In other words, the related ``Category``
object will be stored on the ``$category`` property, but behind the scenes,
Doctrine will persist this relationship by storing the category's id value
on a ``category_id`` column of the ``product`` table.
现在，看一下``Product``类中``$category``属性上方的metadata；这个信息高速doctrine相关联的类
是``Category``，并且它应当将category记录中的``id``存储到``product``表的``category_id``字段中。
换句话说，这个相关联的``Category``对象会被存储至``$category``属性中，但是在后台，doctrine
会将这个关联模式载入数据库，将category的id值存储到``product``表的``category_id``字段中。

.. image:: /images/book/doctrine_image_2.png
   :align: center

The metadata above the ``$products`` property of the ``Category`` object
is less important, and simply tells Doctrine to look at the ``Product.category``
property to figure out how the relationship is mapped.
``Category``对象中``$products``属性上方的metadata不是很重要，它只是告诉doctrine检查
``Product.category``属性，从而知道是如何关联的。

Before you continue, be sure to tell Doctrine to add the new ``category``
table, and ``product.category_id`` column, and new foreign key:
在继续之前，确保告诉doctrine添加新的``category``表、``product.category_id``字段、
以及新的foreign key:

.. code-block:: bash

    php app/console doctrine:schema:update --force

.. note::

    This task should only be really used during development. For a more robust
    method of systematically updating your production database, read about
    :doc:`Doctrine migrations</bundles/DoctrineMigrationsBundle/index>`.
    这个工作只能在开发（development）的时候才能被运用。要想有一个更稳健的方法来系统地更新你的生成（production）
    数据库，请参阅:doc:`Doctrine migrations</bundles/DoctrineMigrationsBundle/index>`。
    

Saving Related Entities
保存关联数据库
~~~~~~~~~~~~~~~~~~~~~~~

Now, let's see the code in action. Imagine you're inside a controller::
现在分析一下这个代码。设想你的控制器::

    // ...
    use Acme\StoreBundle\Entity\Category;
    use Acme\StoreBundle\Entity\Product;
    use Symfony\Component\HttpFoundation\Response;
    // ...

    class DefaultController extends Controller
    {
        public function createProductAction()
        {
            $category = new Category();
            $category->setName('Main Products');
            
            $product = new Product();
            $product->setName('Foo');
            $product->setPrice(19.99);
            // relate this product to the category
            $product->setCategory($category);
            
            $em = $this->getDoctrine()->getManager();
            $em->persist($category);
            $em->persist($product);
            $em->flush();
            
            return new Response(
                'Created product id: '.$product->getId().' and category id: '.$category->getId()
            );
        }
    }

Now, a single row is added to both the ``category`` and ``product`` tables.
The ``product.category_id`` column for the new product is set to whatever
the ``id`` is of the new category. Doctrine manages the persistence of this
relationship for you.
现在一条记录已经被添加到``category``和``product``表中。新的product的``product.category_id``列
的值被设置为新的category``id``列的值。doctrine处理了这个载入工作。

Fetching Related Objects
取出关联数据
~~~~~~~~~~~~~~~~~~~~~~~~

When you need to fetch associated objects, your workflow looks just like it
did before. First, fetch a ``$product`` object and then access its related
``Category``::
当你需要取出关联的对象时，你的工作流程和前面的差不多。首先，取一个``$product``对象
并取出相关联的``Category``::

    public function showAction($id)
    {
        $product = $this->getDoctrine()
            ->getRepository('AcmeStoreBundle:Product')
            ->find($id);

        $categoryName = $product->getCategory()->getName();
        
        // ...
    }

In this example, you first query for a ``Product`` object based on the product's
``id``. This issues a query for *just* the product data and hydrates the
``$product`` object with that data. Later, when you call ``$product->getCategory()->getName()``,
Doctrine silently makes a second query to find the ``Category`` that's related
to this ``Product``. It prepares the ``$category`` object and returns it to
you.
在这个例子中，你首先根据这个product的``id``请求了Product对象。这提交了一个要取出这个
product数据的请求并将得到的数据载入了那个``$product``对象。然后，当你执行``$product->getCategory()->getName()``
时，doctrine在后台发送了一个查找相关联的Category对象的请求。它将``$category``对象返回给你。

.. image:: /images/book/doctrine_image_3.png
   :align: center

What's important is the fact that you have easy access to the product's related
category, but the category data isn't actually retrieved until you ask for
the category (i.e. it's "lazily loaded").
重要的是，你很容易就能进入product的相关联category，但是这个category数据并没有真正被从数据库
取出来，直到你请求（这就是"lazy loaded"）。

You can also query in the other direction::

    public function showProductAction($id)
    {
        $category = $this->getDoctrine()
            ->getRepository('AcmeStoreBundle:Category')
            ->find($id);

        $products = $category->getProducts();
    
        // ...
    }

In this case, the same things occurs: you first query out for a single ``Category``
object, and then Doctrine makes a second query to retrieve the related ``Product``
objects, but only once/if you ask for them (i.e. when you call ``->getProducts()``).
The ``$products`` variable is an array of all ``Product`` objects that relate
to the given ``Category`` object via their ``category_id`` value.
在这个例子中也是一样:首先你请求了一个单独的Category对象，然后doctrine发送了第二个请求要
获得相关联的Product对象，但是仅仅/如果你请求（也就是当你执行``->getProducts()``的时候）。
这个$products变量是所有通过category_id关联到Category对象的product对象的集合（数组的形式）。

.. sidebar:: Relationships and Proxy Classes

    This "lazy loading" is possible because, when necessary, Doctrine returns
    a "proxy" object in place of the true object. Look again at the above
    example::
    "延迟加载（lazy loading）"是可能的，因为，当需要的时候，doctrine返回一个"代理(proxy)"对象
    来代替真正的对象。在以上例子中::
    
        $product = $this->getDoctrine()
            ->getRepository('AcmeStoreBundle:Product')
            ->find($id);

        $category = $product->getCategory();

        // prints "Proxies\AcmeStoreBundleEntityCategoryProxy"
        echo get_class($category);

    This proxy object extends the true ``Category`` object, and looks and
    acts exactly like it. The difference is that, by using a proxy object,
    Doctrine can delay querying for the real ``Category`` data until you
    actually need that data (e.g. until you call ``$category->getName()``).
    这个代理对象扩展了真实的category对象，并几乎像它一样工作。不同的是，通过代理
    对象，doctrine可以延迟请求获得category的数据直到你需要它（也就是当你执行``$category->getName()``时）。

    The proxy classes are generated by Doctrine and stored in the cache directory.
    And though you'll probably never even notice that your ``$category``
    object is actually a proxy object, it's important to keep in mind.
    这个代理对象被doctrine集成并存放在缓存目录中。尽管你可能永远都不会注意到你的$category对象
    实际上是一个代理对象，但记住它很重要。

    In the next section, when you retrieve the product and category data
    all at once (via a *join*), Doctrine will return the *true* ``Category``
    object, since nothing needs to be lazily loaded.
    下一节中，当你要同时获取product和category的数据时（通过join方法），doctrine会返回
    真正的category对象，因为没有东西需要延迟加载。

Joining to Related Records
联接相关记录
~~~~~~~~~~~~~~~~~~~~~~~~~~

In the above examples, two queries were made - one for the original object
(e.g. a ``Category``) and one for the related object(s) (e.g. the ``Product``
objects).
在上面的例子中，两个请求被发送——一个是来源对象的（category），一个是被关联对象的（product）。

.. tip::

    Remember that you can see all of the queries made during a request via
    the web debug toolbar.
    记住你可以在web debug工具条中看见所有被发送的请求。

Of course, if you know up front that you'll need to access both objects, you
can avoid the second query by issuing a join in the original query. Add the
following method to the ``ProductRepository`` class::
当然，如果你事先知道你需要获取两个对象，你可以通过join方法避免第二次请求。
将以下方法加入到``ProductRepository``类中::

    // src/Acme/StoreBundle/Repository/ProductRepository.php
    
    public function findOneByIdJoinedToCategory($id)
    {
        $query = $this->getManager()
            ->createQuery('
                SELECT p, c FROM AcmeStoreBundle:Product p
                JOIN p.category c
                WHERE p.id = :id'
            )->setParameter('id', $id);
        
        try {
            return $query->getSingleResult();
        } catch (\Doctrine\ORM\NoResultException $e) {
            return null;
        }
    }

Now, you can use this method in your controller to query for a ``Product``
object and its related ``Category`` with just one query::
现在你可以在你的控制器中使用这个方法，从而通过一个请求就可以请求product对象和它的
关联的category对象::


    public function showAction($id)
    {
        $product = $this->getDoctrine()
            ->getRepository('AcmeStoreBundle:Product')
            ->findOneByIdJoinedToCategory($id);

        $category = $product->getCategory();
    
        // ...
    }    

More Information on Associations
关于关联的更多信息
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

This section has been an introduction to one common type of entity relationship,
the one-to-many relationship. For more advanced details and examples of how
to use other types of relations (e.g. ``one-to-one``, ``many-to-many``), see
Doctrine's `Association Mapping Documentation`_.
这个章节是一个对实体类关联类型（一对多）的简单介绍。要了解更多（如一对一、多对多），请
参阅doctrine的文档`Association Mapping Documentation`_。

.. note::

    If you're using annotations, you'll need to prepend all annotations with
    ``ORM\`` (e.g. ``ORM\OneToMany``), which is not reflected in Doctrine's
    documentation. You'll also need to include the ``use Doctrine\ORM\Mapping as ORM;``
    statement, which *imports* the ``ORM`` annotations prefix.
    如果你要使用注释，你必须为所有注释都加上``ORM\`` (e.g. ``ORM\OneToMany``)前缀，这在doctrine文档中
    是没有叙述的。你还必须包含``use Doctrine\ORM\Mapping as ORM;``这个语句，它能够导入ORM注释前缀。

Configuration
配置
-------------

Doctrine is highly configurable, though you probably won't ever need to worry
about most of its options. To find out more about configuring Doctrine, see
the Doctrine section of the :doc:`reference manual</reference/configuration/doctrine>`.
doctrine很容易配置，虽然你可能永远不会更改它的选项。要了解更多如何配置doctrine的信息，请参阅
doctrine文档:doc:`reference manual</reference/configuration/doctrine>`。

Lifecycle Callbacks
lifecycle callbacks
-------------------

Sometimes, you need to perform an action right before or after an entity
is inserted, updated, or deleted. These types of actions are known as "lifecycle"
callbacks, as they're callback methods that you need to execute during different
stages of the lifecycle of an entity (e.g. the entity is inserted, updated,
deleted, etc).
有时候你需要在一个实体类被在数据库插入、更新或删除后马上执行一个方法，这些方法被称作
lifecycle callback，因为它们是callback方法，你必须在一个实体类的不同生命周期中执行它
（比如当这个实体被插入、更新或删除时）。

If you're using annotations for your metadata, start by enabling the lifecycle
callbacks. This is not necessary if you're using YAML or XML for your mapping:
如果你使用注释描述metadata，你首先要插入lifecycle callback的注释语句。如果你用的是YAML或XML
就不必了:

.. code-block:: php-annotations

    /**
     * @ORM\Entity()
     * @ORM\HasLifecycleCallbacks()
     */
    class Product
    {
        // ...
    }

Now, you can tell Doctrine to execute a method on any of the available lifecycle
events. For example, suppose you want to set a ``created`` date column to
the current date, only when the entity is first persisted (i.e. inserted):
现在你可以告诉doctrine对任何可用的lifecycle事件执行方法。比如，假设你想要创建一个
created数据列，里面存放当前的日期，只在当这个实体类被第一次载入时（就是被插入）:


.. configuration-block::

    .. code-block:: php-annotations

        /**
         * @ORM\PrePersist
         */
        public function setCreatedValue()
        {
            $this->created = new \DateTime();
        }

    .. code-block:: yaml

        # src/Acme/StoreBundle/Resources/config/doctrine/Product.orm.yml
        Acme\StoreBundle\Entity\Product:
            type: entity
            # ...
            lifecycleCallbacks:
                prePersist: [ setCreatedValue ]

    .. code-block:: xml

        <!-- src/Acme/StoreBundle/Resources/config/doctrine/Product.orm.xml -->
        <!-- ... -->
        <doctrine-mapping>

            <entity name="Acme\StoreBundle\Entity\Product">
                    <!-- ... -->
                    <lifecycle-callbacks>
                        <lifecycle-callback type="prePersist" method="setCreatedValue" />
                    </lifecycle-callbacks>
            </entity>
        </doctrine-mapping>

.. note::

    The above example assumes that you've created and mapped a ``created``
    property (not shown here).
    以上范例假设你已经创建并映射了一个created属性（这里没有写出来）。

Now, right before the entity is first persisted, Doctrine will automatically
call this method and the ``created`` field will be set to the current date.
现在，在这个实体类被第一次载入之前，doctrine会自动执行这个方法，当前日期
会被插入created字段。

This can be repeated for any of the other lifecycle events, which include:
对于其他lifecycle事件都可以这样做，包括:

* ``preRemove``
* ``postRemove``
* ``prePersist``
* ``postPersist``
* ``preUpdate``
* ``postUpdate``
* ``postLoad``
* ``loadClassMetadata``

For more information on what these lifecycle events mean and lifecycle callbacks
in general, see Doctrine's `Lifecycle Events documentation`_
要了解更多有关lifecycle事件信息，请参阅doctrine文档`Lifecycle Events documentation`_。

.. sidebar:: Lifecycle Callbacks and Event Listeners

    Notice that the ``setCreatedValue()`` method receives no arguments. This
    is always the case for lifecycle callbacks and is intentional: lifecycle
    callbacks should be simple methods that are concerned with internally
    transforming data in the entity (e.g. setting a created/updated field,
    generating a slug value).
    注意这个``setCreatedValue()``方法不接受参数。lifecycle callback都不接受参数:
    lifecycle callback应该是实体内部转换数据的简单方法，所以不应该有参数（比如创建一个created
    /updated字段，集成一个slug值）。
    
    If you need to do some heavier lifting - like perform logging or send
    an email - you should register an external class as an event listener
    or subscriber and give it access to whatever resources you need. For
    more information, see :doc:`/cookbook/doctrine/event_listeners_subscribers`.
    如果你想要做更多的事情——比如记录日志或发送邮件——你应该注册一个外部类来作为
    event listener或者subscriber，从而使它能够进入任何你需要的资源。请参阅:doc:`/cookbook/doctrine/event_listeners_subscribers`。

Doctrine Extensions: Timestampable, Sluggable, etc.
doctrine扩展: Timestampable, Sluggable,等等。
---------------------------------------------------

Doctrine is quite flexible, and a number of third-party extensions are available
that allow you to easily perform repeated and common tasks on your entities.
These include thing such as *Sluggable*, *Timestampable*, *Loggable*, *Translatable*,
and *Tree*.
doctrine十分灵活，并且它还有许多能让你更容易的进行重复和常见工作的第三方扩展。这些包括*Sluggable*, *Timestampable*, *Loggable*, *Translatable*,
, *Tree*。

For more information on how to find and use these extensions, see the cookbook
article about :doc:`using common Doctrine extensions</cookbook/doctrine/common_extensions>`.
要知道如何使用这些扩展，请参阅:doc:`using common Doctrine extensions</cookbook/doctrine/common_extensions>`。

.. _book-doctrine-field-types:

Doctrine Field Types Reference
doctrine字段类型索引
------------------------------

Doctrine comes with a large number of field types available. Each of these
maps a PHP data type to a specific column type in whatever database you're
using. The following types are supported in Doctrine:
doctrine有许多字段类型。每个字段类型都与任何你所使用的数据库的列类型相对应。以下是doctrine支持的一些类型:

* **Strings**

  * ``string`` (used for shorter strings)
  * ``text`` (used for larger strings)

* **Numbers**

  * ``integer``
  * ``smallint``
  * ``bigint``
  * ``decimal``
  * ``float``

* **Dates and Times** (use a `DateTime`_ object for these fields in PHP)

  * ``date``
  * ``time``
  * ``datetime``

* **Other Types**

  * ``boolean``
  * ``object`` (serialized and stored in a ``CLOB`` field)
  * ``array`` (serialized and stored in a ``CLOB`` field)

For more information, see Doctrine's `Mapping Types documentation`_.
更多信息请参阅`Mapping Types documentation`_。

Field Options
字段选项
~~~~~~~~~~~~~

Each field can have a set of options applied to it. The available options
include ``type`` (defaults to ``string``), ``name``, ``length``, ``unique``
and ``nullable``. Take a few examples:
每个字段都有一系列选项。这些选项包括``type`` (默认为 ``string``), ``name``, ``length``, ``unique``
and ``nullable``。

.. configuration-block::

    .. code-block:: php-annotations

        /**
         * A string field with length 255 that cannot be null
         * (reflecting the default values for the "type", "length" and *nullable* options)
         * 
         * @ORM\Column()
         */
        protected $name;
    
        /**
         * A string field of length 150 that persists to an "email_address" column
         * and has a unique index.
         *
         * @ORM\Column(name="email_address", unique=true, length=150)
         */
        protected $email;

    .. code-block:: yaml

        fields:
            # A string field length 255 that cannot be null
            # (reflecting the default values for the "length" and *nullable* options)
            # type attribute is necessary in yaml definitions
            name:
                type: string

            # A string field of length 150 that persists to an "email_address" column
            # and has a unique index.
            email:
                type: string
                column: email_address
                length: 150
                unique: true

.. note::

    There are a few more options not listed here. For more details, see
    Doctrine's `Property Mapping documentation`_

.. index::
   single: Doctrine; ORM Console Commands
   single: CLI; Doctrine ORM

Console Commands
控制台命令
----------------

The Doctrine2 ORM integration offers several console commands under the
``doctrine`` namespace. To view the command list you can run the console
without any arguments:
doctrine2 ORM在doctrine命名空间下提供了一些控制台命令。要查阅这些命令，你可以
运行以下命令:

.. code-block:: bash

    php app/console

A list of available command will print out, many of which start with the
``doctrine:`` prefix. You can find out more information about any of these
commands (or any Symfony command) by running the ``help`` command. For example,
to get details about the ``doctrine:database:create`` task, run:
一系列的可用命令会被输出，许多都有doctrine:前缀。你可以输入help命令来查找更多有关这些命令的
信息（或者其他symfony命令）。比如，如果你想要``doctrine:database:create``任务的详细信息，输入:

.. code-block:: bash

    php app/console help doctrine:database:create

Some notable or interesting tasks include:
一些有用的命令包括:

* ``doctrine:ensure-production-settings`` - checks to see if the current
  environment is configured efficiently for production. This should always
  be run in the ``prod`` environment:
* ``doctrine:ensure-production-settings``——检查当前的环境是否被配置为生成环境以及配置情况是否良好。这个命令
  必须在生成（prod）环境中运行:  
  
  .. code-block:: bash
  
    php app/console doctrine:ensure-production-settings --env=prod

* ``doctrine:mapping:import`` - allows Doctrine to introspect an existing
  database and create mapping information. For more information, see
  :doc:`/cookbook/doctrine/reverse_engineering`.
* ``doctrine:mapping:import`` ——允许doctrine监测内部数据库并创建映射信息。更多请参阅:doc:`/cookbook/doctrine/reverse_engineering`。

* ``doctrine:mapping:info`` - tells you all of the entities that Doctrine
  is aware of and whether or not there are any basic errors with the mapping.
* ``doctrine:mapping:info`` ——告诉你所有doctrine已知的实体类以及可能出现的基本映射错误。

* ``doctrine:query:dql`` and ``doctrine:query:sql`` - allow you to execute
  DQL or SQL queries directly from the command line.
* ``doctrine:query:dql``和``doctrine:query:sql``——允许你直接从命令行执行DQL或SQL请求。

.. note::

   To be able to load data fixtures to your database, you will need to have
   the ``DoctrineFixturesBundle`` bundle installed. To learn how to do it,
   read the ":doc:`/bundles/DoctrineFixturesBundle/index`" entry of the
   documentation.
   要想向数据库载入数据fixture，你必须安装``DoctrineFixturesBundle``这个bundle。请参阅":doc:`/bundles/DoctrineFixturesBundle/index`".

Summary
总结
-------

With Doctrine, you can focus on your objects and how they're useful in your
application and worry about database persistence second. This is because
Doctrine allows you to use any PHP object to hold your data and relies on
mapping metadata information to map an object's data to a particular database
table.
通过doctrine，你可以专注于如何使用你的实体类，并能够查看数据库操作所需时间。这是因为doctrine允许你使用PHP对象
来存储数据，并且这个对象可以被映射到数据库表中。

And even though Doctrine revolves around a simple concept, it's incredibly
powerful, allowing you to create complex queries and subscribe to events
that allow you to take different actions as objects go through their persistence
lifecycle.
尽管doctrine概念很简单，但它很强大。它允许你在对象的生命周期中创建复杂的请求并提交事件。

For more information about Doctrine, see the *Doctrine* section of the
:doc:`cookbook</cookbook/index>`, which includes the following articles:
更多关于doctrine的信息请参阅:doc:`cookbook</cookbook/index>`，该文档包含了以下文章:

* :doc:`/bundles/DoctrineFixturesBundle/index`
* :doc:`/cookbook/doctrine/common_extensions`

.. _`Doctrine`: http://www.doctrine-project.org/
.. _`MongoDB`: http://www.mongodb.org/
.. _`Basic Mapping Documentation`: http://docs.doctrine-project.org/projects/doctrine-orm/en/latest/reference/basic-mapping.html
.. _`Query Builder`: http://docs.doctrine-project.org/projects/doctrine-orm/en/latest/reference/query-builder.html
.. _`Doctrine Query Language`: http://docs.doctrine-project.org/projects/doctrine-orm/en/latest/reference/dql-doctrine-query-language.html
.. _`Association Mapping Documentation`: http://docs.doctrine-project.org/projects/doctrine-orm/en/latest/reference/association-mapping.html
.. _`DateTime`: http://php.net/manual/en/class.datetime.php
.. _`Mapping Types Documentation`: http://docs.doctrine-project.org/projects/doctrine-orm/en/latest/reference/basic-mapping.html#doctrine-mapping-types
.. _`Property Mapping documentation`: http://docs.doctrine-project.org/projects/doctrine-orm/en/latest/reference/basic-mapping.html#property-mapping
.. _`Lifecycle Events documentation`: http://docs.doctrine-project.org/projects/doctrine-orm/en/latest/reference/events.html#lifecycle-events
.. _`Reserved SQL keywords documentation`: http://docs.doctrine-project.org/projects/doctrine-orm/en/latest/reference/basic-mapping.html#quoting-reserved-words
