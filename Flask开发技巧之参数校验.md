# Flask开发技巧之参数校验

[TOC]

本人平时开发中使用的，或者学习到的一些flask开发技巧整理，需要已有较为扎实的flask基础。

## 1、请求参数分类

一般来说，前端发送过来的请求中，大致包含以下三种类型的参数，分别是url路径参数，url查询参数，还有目前前后端分离开发中最常见的json格式的数据。

- url路径参数

```url
/v1/user/1
```

url路径参数即类似于上述例子中的参数，直接带在url路径中，可变化，flask针对这种参数，已经直接提供了支持，例：

```url
@app.route('/v1/user/<int:id>')
```

- url查询参数

```url
/v1/user?page=1&pageSize=10
```

类似于这种带在url中的问号后面的键值对并且用&连接的参数称为url查询参数

- json格式的参数

```json
{
    "name": "xiaowang",
    "age": 1
}
```

而json格式的参数就更不用多说了，header中带有Content-Type：Application/json传输过来的json格式的数据就是这样的。


## 2、解决方案使用到的库

这里我们为了解决参数校验的问题，一定是要将参数校验的部分抽离出来，按照面向对象的思想，隐藏参数校验的具体过程，交给特定的类去解决。这样，我们在视图函数中，不会出现冗余的参数校验代码，会使整个视图函数显得简短易读。

这里我们需要安装两个库

```shell
pip install WTForms
pip install WTForms-JSON
```

后续方法建立在wtforms库上扩展，所有wtforms库原有的操作，全部都有效，可以继续使用。如果不熟悉wtforms，需要先学习一下。

## 3、针对url查询参数与一般json格式

首先解释一下，经过我的探究（本人能力有限，可能无法扩展实现），使用普通的wtforms库，无法接受复杂格式的json数据，只能接受普通格式的json数据以及url查询参数进行校验。

- 普通格式的json参数举例

```json
{
    "name": "xiaowang",
    "age": 1,
    "address": "beijing"
}
```

- 复杂格式的json参数举例

```json
{
    "category": {
        "category_name": "电脑",
        "category_id": 2
    },
    "address_list": [
        "beijing",
        "shanghai"
    ]
    "name": "xiaohong",
    "age": 1,
}
```

实现方法，继承wtforms库中的Form，实现自己的基类参数验证类BaseForm

```python
class BaseForm(Form):
    def __init__(self):
        data = request.get_json()
        args = request.args.to_dict()
        super(BaseForm, self).__init__(data=data, **args)

    def validate_for_api(self):
        valid = super(BaseForm, self).validate()
        if not valid:
            raise ParameterException(msg=self.errors)
        return self
```

这里进行一下说明，BaseForm的__init__方法实例化对象的时候首先通过flask中的request对象将普通json数据和查询参数args拿到，通过调用父类的方法将参数初始化。

而validate_for_api()方法则调用父类中的validate()进行参数校验，如果校验结果不通过，那么将错误信息放入msg交给异常类400处理，异常处理我们已经在上一篇详细讲述。如果校验通过，那么就将校验完成的form返回。

**使用举例**

针对一个请求url为
```url
/v1/user?user_id=1
```
请求体为
```json
{
    "username": "xiaoming",
    "age": 1
}
```
那么使用如下类:
```python
class UserForm(BaseForm):
    user_id = IntegerField()
    username = StringField()
    age = IntegerField()


form = UserForm().validate_for_api()
```
即可完成参数校验，如果校验出错，会直接向前端返回400，并且错误信息也会附带返回。

## 4、针对复杂json格式数据

单纯的使用wtforms库无法实现复杂json格式数据的处理，于是在我的探索下，发现还有一个wtforms的扩展库，叫wtforms-json，通过这个库可以实现。

于是扩展原先的BaseForm，使用wtforms-json，仿照原先基类，我实现的新基类如下。

```python
import wtforms_json

class JsonForm(Form):

    @classmethod
    def init_and_validate(cls):
        wtforms_json.init()
        form = cls.from_json(request.get_json())
        valid = form.validate()
        if not valid:
            raise ParameterException(msg=form.errors)
        return form
```

继承上述新的基类，这样的Form就可以实现任意json格式的数据的校验了。

**使用举例**

针对一个请求，请求体如果为

```json
{
    "username": "xiaochen",
    "age": 1,
    "address_list": [
        "beijing",
        "shanghai"
    ],
    "info": {
        "name": "hi",
        "length": 5
    },
    "area_list": [
        {
            "level1": "北京",
            "level2": "朝阳"
        },
        {
            "level1": "北京",
            "level2": "海淀"
        }
    ]
}
```

通过如下Form就可以实现校验

```python
class InfoForm(JsonForm):
    name = StringField()
    length = IntegerField()

class AreaForm(JsonForm):
    level1 = StringField()
    levle2 = StringField()

class DemoForm(JsonForm):
    username = StringField()
    age = IntegerField()
    address_list = FieldList(
        StringField(),
        min_entries=1
    )
    info = FormField(InfoForm)
    area_list = FieldList(
        FormField(AreaForm),
        min_entries=1
    )


form = DemoForm().init_and_validate()
```
如此就可以实现复杂json数据的校验

关于flask的参数校验，以上就是我目前掌握的一些技巧，如有错误欢迎指出。

-----

博客园: https://www.cnblogs.com/luyuze95/

GitHub: https://github.com/luyuze95