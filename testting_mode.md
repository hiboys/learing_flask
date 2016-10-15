#Flask测试模式
Flask测试模式，主要用于单元测试，http请求，http api测试。

##开启测试模式
如何开始Flask测试模式？主要通过Flask类实例的testing属性。

```python
import flask

app = flask.Flask(__name__)
app.testing = True
```

##testing属性对测试的影响
Flask框架为了更好更好支持测试，在Flask类中提供了test_client方法

```python
def test_client(self, use_cookies=True, **kwargs):
```

该方法会返回一个http client，用于发http请求，进行测试，test_client的简单使用如下：

```python
@app.route("/index")
def index():
    return "hello world"

client = app.test_client()
response = client.get('/index')

```

最终在response对象的data属性，我们可以取到index函数返回的“***hello world***”字符串,response.data实际保存的就是对应请求服务端处理后返回的文本。

**那么testing属性究竟对测试会有什么影响呢**？

Flask的http handler函数，除了正常的http处理逻辑外，也可能会抛出未处理的异常。
如果设置了testing属性为True，test client就能感知handler抛出的异常。否则，handler中抛出的异常会被Flask框架处理掉，response.data中我们只能得到类似服务器内部错误信息，是无法知道请求对应的handler中究竟抛出了什么异常。

换句话说，设置了testing属性为True后，服务器端handler抛出的异常，会冒泡到client。

```python
app = flask.Flask(__name__)
app.testing = True

@app.route("/index")
def index():
    raise Exception('unhandled exception')

client = app.test_client()

try:
    response = client.get('/index')
except Exception, e:
    print e
```

上面代码中捕获的Exception e，就是index函数中抛出的Exception了。