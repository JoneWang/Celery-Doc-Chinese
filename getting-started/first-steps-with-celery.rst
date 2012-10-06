.. _tut-celery:
.. _first-steps:

=========================
第一步
=========================

Celery是一个任务队列的控制器。它非常容易使用，所以起初你不用去学习解决问题中的复杂性......

Celery is a task queue with batteries included.
It is easy to use so that you can get started without learning
the full complexities of the problem it solves. It is designed
around best practices so that your product can scale
and integrate with other languages, and it comes with the
tools and support you need to run such a system in production.

In this tutorial you will learn the absolute basics of using Celery.
You will learn about;

- Choosing and installing a message broker.
- Installing Celery and creating your first task
- Starting the worker and calling tasks.
- Keeping track of tasks as they transition through different states,
  and inspecting return values.

Celery may seem daunting at first - but don't worry - this tutorial
will get you started in no time. It is deliberately kept simple, so
to not confuse you with advanced features.
After you have finished this tutorial
it's a good idea to browse the rest of the documentation,
for example the :ref:`next-steps` tutorial, which will
showcase Celery's capabilities.

.. contents::
    :local:

.. _celerytut-broker:

选择一个Broker
=================

Celery需要一个发送和接收消息的解决方案，通常是一个独立服务调用一个消息broker的
形式。

这里有几个可用的选择：

RabbitMQ
--------

`RabbitMQ`_ 是功能全面、稳定、持久和易于安装的，是生产环境中极佳的选择，关于在
Celery中使用RabbitMQ的详细信息:

    :ref:`broker-rabbitmq`

.. _`RabbitMQ`: http://www.rabbitmq.com/

如果你使用的是Ubuntu或者Debian系统来安装RabbitMQ，可以执行下面的命令::

.. code-block:: bash

    $ sudo apt-get install rabbitmq-server

在命令执行完毕时，broker就已经在后台准备好为传输消息了 。

如果你使用的不是Ubuntu或者Debian，你可以去网站上找到在其他系统上同样简单的安装
说明，其中也包括windows下的：

    http://www.rabbitmq.com/download.html

Redis
-----

`Redis`_ 的功能也很全面，但是在意外终止或者停电的情况下更容易数据丢失，关于
Redis的详细信息::

    :ref:`broker-redis`

.. _`Redis`: http://redis.io/


使用数据库
----------------

不推荐使用数据库作为消息队列，但是可以最小化安装，可以使用的库:

* :ref:`broker-sqlalchemy`
* :ref:`broker-django`

例如如果你准备用Django数据库，用它作为你的消息broker可以很方便的进行开发，即使
你的产品中你使用了更强大的系统。

其他broker
-------------

In addition to the above, there are other transport implementations
to choose from, including

* :ref:`Amazon SQS <broker-sqs>`
* :ref:`broker-mongodb`

See also `Transport Comparison`_.

.. _`Transport Comparison`: http://kombu.readthedocs.org/en/latest/introduction.html#transport-comparison

.. _celerytut-installation:

安装Celery
=================

Celery在pypi上，因此可以使用像 ``pip`` 或者 ``easy_install`` 这样的标准Python
工具进行安装:

.. code-block:: bash

    $ pip install celery

应用
===========

首先你需要的是一个Celery实例，这被称为celery应用程序或者app，此后由此实例作为
入口点在Celery中作你想做的任何事，像创建任务和管理worker，它必须能够被其他模块
导入。

这个教程里你将，但是在较大的项目里你需要创建一个 `dedicated module <project-layout>` 。

创建一个文件:file:`tasks.py`:

Let's create the file :file:`tasks.py`:

.. code-block:: python

    from celery import Celery

    celery = Celery('tasks', broker='amqp://guest@localhost//')

    @celery.task
    def add(x, y):
        return x + y

:class:`~celery.app.Celery` 的一个参数是当前模块(当前文件)的名称，此参数是
必需的，这样就可以自动生成名称；第二个参数是broker keyword[?]参数，它指定了你
所使用的broker的url，在这里默认使用的是RabbitMQ，这里(:ref:`celerytut-broker`)
有更多的选择，例如redis ``redis://localhost`` 或者 ``mongodb://localhost`` 。

你现在声明了一个单一的任务，叫做 ``add`` ，返回两个数字之和。

.. _celerytut-running-celeryd:

运行celery生产者服务器
================================

你现在通过执行我们的程序和 ``worker`` 参数来运行生产者:

.. code-block:: bash

    $ celery -A tasks worker --loglevel=info

在生产环境中，你可能会想以在后台以守护进程的方式运行生产者，要做到这点你需要使
用你平台所使用的工具或者 `supervisord`_ (了解相关信息 :ref:`daemonizing` )。

要察看完整的worker命令行可选参数，可以执行:

.. code-block:: bash

    $  celery worker --help

其他一些可用参数和帮助可以执行:

.. code-block:: bash

    $ celery help

.. _`supervisord`: http://supervisord.org

.. _celerytut-calling:

调用任务
================

要调用任务你可以使用 :meth:`~@Task.delay` 方法。

这是使用 :meth:`~@Task.apply_async` 方法的一条捷径，对任务的运行有着更好的
控制(查看 :ref:`guide-calling`)::

    >>> from tasks import add
    >>> add.delay(4, 4)

任务现在已经在你执行它的时候被生产者处理了，你可以通过在控制台输出亦一些信息来
验证一下。

调用一个任务会返回一个 :class:`~@AsyncResult` 实例，可以用于查看任务的状态，如
等待到执行完毕或者获取任务的返回值(如果任务失败会返回异常追踪信息)，但是这不是
默认启用的，必须使用一个resulut backend Celery。。。下一节将详细描述。

.. _celerytut-keeping-results:

保留结果
===============

如果想保留任务的状态轨迹，Celery需要存储或者发送状态到某处，有几个内建
result backend 的可选：`SQLAlchemy`_/`Django`_ ORM, `Memcached`_, `Redis`_,
AMQP (`RabbitMQ`_), and `MongoDB`_ -- 或者你自己定义。

.. _`Memcached`: http://memcached.org
.. _`MongoDB`: http://www.mongodb.org
.. _`SQLAlchemy`: http://www.sqlalchemy.org/
.. _`Django`: http://djangoproject.com

这个例子里会使用 `amqp` result backend 把状态作为消息发送，这需要通过 
在 :class:`@Celery` 中指定 ``backend`` 参数(如果你选择使用配置模块，也可以
设置 :setting:`CELERY_RESULT_BACKEND` 配置项)::

    celery = Celery('tasks', backend='amqp', broker='amqp://')

或者说你想使用Redis作为 result backend，却又想使用RabbitMQ作为消息broker (流行
的组合)，可以使用如下方式::

    celery = Celery('tasks', backend='redis://localhost', broker='amqp://')

了解更多关于 result backends 可以看 :ref:`task-result-backends` 。

现在我们配置好了 result backend，开始定义任务吧，这回当你调用任务时，
:class:`~@AsyncResult` 实例会被返回

    >>> result = add.delay(4, 4)

 :meth:`~@AsyncResult.ready` 方法会返回任务是否已经被处理完成::

    >>> result.ready()
    False

你也可以同步的方式等待结果完成，但是由于是将异步变成了同步的方式调用，所以这
种方式很少被使用::

    >>> result.get(timeout=1)
    4

假如任务抛出了一个异常， :meth:`~@AsyncResult.get` 也会抛出异常，但是你可以
指定 ``propagate`` 参数使其无效::

    >>> result.get(propagate=True)

如果任务抛出了一个异常，你也能追溯到错误来源::

    >>> result.traceback
    ...

这里 :mod:`celery.result` 可以查看完整的result对象信息。

.. _celerytut-configuration:

配置
=============

Celery 像一个用户应用一样不需要太多的操作，但你先你必须连接到broker和 result backend 
用于输入和输出消息，但如果你看......

Celery, like a consumer appliance doesn't need much to be operated.
It has an input and an output, where you must connect the input to a broker and maybe
the output to a result backend if so wanted.  But if you look closely at the back
there is a lid revealing lots of sliders, dials and buttons: this is the configuration.

默认配置适用于大多数的情况，但是有时候你需要对Celery做一些调整来适应你在实际工
作中的需要，可以查看 :ref:`configuration` 来熟悉一些配置项的作用。

配置可以直接或者使用专门的配置模块来设置。配置默认 序列化任务 序列化......

The configuration can be set on the app directly or by using a dedicated
configuration module.
As an example you can configure the default serializer used for serializing
task payloads by changing the :setting:`CELERY_TASK_SERIALIZER` setting:

.. code-block:: python

    celery.conf.CELERY_TASK_SERIALIZER = 'json'

如果你需要配置多项可以使用 ``update`` :

.. code-block:: python

    celery.conf.update(
        CELERY_TASK_SERIALIZER='json',
        CELERY_RESULT_SERIALIZER='json',
        CELERY_TIMEZONE='Europe/Oslo',
        CELERY_ENABLE_UTC=True,
    )

在较大项目中最好使用专门的配置模块，配置项都集中在一个模块中使使用者能方便的
进行配置以达到预期效果，你也可以想象一下，在你的系统发生问题时系统管理员可以
对其简单的修改。

调用:meth:`~@Celery.config_from_object` 方法可以告诉你的Celery实例所使用的配置
模块名称:

.. code-block:: python

    celery.config_from_object('celeryconfig')

这个模块一般被叫做"``celeryconfig``"，不过你也可以取其他名称。

从当前文件夹或者Python的模块搜索路径寻找名为 ``celeryconfig.py`` 的模块，模块的
内容如:

:file:`celeryconfig.py`:

.. code-block:: python

    BROKER_URL = 'amqp://'
    CELERY_RESULT_BACKEND = 'amqp://'

    CELERY_TASK_SERIALIZER = 'json'
    CELERY_RESULT_SERIALIZER = 'json'
    CELERY_TIMEZONE = 'Europe/Oslo'
    CELERY_ENABLE_UTC = True

要验证你的配置文件是否能正确的运行，并且检查是否有语法错误，你可以尝试导入模块:

.. code-block:: bash

    $ python -m celeryconfig

配置项的完整信息 :ref:`configuration` 。

To demonstrate the power of configuration files, this how you would
route a misbehaving task to a dedicated queue:

:file:`celeryconfig.py`:

.. code-block:: python

    CELERY_ROUTES = {
        'tasks.add': 'low-priority',
    }

Or instead of routing it you could rate limit the task
instead, so that only 10 tasks of this type can be processed in a minute
(10/m):

:file:`celeryconfig.py`:

.. code-block:: python

    CELERY_ANNOTATIONS = {
        'tasks.add': {'rate_limit': '10/m'}
    }

If you are using RabbitMQ, Redis or MongoDB as the
broker then you can also direct the workers to set a new rate limit
for the task at runtime:

.. code-block:: bash

    $ celery control rate_limit tasks.add 10/m
    worker.example.com: OK
        new rate limit set successfully

See :ref:`guide-routing` to read more about task routing,
and the :setting:`CELERY_ANNOTATIONS` setting for more about annotations,
or :ref:`guide-monitoring` for more about remote control commands,
and how to monitor what your workers are doing.

Where to go from here
=====================

If you want to learn more you should continue to the
:ref:`Next Steps <next-steps>` tutorial, and after that you
can study the :ref:`User Guide <guide>`.
