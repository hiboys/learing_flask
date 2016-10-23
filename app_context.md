#应用上下文app context
AppContext将一个应用上下文绑定到当前的线程或者greenlet。当一个请求上下文构建的时候，一个应用上下文也会隐式构造。

AppContext会在初始化的时候创建url_adapter，使得在测试环境中，即使没有http 请求，也可以通过url_for动态生成url。

##AppContext的生命周期


app.py模块中的wsgi_app函数如下：

```python
	def wsgi_app(self, environ, start_response):
        ctx = self.request_context(environ)
        ctx.push()
        error = None
        try:
            try:
                response = self.full_dispatch_request()
            except Exception as e:
                error = e
                response = self.handle_exception(e)
            return response(environ, start_response)
        finally:
            if self.should_ignore_error(error):
                error = None
            ctx.auto_pop(error)

```

wsgi_app函数是著名的wsgi application入口函数。ctx.push就是将请求上下文进栈.
在请求上下文push进栈的时候，如果发现应用上下文（AppContext）没有构造，那么就构造一个应用上下文进栈。

我们再看看ctx.auto_pop位于finally块作用域，**实际当请求上下文从栈顶pop出来的时候，AppContext也从栈里面pop出去**。


这说明了AppContext不会再不同的请求间共享。AppContext相伴着RequestContext而产生。

AppContext本质是一种**线程局部对象**

##AppContext如何使用

在global.py模块中，构造了一个**g**对象，实际它就是一个AppContext的实例。 可以借助g对象这个线程局部存储对象，在不同的函数间传递数据，但前提是在同一个请求内。