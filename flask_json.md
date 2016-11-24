#flask中的json序列化

通常的restfull api是发起一个api请求，然后返回一个json字符串的数据。

通常涉及到两个问题

1. json序列化
2. api异常处理

我们先来看看json序列化

一般的api请求是从数据库中获取模型实例，然后将model序列化成json返回给客户端。那么我们就来看看这个model应该如何序列化。

在flask框架中的json模块中提供了一个工具函数jsonify，负责将参数转换成一个flask repsonse对象。那么这个response对象的类型mimetype默认是"application/json"。

jsonify的参数必须同时是位置参数，或者命名参数。如果是位置参数，那么会打包成为一个数组。如果都是命名参数，那么会打包成为一个字典。如果既有位置参数，又有命名参数，则会抛出异常。

什么是json\_encoder，为什么需要json_encoder?

flask中实际使用了[simplejson](https://github.com/simplejson/simplejson)模块，进行json的序列化。

json_encoder将python的数据类型转换成json的数据类型。基本的转换类型如下:

| JSON | Python2 |
| :---: | :---: |
| object | dict |
| array | list |
| string | unicode |
| number (int) | int, long |
| number (real) | float |
| true | True |
| false | False |
| null | None |

那么我们如果要将复杂的对象实例转换成json字符串呢？那么此时就需要用到json_encoder。

在flask框架中提供了一个接口，在app.py模块中，Flask的类属性json_encoder

```python
json_encoder = json.JSONEncoder
```

默认的json_encoder使用的flask框架json.py模块中的JSONEncoder类。

JSONEncoder的实现如下:

```python
class JSONEncoder(_json.JSONEncoder):
     def default(self, o):
         if isinstance(o, date):
             return http_date(o.timetuple())
         if isinstance(o, uuid.UUID):
             return str(o)
         if hasattr(o, '__html__'):
             return text_type(o.__html__())
         return _json.JSONEncoder.default(self, o)
```

JSONEncoder中的主要方法是default，它将一个python的复杂类型，转换成simplejson可以序列化的简单类型，如上表格中的:dict,list,unicode,int, long, float, True, False, None等。

如果要让JSONEncoder支持复杂类型的序列化，那么需要扩展重写default方法:

我们来看看[overholt](https://github.com/mattupstate/overholt)项目中helpers.py模块中对JSONEncoder的扩展。

```python
class JSONEncoder(BaseJSONEncoder):
     def default(self, obj):
         if isinstance(obj, JsonSerializer):
             return obj.to_json()
         return super(JSONEncoder, self).default(obj)
         
class JsonSerializer(object):
 	  __json_public__ = None
     __json_hidden__ = None
     __json_modifiers__ = None

     def get_field_names(self):
         for p in self.__mapper__.iterate_properties:
             yield p.key

     def to_json(self):
         field_names = self.get_field_names()

         public = self.__json_public__ or field_names
         hidden = self.__json_hidden__ or []
         modifiers = self.__json_modifiers__ or dict()

         rv = dict()
         for key in public:
             rv[key] = getattr(self, key)
         for key, modifier in modifiers.items():
             value = getattr(self, key)
             rv[key] = modifier(value, self)
         for key in hidden:
             rv.pop(key, None)
         return rv
```
overholt中的JSONEncoder是原生的flask JSONEncoder的子类，重写了default方法。
判断一个对象如果是JsonSerializer的实例，那么调用to_json方法。

而JsonSerializer实际是一个sqlalchemy model的mixin。

只要sqlalchemy的model继承了JsonSerializer，那么这个model的实例就具有了to_json方法。to_json方法作用是将model中向json序列化暴露的属性，转换成一个字典。那么model实例，便可以真正通过flask的jsonify这个统一的json序列化接口进行model实例的序列化。

