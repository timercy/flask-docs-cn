.. _larger-applications:

较大的应用
===================

对于比较大型的应用，使用包而不是模块来管理代码是一个很好的主意。
这非常简单，设想一个如下结构的应用::

    /yourapplication
        /yourapplication.py
        /static
            /style.css
        /templates
            layout.html
            index.html
            login.html
            ...

简单的包
---------------

将一个项目改为一个更大的包，仅仅创建一个新的 `yourapplication` 文件夹在
已经存在的那个下面，然后将所有的的文件都移动到它下面。然后将 `yourapplication.py`
重命名为 `__init__.py` (确保先删除了其中所有的 `.pyc` 文件，否则可能导致
错误的结果)

您最后得到的东西应该向下面这样::

    /yourapplication
        /yourapplication
            /__init__.py
            /static
                /style.css
            /templates
                layout.html
                index.html
                login.html
                ...

如何在此种方式下运行您的应用？原来的 ``python yourapplication/__init__.py`` 
不能再工作了。这是由于 Python 不希望在包中的模块成为初始运行的文件。但这
不是一个大问题，仅仅添加一个名叫 `runserver.py` 的新文件，把这个文件放在
`yourapplication` 文件夹里，并添加如下功能::

    from yourapplication import app
    app.run(debug=True)

What did we gain from this?  Now we can restructure the application a bit
into multiple modules.  The only thing you have to remember is the
following quick checklist:

然后，我们又能对应用做什么呢？现在我们可以重新构造我们的应用，将其
改造为多个模块。你需要记住的唯一一件东西就是如下的快速备忘列表。

1. `Flask` 程序对象的创建必须在 `__init__.py` 文件里完成，
   这样我们就可以安全的导入每个模块，而 `__name__` 变量将
   会被分配给正确的包。
2. 所有的视图函数(有 `~flask.Flask.route` 装饰器在上面的那些)必须被
   导入到 `__init__.py` 文件。此时，请通过模块而不是对象本身作为路径
   导入这些视图函数。视图模块 **必须在应用程序创建之后** 才能导入。

这里是 `__init__.py` 的一个例子::

    from flask import Flask
    app = Flask(__name__)

    import yourapplication.views

而 `views.py` 应该看起来像这样::

    from yourapplication import app

    @app.route('/')
    def index():
        return 'Hello World!'

您最终应该得到的程序结构应该是这样::

    /yourapplication
        /runserver.py
        /yourapplication
            /__init__.py
            /views.py
            /static
                /style.css
            /templates
                layout.html
                index.html
                login.html
                ...

.. admonition:: 循环导入

   每个 Python 程序员都会讨厌他们，而我们反而还添加了几个进去:
   循环导入(在两个模块相互依赖对方的时候，就会发生循环导入)。在这里
   `views.py` 依赖于 `__init__.py`。通常这被认为是个不好的主意，但是
   在这里实际上不会造成问题。之所以如此，是因为我们实际上没有在
   `__init__.py` 里使用这些视图，而仅仅是保证模块被导入了。并且，我们是
   在文件的结尾这么做的。

   这种做法仍然有些问题，但是如果您想要使用修饰器，那么没有
   其他更好的方法了。检查 :ref:`becomingbig` 这一章来寻找解决
   问题的些许灵感吧。


.. _working-with-modules:

与蓝图一起工作
-----------------------

如果您有规模较大的应用，建议您将他们分拆成小的组，让每个组
接口于蓝图提供的辅助功能。关于这一主题进一步的介绍请参考
:ref:`blueprints` 这一章节的文档
