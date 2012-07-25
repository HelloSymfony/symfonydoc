Using Data Transformers
使用数据转换
=======================

You'll often find the need to transform the data the user entered in a form into
something else for use in your program. You could easily do this manually in your
controller, but what if you want to use this specific form in different places?
你往往需要将用户输入到表单的数据转换成其他的东西。你可以在控制器中做这个工作，但是如果
你想在不同地方使用这个特定表单呢？

Say you have a one-to-one relation of Task to Issue, e.g. a Task optionally has an
issue linked to it. Adding a listbox with all possible issues can eventually lead to
a really long listbox in which it is impossible to find something. You'll rather want
to add a textbox, in which the user can simply enter the number of the issue. In the
controller you can convert this issue number to an actual task, and eventually add
errors to the form if it was not found, but of course this is not really clean.
假如你有一个一对一的task-issue数据库关系，一个task有一个issue联系到它。但如果添加一个包含所有issue的
列表框会产生非常长的列表，这样查找起来就会很不方便。你可以添加一个文本框，用户可以在里面输入一个代表
issue的数字，然后在控制器中将issue的数字转换为一个task，如果找不到，则在表单中添加错误信息。但是这样做
并不是很清爽。

It would be better if this issue was automatically looked up and converted to an
Issue object, for use in your action. This is where Data Transformers come into play.
假如能让这个issue自动查找并转换为一个issue对象就好了。这时就要用到Data Transformer。

First, create a custom form type which has a Data Transformer attached to it, which
returns the Issue by number: the issue selector type. Eventually this will simply be
a text field, as we configure the fields' parent to be a "text" field, in which you
will enter the issue number. The field will display an error if a non existing number
was entered::
首先创建一个附加了Data Transformer的表单类型（form type）,它会根据数字来返回issue：IssueSelectorType。
它最终会是一个简单的文本框，因为我们将这个字段的父类配置为text字段，你可以在这个字段中输入issue的数字。
如果输入了不存在的数字则会显示一个错误::

    // src/Acme/TaskBundle/Form/Type/IssueSelectorType.php
    namespace Acme\TaskBundle\Form\Type;

    use Symfony\Component\Form\AbstractType;
    use Symfony\Component\Form\FormBuilder;
    use Acme\TaskBundle\Form\DataTransformer\IssueToNumberTransformer;
    use Doctrine\Common\Persistence\ObjectManager;

    class IssueSelectorType extends AbstractType
    {
        /**
         * @var ObjectManager
         */
        private $om;

        /**
         * @param ObjectManager $om
         */
        public function __construct(ObjectManager $om)
        {
            $this->om = $om;
        }

        public function buildForm(FormBuilder $builder, array $options)
        {
            $transformer = new IssueToNumberTransformer($this->om);
            $builder->appendClientTransformer($transformer);
        }

        public function getDefaultOptions()
        {
            return array(
                'invalid_message' => 'The selected issue does not exist',
            );
        }

        public function getParent(array $options)
        {
            return 'text';
        }

        public function getName()
        {
            return 'issue_selector';
        }
    }

.. tip::

    You can also use transformers without creating a new custom form type
    by calling ``appendClientTransformer`` on any field builder::
    你还可以不创建表单类型，而通过在字段builder中添加``appendClientTransformer``来进行转换::

        use Acme\TaskBundle\Form\DataTransformer\IssueToNumberTransformer;

        class TaskType extends AbstractType
        {
            public function buildForm(FormBuilder $builder, array $options)
            {
                // ...

                // this assumes that the entity manager was passed in as an option
                $entityManager = $options['em'];
                $transformer = new IssueToNumberTransformer($entityManager);

                // use a normal text field, but transform the text into an issue object
                $builder
                    ->add('issue', 'text')
                    ->appendClientTransformer($transformer)
                ;
            }

            // ...
        }

Next, we create the data transformer, which does the actual conversion::
下面我们创建数据转换器，它会做这个转换的工作::

    // src/Acme/TaskBundle/Form/DataTransformer/IssueToNumberTransformer.php

    namespace Acme\TaskBundle\Form\DataTransformer;

    use Symfony\Component\Form\DataTransformerInterface;
    use Symfony\Component\Form\Exception\TransformationFailedException;
    use Doctrine\Common\Persistence\ObjectManager;
    use Acme\TaskBundle\Entity\Issue;

    class IssueToNumberTransformer implements DataTransformerInterface
    {
        /**
         * @var ObjectManager
         */
        private $om;

        /**
         * @param ObjectManager $om
         */
        public function __construct(ObjectManager $om)
        {
            $this->om = $om;
        }

        /**
         * Transforms an object (issue) to a string (number).
         *
         * @param  Issue|null $issue
         * @return string
         */
        public function transform($issue)
        {
            if (null === $issue) {
                return "";
            }

            return $issue->getNumber();
        }

        /**
         * Transforms a string (number) to an object (issue).
         *
         * @param  string $number
         * @return Issue|null
         * @throws TransformationFailedException if object (issue) is not found.
         */
        public function reverseTransform($number)
        {
            if (!$number) {
                return null;
            }

            $issue = $this->om
                ->getRepository('AcmeTaskBundle:Issue')
                ->findOneBy(array('number' => $number))
            ;

            if (null === $issue) {
                throw new TransformationFailedException(sprintf(
                    'An issue with number "%s" does not exist!',
                    $number
                ));
            }

            return $issue;
        }
    }

Finally, since we've decided to create a custom form type that uses the data
transformer, register the Type in the service container, so that the entity
manager can be automatically injected:
最后，由于我们创建了使用数据转换器的表单类型，需要在服务容器中注册这个类型以
自动注入entity manager:

.. configuration-block::

    .. code-block:: yaml

        services:
            acme_demo.type.issue_selector:
                class: Acme\TaskBundle\Form\Type\IssueSelectorType
                arguments: ["@doctrine.orm.entity_manager"]
                tags:
                    - { name: form.type, alias: issue_selector }

    .. code-block:: xml

        <service id="acme_demo.type.issue_selector" class="Acme\TaskBundle\Form\Type\IssueSelectorType">
            <argument type="service" id="doctrine.orm.entity_manager"/>
            <tag name="form.type" alias="issue_selector" />
        </service>

You can now add the type to your form by its alias as follows::
现在你可以在你的表单中添加这个类型了（使用alias）::

    // src/Acme/TaskBundle/Form/Type/TaskType.php

    namespace Acme\TaskBundle\Form\Type;

    use Symfony\Component\Form\AbstractType;
    use Symfony\Component\Form\FormBuilder;

    class TaskType extends AbstractType
    {
        public function buildForm(FormBuilder $builder, array $options)
        {
            $builder
                ->add('task')
                ->add('dueDate', null, array('widget' => 'single_text'));
                ->add('issue', 'issue_selector')
            ;
        }

        public function getName()
        {
            return 'task';
        }
    }

Now it will be very easy at any random place in your application to use this
selector type to select an issue by number. No logic has to be added to your
Controller at all.
现在就可以在你的应用中的任何地方所使用这个表单类型了，它可以通过选择数字来选择issue，
不需要在你的控制器中添加逻辑。

If you want a new issue to be created when an unknown number is entered, you
can instantiate it rather than throwing the TransformationFailedException, and
even persist it to your entity manager if the task has no cascading options
for the issue.
如果你想要在一个不可知的数字输入时创建新的issue，你可以将它初始化，假如这个task没有针对issue的cascade，
还可以将它persist到你的entity manager，而不是抛出TransformationFailedException。
