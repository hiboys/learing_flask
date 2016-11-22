#flask-sqlalchemy

flask-sqlchemy的基本用法如下:

引进扩展包:

```python
from flask_sqlalchemy import  SQLAlchemy
```

基本的配置如下:

```python
app = Flask(__name__)
app.config["SQLALCHEMY_DATABASE_URI"]='sqlite://'

db = SQLAlchemy(app)
```

其中一个配置是SQLALCHEMY\_DATABASE\_URI，指定数据库连接字符串。注意上面的db实例。

抽象基类模型供继承:

```python
class Base(db.Model):
    __abstract__ = True
    id = db.Column(db.Integer, primary_key=True)
    created_at = db.Column(db.DateTime, default=db.func.current_timestamp())
    modified_at = db.Column(db.DateTime, default=db.func.current_timestamp(),
                            onupdate=db.func.current_timestamp())
                            
class Role(Base):
    __tablename__ = 'auth_role'
    name = db.Column(db.String(80), nullable=False, unique=True)
    description = db.Column(db.String(255))

    def __init__(self, name):
        self.name = name

    def __repr__(self):
        return '<Role %r>' % self.name

```

类属性：\_\_abstract__，表明了这是一个模型抽象基类，可以继承。这个基类定义了三个字段，主键id，创建时间，修改时间。注意里面日期默认为当前时间戳的写法，使用了db.func。

常用的一些db.Column参数如下:

1. primary_key:是否主键
2. default:默认值
3. unique:是否唯一键
4. nullable:是否可空，默认可空。

那么如何对定义好的实体类，建表?

```python
db.create_all()
```

如何取到数据库会话实例?

```python
db.session
```

注意每次数据库查询要提交的时候，使用db.session.commit()

如何进行基本的数据库模型查询操作?

```python
Role.query.first()
```

常用的数据库操作如下:

1.新增model:

```python
db.session.add(model)
```

2.查询所有model实例:

```python
Role.query.all()
```

3.根据id获取唯一的一个实例:

```python
Role.query.get(id)
```

4.根据id数组获取多个模型实例:

```python
Role.query.filter(Role.id.in_(ids)).all()
```

我们可以注意到sql中的in已经映射成了id属性的一个方法，这一点比较有意思。

5.根据组合条件来查询:

```python
Role.query.filter_by(**kwargs)
```

6.更新操作:

更新操作是直接修改模型实例的属性，这一点与django中的模型更新操作是一样的。

7.删除操作如下:

```python
db.session.delete(model)
```




