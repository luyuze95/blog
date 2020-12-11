# Flask开发技巧之异常处理

[TOC]

本人平时开发中使用的，或者学习到的一些flask开发技巧整理，需要已有较为扎实的flask基础。

## 1、Flask内置异常处理

要想在Flask中处理好异常，有一套自己的异常处理机制，首先，我们必须先知道Flask自己是如何处理异常的。去flask的源码里找一找会发现，在flask源码的app.py文件下，有很多会抛出异常的方法，其中拿一个举例:

```python
    def handle_exception(self, e):
        """Default exception handling that kicks in when an exception
        occurs that is not caught.  In debug mode the exception will
        be re-raised immediately, otherwise it is logged and the handler
        for a 500 internal server error is used.  If no such handler
        exists, a default 500 internal server error message is displayed.

        .. versionadded:: 0.3
        """
        exc_type, exc_value, tb = sys.exc_info()

        got_request_exception.send(self, exception=e)
        handler = self._find_error_handler(InternalServerError())

        if self.propagate_exceptions:
            # if we want to repropagate the exception, we can attempt to
            # raise it with the whole traceback in case we can do that
            # (the function was actually called from the except part)
            # otherwise, we just raise the error again
            if exc_value is e:
                reraise(exc_type, exc_value, tb)
            else:
                raise e

        self.log_exception((exc_type, exc_value, tb))
        if handler is None:
            return InternalServerError()
        return self.finalize_request(handler(e), from_error_handler=True)
```

我们发现在flask内部对于500异常，会抛出这样一个错误类`InternalServerError()`

```python
class InternalServerError(HTTPException):
    ......
```

至此我们发现flask内部异常通过继承这个HTTPException类来处理，那么这个HTTPException类就是我们研究的重点。

## 2、HTTPException类分析

```python
@implements_to_string
class HTTPException(Exception):
    """Baseclass for all HTTP exceptions.  This exception can be called as WSGI
    application to render a default error page or you can catch the subclasses
    of it independently and render nicer error messages.
    """

    code = None
    description = None

    def __init__(self, description=None, response=None):
        super(HTTPException, self).__init__()
        if description is not None:
            self.description = description
        self.response = response

    @classmethod
    def wrap(cls, exception, name=None):
        """Create an exception that is a subclass of the calling HTTP
        exception and the ``exception`` argument.

        The first argument to the class will be passed to the
        wrapped ``exception``, the rest to the HTTP exception. If
        ``e.args`` is not empty and ``e.show_exception`` is ``True``,
        the wrapped exception message is added to the HTTP error
        description.

        .. versionchanged:: 0.15.5
            The ``show_exception`` attribute controls whether the
            description includes the wrapped exception message.

        .. versionchanged:: 0.15.0
            The description includes the wrapped exception message.
        """

        class newcls(cls, exception):
            _description = cls.description
            show_exception = False

            def __init__(self, arg=None, *args, **kwargs):
                super(cls, self).__init__(*args, **kwargs)

                if arg is None:
                    exception.__init__(self)
                else:
                    exception.__init__(self, arg)

            @property
            def description(self):
                if self.show_exception:
                    return "{}\n{}: {}".format(
                        self._description, exception.__name__, exception.__str__(self)
                    )

                return self._description

            @description.setter
            def description(self, value):
                self._description = value

        newcls.__module__ = sys._getframe(1).f_globals.get("__name__")
        name = name or cls.__name__ + exception.__name__
        newcls.__name__ = newcls.__qualname__ = name
        return newcls

    @property
    def name(self):
        """The status name."""
        from .http import HTTP_STATUS_CODES

        return HTTP_STATUS_CODES.get(self.code, "Unknown Error")

    def get_description(self, environ=None):
        """Get the description."""
        return u"<p>%s</p>" % escape(self.description).replace("\n", "<br>")

    def get_body(self, environ=None):
        """Get the HTML body."""
        return text_type(
            (
                u'<!DOCTYPE HTML PUBLIC "-//W3C//DTD HTML 3.2 Final//EN">\n'
                u"<title>%(code)s %(name)s</title>\n"
                u"<h1>%(name)s</h1>\n"
                u"%(description)s\n"
            )
            % {
                "code": self.code,
                "name": escape(self.name),
                "description": self.get_description(environ),
            }
        )

    def get_headers(self, environ=None):
        """Get a list of headers."""
        return [("Content-Type", "text/html; charset=utf-8")]

    def get_response(self, environ=None):
        """Get a response object.  If one was passed to the exception
        it's returned directly.

        :param environ: the optional environ for the request.  This
                        can be used to modify the response depending
                        on how the request looked like.
        :return: a :class:`Response` object or a subclass thereof.
        """
        from .wrappers.response import Response

        if self.response is not None:
            return self.response
        if environ is not None:
            environ = _get_environ(environ)
        headers = self.get_headers(environ)
        return Response(self.get_body(environ), self.code, headers)

    ......
```

- 截取这个类比较重要的几个方法分析，`get_headers`方法定义了这个返回的响应头，返回的是html文档。

- `get_body`方法定义了返回的响应体，对应也是一段html的内容。

- 最后在Response中将响应体，状态码，响应头定义好返回。

分析至此，其实这个HTTPException中做的事也不难理解，就是定义好响应体，状态码，还有响应头，做了一个返回。当然这个类返回是html类型的，现在前后端分离交互都是json形式的返回，所以我们可以继承自这个类，定义我们自己的异常处理类。

## 3、自定义异常处理类

首先我们理解我们自己的这个异常处理类，应该继承自HTTPException来改写。而我们自定义的内容应该包含以下几点:

- 需要定义我们自己想要返回的错误信息的json格式，比如内部错误码、错误信息等我们想记录的信息。
- 需要更改返回的响应头，返回json格式的信息响应头就应该设为`'Content-Type': 'application/json'`
- 同样需要和HTTPException一样定义好状态码

如下定义我们自己的异常类APIException，返回的信息包括内部错误码，错误信息，请求的url

```python
class APIException(HTTPException):
    code = 500
    msg = 'sorry, we made a mistake!'
    error_code = 999

    def __init__(self, msg=None, code=None, error_code=None, headers=None):
        if code:
            self.code = code
        if error_code:
            self.error_code = error_code
        if msg:
            self.msg = msg
        super(APIException, self).__init__(msg, None)

    def get_body(self, environ=None):
        body = dict(
            msg=self.msg,
            error_code=self.error_code,
            request=request.method + ' ' + self.get_url_no_param()
        )
        text = json.dumps(body)
        return text

    def get_headers(self, environ=None):
        """Get a list of headers."""
        return [('Content-Type', 'application/json')]

    @staticmethod
    def get_url_no_param():
        full_path = str(request.full_path)
        main_path = full_path.split('?')
        return main_path[0]
```

## 4、方便的定义自己的错误类

有了上面我们改写好的APIException类，我们就可以自由的定义各种状态码的错误以及对应的错误信息，然后在合适的位置抛出。比如：

```python
class Success(APIException):
    code = 201
    msg = 'ok'
    error_code = 0


class DeleteSuccess(APIException):
    code = 202
    msg = 'delete ok'
    error_code = 1


class UpdateSuccess(APIException):
    code = 200
    msg = 'update ok'
    error_code = 2


class ServerError(APIException):
    code = 500
    msg = 'sorry, we made a mistake!'
    error_code = 999


class ParameterException(APIException):
    code = 400
    msg = 'invalid parameter'
    error_code = 1000


class NotFound(APIException):
    code = 404
    msg = 'the resource are not found'
    error_code = 1001


class AuthFailed(APIException):
    code = 401
    msg = 'authorization failed'
    error_code = 1005


class Forbidden(APIException):
    code = 403
    error_code = 1004
    msg = 'forbidden, not in scope'
```

有了这些自定义的错误类，我们不仅可以直接在需要的地方抛出，而且有了自定义的错误码，发生错误时，只要对照错误码去查找对应的错误类，非常方便。而且特别说明的是，虽然说是错误类，但是也是可以定义响应成功的返回的，比如上面定义的200，201的类，同样可以作为一个成功的返回。

使用演示：

```python
user = User.query.first()
if not user:
    raise NotFound()
```

## 5、注意事项

尽管我们可以在我们认为可能出错的所有地方，继承自己的异常类，定义自己的错误类，然后抛出，但是也不是所有的异常都是我们可以提前预知的。比如我们接受前端传来的参数，参数类型或取值范围不正确，这些我们可以预知并处理好，但是如果是逻辑处理中出现了问题，这些不是我们程序员可以控制并处理。所以光有自定义错误类还不够，我们还需要在全局捕获异常来判断，利用AOP思想。

```python
# 全局错误AOP处理
@app.errorhandler(Exception)
def framework_error(e):
    api_logger.error("error info: %s" % e) # 对错误进行日志记录
    if isinstance(e, APIException):
        return e
    if isinstance(e, HTTPException):
        code = e.code
        msg = e.description
        error_code = 1007
        return APIException(msg, code, error_code)
    else:
        if not app.config['DEBUG']:
            return ServerError()
        else:
            return e
```

这里对于flask中抛出的所有的错误进行捕获，然后先进行日志的记录。然后判断如果是我们自定义的APIException，就直接返回。如果不是我们自定义的，但是是flask处理的HTTPException，包装成我们自定义的APIException再返回。如果都不是的话，说明是服务器出现的其他错误，问题一般出在我们的代码上，在生产环境下，一般统一返回一个500错误，在调试模式下，可以原样返回，便于我们定位修改自己的代码。

关于flask的异常处理，以上就是我目前掌握的一些技巧，如有错误欢迎指出。

-----

博客园: https://www.cnblogs.com/luyuze95/

GitHub: https://github.com/luyuze95