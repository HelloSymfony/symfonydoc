.. index::
   single: Security; User Provider
   single: Security; Entity Provider

How to load Security Users from the Database (the Entity Provider)
如何从数据库加载安全用户（实体提供方）
==================================================================

The security layer is one of the smartest tools of Symfony. It handles two
things: the authentication and the authorization processes. Although it may
seem difficult to understand how it works internally, the security system
is very flexible and allows you to integrate your application with any authentication
backend, like Active Directory, an OAuth server or a database.
安全层（the security layer）是symfony最为灵巧的工具之一。它做两件事情：
身份验证（authentication）和授权（authorization）。尽管不容易理解安全层的内部运行流程，
它很容易运用。你可以在你的代码中应用身份验证的后台，比如Active Directory，QAuth server 
或者数据库。


Introduction
介绍
------------

This article focuses on how to authenticate users against a database table
managed by a Doctrine entity class. The content of this cookbook entry is split
in three parts. The first part is about designing a Doctrine ``User`` entity
class and making it usable in the security layer of Symfony. The second part
describes how to easily authenticate a user with the Doctrine
:class:`Symfony\\Bridge\\Doctrine\\Security\\User\\EntityUserProvider` object
bundled with the framework and some configuration.
Finally, the tutorial will demonstrate how to create a custom
:class:`Symfony\\Bridge\\Doctrine\\Security\\User\\EntityUserProvider` object to
retrieve users from a database with custom conditions.
本文介绍如何用Doctrine实体类（entity class）和数据库来对用户进行身份验证。本文的
内容分三个部分：第一部分讲如何设置一个Doctrine User 实体类并将它运用到安全层；第二
部分讲如何使用Doctrine :class:`Symfony\\Bridge\\Doctrine\\Security\\User\\EntityUserProvider`
类来对用户进行身份验证；最后演示一个范例，表明如何创建一个定制的
:class:`Symfony\\Bridge\\Doctrine\\Security\\User\\EntityUserProvider`
类，从而可以用你自己定制的方法进行用户的身份验证。

This tutorial assumes there is a bootstrapped and loaded
``Acme\UserBundle`` bundle in the application kernel.
本文假设你已经设置了一个Acme\UserBundle bundle。

The Data Model
数据模型
--------------

For the purpose of this cookbook, the ``AcmeUserBundle`` bundle contains a
``User`` entity class with the following fields: ``id``, ``username``, ``salt``,
``password``, ``email`` and ``isActive``. The ``isActive`` field tells whether
or not the user account is active.
为了方便论述，本文假设这个``AcmeUserBundle`` bundle 包含了一个``User`` 实体类，这个类
又包含以下字段：``id``，``username``，``salt``，``password``，``email``和``isActive``。
其中``isActive``字段表示User账户是否已经激活。

To make it shorter, the getter and setter methods for each have been removed to
focus on the most important methods that come from the
:class:`Symfony\\Component\\Security\\Core\\User\\UserInterface`.
简短起见，这些字段的getter和setter方法都被去掉，只保留:class:`Symfony\\Component\\Security\\Core\\User\\UserInterface`
中的最重要的方法。

.. code-block:: php

    // src/Acme/UserBundle/Entity/User.php

    namespace Acme\UserBundle\Entity;

    use Doctrine\ORM\Mapping as ORM;
    use Symfony\Component\Security\Core\User\UserInterface;

    /**
     * Acme\UserBundle\Entity\User
     *
     * @ORM\Table(name="acme_users")
     * @ORM\Entity(repositoryClass="Acme\UserBundle\Entity\UserRepository")
     */
    class User implements UserInterface
    {
        /**
         * @ORM\Column(type="integer")
         * @ORM\Id
         * @ORM\GeneratedValue(strategy="AUTO")
         */
        private $id;

        /**
         * @ORM\Column(type="string", length=25, unique=true)
         */
        private $username;

        /**
         * @ORM\Column(type="string", length=32)
         */
        private $salt;

        /**
         * @ORM\Column(type="string", length=40)
         */
        private $password;

        /**
         * @ORM\Column(type="string", length=60, unique=true)
         */
        private $email;

        /**
         * @ORM\Column(name="is_active", type="boolean")
         */
        private $isActive;

        public function __construct()
        {
            $this->isActive = true;
            $this->salt = md5(uniqid(null, true));
        }

        /**
         * @inheritDoc
         */
        public function getUsername()
        {
            return $this->username;
        }

        /**
         * @inheritDoc
         */
        public function getSalt()
        {
            return $this->salt;
        }

        /**
         * @inheritDoc
         */
        public function getPassword()
        {
            return $this->password;
        }

        /**
         * @inheritDoc
         */
        public function getRoles()
        {
            return array('ROLE_USER');
        }

        /**
         * @inheritDoc
         */
        public function eraseCredentials()
        {
        }
    }

In order to use an instance of the ``AcmeUserBundle:User`` class in the Symfony
security layer, the entity class must implement the
:class:`Symfony\\Component\\Security\\Core\\User\\UserInterface`. This
interface forces the class to implement the five following methods:
为了能够在symfony 安全层中应用``AcmeUserBundle:User``类的实例，这个User类必须
植入:class:`Symfony\\Component\\Security\\Core\\User\\UserInterface`接口。这个interface强制使User类植入了以下方法：

* ``getRoles()``,
* ``getPassword()``,
* ``getSalt()``,
* ``getUsername()``,
* ``eraseCredentials()``

For more details on each of these, see :class:`Symfony\\Component\\Security\\Core\\User\\UserInterface`.
详情请见:class:`Symfony\\Component\\Security\\Core\\User\\UserInterface`。

.. versionadded:: 2.1

    In Symfony 2.1, the ``equals`` method was removed from ``UserInterface``.
    If you need to override the default implementation of comparison logic,
    implement the new :class:`Symfony\\Component\\Security\\Core\\User\\EquatableInterface`
    interface and implement the ``isEqualTo`` method.
    在symfony2.1中，这个``equals``方法被从``UserInterface``中移除了。如果你需要覆盖这个默认的
    比较逻辑，可以植入:class:`Symfony\\Component\\Security\\Core\\User\\EquatableInterface` interface
    并植入``isEqualTo``方法。

.. code-block:: php

    // src/Acme/UserBundle/Entity/User.php

    namespace Acme\UserBundle\Entity;

    use Symfony\Component\Security\Core\User\EquatableInterface;

    // ...

    public function isEqualTo(UserInterface $user)
    {
        return $this->username === $user->getUsername();
    }

Below is an export of my ``User`` table from MySQL. For details on how to
create user records and encode their password, see :ref:`book-security-encoding-user-password`.
以下是从MySQL中导出的数据。关于如何创建用户记录以及加密密码，详情请见:ref:`book-security-encoding-user-password`

.. code-block:: text

    mysql> select * from user;
    +----+----------+----------------------------------+------------------------------------------+--------------------+-----------+
    | id | username | salt                             | password                                 | email              | is_active |
    +----+----------+----------------------------------+------------------------------------------+--------------------+-----------+
    |  1 | hhamon   | 7308e59b97f6957fb42d66f894793079 | 09610f61637408828a35d7debee5b38a8350eebe | hhamon@example.com |         1 |
    |  2 | jsmith   | ce617a6cca9126bf4036ca0c02e82dee | 8390105917f3a3d533815250ed7c64b4594d7ebf | jsmith@example.com |         1 |
    |  3 | maxime   | cd01749bb995dc658fa56ed45458d807 | 9764731e5f7fb944de5fd8efad4949b995b72a3c | maxime@example.com |         0 |
    |  4 | donald   | 6683c2bfd90c0426088402930cadd0f8 | 5c3bcec385f59edcc04490d1db95fdb8673bf612 | donald@example.com |         1 |
    +----+----------+----------------------------------+------------------------------------------+--------------------+-----------+
    4 rows in set (0.00 sec)

The database now contains four users with different usernames, emails and
statuses. The next part will focus on how to authenticate one of these users
thanks to the Doctrine entity user provider and a couple of lines of
configuration.
现在这个数据库包含了四个用户，每个用户有不同的username，email，和status（is_active）。
下一部分介绍如何使用Doctrine实体用户提供方（entity user provider）和一些配置代码来对其中一个用户进行验证。

Authenticating Someone against a Database
对数据库中某个用户进行验证
-----------------------------------------

Authenticating a Doctrine user against the database with the Symfony security
layer is a piece of cake. Everything resides in the configuration of the
:doc:`SecurityBundle</reference/configuration/security>` stored in the
``app/config/security.yml`` file.
在symfony中，对通过Doctrine导入数据库的用户进行验证十分简单。所有的
:doc:`SecurityBundle</reference/configuration/security>`的配置都存放在``app/config/security.yml``文件中。

Below is an example of configuration where the user will enter his/her
username and password via HTTP basic authentication. That information will
then be checked against our User entity records in the database:
以下是一个验证配置文件的范例，通过这个配置文件，用户可以输入username和
password，通过HTTP basic authentication进行验证。这个信息可以通过我们数据库里的user实体
记录来检验:

.. configuration-block::

    .. code-block:: yaml

        # app/config/security.yml

        security:
            encoders:
                Acme\UserBundle\Entity\User:
                    algorithm:        sha1
                    encode_as_base64: false
                    iterations:       1

            role_hierarchy:
                ROLE_ADMIN:       ROLE_USER
                ROLE_SUPER_ADMIN: [ ROLE_USER, ROLE_ADMIN, ROLE_ALLOWED_TO_SWITCH ]

            providers:
                administrators:
                    entity: { class: AcmeUserBundle:User, property: username }

            firewalls:
                admin_area:
                    pattern:    ^/admin
                    http_basic: ~

            access_control:
                - { path: ^/admin, roles: ROLE_ADMIN }

The ``encoders`` section associates the ``sha1`` password encoder to the entity
class. This means that Symfony will expect the password that's encoded in
the database to be encoded using this algorithm. For details on how to create
a new User object with a properly encoded password, see the
:ref:`book-security-encoding-user-password` section of the security chapter.
以上代码段中，``encoders``字段表示将使用``sha1``作为实体类的编码器（encoder）。即，
symfony将使用这个编码器对提交到数据库的password进行编码。对于如何对创建的
User类的password进行编码，详情请见:ref:`book-security-encoding-user-password`。

The ``providers`` section defines an ``administrators`` user provider. A
user provider is a "source" of where users are loaded during authentication.
In this case, the ``entity`` keyword means that Symfony will use the Doctrine
entity user provider to load User entity objects from the database by using
the ``username`` unique field. In other words, this tells Symfony how to
fetch the user from the database before checking the password validity.
``Providers``字段定义了一个名为``administrators``的用户提供方（user provider）。User provider
是用户验证过程中用户信息的“来源（source）”。在这个范例中，``entity``字段表明symfony会用
Doctrine实体用户提供方（entity user provider）和``username``这个unique字段来从数据库中提
取User实体类。也就是说，这段代码告诉symfony在验证密码之前如何从数据库中提取用户数据。

This code and configuration works but it's not enough to secure the application
for **active** users. As of now, we still can authenticate with ``maxime``. The
next section explains how to forbid non active users.
以上这段配置代码还不足以对**被激活的**（active）用户加密。也就是说，现在我们仍然对maxime这个
非激活用户（non-active user）通过验证了。下一节将解释如何禁止非激活用户的验证。

Forbid non Active Users
禁止非激活用户
-----------------------

The easiest way to exclude non active users is to implement the
:class:`Symfony\\Component\\Security\\Core\\User\\AdvancedUserInterface`
interface that takes care of checking the user's account status.
The :class:`Symfony\\Component\\Security\\Core\\User\\AdvancedUserInterface`
extends the :class:`Symfony\\Component\\Security\\Core\\User\\UserInterface`
interface, so you just need to switch to the new interface in the ``AcmeUserBundle:User``
entity class to benefit from simple and advanced authentication behaviors.
禁止非激活用户最简单的方法就是植入:class:`Symfony\\Component\\Security\\Core\\User\\AdvancedUserInterface`。
AdvancedUserInterface能够检查用户的账户状态。:class:`Symfony\\Component\\Security\\Core\\User\\AdvancedUserInterface`
是:class:`Symfony\\Component\\Security\\Core\\User\\UserInterface`的扩展，所以只要把``AcmeUserBundle:User``
这个实体类中植入的interface改成AdvancedUserInterface就可以了。

The :class:`Symfony\\Component\\Security\\Core\\User\\AdvancedUserInterface`
interface adds four extra methods to validate the account status:
:class:`Symfony\\Component\\Security\\Core\\User\\AdvancedUserInterface`加了四个验证用户账户状态的方法：

* ``isAccountNonExpired()`` checks whether the user's account has expired,
  ``isAccountNonExpired()``检查用户的账户是否过期
* ``isAccountNonLocked()`` checks whether the user is locked,
  ``isAccountNonLocked()``检查用户是否被锁定
* ``isCredentialsNonExpired()`` checks whether the user's credentials (password)
  ``isCredentialsNonExpired()`` 检查用户的密码是否过期
* ``isEnabled()`` checks whether the user is enabled.
  ``isEnabled()``检查用户是否被允许

For this example, the first three methods will return ``true`` whereas the
``isEnabled()`` method will return the boolean value in the ``isActive`` field.
以下范例中，前三个方法都会返回``true``，但``isEnabled（）``方法会返回``isActive``字段的boolean值。

.. code-block:: php

    // src/Acme/UserBundle/Entity/User.php

    namespace Acme\Bundle\UserBundle\Entity;

    // ...
    use Symfony\Component\Security\Core\User\AdvancedUserInterface;

    // ...
    class User implements AdvancedUserInterface
    {
        // ...
        public function isAccountNonExpired()
        {
            return true;
        }

        public function isAccountNonLocked()
        {
            return true;
        }

        public function isCredentialsNonExpired()
        {
            return true;
        }

        public function isEnabled()
        {
            return $this->isActive;
        }
    }

If we try to authenticate a ``maxime``, the access is now forbidden as this
user does not have an enabled account. The next session will focus on how
to write a custom entity provider to authenticate a user with his username
or his email address.
现在，如果我们要验证``maxime``这个用户，该用户会被禁止通过验证，因为他没有被激活。
下一节将演示如何用自定义的实体提供方（entity provider）来对用户进行验证，这个
实体提供方将使用username或者email来对用户验证。

Authenticating Someone with a Custom Entity Provider
使用自定义的实体提供方进行验证
----------------------------------------------------

The next step is to allow a user to authenticate with his username or his email
address as they are both unique in the database. Unfortunately, the native
entity provider is only able to handle a single property to fetch the user from
the database.
数据库中，用户的username字段和email字段都是唯一（unique）的，下面我们将阐述如何
使用username或者email来对用户进行验证。我们不能用本地的实体提供方（entity provider）
，因为它只能针对一个字段从数据库取数据并进行验证。

To accomplish this, create a custom entity provider that looks for a user
whose username *or* email field matches the submitted login username.
The good news is that a Doctrine repository object can act as an entity user
provider if it implements the
:class:`Symfony\\Component\\Security\\Core\\User\\UserProviderInterface`. This
interface comes with three methods to implement: ``loadUserByUsername($username)``,
``refreshUser(UserInterface $user)``, and ``supportsClass($class)``. For
more details, see :class:`Symfony\\Component\\Security\\Core\\User\\UserProviderInterface`.
为了达到这个目的，可以创建一个自定义实体提供方，这个自定义的实体提供方可以通过查询用户
的username字段*或者*email字段来进行验证。只要在Doctrine的repository对象中植入
:class:`Symfony\\Component\\Security\\Core\\User\\UserProviderInterface`，
它就可以被作为实体提供方了。这个interface有三个方法：``loadUserByUsername($username)``，
``refreshUser(Userinterface $user)``，以及``supportsClass($class)``。详情请见
:class:`Symfony\\Component\\Security\\Core\\User\\UserProviderInterface`。

The code below shows the implementation of the
:class:`Symfony\\Component\\Security\\Core\\User\\UserProviderInterface` in the
``UserRepository`` class::
以下代码展示了如何使用:class:`Symfony\\Component\\Security\\Core\\User\\UserProviderInterface`::

    // src/Acme/UserBundle/Entity/UserRepository.php

    namespace Acme\UserBundle\Entity;

    use Symfony\Component\Security\Core\User\UserInterface;
    use Symfony\Component\Security\Core\User\UserProviderInterface;
    use Symfony\Component\Security\Core\Exception\UsernameNotFoundException;
    use Symfony\Component\Security\Core\Exception\UnsupportedUserException;
    use Doctrine\ORM\EntityRepository;
    use Doctrine\ORM\NoResultException;

    class UserRepository extends EntityRepository implements UserProviderInterface
    {
        public function loadUserByUsername($username)
        {
            $q = $this
                ->createQueryBuilder('u')
                ->where('u.username = :username OR u.email = :email')
                ->setParameter('username', $username)
                ->setParameter('email', $username)
                ->getQuery()
            ;

            try {
                // The Query::getSingleResult() method throws an exception
                // if there is no record matching the criteria.
                $user = $q->getSingleResult();
            } catch (NoResultException $e) {
                throw new UsernameNotFoundException(sprintf('Unable to find an active admin AcmeUserBundle:User object identified by "%s".', $username), null, 0, $e);
            }

            return $user;
        }

        public function refreshUser(UserInterface $user)
        {
            $class = get_class($user);
            if (!$this->supportsClass($class)) {
                throw new UnsupportedUserException(sprintf('Instances of "%s" are not supported.', $class));
            }

            return $this->loadUserByUsername($user->getUsername());
        }

        public function supportsClass($class)
        {
            return $this->getEntityName() === $class || is_subclass_of($class, $this->getEntityName());
        }
    }

To finish the implementation, the configuration of the security layer must be
changed to tell Symfony to use the new custom entity provider instead of the
generic Doctrine entity provider. It's trival to achieve by removing the
``property`` field in the ``security.providers.administrators.entity`` section
of the ``security.yml`` file.
注意还要改变安全层（security layer）的配置，这样的话symfony才知道要使用自定义的
实体提供方，而不是Doctrine默认的实体提供方。你可以把``security.yml``中
``security.providers.administrators.entity``中的``property``去掉。

.. configuration-block::

    .. code-block:: yaml

        # app/config/security.yml
        security:
            # ...
            providers:
                administrators:
                    entity: { class: AcmeUserBundle:User }
            # ...

By doing this, the security layer will use an instance of ``UserRepository`` and
call its ``loadUserByUsername()`` method to fetch a user from the database
whether he filled in his username or email address.
这样，不管用户输入的是username还是email，安全层都可以使用``UserRepository``的实例，
并且使用它的``loadUserByUsername()``方法来获取用户数据了。

Managing Roles in the Database
管理数据库中的角色
------------------------------

The end of this tutorial focuses on how to store and retrieve a list of roles
from the database. As mentioned previously, when your user is loaded, its
``getRoles()`` method returns the array of security roles that should be
assigned to the user. You can load this data from anywhere - a hardcoded
list used for all users (e.g. ``array('ROLE_USER')``), a Doctrine array
property called ``roles``, or via a Doctrine relationship, as we'll learn
about in this section.
本节阐述如何在数据库存储和获取用户的一系列角色。前面提到，当一个用户被载入时，
他的``getRoles()``方法会以数组的形式返回用户的roles。你可以通过各种方法载入这些数据
——比如，一个硬编码的list（如``array('ROLE_USER')``），它会给所有用户都加上这个
'ROLE_USER'角色；再比如，用一个Doctrine实体类的property，把这个property命名为``roles``
（注意这个property的类型必须是array）；或者通过Doctrine关系型数据库。

.. caution::

    In a typical setup, you should always return at least 1 role from the ``getRoles()``
    method. By convention, a role called ``ROLE_USER`` is usually returned.
    If you fail to return any roles, it may appear as if your user isn't
    authenticated at all.
    一般的，``getRoles()``方法起码要返回一个角色。按照惯例，一个名叫``ROLE_USER``的角色要被
    返回。如果你没有返回任何角色，那么用户就不会被通过验证。

In this example, the ``AcmeUserBundle:User`` entity class defines a
many-to-many relationship with a ``AcmeUserBundle:Group`` entity class. A user
can be related several groups and a group can be composed of one or
more users. As a group is also a role, the previous ``getRoles()`` method now
returns the list of related groups::
下面这个范例中，``AcmeUserBundle:User``实体类定义了一个关于``AcmeUserBundle:Group``实体类的多对多
（many-to-many）关系。一个user可以有多个group，一个group也可以有多个user。该范例中将group定义
为角色，那么``getRoles()``方法就会返回一系列group对象。

    // src/Acme/UserBundle/Entity/User.php

    namespace Acme\Bundle\UserBundle\Entity;

    use Doctrine\Common\Collections\ArrayCollection;

    // ...
    class User implements AdvancedUserInterface
    {
        /**
         * @ORM\ManyToMany(targetEntity="Group", inversedBy="users")
         *
         */
        private $groups;

        public function __construct()
        {
            $this->groups = new ArrayCollection();
        }

        // ...

        public function getRoles()
        {
            return $this->groups->toArray();
        }
    }

The ``AcmeUserBundle:Group`` entity class defines three table fields (``id``,
``name`` and ``role``). The unique ``role`` field contains the role name used by
the Symfony security layer to secure parts of the application. The most
important thing to notice is that the ``AcmeUserBundle:Group`` entity class
implements the :class:`Symfony\\Component\\Security\\Core\\Role\\RoleInterface`
that forces it to have a ``getRole()`` method::
在``AcmeUserBundle:Group``实体类中定义了三个字段：``id``，``name``，``role``。这个唯一（unique）
的``role``字段包含了symfony安全层配置文件中使用的role名称。注意：``AcmeUserBundle:Group``
实体类植入了:class:`Symfony\\Component\\Security\\Core\\Role\\RoleInterface`，这样的话
就能强制它使用``getRole()``这个方法：

    namespace Acme\Bundle\UserBundle\Entity;

    use Symfony\Component\Security\Core\Role\RoleInterface;
    use Doctrine\Common\Collections\ArrayCollection;
    use Doctrine\ORM\Mapping as ORM;

    /**
     * @ORM\Table(name="acme_groups")
     * @ORM\Entity()
     */
    class Group implements RoleInterface
    {
        /**
         * @ORM\Column(name="id", type="integer")
         * @ORM\Id()
         * @ORM\GeneratedValue(strategy="AUTO")
         */
        private $id;

        /**
         * @ORM\Column(name="name", type="string", length=30)
         */
        private $name;

        /**
         * @ORM\Column(name="role", type="string", length=20, unique=true)
         */
        private $role;

        /**
         * @ORM\ManyToMany(targetEntity="User", mappedBy="groups")
         */
        private $users;

        public function __construct()
        {
            $this->users = new ArrayCollection();
        }

        // ... getters and setters for each property

        /**
         * @see RoleInterface
         */
        public function getRole()
        {
            return $this->role;
        }
    }

To improve performances and avoid lazy loading of groups when retrieving a user
from the custom entity provider, the best solution is to join the groups
relationship in the ``UserRepository::loadUserByUsername()`` method. This will
fetch the user and his associated roles / groups with a single query::
为了提高性能，避免从自定义实体提供方获取user时group的延迟加载（lazy loading），
最好能在``UserRepository::loadUserByUsername()``方法中使用join，这样的话通过简单的
一个请求就可以获取user和roles/groups数据::

    // src/Acme/UserBundle/Entity/UserRepository.php

    namespace Acme\Bundle\UserBundle\Entity;

    // ...

    class UserRepository extends EntityRepository implements UserProviderInterface
    {
        public function loadUserByUsername($username)
        {
            $q = $this
                ->createQueryBuilder('u')
                ->select('u, g')
                ->leftJoin('u.groups', 'g')
                ->where('u.username = :username OR u.email = :email')
                ->setParameter('username', $username)
                ->setParameter('email', $username)
                ->getQuery()
            ;

            // ...
        }

        // ...
    }

The ``QueryBuilder::leftJoin()`` method joins and fetches related groups from
the ``AcmeUserBundle:User`` model class when a user is retrieved with his email
address or username.
当用户输入username或者email后，``QueryBuilder::leftJoin()``方法联接并从
``AcmeUserBundle:User``中获取了相关groups数据。
