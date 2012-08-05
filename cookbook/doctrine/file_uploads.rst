How to handle File Uploads with Doctrine
如何使用doctrine来处理文件上传
========================================

Handling file uploads with Doctrine entities is no different than handling
any other file upload. In other words, you're free to move the file in your
controller after handling a form submission. For examples of how to do this,
see the :doc:`file type reference</reference/forms/types/file>` page.
使用doctrine来处理文件上传和其他处理文件上传的方式没什么区别。换句话说，你可以在处理完表单提交
后自由移动文件。查看范例请参见:doc:`file type reference</reference/forms/types/file>`。

If you choose to, you can also integrate the file upload into your entity
lifecycle (i.e. creation, update and removal). In this case, as your entity
is created, updated, and removed from Doctrine, the file uploading and removal
processing will take place automatically (without needing to do anything in
your controller);
如果你需要，你还可以将文件上传合并到你的实体类lifecycle中（也就是创建、更新和删除）。在这种情况下，
由于你的实体类被创建、更新和删除了，文件上传和删除过程也会自动运行（你不必在控制器中做任何事情）；

To make this work, you'll need to take care of a number of details, which
will be covered in this cookbook entry.
要达到这个目的，你需要注意一些细节，本章将详细讲述。

Basic Setup
基本设置
-----------

First, create a simple Doctrine Entity class to work with::
首先，创建一个简单的doctrine实体类::

    // src/Acme/DemoBundle/Entity/Document.php
    namespace Acme\DemoBundle\Entity;

    use Doctrine\ORM\Mapping as ORM;
    use Symfony\Component\Validator\Constraints as Assert;

    /**
     * @ORM\Entity
     */
    class Document
    {
        /**
         * @ORM\Id
         * @ORM\Column(type="integer")
         * @ORM\GeneratedValue(strategy="AUTO")
         */
        public $id;

        /**
         * @ORM\Column(type="string", length=255)
         * @Assert\NotBlank
         */
        public $name;

        /**
         * @ORM\Column(type="string", length=255, nullable=true)
         */
        public $path;

        public function getAbsolutePath()
        {
            return null === $this->path ? null : $this->getUploadRootDir().'/'.$this->path;
        }

        public function getWebPath()
        {
            return null === $this->path ? null : $this->getUploadDir().'/'.$this->path;
        }

        protected function getUploadRootDir()
        {
            // the absolute directory path where uploaded documents should be saved
            return __DIR__.'/../../../../web/'.$this->getUploadDir();
        }

        protected function getUploadDir()
        {
            // get rid of the __DIR__ so it doesn't screw when displaying uploaded doc/image in the view.
            return 'uploads/documents';
        }
    }

The ``Document`` entity has a name and it is associated with a file. The ``path``
property stores the relative path to the file and is persisted to the database.
The ``getAbsolutePath()`` is a convenience method that returns the absolute
path to the file while the ``getWebPath()`` is a convenience method that
returns the web path, which can be used in a template to link to the uploaded
file.
Document实体类有一个name字段并关联到一个文件。path属性存储文件的相对路径并persist到数据库。
getAbsolutePath()是一个返回文件绝对路径的便捷方法，getWebPath()则是一个返回web路径的便捷方法，它可以被
应用于模板中，从而返回被上传文件的链接。

.. tip::

    If you have not done so already, you should probably read the
    :doc:`file</reference/forms/types/file>` type documentation first to
    understand how the basic upload process works.
    如果你还没有这样做，你应该阅读:doc:`file</reference/forms/types/file>` type文档，从而了解
    基本上传过程是怎样的。
    
.. note::

    If you're using annotations to specify your validation rules (as shown
    in this example), be sure that you've enabled validation by annotation
    (see :ref:`validation configuration<book-validation-configuration>`).
    如果你使用注释来指定验证规则（本例中所展示的），请确保你已经激活了注释验证（参见:ref:`validation configuration<book-validation-configuration>`）。
    
To handle the actual file upload in the form, use a "virtual" ``file`` field.
For example, if you're building your form directly in a controller, it might
look like this::
要实际在表单中处理文件上传，可以使用“虚拟”file字段。比如，如果你直接在控制器中创建表单，它
看起来会像这样::

    public function uploadAction()
    {
        // ...

        $form = $this->createFormBuilder($document)
            ->add('name')
            ->add('file')
            ->getForm()
        ;

        // ...
    }

Next, create this property on your ``Document`` class and add some validation
rules::
接下来，在你的Document实体类中创建这个属性并添加一些验证规则::

    // src/Acme/DemoBundle/Entity/Document.php

    // ...
    class Document
    {
        /**
         * @Assert\File(maxSize="6000000")
         */
        public $file;

        // ...
    }

.. note::

    As you are using the ``File`` constraint, Symfony2 will automatically guess
    that the form field is a file upload input. That's why you did not have
    to set it explicitly when creating the form above (``->add('file')``).
    由于你使用了File规则，symfony2会自动猜测表单字段是一个文件上传input。所以你不必在创建以上
    表单时显性地设置它（->add('file')）。
    
The following controller shows you how to handle the entire process::
以下的控制器向你展示了如何来处理整个过程::

    use Acme\DemoBundle\Entity\Document;
    use Sensio\Bundle\FrameworkExtraBundle\Configuration\Template;
    // ...

    /**
     * @Template()
     */
    public function uploadAction()
    {
        $document = new Document();
        $form = $this->createFormBuilder($document)
            ->add('name')
            ->add('file')
            ->getForm()
        ;

        if ($this->getRequest()->getMethod() === 'POST') {
            $form->bindRequest($this->getRequest());
            if ($form->isValid()) {
                $em = $this->getDoctrine()->getManager();

                $em->persist($document);
                $em->flush();

                $this->redirect($this->generateUrl('...'));
            }
        }

        return array('form' => $form->createView());
    }

.. note::

    When writing the template, don't forget to set the ``enctype`` attribute:
    创建模板时不要忘了设置enctype属性:
    
    .. code-block:: html+php

        <h1>Upload File</h1>

        <form action="#" method="post" {{ form_enctype(form) }}>
            {{ form_widget(form) }}

            <input type="submit" value="Upload Document" />
        </form>

The previous controller will automatically persist the ``Document`` entity
with the submitted name, but it will do nothing about the file and the ``path``
property will be blank.
以上控制器会自动persist这个Document实体类所提交的name，但它不会针对文件做任何事情，且path
属性是空的。

An easy way to handle the file upload is to move it just before the entity is
persisted and then set the ``path`` property accordingly. Start by calling
a new ``upload()`` method on the ``Document`` class, which you'll create
in a moment to handle the file upload::
一个处理文件上传更简单的方法就是在实体类被persist之前移动它，并据此来设置path的值。首先对Document类
执行upload()方法，待会我们再创建该方法::

    if ($form->isValid()) {
        $em = $this->getDoctrine()->getManager();

        $document->upload();

        $em->persist($document);
        $em->flush();

        $this->redirect('...');
    }

The ``upload()`` method will take advantage of the :class:`Symfony\\Component\\HttpFoundation\\File\\UploadedFile`
object, which is what's returned after a ``file`` field is submitted::
upload()方法利用了:class:`Symfony\\Component\\HttpFoundation\\File\\UploadedFile`对象，该对象在file字段
被提交后返回::

    public function upload()
    {
        // the file property can be empty if the field is not required
        if (null === $this->file) {
            return;
        }

        // we use the original file name here but you should
        // sanitize it at least to avoid any security issues
        
        // move takes the target directory and then the target filename to move to
        $this->file->move($this->getUploadRootDir(), $this->file->getClientOriginalName());

        // set the path property to the filename where you'ved saved the file
        $this->path = $this->file->getClientOriginalName();

        // clean up the file property as you won't need it anymore
        $this->file = null;
    }

Using Lifecycle Callbacks
使用lifecycle回调函数
-------------------------

Even if this implementation works, it suffers from a major flaw: What if there
is a problem when the entity is persisted? The file would have already moved
to its final location even though the entity's ``path`` property didn't
persist correctly.
虽然以上方法可以工作，但是它有一个主要缺陷：如果在实体类被persist的时候出问题怎么办？
实体类的path没有被存入数据库，而文件却已经被移动到它的最终位置了。

To avoid these issues, you should change the implementation so that the database
operation and the moving of the file become atomic: if there is a problem
persisting the entity or if the file cannot be moved, then *nothing* should
happen.
要解决这个问题，你应该改变以下方式以使数据库操作和文件移动单元化:如果在persist实体类的时候出问题，
或者文件无法移动，则什么也不返回。

To do this, you need to move the file right as Doctrine persists the entity
to the database. This can be accomplished by hooking into an entity lifecycle
callback::
要解决这个问题，你可以使用实体类lifecycle回调函数::

    /**
     * @ORM\Entity
     * @ORM\HasLifecycleCallbacks
     */
    class Document
    {
    }

Next, refactor the ``Document`` class to take advantage of these callbacks::
然后，重构Document实体类来利用这些回调函数::

    use Symfony\Component\HttpFoundation\File\UploadedFile;

    /**
     * @ORM\Entity
     * @ORM\HasLifecycleCallbacks
     */
    class Document
    {
        /**
         * @ORM\PrePersist()
         * @ORM\PreUpdate()
         */
        public function preUpload()
        {
            if (null !== $this->file) {
                // do whatever you want to generate a unique name
                $this->path = uniqid().'.'.$this->file->guessExtension();
            }
        }

        /**
         * @ORM\PostPersist()
         * @ORM\PostUpdate()
         */
        public function upload()
        {
            if (null === $this->file) {
                return;
            }

            // if there is an error when moving the file, an exception will
            // be automatically thrown by move(). This will properly prevent
            // the entity from being persisted to the database on error
            $this->file->move($this->getUploadRootDir(), $this->path);

            unset($this->file);
        }

        /**
         * @ORM\PostRemove()
         */
        public function removeUpload()
        {
            if ($file = $this->getAbsolutePath()) {
                unlink($file);
            }
        }
    }

The class now does everything you need: it generates a unique filename before
persisting, moves the file after persisting, and removes the file if the
entity is ever deleted.
这个实体类现在可以做许多你需要的工作了：它可以在persist之前集成一个唯一文件名，在persist之后
移动文件，在这个实体类被删除之后删除文件。

Now that the moving of the file is handled atomically by the entity, the
call to ``$document->upload()`` should be removed from the controller::
现在既然文件移动已经可以由实体类自动完成，``$document->upload()``可以被从控制器中移除了::

    if ($form->isValid()) {
        $em = $this->getDoctrine()->getEntityManager();

        $em->persist($document);
        $em->flush();

        $this->redirect('...');
    }

.. note::

    The ``@ORM\PrePersist()`` and ``@ORM\PostPersist()`` event callbacks are
    triggered before and after the entity is persisted to the database. On the
    other hand, the ``@ORM\PreUpdate()`` and ``@ORM\PostUpdate()`` event
    callbacks are called when the entity is updated.
    ``@ORM\PrePersist()``和``@ORM\PostPersist()``回调是在实体类被persist到数据库之前或之后激发的；而
    ``@ORM\PreUpdate()``和``@ORM\PostUpdate()``则是在实体类被更新的时候激发的。

.. caution::

    The ``PreUpdate`` and ``PostUpdate`` callbacks are only triggered if there
    is a change in one of the entity's field that are persisted. This means
    that, by default, if you modify only the ``$file`` property, these events
    will not be triggered, as the property itself is not directly persisted
    via Doctrine. One solution would be to use an ``updated`` field that's
    persisted to Doctrine, and to modify it manually when changing the file.
    ``PreUpdate``和``PostUpdate``回调只在已经存储于数据库中的实体类字段有改变的时候才会被激发。
    这表示如果你仅修改$file属性，这些事件是不会被激发的，因为该属性本身没有直接通过doctrine被persist到数据库中。
    一个解决方法就是使用一个能通过doctrine被persist到数据库的updated字段，并在修改file的时候手动修改它。
    
Using the ``id`` as the filename
使用id作为文件名
--------------------------------

If you want to use the ``id`` as the name of the file, the implementation is
slightly different as you need to save the extension under the ``path``
property, instead of the actual filename::
如果你想要使用id作为文件的名称，方法就略有不同了。因为你需要在path属性下保存这个扩展，而不是
确切的文件名::

    use Symfony\Component\HttpFoundation\File\UploadedFile;

    /**
     * @ORM\Entity
     * @ORM\HasLifecycleCallbacks
     */
    class Document
    {
        // a property used temporarily while deleting
        private $filenameForRemove;

        /**
         * @ORM\PrePersist()
         * @ORM\PreUpdate()
         */
        public function preUpload()
        {
            if (null !== $this->file) {
                $this->path = $this->file->guessExtension();
            }
        }

        /**
         * @ORM\PostPersist()
         * @ORM\PostUpdate()
         */
        public function upload()
        {
            if (null === $this->file) {
                return;
            }

            // you must throw an exception here if the file cannot be moved
            // so that the entity is not persisted to the database
            // which the UploadedFile move() method does
            $this->file->move($this->getUploadRootDir(), $this->id.'.'.$this->file->guessExtension());

            unset($this->file);
        }

        /**
         * @ORM\PreRemove()
         */
        public function storeFilenameForRemove()
        {
            $this->filenameForRemove = $this->getAbsolutePath();
        }

        /**
         * @ORM\PostRemove()
         */
        public function removeUpload()
        {
            if ($this->filenameForRemove) {
                unlink($this->filenameForRemove);
            }
        }

        public function getAbsolutePath()
        {
            return null === $this->path ? null : $this->getUploadRootDir().'/'.$this->id.'.'.$this->path;
        }
    }

You'll notice in this case that you need to do a little bit more work in
order to remove the file. Before it's removed, you must store the file path
(since it depends on the id). Then, once the object has been fully removed
from the database, you can safely delete the file (in ``PostRemove``).
在这种情况下你需要做更多工作来移动文件。在移动之前，你必须存储文件路径（因为它取决于id）。
然后，一旦对象被彻底从数据库中删除，你可以安全地删除文件（在PostRemove中）。