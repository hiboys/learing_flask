#url_for

url_for函数是Flask框架中动态生成url的一个helper函数。在web模板中，为了避免url硬编码，往往需要借助url_for函数。

##url_for函数签名

```python
def url_for(endpoint, **values):
```

第一个参数是必要参数路由规制名称**endpoint**, 接下来是可选的命名参数

##url_for生成静态文件url

web项目中会涉及到静态文件如css文件，那么如何借助url_for生成静态文件url？

```python
url_for('static', filename='style.css')
```

Flask框架在启动的时候，会自动注册一个路由endpoint **static**， 第二个命名参数fileName表明在static静态文件根目录下的style.css文件，以此生成静态文件url。

**默默问一句filename参数，究竟是如何起作用的？？**

filename实际是Flask路由规则中的url参数(不是http的query参数)。

我们从app.py模块中的一段代码，可以看出端倪。

```python
 if self.has_static_folder:
 	self.add_url_rule(self.static_url_path + '/<path:filename>',
 	                               endpoint='static',
 	                               view_func=self.send_static_file)
```

注意上面代码中的*/\<path:filename\>*

假设有如下的http handler

```python
@app.route('/<path:name>')
def index(name):
    return render_template('index.html', name=name);
```

那么在模板中如何为这个index handler动态生成url？

```python
url_for('index', name=name)
```