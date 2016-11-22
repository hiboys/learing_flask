#flask-wtf 处理表单

[flask-wtf](https://github.com/lepture/flask-wtf)实际整合了[wtforms](https://github.com/wtforms/wtforms)，方便在flask框架中使用wtforms，在表单中加入了csrf token，使得处理表单更加安全。支持表单验证码，但是由于这个验证码是使用了Google的验证码api，因此是没法使用的。支持表单文件上传，但是依赖(Flask-uploads)，以及国际化。

##flask-wtf的使用
flask-wtf中主要封装了FlaskForm类，定义一个表单对象。

```python
from flask_wtf import FlaskForm,RecaptchaField
```

RecaptchaField也是Flask-wtf中额外封装的，但是一些常用的表单字段类型还是得从wtforms中引入。

```python
from wtforms import StringField
```

以及表单的数据验证：

```python
from wtforms.validators import DataRequired
```

使用flask-wtf相当简单，首先自定义表单类。

```python
class MyForm(FlaskForm):
    name=StringField('name', validators=[DataRequired()])
```

如上的表单类，只是定义了一个字符串字段，这个表单字段是必须的。

我们先看看在模板中表单是如何使用的。

```html
<form method="post" action="/">
    {{form.csrf_token}}
    {{form.name.label}}
    {{form.name(size=20 ) }}
    <input type="submit" value="Go">
</form>
```

从上面我们可以看到flask-wtf在模板中的使用，其实并没有非常智能，需要列出所有的表单字段。

每个字段会渲染在一个input标签，但是如果需要使用label标签，那么需要额外渲染label。

同时也需要在表单中显式使用csrf_token。

接下来使用在flask的视图函数中渲染表单和验证表单呢?


```python
@app.route('/', methods=("get", "post"))
def home():
    form = MyForm()
    if form.validate_on_submit():
        return "success!"
    return render_template('index.html', form=form)

```





