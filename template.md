#Flask模板
框架中的jinja2模板，实际是通过jinja2的**Environment**加载渲染的。

##Environment什么时候被构建

在Flask的app.py模块中，我们可以看到Flask类有一个jinja_env的方法

```python
 @locked_cached_property
 def jinja_env(self):
 	"""The Jinja2 environment used to load templates."""
    return self.create_jinja_environment
```

jinja_env方法被函数装饰器locked\_cached\_property装饰，表明此方法第一次调用会被计算，然后缓存下来，作为属性。后续的调用，直接取的是缓存的结果。

也就是说，如果没有使用模板，那么jinja_env永远不会加载。

##Environment加载的时候究竟做了什么事情

从templating.py文件中，我们可以看到jinja2的Environment类在Flask框架中，实际已经被继承，在继承后的构造函数做了一些事情。

```python
class Environment(BaseEnvironment):
    """Works like a regular Jinja2 environment but has some additional
    knowledge of how Flask's blueprint works so that it can prepend the
    name of the blueprint to referenced templates if necessary.
    """

    def __init__(self, app, **options):
        if 'loader' not in options:
            options['loader'] = app.create_global_jinja_loader()
        BaseEnvironment.__init__(self, **options)
        self.app = app
```

如果loader没有创建的话，会调用Flask类的create_global_jinja_loader方法，创建loader，同时往Environment中注入了Flask的app实例。

模板引擎的loader作用，一般用于从文件系统中加载模板文件。**Flask究竟从什么文件系统目录搜索模板文件？**我们应该可以从create\_global\_jinja\_loader方法中找到模板根路径的设置。

```python
  def create_global_jinja_loader(self):
       return DispatchingJinjaLoader(self)
```

DispatchingJinjaLoader类继承jinja2.BaseLoader

从DispatchingJinjaLoader的源代码实现中，真正的模板loader实际是app.jinja\_loader或者blueprint.jinja\_loader。

```python
@locked_cached_property
    def jinja_loader(self):
        if self.template_folder is not None:
            return FileSystemLoader(os.path.join(self.root_path,
                                                 self.template_folder)
```

jinja\_loader实际也是Flask种的一个缓存属性，最终所构建的是FileSystemLoader实例。

从上文的代码中，**我们可以看到root\_path和template\_folder组成的绝对路径，就是Flask模板的根路径**。

而template\_folder是通过Flask的构造函数传入的，默认是"templates"



