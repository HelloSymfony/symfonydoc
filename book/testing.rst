.. index::
   single: Tests

Testing
测试
=======

Whenever you write a new line of code, you also potentially add new bugs.
To build better and more reliable applications, you should test your code
using both functional and unit tests.
每当你多写一行代码，你就可能多一个错误。要创建一个稳定的应用，你还应该使用功能测试和单元测试。

The PHPUnit Testing Framework
PHPUnit测试框架
-----------------------------

Symfony2 integrates with an independent library - called PHPUnit - to give
you a rich testing framework. This chapter won't cover PHPUnit itself, but
it has its own excellent `documentation`_.
symfony2与一个名叫PHPUnit的独立库集成，从而使你能够更方便的进行测试工作。
本章不会讲述PHPUnit如何工作，但它有自己的文档，参见`documentation`_。

.. note::

    Symfony2 works with PHPUnit 3.5.11 or later, though version 3.6.4 is
    needed to test the Symfony core code itself.
    symfony2使用PHPUnit3.5.11以上的版本，但你需要3.6.4版本才能测试symfony2的核心代码。

Each test - whether it's a unit test or a functional test - is a PHP class
that should live in the `Tests/` subdirectory of your bundles. If you follow
this rule, then you can run all of your application's tests with the following
command:
每个测试——不管它是单元测试还是功能测试——都是一个php类，你必须将这个类放置在你的bundle的Tests/子目录
下。这样你就可以使用以下命令行来运行你应用的所有测试：

.. code-block:: bash

    # specify the configuration directory on the command line
    $ phpunit -c app/

The ``-c`` option tells PHPUnit to look in the ``app/`` directory for a configuration
file. If you're curious about the PHPUnit options, check out the ``app/phpunit.xml.dist``
file.
-c选项告诉PHPUnit要查看app/目录中有没有配置文件。如果你想知道PHPUnit的选项，可以查看``app/phpunit.xml.dist``文件。

.. tip::

    Code coverage can be generated with the ``--coverage-html`` option.
    如果你想集成代码覆盖（code coverage），可以运行``--coverage-html``选项。

.. index::
   single: Tests; Unit Tests

Unit Tests
单元测试
----------

A unit test is usually a test against a specific PHP class. If you want to
test the overall behavior of your application, see the section about `Functional Tests`_.
一个单元测试通常是对一个php类的测试。如果你想测试应用的所有行为，参见`Functional Tests`_。

Writing Symfony2 unit tests is no different than writing standard PHPUnit
unit tests. Suppose, for example, that you have an *incredibly* simple class
called ``Calculator`` in the ``Utility/`` directory of your bundle::
编写一个symfony2单元测试和编写一个标准的PHPUnit单元测试一样。比如，假设你在bundle的Utility/目录下
有一个非常简单的Calculator类::

    // src/Acme/DemoBundle/Utility/Calculator.php
    namespace Acme\DemoBundle\Utility;
    
    class Calculator
    {
        public function add($a, $b)
        {
            return $a + $b;
        }
    }

To test this, create a ``CalculatorTest`` file in the ``Tests/Utility`` directory
of your bundle::
要测试它，只要在bundle的Tests/Utility中创建一个CalculatorTest文件就可以了::

    // src/Acme/DemoBundle/Tests/Utility/CalculatorTest.php
    namespace Acme\DemoBundle\Tests\Utility;

    use Acme\DemoBundle\Utility\Calculator;

    class CalculatorTest extends \PHPUnit_Framework_TestCase
    {
        public function testAdd()
        {
            $calc = new Calculator();
            $result = $calc->add(30, 12);

            // assert that our calculator added the numbers correctly!
            $this->assertEquals(42, $result);
        }
    }

.. note::

    By convention, the ``Tests/`` sub-directory should replicate the directory
    of your bundle. So, if you're testing a class in your bundle's ``Utility/``
    directory, put the test in the ``Tests/Utility/`` directory.
    通常情况下，Tests/子目录必须复制你bundle的目录，所以如果你要测试bundle中Utility/目录的
    类，就将测试放在Tests/Utility目录下。

Just like in your real application - autoloading is automatically enabled
via the ``bootstrap.php.cache`` file (as configured by default in the ``phpunit.xml.dist``
file).
就像一个真正应用一样，autoloading通过``bootstrap.php.cache``文件自动激活（这在``phpunit.xml.dist``文件中
是默认配置）。

Running tests for a given file or directory is also very easy:
对一个给定的文件或命令执行测试非常简单：

.. code-block:: bash

    # run all tests in the Utility directory
    $ phpunit -c app src/Acme/DemoBundle/Tests/Utility/

    # run tests for the Calculator class
    $ phpunit -c app src/Acme/DemoBundle/Tests/Utility/CalculatorTest.php

    # run all tests for the entire Bundle
    $ phpunit -c app src/Acme/DemoBundle/

.. index::
   single: Tests; Functional Tests

Functional Tests
功能测试
----------------

Functional tests check the integration of the different layers of an
application (from the routing to the views). They are no different from unit
tests as far as PHPUnit is concerned, but they have a very specific workflow:
功能测试检查一个应用的不同层（从路径到模板）。它们的原理和单元测试一样，只不过流程不同：

* 创建请求;
* 测试响应;
* 点击链接或发送表单;
* 测试响应;
* 清洗并重来。

Your First Functional Test
你的第一个功能测试
~~~~~~~~~~~~~~~~~~~~~~~~~~

Functional tests are simple PHP files that typically live in the ``Tests/Controller``
directory of your bundle. If you want to test the pages handled by your
``DemoController`` class, start by creating a new ``DemoControllerTest.php``
file that extends a special ``WebTestCase`` class.
功能测试都是放置在``Tests/Controller``目录下的简单php文件。如果你需要测试一个``DemoController``
类处理的页面，首先，你要创建一个新的``DemoControllerTest.php``文件并扩展``WebTestCase``类。 

For example, the Symfony2 Standard Edition provides a simple functional test
for its ``DemoController`` (`DemoControllerTest`_) that reads as follows::
symfony2里面有个为它的``DemoController``测试的文件(`DemoControllerTest`_)：

    // src/Acme/DemoBundle/Tests/Controller/DemoControllerTest.php
    namespace Acme\DemoBundle\Tests\Controller;

    use Symfony\Bundle\FrameworkBundle\Test\WebTestCase;

    class DemoControllerTest extends WebTestCase
    {
        public function testIndex()
        {
            $client = static::createClient();

            $crawler = $client->request('GET', '/demo/hello/Fabien');

            $this->assertTrue($crawler->filter('html:contains("Hello Fabien")')->count() > 0);
        }
    }

.. tip::

    To run your functional tests, the ``WebTestCase`` class bootstraps the
    kernel of your application. In most cases, this happens automatically.
    However, if your kernel is in a non-standard directory, you'll need
    to modify your ``phpunit.xml.dist`` file to set the ``KERNEL_DIR`` environment
    variable to the directory of your kernel::
    WebTestCase类引导你应用的核心（kernel），从而执行你的功能测试。大多数情况下，这些是自动完成的。
    但是，如果你的kernel不在标准目录下，你还要修改你的``phpunit.xml.dist``文件，将``KERNEL_DIR``环境变量
    设置为你的kernel的目录::

        <phpunit>
            <!-- ... -->
            <php>
                <server name="KERNEL_DIR" value="/path/to/your/app/" />
            </php>
            <!-- ... -->
        </phpunit>

The ``createClient()`` method returns a client, which is like a browser that
you'll use to crawl your site::
``createClient()``方法返回一个客户端，它会像浏览器一样针对你的网站做一些动作::

    $crawler = $client->request('GET', '/demo/hello/Fabien');

The ``request()`` method (see :ref:`more about the request method<book-testing-request-method-sidebar>`)
returns a :class:`Symfony\\Component\\DomCrawler\\Crawler` object which can
be used to select elements in the Response, click on links, and submit forms.
request()方法(参见 :ref:`more about the request method<book-testing-request-method-sidebar>`)返回一个
:class:`Symfony\\Component\\DomCrawler\\Crawler`对象，它可以在响应中选取变量，点击链接，并提交表单。

.. tip::

    The Crawler only works when the response is an XML or an HTML document.
    To get the raw content response, call ``$client->getResponse()->getContent()``.
    Crawl只在response是一个XML或一个HTML文件的时候能够工作。要返回原始的响应内容，执行``$client->getResponse()->getContent()``。

Click on a link by first selecting it with the Crawler using either an XPath
expression or a CSS selector, then use the Client to click on it. For example,
the following code finds all links with the text ``Greet``, then selects
the second one, and ultimately clicks on it::
通过使用XPath或CSS选择器选中一个链接就可以使用Client点击它。比如，以下的代码会查找所有包含Greet的链接，然后
选择第二个并点击它::

    $link = $crawler->filter('a:contains("Greet")')->eq(1)->link();

    $crawler = $client->click($link);

Submitting a form is very similar; select a form button, optionally override
some form values, and submit the corresponding form::
提交一个表单也很类似，选择一个表单按钮，输入表单的值，然后提交相应的表单::

    $form = $crawler->selectButton('submit')->form();

    // set some values
    $form['name'] = 'Lucas';
    $form['form_name[subject]'] = 'Hey there!';

    // submit the form
    $crawler = $client->submit($form);

.. tip::

    The form can also handle uploads and contains methods to fill in different types
    of form fields (e.g. ``select()`` and ``tick()``). For details, see the
    `Forms`_ section below.
    表单还可以处理上传并包含了填充不同类型字段的方法（e.g. ``select()``和``tick()``）。详情请见
    下面的`Forms`_一节。

Now that you can easily navigate through an application, use assertions to test
that it actually does what you expect it to. Use the Crawler to make assertions
on the DOM::
现在你可以自由的游历一个应用了，你可以对DOM使用判断语句（assertion）来测试它是否根据你希望的
方式来进行::

    // Assert that the response matches a given CSS selector.
    $this->assertTrue($crawler->filter('h1')->count() > 0);

Or, test against the Response content directly if you just want to assert that
the content contains some text, or if the Response is not an XML/HTML
document::
或者如果你想要判断响应内容是否包含某些文字，或响应是否是XML/HTML文档::

    $this->assertRegExp('/Hello Fabien/', $client->getResponse()->getContent());

.. _book-testing-request-method-sidebar:

.. sidebar:: More about the ``request()`` method:

    The full signature of the ``request()`` method is::

        request(
            $method,
            $uri, 
            array $parameters = array(), 
            array $files = array(), 
            array $server = array(), 
            $content = null, 
            $changeHistory = true
        )

    server数组就是你可以在php的`$_SERVER`_全局变量中找到的值。比如，要设置`Content-Type`和
    `Referer` HTTP头文件，你可以::

        $client->request(
            'GET',
            '/demo/hello/Fabien',
            array(),
            array(),
            array(
                'CONTENT_TYPE' => 'application/json',
                'HTTP_REFERER' => '/foo/bar',
            )
        );

.. index::
   single: Tests; Assertions

.. sidebar: Useful Assertions

    To get you started faster, here is a list of the most common and
    useful test assertions::
    以下是一系列的常用测试判断语句::

        // Assert that there is more than one h2 tag with the class "subtitle"
        $this->assertTrue($crawler->filter('h2.subtitle')->count() > 0);

        // Assert that there are exactly 4 h2 tags on the page
        $this->assertEquals(4, $crawler->filter('h2')->count());

        // Assert the the "Content-Type" header is "application/json"
        $this->assertTrue($client->getResponse()->headers->contains('Content-Type', 'application/json'));

        // Assert that the response content matches a regexp.
        $this->assertRegExp('/foo/', $client->getResponse()->getContent());

        // Assert that the response status code is 2xx
        $this->assertTrue($client->getResponse()->isSuccessful());
        // Assert that the response status code is 404
        $this->assertTrue($client->getResponse()->isNotFound());
        // Assert a specific 200 status code
        $this->assertEquals(200, $client->getResponse()->getStatusCode());

        // Assert that the response is a redirect to /demo/contact
        $this->assertTrue($client->getResponse()->isRedirect('/demo/contact'));
        // or simply check that the response is a redirect to any URL
        $this->assertTrue($client->getResponse()->isRedirect());

.. index::
   single: Tests; Client

Working with the Test Client
使用测试客户端
-----------------------------

The Test Client simulates an HTTP client like a browser and makes requests
into your Symfony2 application::
测试客户端（The Test Client）创建了一个像浏览器一样工作的HTTP客户端，并在你的symfony2应用中创建请求::

    $crawler = $client->request('GET', '/hello/Fabien');

The ``request()`` method takes the HTTP method and a URL as arguments and
returns a ``Crawler`` instance.
request()方法使用这个HTTP方法，并将URL作为参数，然后返回了一个Crawler实例。

Use the Crawler to find DOM elements in the Response. These elements can then
be used to click on links and submit forms::
使用Crawler在响应中查找DOM元素。这些元素可以被用来点击链接并提交表单::

    $link = $crawler->selectLink('Go elsewhere...')->link();
    $crawler = $client->click($link);

    $form = $crawler->selectButton('validate')->form();
    $crawler = $client->submit($form, array('name' => 'Fabien'));

The ``click()`` and ``submit()`` methods both return a ``Crawler`` object.
These methods are the best way to browse your application as it takes care
of a lot of things for you, like detecting the HTTP method from a form and
giving you a nice API for uploading files.
``click()``和``submit()``方法都返回一个Crawler对象。这些方法对于浏览你的应用都很有效，
因为它们可以为你做很多工作，比如从表单中检测HTTP方法并给你一个API来上传文件。

.. tip::

    You will learn more about the ``Link`` and ``Form`` objects in the
    :ref:`Crawler<book-testing-crawler>` section below.
    在下面的:ref:`Crawler<book-testing-crawler>`一节中你将学习更多关于Link和Form对象的知识。

The ``request`` method can also be used to simulate form submissions directly
or perform more complex requests::
request方法可以被用来直接提交表单或做其他复杂工作::

    // Directly submit a form (but using the Crawler is easier!)
    $client->request('POST', '/submit', array('name' => 'Fabien'));

    // Form submission with a file upload
    use Symfony\Component\HttpFoundation\File\UploadedFile;

    $photo = new UploadedFile(
        '/path/to/photo.jpg',
        'photo.jpg',
        'image/jpeg',
        123
    );
    // or
    $photo = array(
        'tmp_name' => '/path/to/photo.jpg',
        'name' => 'photo.jpg',
        'type' => 'image/jpeg',
        'size' => 123,
        'error' => UPLOAD_ERR_OK
    );
    $client->request(
        'POST',
        '/submit',
        array('name' => 'Fabien'),
        array('photo' => $photo)
    );

    // Perform a DELETE requests, and pass HTTP headers
    $client->request(
        'DELETE',
        '/post/12',
        array(),
        array(),
        array('PHP_AUTH_USER' => 'username', 'PHP_AUTH_PW' => 'pa$$word')
    );

Last but not least, you can force each request to be executed in its own PHP
process to avoid any side-effects when working with several clients in the same
script::
还有，你可以强迫每个请求都在它自己的php过程中执行（隔离），这样可以避免多个客户端在同一个脚本中一起执行的副作用::

    $client->insulate();

Browsing
浏览
~~~~~~~~

The Client supports many operations that can be done in a real browser::
客户端支持许多可以在真实的浏览器中完成的工作::

    $client->back();
    $client->forward();
    $client->reload();

    // Clears all cookies and the history
    $client->restart();

Accessing Internal Objects
访问内部对象
~~~~~~~~~~~~~~~~~~~~~~~~~~

If you use the client to test your application, you might want to access the
client's internal objects::
如果你使用客户端来测试你的应用，你你会需要访问客户端的内部对象::

    $history   = $client->getHistory();
    $cookieJar = $client->getCookieJar();

You can also get the objects related to the latest request::
你还可以获取关于最新请求的对象::

    $request  = $client->getRequest();
    $response = $client->getResponse();
    $crawler  = $client->getCrawler();

If your requests are not insulated, you can also access the ``Container`` and
the ``Kernel``::
如果你的请求没有被隔离，你还可以访问Container和Kernel::

    $container = $client->getContainer();
    $kernel    = $client->getKernel();

Accessing the Container
访问Container
~~~~~~~~~~~~~~~~~~~~~~~

It's highly recommended that a functional test only tests the Response. But
under certain very rare circumstances, you might want to access some internal
objects to write assertions. In such cases, you can access the dependency
injection container::
强烈推荐功能测试仅用来来测试响应（Response）。但是在极少数情况下你还可能需要访问内部
对象来编写判断语句。在这种情况下，你可以访问dependency
injection container::

    $container = $client->getContainer();

Be warned that this does not work if you insulate the client or if you use an
HTTP layer. For a list of services available in your application, use the
``container:debug`` console task.
要注意如果你隔离了客户端或者使用了HTTP层，这个方法是不能使用的。使用``container:debug``命令行
来查看你应用中的可用服务。

.. tip::

    If the information you need to check is available from the profiler, use
    it instead.
    如果你需要检查的信息在profiler中存在，你就查看profiler（参见http://symfony.com/doc/current/book/internals.html#visualizing-profiling-data）。

Accessing the Profiler Data
访问profiler数据
~~~~~~~~~~~~~~~~~~~~~~~~~~~

On each request, the Symfony profiler collects and stores a lot of data about
the internal handling of that request. For example, the profiler could be
used to verify that a given page executes less than a certain number of database
queries when loading.
对于每个请求，symfonyprofiler都会存储许多关于内部处理那个请求的数据。比如，profiler
可以确认一个给定的页面执行了比某个数量少的数据库请求。

To get the Profiler for the last request, do the following::
要获取最新请求的profiler，可以::

    $profile = $client->getProfile();

For specific details on using the profiler inside a test, see the
:doc:`/cookbook/testing/profiling` cookbook entry.
要了解更多关于在测试中使用profiler的信息，参见:doc:`/cookbook/testing/profiling`。

Redirecting
重定向
~~~~~~~~~~~

When a request returns a redirect response, the client does not follow
it automatically. You can examine the response and force a redirection
afterwards  with the ``followRedirect()`` method::
当一个请求返回了一个重定向响应时，客户端并不会自动跟随。你可以检查响应并通过``followRedirect()``方法强制
一个重定向::

    $crawler = $client->followRedirect();
    
If you want the client to automatically follow all redirects, you can 
force him with the ``followRedirects()`` method::
如果你需要客户端自动跟随所有的重定向，可以::

    $client->followRedirects();

.. index::
   single: Tests; Crawler

.. _book-testing-crawler:

The Crawler
-----------

A Crawler instance is returned each time you make a request with the Client.
It allows you to traverse HTML documents, select nodes, find links and forms.
每当你请求Client时，一个Crawler实例就会被返回。它允许你遍历HTML文档，选择节点，查找链接和表单。

Traversing
遍历
~~~~~~~~~~

Like jQuery, the Crawler has methods to traverse the DOM of an HTML/XML
document. For example, the following finds all ``input[type=submit]`` elements,
selects the last one on the page, and then selects its immediate parent element::
像jQuery一样，Crawler可以通过一些方法来遍历HTML/XML的DOM。比如，下面的代码查找所有的``input[type=submit]``
元素，选择页面上最后一个元素，并选择它的父元素::

    $newCrawler = $crawler->filter('input[type=submit]')
        ->last()
        ->parents()
        ->first()
    ;

Many other methods are also available:
还有许多其他方法：

+------------------------+----------------------------------------------------+
| Method                 | Description                                        |
+========================+====================================================+
| ``filter('h1.title')`` | Nodes that match the CSS selector                  |
+------------------------+----------------------------------------------------+
| ``filterXpath('h1')``  | Nodes that match the XPath expression              |
+------------------------+----------------------------------------------------+
| ``eq(1)``              | Node for the specified index                       |
+------------------------+----------------------------------------------------+
| ``first()``            | First node                                         |
+------------------------+----------------------------------------------------+
| ``last()``             | Last node                                          |
+------------------------+----------------------------------------------------+
| ``siblings()``         | Siblings                                           |
+------------------------+----------------------------------------------------+
| ``nextAll()``          | All following siblings                             |
+------------------------+----------------------------------------------------+
| ``previousAll()``      | All preceding siblings                             |
+------------------------+----------------------------------------------------+
| ``parents()``          | Returns the parent nodes                           |
+------------------------+----------------------------------------------------+
| ``children()``         | Returns children nodes                             |
+------------------------+----------------------------------------------------+
| ``reduce($lambda)``    | Nodes for which the callable does not return false |
+------------------------+----------------------------------------------------+

Since each of these methods returns a new ``Crawler`` instance, you can
narrow down your node selection by chaining the method calls::
由于这些方法每个都返回一个新的Crawler实例，你可以缩小节点的选择范围::

    $crawler
        ->filter('h1')
        ->reduce(function ($node, $i)
        {
            if (!$node->getAttribute('class')) {
                return false;
            }
        })
        ->first();

.. tip::

    Use the ``count()`` function to get the number of nodes stored in a Crawler:
    ``count($crawler)``
    使用count()方法来获取Crawler中节点的数量：count($crawler)

Extracting Information
提取信息
~~~~~~~~~~~~~~~~~~~~~~

The Crawler can extract information from the nodes::
Crawler可以从节点中提取信息::

    // Returns the attribute value for the first node
    $crawler->attr('class');

    // Returns the node value for the first node
    $crawler->text();

    // Extracts an array of attributes for all nodes (_text returns the node value)
    // returns an array for each element in crawler, each with the value and href
    $info = $crawler->extract(array('_text', 'href'));

    // Executes a lambda for each node and return an array of results
    $data = $crawler->each(function ($node, $i)
    {
        return $node->attr('href');
    });

Links
链接
~~~~~

To select links, you can use the traversing methods above or the convenient
``selectLink()`` shortcut::
要选择链接，你可以使用遍历方法或便捷的selectLink()方法::

    $crawler->selectLink('Click here');

This selects all links that contain the given text, or clickable images for
which the ``alt`` attribute contains the given text. Like the other filtering
methods, this returns another ``Crawler`` object.
这可以选择所有的包含给定文本的链接，或者alt属性中包含有给定文本的图像链接。这也会
返回一个Crawler对象。

Once you've selected a link, you have access to a special ``Link`` object,
which has helpful methods specific to links (such as ``getMethod()`` and
``getUri()``). To click on the link, use the Client's ``click()`` method
and pass it a ``Link`` object::
一旦你选择了一个链接，你就可以访问一个特定的Link对象了，这个对象有一些针对链接的方法（如getMethod()和getUri()）。
要点击这些链接，可以使用Client的click()方法并将它传递给Link对象::

    $link = $crawler->selectLink('Click here')->link();

    $client->click($link);

Forms
表单
~~~~~

Just like links, you select forms with the ``selectButton()`` method::
像链接一样，你使用selectButton()方法来选择表单::

    $buttonCrawlerNode = $crawler->selectButton('submit');

.. note::

    Notice that we select form buttons and not forms as a form can have several
    buttons; if you use the traversing API, keep in mind that you must look for a
    button.
    注意我们选择表单的button而不是表单，因为表单可能有多个button；如果你使用遍历API，记住你必须选择button。

The ``selectButton()`` method can select ``button`` tags and submit ``input``
tags. It uses several different parts of the buttons to find them:
selectButton()方法可以选择button标签并提交input标签。它使用button的几个不同部分来查找它：

* ``value``属性值;

* 图像的``id``或``alt``属性值 ;

* button标签的``id``或``name``属性值。

Once you have a Crawler representing a button, call the ``form()`` method
to get a ``Form`` instance for the form wrapping the button node::
一旦你有了一个代表button的Crawler实例，就可以通过执行form()方法来获取一个包围这个button节点的Form实例::

    $form = $buttonCrawlerNode->form();

When calling the ``form()`` method, you can also pass an array of field values
that overrides the default ones::
当执行form()方法时，你还可以传递覆盖默认值的字段值::

    $form = $buttonCrawlerNode->form(array(
        'name'              => 'Fabien',
        'my_form[subject]'  => 'Symfony rocks!',
    ));

And if you want to simulate a specific HTTP method for the form, pass it as a
second argument::
如果你想对表单模拟一个特定的HTTP方法，给它传递第二个参数::

    $form = $buttonCrawlerNode->form(array(), 'DELETE');

The Client can submit ``Form`` instances::
Client可以提交Form对象::

    $client->submit($form);

The field values can also be passed as a second argument of the ``submit()``
method::
字段值还可以作为第二个参数传递给submit()方法::

    $client->submit($form, array(
        'name'              => 'Fabien',
        'my_form[subject]'  => 'Symfony rocks!',
    ));

For more complex situations, use the ``Form`` instance as an array to set the
value of each field individually::
对于更复杂的情况，可以将Form对象作为数组来设置每个字段的值::

    // Change the value of a field
    $form['name'] = 'Fabien';
    $form['my_form[subject]'] = 'Symfony rocks!';

There is also a nice API to manipulate the values of the fields according to
their type::
还有API可以根据字段类型操作字段的值::

    // Select an option or a radio
    $form['country']->select('France');

    // Tick a checkbox
    $form['like_symfony']->tick();

    // Upload a file
    $form['photo']->upload('/path/to/lucas.jpg');

.. tip::

    You can get the values that will be submitted by calling the ``getValues()``
    method on the ``Form`` object. The uploaded files are available in a
    separate array returned by ``getFiles()``. The ``getPhpValues()`` and
    ``getPhpFiles()`` methods also return the submitted values, but in the
    PHP format (it converts the keys with square brackets notation - e.g.
    ``my_form[subject]`` - to PHP arrays).
    你可以通过getValues()方法来获取Form对象上要提交的数据。上传的文件都存放在getValues()返回的
    一个数组中。getPhpValues()和getPhpFiles()方法也返回提交的值，但是是以php格式的形式（它将key转换成为
    方括号，比如my_form[subject]）。

.. index::
   pair: Tests; Configuration

Testing Configuration
测试配置
---------------------

The Client used by functional tests creates a Kernel that runs in a special
``test`` environment. Since Symfony loads the ``app/config/config_test.yml``
in the ``test`` environment, you can tweak any of your application's settings
specifically for testing.
功能测试的Client创建了一个在test环境中运行的kernel。由于symfony在test环境中载入的是
``app/config/config_test.yml``，你可以将应用中的配置改变一下以适应测试。

For example, by default, the swiftmailer is configured to *not* actually
deliver emails in the ``test`` environment. You can see this under the ``swiftmailer``
configuration option:
比如，默认情况下，swiftmail在test环境下被配置为不自动发送邮件。你可以在swiftmailer下看见：

.. configuration-block::

    .. code-block:: yaml

        # app/config/config_test.yml
        # ...

        swiftmailer:
            disable_delivery: true

    .. code-block:: xml

        <!-- app/config/config_test.xml -->
        <container>
            <!-- ... -->

            <swiftmailer:config disable-delivery="true" />
        </container>

    .. code-block:: php

        // app/config/config_test.php
        // ...

        $container->loadFromExtension('swiftmailer', array(
            'disable_delivery' => true
        ));

You can also use a different environment entirely, or override the default
debug mode (``true``) by passing each as options to the ``createClient()``
method::
你可以给createClient()方法传递参数，以使用一个不同的环境，或覆盖默认的调试模式（true）::

    $client = static::createClient(array(
        'environment' => 'my_test_env',
        'debug'       => false,
    ));

If your application behaves according to some HTTP headers, pass them as the
second argument of ``createClient()``::
如果你的应用根据一些HTTP头文件来动作，可以将它们作为createClient()的第二个参数传递::

    $client = static::createClient(array(), array(
        'HTTP_HOST'       => 'en.example.com',
        'HTTP_USER_AGENT' => 'MySuperBrowser/1.0',
    ));

You can also override HTTP headers on a per request basis::
你还可以根据每个请求来覆盖HTTP头文件::

    $client->request('GET', '/', array(), array(), array(
        'HTTP_HOST'       => 'en.example.com',
        'HTTP_USER_AGENT' => 'MySuperBrowser/1.0',
    ));

.. tip::

    The test client is available as a service in the container in the ``test``
    environment (or wherever the :ref:`framework.test<reference-framework-test>`
    option is enabled). This means you can override the service entirely
    if you need to.
    test client在test环境中是作为容器中的一个服务存在的（或者任何:ref:`framework.test<reference-framework-test>`被激活的地方）。
    这表示你可以覆盖整个test服务。

.. index::
   pair: PHPUnit; Configuration

PHPUnit Configuration
PHPUnit配置
~~~~~~~~~~~~~~~~~~~~~

Each application has its own PHPUnit configuration, stored in the
``phpunit.xml.dist`` file. You can edit this file to change the defaults or
create a ``phpunit.xml`` file to tweak the configuration for your local machine.
每个应用都有自己的PHPUnit配置，它存放在``phpunit.xml.dist``文件中。你可以通过编辑这个文件来修改它的
默认配置或创建一个phpunit.xml文件来为你的本地机器修改配置。

.. tip::

    Store the ``phpunit.xml.dist`` file in your code repository, and ignore the
    ``phpunit.xml`` file.
    将``phpunit.xml.dist``文件存放在你的bundle中，并忽略phpunit.xml文件。

By default, only the tests stored in "standard" bundles are run by the
``phpunit`` command (standard being tests in the ``src/*/Bundle/Tests`` or
``src/*/Bundle/*Bundle/Tests`` directories) But you can easily add more
directories. For instance, the following configuration adds the tests from
the installed third-party bundles:
默认情况下，只有存放在标准bundle的测试（也就是存放在``src/*/Bundle/Tests``或``src/*/Bundle/*Bundle/Tests``目录下）
能够通过phpunit命令行运行。但你还可以添加更多目录。比如，以下的配置会从被安装的第三方bundle中添加测试：

.. code-block:: xml

    <!-- hello/phpunit.xml.dist -->
    <testsuites>
        <testsuite name="Project Test Suite">
            <directory>../src/*/*Bundle/Tests</directory>
            <directory>../src/Acme/Bundle/*Bundle/Tests</directory>
        </testsuite>
    </testsuites>

To include other directories in the code coverage, also edit the ``<filter>``
section:
要在代码覆盖（code coverage）中包含其他目录，还要编辑<filter>部分：

.. code-block:: xml

    <filter>
        <whitelist>
            <directory>../src</directory>
            <exclude>
                <directory>../src/*/*Bundle/Resources</directory>
                <directory>../src/*/*Bundle/Tests</directory>
                <directory>../src/Acme/Bundle/*Bundle/Resources</directory>
                <directory>../src/Acme/Bundle/*Bundle/Tests</directory>
            </exclude>
        </whitelist>
    </filter>

Learn more from the Cookbook
----------------------------

* :doc:`/cookbook/testing/http_authentication`
* :doc:`/cookbook/testing/insulating_clients`
* :doc:`/cookbook/testing/profiling`


.. _`DemoControllerTest`: https://github.com/symfony/symfony-standard/blob/master/src/Acme/DemoBundle/Tests/Controller/DemoControllerTest.php
.. _`$_SERVER`: http://php.net/manual/en/reserved.variables.server.php
.. _`documentation`: http://www.phpunit.de/manual/3.5/en/
