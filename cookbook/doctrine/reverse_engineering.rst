.. index::
   single: Doctrine; Generating entities from existing database

How to generate Entities from an Existing Database
如何从已存在的数据库中集成实体类
==================================================

When starting work on a brand new project that uses a database, two different
situations comes naturally. In most cases, the database model is designed
and built from scratch. Sometimes, however, you'll start with an existing and
probably unchangeable database model. Fortunately, Doctrine comes with a bunch
of tools to help generate model classes from your existing database.
当开始一个已有数据库的新项目时，有两种情况，大多数时候你要从头开始创建数据库，但是也有可能
要使用已有的数据库模型。doctrine有一些工具可以帮助你从已有数据库集成实体类。

.. note::

    As the `Doctrine tools documentation`_ says, reverse engineering is a
    one-time process to get started on a project. Doctrine is able to convert
    approximately 70-80% of the necessary mapping information based on fields,
    indexes and foreign key constraints. Doctrine can't discover inverse
    associations, inheritance types, entities with foreign keys as primary keys
    or semantical operations on associations such as cascade or lifecycle
    events. Some additional work on the generated entities will be necessary
    afterwards to design each to fit your domain model specificities.
    在`Doctrine tools documentation`_中讲到，反向工程（reverse engineering）是一个一次性过程。
    doctrine能够转换70-80%的映射信息，如字段、索引、外键规则（foreign key）。但它不能转换inverse关联、
    继承类型、以外键作为主键（primary key）的实体类、对关联表语义上的操作——如cascade和lifecycle等。
    要使创建的数据库完全符合已有的数据库模型，你还要做一些额外的工作。

This tutorial assumes you're using a simple blog application with the following
two tables: ``blog_post`` and ``blog_comment``. A comment record is linked
to a post record thanks to a foreign key constraint.
本章假设你已经使用了一个简单博客应用，该应用有两个表：blog_post和blog_comment。每个comment记录都使用外键规则被关联
到一个post记录。

.. code-block:: sql

    CREATE TABLE `blog_post` (
      `id` bigint(20) NOT NULL AUTO_INCREMENT,
      `title` varchar(100) COLLATE utf8_unicode_ci NOT NULL,
      `content` longtext COLLATE utf8_unicode_ci NOT NULL,
      `created_at` datetime NOT NULL,
      PRIMARY KEY (`id`),
    ) ENGINE=InnoDB AUTO_INCREMENT=1 DEFAULT CHARSET=utf8 COLLATE=utf8_unicode_ci;

    CREATE TABLE `blog_comment` (
      `id` bigint(20) NOT NULL AUTO_INCREMENT,
      `post_id` bigint(20) NOT NULL,
      `author` varchar(20) COLLATE utf8_unicode_ci NOT NULL,
      `content` longtext COLLATE utf8_unicode_ci NOT NULL,
      `created_at` datetime NOT NULL,
      PRIMARY KEY (`id`),
      KEY `blog_comment_post_id_idx` (`post_id`),
      CONSTRAINT `blog_post_id` FOREIGN KEY (`post_id`) REFERENCES `blog_post` (`id`) ON DELETE CASCADE
    ) ENGINE=InnoDB AUTO_INCREMENT=1 DEFAULT CHARSET=utf8 COLLATE=utf8_unicode_ci;

Before diving into the recipe, be sure your database connection parameters are
correctly setup in the ``app/config/parameters.yml`` file (or wherever your
database configuration is kept) and that you have initialized a bundle that
will host your future entity class. In this tutorial, we will assume that
an ``AcmeBlogBundle`` exists and is located under the ``src/Acme/BlogBundle``
folder.
在开始之前请确保你的数据库连接参数都在``app/config/parameters.yml``（或其他任何你自己的数据库配置位置中）中被正确地设置了，
并确保你已经初始化了一个存储实体类的bundle。在本例中，我们假设一个AcmeBlogBundle已经存在并被放置在``src/Acme/BlogBundle``中。

The first step towards building entity classes from an existing database
is to ask Doctrine to introspect the database and generate the corresponding
metadata files. Metadata files describe the entity class to generate based on
tables fields.
第一步就是要让doctrine检测已有的数据库并集成相应的元数据（matadata）文件。元数据文件根据
表的字段来描述要集成的实体类。

.. code-block:: bash

    php app/console doctrine:mapping:convert xml ./src/Acme/BlogBundle/Resources/config/doctrine/metadata/orm --from-database --force

This command line tool asks Doctrine to introspect the database and generate
the XML metadata files under the ``src/Acme/BlogBundle/Resources/config/doctrine/metadata/orm``
folder of your bundle.
命令行工具使doctrine能检测数据库并在你的bundle下的``src/Acme/BlogBundle/Resources/config/doctrine/metadata/orm``
目录中集成XML元数据文件。

.. tip::

    It's also possible to generate metadata class in YAML format by changing the
    first argument to `yml`.
    通过将第一个参数修改为yml可以使集成的元数据类以YAML的格式呈现。

The generated ``BlogPost.dcm.xml`` metadata file looks as follows:
被集成的``BlogPost.dcm.xml``元数据文件看起来会像是这样:

.. code-block:: xml

    <?xml version="1.0" encoding="utf-8"?>
    <doctrine-mapping>
      <entity name="BlogPost" table="blog_post">
        <change-tracking-policy>DEFERRED_IMPLICIT</change-tracking-policy>
        <id name="id" type="bigint" column="id">
          <generator strategy="IDENTITY"/>
        </id>
        <field name="title" type="string" column="title" length="100"/>
        <field name="content" type="text" column="content"/>
        <field name="isPublished" type="boolean" column="is_published"/>
        <field name="createdAt" type="datetime" column="created_at"/>
        <field name="updatedAt" type="datetime" column="updated_at"/>
        <field name="slug" type="string" column="slug" length="255"/>
        <lifecycle-callbacks/>
      </entity>
    </doctrine-mapping>

Once the metadata files are generated, you can ask Doctrine to import the
schema and build related entity classes by executing the following two commands.
一旦元数据文件被集成，你就可以让doctrine导入数据库结构并据此来创建实体类了。请执行以下两行命令。

.. code-block:: bash

    php app/console doctrine:mapping:import AcmeBlogBundle annotation
    php app/console doctrine:generate:entities AcmeBlogBundle

The first command generates entity classes with an annotations mapping, but
you can of course change the ``annotation`` argument to ``xml`` or ``yml``.
The newly created ``BlogComment`` entity class looks as follow:
第一个命令行会使用注释的方式来集成实体类映射，但你也可以将annotation参数修改为xml或yml。
新创建的BlogComment实体类会看起来像这样:

.. code-block:: php

    <?php

    // src/Acme/BlogBundle/Entity/BlogComment.php
    namespace Acme\BlogBundle\Entity;

    use Doctrine\ORM\Mapping as ORM;

    /**
     * Acme\BlogBundle\Entity\BlogComment
     *
     * @ORM\Table(name="blog_comment")
     * @ORM\Entity
     */
    class BlogComment
    {
        /**
         * @var bigint $id
         *
         * @ORM\Column(name="id", type="bigint", nullable=false)
         * @ORM\Id
         * @ORM\GeneratedValue(strategy="IDENTITY")
         */
        private $id;

        /**
         * @var string $author
         *
         * @ORM\Column(name="author", type="string", length=100, nullable=false)
         */
        private $author;

        /**
         * @var text $content
         *
         * @ORM\Column(name="content", type="text", nullable=false)
         */
        private $content;

        /**
         * @var datetime $createdAt
         *
         * @ORM\Column(name="created_at", type="datetime", nullable=false)
         */
        private $createdAt;

        /**
         * @var BlogPost
         *
         * @ORM\ManyToOne(targetEntity="BlogPost")
         * @ORM\JoinColumn(name="post_id", referencedColumnName="id")
         */
        private $post;
    }

As you can see, Doctrine converts all table fields to pure private and annotated
class properties. The most impressive thing is that it also discovered the
relationship with the ``BlogPost`` entity class based on the foreign key constraint.
Consequently, you can find a private ``$post`` property mapped with a ``BlogPost``
entity in the ``BlogComment`` entity class.
如你所见，doctrine会将所有表字段转换成单纯的private和具有注释的类属性。最引人注意的是它还根据外键规则来
发现与BlogPost实体类的关系。于是，这个私有的（private）$post属性会在BlogComment实体类中映射到BlogPost实体类中。

The last command generated all getters and setters for your two ``BlogPost`` and
``BlogComment`` entity class properties. The generated entities are now ready to be
used. Have fun!
后面那个命令行会为你的两个实体类（BlogPost和BlogComment）集成所有的getter和setter。现在你就能使用集成的实体类了。

.. _`Doctrine tools documentation`: http://docs.doctrine-project.org/projects/doctrine-orm/en/latest/reference/tools.html#reverse-engineering
