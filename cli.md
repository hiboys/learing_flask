#Flask cli接口
Flask框架像django一样，也提供了cli接口，运行app和打开shell环境。

##Flask cli接口如何使用

通过**pip install flask** 安装flask框架，并不会直接安装flask cli接口。

相关问题请参考，链接[https://github.com/pallets/flask/issues/1278](https://github.com/pallets/flask/issues/1278)。

##Flask cli命令

cli中提供了两个最有用的命令，run和shell

run命令，运行一个开发server

shell命令，在app context环境中，运行一个python解析器。

其中也有两个常用的环境变量，FLASK_APP和FLASK_DEBUG。

具体使用例子如下：

```shell
 export FLASK_APP=hello.py
 export FLASK_DEBUG=1
 flask run
```

上面shell命令的意思是在debug模式下运行一个flask app。

##Flask自定义cli命令
Flask中还提供了一种方式，供开发者自定义cli命令。

```python
@app.cli.command()
def init_db():
    from myblog import models
    Base.metadata.create_all(bind=engine)
```

注意到上面代码使用了函数装饰器app.cli.command()，init_db函数名已经成为了一个Flask cli命令，这一点我们可以从**flask --help** 命令的输出中可以看到。

![cli_custom](img/cli_custom.tiff)