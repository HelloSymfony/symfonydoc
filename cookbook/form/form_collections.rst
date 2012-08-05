.. index::
   single: Form; Embed collection of forms

How to Embed a Collection of Forms
如何嵌入表单集合
==================================

In this entry, you'll learn how to create a form that embeds a collection
of many other forms. This could be useful, for example, if you had a ``Task``
class and you wanted to edit/create/remove many ``Tag`` objects related to
that Task, right inside the same form.
本章你将学习如何创建一个可以嵌入其他表单集合的表单。这很有用，比如，如果你有一个Task类
并且你需要在那个Task的表单中编辑/创建/删除许多关于Task的Tag对象。

.. note::

    In this entry, we'll loosely assume that you're using Doctrine as your
    database store. But if you're not using Doctrine (e.g. Propel or just
    a database connection), it's all very similar. There are only a few parts
    of this tutorial that really care about "persistence".
    本章假定你使用doctrine来存储数据。但是如果你不使用doctrine（比如Propel或仅仅是一个数据库连接），
    这个过程也很相似。本章只有少数部分用到persist。
    
    If you *are* using Doctrine, you'll need to add the Doctrine metadata,
    including the ``ManyToMany`` on the Task's ``tags`` property.
    如果你使用doctrine，你需要添加doctrine metadata，包括在Task的tags属性中
    添加ManyToMany metadata。

Let's start there: suppose that each ``Task`` belongs to multiple ``Tags``
objects. Start by creating a simple ``Task`` class::
假设每个task都属于多个tags对象。首先创建一个Task对象::

    // src/Acme/TaskBundle/Entity/Task.php
    namespace Acme\TaskBundle\Entity;
    
    use Doctrine\Common\Collections\ArrayCollection;

    class Task
    {
        protected $description;

        protected $tags;

        public function __construct()
        {
            $this->tags = new ArrayCollection();
        }
        
        public function getDescription()
        {
            return $this->description;
        }

        public function setDescription($description)
        {
            $this->description = $description;
        }

        public function getTags()
        {
            return $this->tags;
        }

        public function setTags(ArrayCollection $tags)
        {
            $this->tags = $tags;
        }
    }

.. note::

    The ``ArrayCollection`` is specific to Doctrine and is basically the
    same as using an ``array`` (but it must be an ``ArrayCollection``) if
    you're using Doctrine.
    ArrayCollection是doctrine所特有的，它的原理就像使用array一样，但是你必须使用ArrayCollection。

Now, create a ``Tag`` class. As you saw above, a ``Task`` can have many ``Tag``
objects::
现在创建一个Tag类。如上所述，一个Task可以包含多个Tag对象::

    // src/Acme/TaskBundle/Entity/Tag.php
    namespace Acme\TaskBundle\Entity;

    class Tag
    {
        public $name;
    }

.. tip::

    The ``name`` property is public here, but it can just as easily be protected
    or private (but then it would need ``getName`` and ``setName`` methods).
    这里name属性是public的，但是它也可以是private或者protected(此时必须要用getName和setName方法)。

Now let's get to the forms. Create a form class so that a ``Tag`` object
can be modified by the user::
创建一个表单类，这样Tag对象就可以被用户修改了::

    // src/Acme/TaskBundle/Form/Type/TagType.php
    namespace Acme\TaskBundle\Form\Type;

    use Symfony\Component\Form\AbstractType;
    use Symfony\Component\Form\FormBuilder;

    class TagType extends AbstractType
    {
        public function buildForm(FormBuilder $builder, array $options)
        {
            $builder->add('name');
        }

        public function getDefaultOptions()
        {
            return array(
                'data_class' => 'Acme\TaskBundle\Entity\Tag',
            );
        }

        public function getName()
        {
            return 'tag';
        }
    }

With this, we have enough to render a tag form by itself. But since the end
goal is to allow the tags of a ``Task`` to be modified right inside the task
form itself, create a form for the ``Task`` class.
现在已经具备可以提交表单的条件了。但是最终目的还是要使task的tags能够在task表单内部被用户修改，
所以我们还要创建一个task的表单。

Notice that we embed a collection of ``TagType`` forms using the
:doc:`collection</reference/forms/types/collection>` field type::
注意我们使用:doc:`collection</reference/forms/types/collection>`字段嵌入了TagType表单集合::

    // src/Acme/TaskBundle/Form/Type/TaskType.php
    namespace Acme\TaskBundle\Form\Type;

    use Symfony\Component\Form\AbstractType;
    use Symfony\Component\Form\FormBuilder;

    class TaskType extends AbstractType
    {
        public function buildForm(FormBuilder $builder, array $options)
        {
            $builder->add('description');

            $builder->add('tags', 'collection', array('type' => new TagType()));
        }

        public function getDefaultOptions()
        {
            return array(
                'data_class' => 'Acme\TaskBundle\Entity\Task',
            );
        }

        public function getName()
        {
            return 'task';
        }
    }

In your controller, you'll now initialize a new instance of ``TaskType``::
在你的控制器中，你需要初始化一个新的TaskType实例::

    // src/Acme/TaskBundle/Controller/TaskController.php
    namespace Acme\TaskBundle\Controller;
    
    use Acme\TaskBundle\Entity\Task;
    use Acme\TaskBundle\Entity\Tag;
    use Acme\TaskBundle\Form\Type\TaskType;
    use Symfony\Component\HttpFoundation\Request;
    use Symfony\Bundle\FrameworkBundle\Controller\Controller;
    
    class TaskController extends Controller
    {
        public function newAction(Request $request)
        {
            $task = new Task();
            
            // dummy code - this is here just so that the Task has some tags
            // otherwise, this isn't an interesting example
            $tag1 = new Tag();
            $tag1->name = 'tag1';
            $task->getTags()->add($tag1);
            $tag2 = new Tag();
            $tag2->name = 'tag2';
            $task->getTags()->add($tag2);
            // end dummy code
            
            $form = $this->createForm(new TaskType(), $task);
            
            // process the form on POST
            if ('POST' === $request->getMethod()) {
                $form->bindRequest($request);
                if ($form->isValid()) {
                    // maybe do some form processing, like saving the Task and Tag objects
                }
            }
            
            return $this->render('AcmeTaskBundle:Task:new.html.twig', array(
                'form' => $form->createView(),
            ));
        }
    }

The corresponding template is now able to render both the ``description``
field for the task form as well as all the ``TagType`` forms for any tags
that are already related to this ``Task``. In the above controller, I added
some dummy code so that you can see this in action (since a ``Task`` has
zero tags when first created).
对应的模板现在可以提交task表单中的description字段和TagType表单中被关联到Task的字段了。
在以上控制器中，我添加了一些伪代码，这样才会使它运行（因为Task刚创建的时候还还没有tag）。

.. configuration-block::

    .. code-block:: html+jinja

        {# src/Acme/TaskBundle/Resources/views/Task/new.html.twig #}
        {# ... #}

        <form action="..." method="POST" {{ form_enctype(form) }}>
            {# render the task's only field: description #}
            {{ form_row(form.description) }}

            <h3>Tags</h3>
            <ul class="tags">
                {# iterate over each existing tag and render its only field: name #}
                {% for tag in form.tags %}
                    <li>{{ form_row(tag.name) }}</li>
                {% endfor %}
            </ul>

            {{ form_rest(form) }}
            {# ... #}
        </form>

    .. code-block:: html+php

        <!-- src/Acme/TaskBundle/Resources/views/Task/new.html.php -->
        <!-- ... -->

        <form action="..." method="POST" ...>
            <h3>Tags</h3>
            <ul class="tags">
                <?php foreach($form['tags'] as $tag): ?>
                    <li><?php echo $view['form']->row($tag['name']) ?></li>
                <?php endforeach; ?>
            </ul>

            <?php echo $view['form']->rest($form) ?>
        </form>
        
        <!-- ... -->

When the user submits the form, the submitted data for the ``Tags`` fields
are used to construct an ArrayCollection of ``Tag`` objects, which is then
set on the ``tag`` field of the ``Task`` instance.
当用户提交表单时，Tags字段所提交的数据被用来构建一个Tag对象的ArrayCollection，它会被
设置在Task实例的tag字段上。

The ``Tags`` collection is accessible naturally via ``$task->getTags()``
and can be persisted to the database or used however you need.
这个Tags集合可以通过``$task->getTags()``来访问，并且可以被载入数据库。

So far, this works great, but this doesn't allow you to dynamically add new
tags or delete existing tags. So, while editing existing tags will work
great, your user can't actually add any new tags yet.
现在这个代码可以工作良好，但是你只能编辑现有tag，还不能动态地创建新的tag或者删除现有tag。

.. _cookbook-form-collections-new-prototype:

Allowing "new" tags with the "prototype"
使用prototype来创建新的tag
-----------------------------------------

Allowing the user to dynamically add new tags means that we'll need to
use some JavaScript. Previously we added two tags to our form in the controller.
Now we need to let the user add as many tag forms as he needs directly in the browser.
This will be done through a bit of JavaScript.
允许用户动态创建新的tag意味着要提交一些javascript代码。目前我们在这个控制器中添加了两个tag，
现在我们需要在浏览器中添加新的tag。这可以使用javascript实现。


The first thing we need to do is to let the form collection know that it will
receive an unknown number of tags. So far we've added two tags and the form
type expects to receive exactly two, otherwise an error will be thrown:
``This form should not contain extra fields``. To make this flexible, we
add the ``allow_add`` option to our collection field::
首先我们要使表单集合知道它会接收多个tag。目前我们已经添加了两个tag，而且表单也只能接收两个tag，否则一个错误
会被抛出：``This form should not contain extra fields``。我们可以将allow_add选项添加到collection字段中::

    // src/Acme/TaskBundle/Form/Type/TaskType.php
    // ...
    
    public function buildForm(FormBuilder $builder, array $options)
    {
        $builder->add('description');

        $builder->add('tags', 'collection', array(
            'type' => new TagType(),
            'allow_add' => true,
            'by_reference' => false,
        ));
    }

Note that we also added ``'by_reference' => false``. Normally, the form
framework would modify the tags on a `Task` object *without* actually
ever calling `setTags`. By setting :ref:`by_reference<reference-form-types-by-reference>`
to `false`, `setTags` will be called. This will be important later as you'll
see.
注意我们还添加了``'by_reference' => false``。通常，表单系统会修改Task对象中的tags属性，但并不是
通过`setTags`。通过设置:ref:`by_reference<reference-form-types-by-reference>`为false，setTags会
被使用。下面你将看到这一步如何重要。

In addition to telling the field to accept any number of submitted objects, the
``allow_add`` also makes a "prototype" variable available to you. This "prototype"
is a little "template" that contains all the HTML to be able to render any
new "tag" forms. To render it, make the following change to your template:
要告诉字段接收多个提交的对象，allow_add还允许一个“prototype”变量被使用。这个“prototype”变量
其实是一小段提交tag表单的HTML模板代码。在你的模板中：

.. configuration-block::

    .. code-block:: html+jinja
    
        <ul class="tags" data-prototype="{{ form_widget(form.tags.get('prototype')) | e }}">
            ...
        </ul>
    
    .. code-block:: html+php
    
        <ul class="tags" data-prototype="<?php echo $view->escape($view['form']->row($form['tags']->get('prototype'))) ?>">
            ...
        </ul>

.. note::

    If you render your whole "tags" sub-form at once (e.g. ``form_row(form.tags)``),
    then the prototype is automatically available on the outer ``div`` as
    the ``data-prototype`` attribute, similar to what you see above.
    如果你将tags子表单整个提交（比如使用``form_row(form.tags)``），prototype会自动被
    作为包围它的div标签的``data-prototype``属性。

.. tip::

    The ``form.tags.get('prototype')`` is form element that looks and feels just
    like the individual ``form_widget(tag)`` elements inside our ``for`` loop.
    This means that you can call ``form_widget``, ``form_row``, or ``form_label``
    on it. You could even choose to render only one of its fields (e.g. the
    ``name`` field):
    ``form.tags.get('prototype')``跟通过for循环单个提交的``form_widget(tag)``元素差不多。
    这表示你可以在它上面使用``form_widget``, ``form_row``, 或``form_label``。还可以只提交
    它的一个字段(比如name字段):
    
    .. code-block:: html+jinja
    
        {{ form_widget(form.tags.get('prototype').name) | e }}

On the rendered page, the result will look something like this:

.. code-block:: html

    <ul class="tags" data-prototype="&lt;div&gt;&lt;label class=&quot; required&quot;&gt;__name__&lt;/label&gt;&lt;div id=&quot;task_tags___name__&quot;&gt;&lt;div&gt;&lt;label for=&quot;task_tags___name___name&quot; class=&quot; required&quot;&gt;Name&lt;/label&gt;&lt;input type=&quot;text&quot; id=&quot;task_tags___name___name&quot; name=&quot;task[tags][__name__][name]&quot; required=&quot;required&quot; maxlength=&quot;255&quot; /&gt;&lt;/div&gt;&lt;/div&gt;&lt;/div&gt;">

The goal of this section will be to use JavaScript to read this attribute
and dynamically add new tag forms when the user clicks a "Add a tag" link.
To make things simple, we'll use jQuery and assume you have it included somewhere
on your page.
这一节会使用javascript来读取这里的属性并在用户点击"Add a tag"链接时添加新的tag
表单。我们将使用jQuery并假设你已经安装它。

Add a ``script`` tag somewhere on your page so we can start writing some JavaScript.
在你的页面中添加一个script标签，这样我们就可以编写javascript代码了。

First, add a link to the bottom of the "tags" list via JavaScript. Second,
bind to the "click" event of that link so we can add a new tag form (``addTagForm``
will be show next):
首先，在tags列表下通过javascript添加一个链接。然后将click事件和那个链接绑定:

.. code-block:: javascript

    // Get the div that holds the collection of tags
    var collectionHolder = $('ul.tags');

    // setup an "add a tag" link
    var $addTagLink = $('<a href="#" class="add_tag_link">Add a tag</a>');
    var $newLinkLi = $('<li></li>').append($addTagLink);

    jQuery(document).ready(function() {
        // add the "add a tag" anchor and li to the tags ul
        collectionHolder.append($newLinkLi);

        $addTagLink.on('click', function(e) {
            // prevent the link from creating a "#" on the URL
            e.preventDefault();

            // add a new tag form (see next code block)
            addTagForm();
        });
    });

The ``addTagForm`` function's job will be to use the ``data-prototype`` attribute
to dynamically add a new form when this link is clicked. The ``data-prototype``
HTML contains the tag ``text`` input element with a name of ``task[tags][__name__][name]``
and id of ``task_tags___name___name``. The ``__name__`` is a little "placeholder",
which we'll replace with a unique, incrementing number (e.g. ``task[tags][3][name]``).
``addTagForm``方法会使用data-prototype属性来动态添加一个新的表单。data-prototype HTML包含了text input
标签，name是``task[tags][__name__][name]``，id是``task_tags___name___name``。``__name__``是一个placeholder，
我们将使用一个递增的数字来替代它（比如``task[tags][3][name]``）。

.. versionadded:: 2.1
    The placeholder was changed from ``$$name$$`` to ``__name__`` in Symfony 2.1
    在symfony2.1中将$$name$$改成了``__name__``。

The actual code needed to make this all work can vary quite a bit, but here's
one example:
要做这个工作的代码会有点多。以下是一个例子:

.. code-block:: javascript

    function addTagForm() {
        // Get the data-prototype we explained earlier
        var prototype = collectionHolder.attr('data-prototype');

        // Replace '__name__' in the prototype's HTML to
        // instead be a number based on the current collection's length.
        var newForm = prototype.replace(/__name__/g, collectionHolder.children().length);

        // Display the form in the page in an li, before the "Add a tag" link li
        var $newFormLi = $('<li></li>').append(newForm);
        $newLinkLi.before($newFormLi);
    }

.. note:

    It is better to separate your javascript in real JavaScript files than
    to write it inside the HTML as we are doing here.
    最好将javascript代码分离出来，而不是直接在这个模板中编写。

Now, each time a user clicks the ``Add a tag`` link, a new sub form will
appear on the page. When we submit, any new tag forms will be converted into
new ``Tag`` objects and added to the ``tags`` property of the ``Task`` object.
现在，每当一个用户点击``Add a tag``标签时，一个新的子表单就会在页面上显示。当提交表单时，
新的tag表单会被转换成新的Tag对象并添加到Task对象中的tags属性中。

.. sidebar:: Doctrine: Cascading Relations and saving the "Inverse" side

    To get the new tags to save in Doctrine, you need to consider a couple
    more things. First, unless you iterate over all of the new ``Tag`` objects
    and call ``$em->persist($tag)`` on each, you'll receive an error from
    Doctrine:
    要让新的标签在doctrine中被存储，你还要做一些工作。要是你不挨个对每个Tag对象使用
    ``$em->persist($tag)``方法的话，你会接收到如下错误信息：
    
        A new entity was found through the relationship 'Acme\TaskBundle\Entity\Task#tags' that was not configured to cascade persist operations for entity...
    
    To fix this, you may choose to "cascade" the persist operation automatically
    from the ``Task`` object to any related tags. To do this, add the ``cascade``
    option to your ``ManyToMany`` metadata:
    要改进这一点，你可以cascade persist方法，在Task对象中的metadata中添加cascade选项:
    
    .. configuration-block::
    
        .. code-block:: php-annotations

            /**
             * @ORM\ManyToMany(targetEntity="Tag", cascade={"persist"})
             */
            protected $tags;

        .. code-block:: yaml

            # src/Acme/TaskBundle/Resources/config/doctrine/Task.orm.yml
            Acme\TaskBundle\Entity\Task:
                type: entity
                # ...
                oneToMany:
                    tags:
                        targetEntity: Tag
                        cascade:      [persist]
    
    A second potential issue deals with the `Owning Side and Inverse Side`_
    of Doctrine relationships. In this example, if the "owning" side of the
    relationship is "Task", then persistence will work fine as the tags are
    properly added to the Task. However, if the owning side is on "Tag", then
    you'll need to do a little bit more work to ensure that the correct side
    of the relationship is modified.
    还有一点要注意的就是doctrine中`Owning Side and Inverse Side`_问题。在这个例子中，如果
    Task是owning side，persist方法会工作良好；但如果owning side是Tag的话，你还要做一些工作来
    确保Task那一方被修改了。

    The trick is to make sure that the single "Task" is set on each "Tag".
    One easy way to do this is to add some extra logic to ``setTags()``,
    which is called by the form framework since :ref:`by_reference<reference-form-types-by-reference>`
    is set to ``false``::
    这就是说要确保对于每个Tag都设置了一个Task。你可以在setTags()中添加一些逻辑（由于
    你设置:ref:`by_reference<reference-form-types-by-reference>`为false，setTags()就可以使用了）::
    
        // src/Acme/TaskBundle/Entity/Task.php
        // ...

        public function setTags(ArrayCollection $tags)
        {
            foreach ($tags as $tag) {
                $tag->addTask($this);
            }

            $this->tags = $tags;
        }

    Inside ``Tag``, just make sure you have an ``addTask`` method::

        // src/Acme/TaskBundle/Entity/Tag.php
        // ...

        public function addTask(Task $task)
        {
            if (!$this->tasks->contains($task)) {
                $this->tasks->add($task);
            }
        }
    
    If you have a ``OneToMany`` relationship, then the workaround is similar,
    except that you can simply call ``setTask`` from inside ``setTags``.
    如果你使用OneToMany关系,方法是差不多的，但是你可以直接在setTags中使用setTask。

.. _cookbook-form-collections-remove:

Allowing tags to be removed
允许tag被删除
----------------------------

The next step is to allow the deletion of a particular item in the collection.
The solution is similar to allowing tags to be added.
第二步就是允许删除集合中的一个特定项。解决方案和添加差不多。

Start by adding the ``allow_delete`` option in the form Type::
在表单中添加allow_delete选项::
    
    // src/Acme/TaskBundle/Form/Type/TaskType.php
    // ...
    
    public function buildForm(FormBuilder $builder, array $options)
    {
        $builder->add('description');

        $builder->add('tags', 'collection', array(
            'type' => new TagType(),
            'allow_add' => true,
            'allow_delete' => true,
            'by_reference' => false,
        ));
    }

Templates Modifications
模板修改
~~~~~~~~~~~~~~~~~~~~~~~
    
The ``allow_delete`` option has one consequence: if an item of a collection 
isn't sent on submission, the related data is removed from the collection
on the server. The solution is thus to remove the form element from the DOM.
allow_delete选项只有一个结果：如果发送表单的时候集合中的某个项没有被提交，那么这个项就会被
从集合中删除。解决方案就是将表单元素从DOM中删除。

First, add a "delete this tag" link to each tag form:
在每个tag后添加一个"delete this tag"链接:

.. code-block:: javascript

    jQuery(document).ready(function() {
        // add a delete link to all of the existing tag form li elements
        collectionHolder.find('li').each(function() {
            addTagFormDeleteLink($(this));
        });
    
        // ... the rest of the block from above
    });
    
    function addTagForm() {
        // ...
        
        // add a delete link to the new form
        addTagFormDeleteLink($newFormLi);
    }

The ``addTagFormDeleteLink`` function will look something like this:
``addTagFormDeleteLink``方法会是这样的：

.. code-block:: javascript

    function addTagFormDeleteLink($tagFormLi) {
        var $removeFormA = $('<a href="#">delete this tag</a>');
        $tagFormLi.append($removeFormA);

        $removeFormA.on('click', function(e) {
            // prevent the link from creating a "#" on the URL
            e.preventDefault();

            // remove the li for the tag form
            $tagFormLi.remove();
        });
    }

When a tag form is removed from the DOM and submitted, the removed ``Tag`` object
will not be included in the collection passed to ``setTags``. Depending on
your persistence layer, this may or may not be enough to actually remove
the relationship between the removed ``Tag`` and ``Task`` object.
当一个tag表单被从DOM中移除并提交后，这个被移除的Tag对象不会包含在setTags的参数中。
但这个方法是否足以将Tag和Task的关系彻底删除还取决于你的persistence层。

.. sidebar:: Doctrine: Ensuring the database persistence

    When removing objects in this way, you may need to do a little bit more
    work to ensure that the relationship between the Task and the removed Tag
    is properly removed.
    当使用这种方法来删除对象时，你需要做一些工作来确保task和被移除的tag之间的关系（relationship）也被删除了。

    In Doctrine, you have two side of the relationship: the owning side and the
    inverse side. Normally in this case you'll have a ManyToMany relation
    and the deleted tags will disappear and persist correctly (adding new
    tags also works effortlessly).
    在doctrine中，关系有两方面：owning side和inverse side。

    But if you have an ``OneToMany`` relation or a ``ManyToMany`` with a
    ``mappedBy`` on the Task entity (meaning Task is the "inverse" side),
    you'll need to do more work for the removed tags to persist correctly.
    但如果你有一个OneToMany或ManyToMany关系且在Task实体类中有一个mappedBy属性（表示Task是inverse side），
    你还要做更多工作来确保正确地载入数据库。

    In this case, you can modify the controller to remove the relationship
    on the removed tag. This assumes that you have some ``editAction`` which
    is handling the "update" of your Task::
    本例中，你可以通过修改控制器来删除被移除tag和task的关系。这里假设你有一个``editAction``
    来处理Task中的update::

        // src/Acme/TaskBundle/Controller/TaskController.php
        // ...

        public function editAction($id, Request $request)
        {
            $em = $this->getDoctrine()->getManager();
            $task = $em->getRepository('AcmeTaskBundle:Task')->find($id);
    
            if (!$task) {
                throw $this->createNotFoundException('No task found for is '.$id);
            }

            // Create an array of the current Tag objects in the database
            foreach ($task->getTags() as $tag) $originalTags[] = $tag;
          
            $editForm = $this->createForm(new TaskType(), $task);

               if ('POST' === $request->getMethod()) {
                $editForm->bindRequest($this->getRequest());

                if ($editForm->isValid()) {
        
                    // filter $originalTags to contain tags no longer present
                    foreach ($task->getTags() as $tag) {
                        foreach ($originalTags as $key => $toDel) {
                            if ($toDel->getId() === $tag->getId()) {
                                unset($originalTags[$key]);
                            }
                        }
                    }

                    // remove the relationship between the tag and the Task
                    foreach ($originalTags as $tag) {
                        // remove the Task from the Tag
                        $tag->getTasks()->removeElement($task);
    
                        // if it were a ManyToOne relationship, remove the relationship like this
                        // $tag->setTask(null);
                        
                        $em->persist($tag);

                        // if you wanted to delete the Tag entirely, you can also do that
                        // $em->remove($tag);
                    }

                    $em->persist($task);
                    $em->flush();

                    // redirect back to some edit page
                    return $this->redirect($this->generateUrl('task_edit', array('id' => $id)));
                }
            }
            
            // render some form template
        }

    As you can see, adding and removing the elements correctly can be tricky.
    Unless you have a ManyToMany relationship where Task is the "owning" side,
    you'll need to do extra work to make sure that the relationship is properly
    updated (whether you're adding new tags or removing existing tags) on
    each Tag object itself.
    正确地添加和删除元素会有些难度。除非你有一个ManyToMany关系且task是owning side，否则你需要做一些
    额外的工作来确保正确地更新数据表之间的关系（当你添加或删除tag时）。


.. _`Owning Side and Inverse Side`: http://docs.doctrine-project.org/en/latest/reference/unitofwork-associations.html