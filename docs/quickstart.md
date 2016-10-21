# Quick Start

## Welcome

    from flask import Flask
    from flask_restaction import Api

    app = Flask(__name__)
    api = Api(app)

    class Welcome:

        def __init__(self, name):
            self.name = name
            self.message = "Hello %s, Welcome to flask-restaction!" % name

    # create a resource
    class Hello:
        """Hello world"""

        # create an action
        def get(self, name):
            """
            Get welcome message

            $input:
                name?str&default="world": Your name
            $output:
                message?str: Welcome message
            """
            return Welcome(name)

    api.add_resource(Hello)
    # config route of API document
    app.route('/')(api.meta_view)

    if __name__ == '__main__':
        app.run(debug=True)

Save as `hello.py`, then run it:

    $ python hello.py
     * Running on http://127.0.0.1:5000/
     * Restarting with reloader

Visit http://127.0.0.1:5000/hello:

    {
      "message": "Hello world, Welcome to flask-restaction!"
    }

Visit http://127.0.0.1:5000/hello?name=kk

you will see:

    {
      "message": "Hello kk, Welcome to flask-restaction!"
    }

Visit http://127.0.0.1:5000 for generated API document.


#### Two concept

**resource**

    eg: class Hello, represent a kind of resource

**action**

    eg: get, post, delete, get_list, post_login.
    HTTP method + '_' + anything is OK.


## Validation and Serialization

Use `$shared` to describe shared schema in doc string and register it via
`Api(docs=__doc__)` .

Use `$shared` in resource's doc string to describe shared schema for resource.

In action's doc string, use `$input`, `$output` to describe request and response
data struct, use `$error` to describe exceptions.

**$input**
    
    Request data struct, if no $input, then won't validate request data and
    call action without params.
    If HTTP method is GET,DELETE, request data is query string. If HTTP method
    is POST,PUT,PATCH, request data is request body, and Content-Type should
    be application/json.
    
**$output**
    
    Response data struce, if no $output, then won't validate and serialize
    returns value of action.

**$error**
    
    Describe exceptions, only used for API document, eg:

        $error:
            400.InvalidData: invalid request data
            403.PermissionDeny: permission deny

    Syntax is: status.Error: message

If request data validation fail, then response is:

    {
        "status": 400,
        "error": "InvalidData",
        "message": "xxx xxxx"
    }

If response data validation fail, then response is:

    {
        "status": 500,
        "error": "ServerError",
        "message": "xxx xxxx"
    }

Schema is [YAML](https://zh.wikipedia.org/wiki/YAML) text, see [Schema](schema.md).

#### Custom validator

Validr's document had describe custom validator, see [Validr](https://github.com/guyskk/validr).

All custom validators is registered via `Api(validators=validators)`.

## URL rules

use `url_for(endpoint)`` of flask to build url for action.

endpoint is `resource@action_name`

**resource**: resource classname in lowercase
**action_name**: the last part of action(split via '_')

Usage:

    url_for("resource@action_name") -> /resource/action_name

Example:

    url_for("hello") -> /hello
    url_for("hello@login") -> /hello/login


## Response errors

    from flask_restaction import abort

    # function prototype
    abort(code, error=None, message=None)

If param error is None, the effect is the same as `flask.abort(code)`.
If error is instance of `flask.Response`, the effect is the same as `flask.abort(code, error)`.

Otherwise, response:

    {
        "status": code,
        "error": error,
        "message": message
    }

The response data will be serialized to appropriate format.


## Permission control

### 举个栗子

meta.json 设定角色和权限

    {
        "$roles": {
            "admin": {
                "hello": ["get", "post"],
                "user": ["post"]
            },
            "guest": {
                "user": ["post"]
            }
        }
    }


__init__.py 根据token确定角色

    from flask_restaction import Api, TokenAuth

    api = Api(metafile='meta.json')
    auth = TokenAuth(api)

    @auth.get_role
    def get_role(token):
        if token:
            return token["role"]
        else:
            return "guest"

hello.py 业务代码

    class Hello:

        def get(self):
            pass

        def post(self):
            pass

user.py 登录接口

    from flask import g

    class User:

        def __init__(self, api):
            self.api = api

        def post(self, username, password):
            # query user from database
            g.token = {"id": user.id, "role": user.role}
            return user


### 使用情景

用户直接请求 `hello.get` 接口, 框架收到请求后, 从请求头的 `Authorization` 中取 `token` , 
此时 `token` 为 `None`, 然后框架调用 `get_role(None)` , 得到角色 `guest` , 再判断
`meta["$roles"]["guest"]["hello"]` 中有没有 `get`, 发现没有, 框架直接拒绝此次请求.

用户请求 `user.post` 接口, 处理流程同上, 请求到达 `user.post` 方法, 验证用户名和密码, 如果验证成功, 就设置
`g.token`, `token` 里面保存了用户ID, 角色和过期时间.TokenAuth会将 `g.token` 用JWT进行签名, 
然后通过响应头的 `Authorization` 返回给用户.

用户再次请求 `hello.get` 接口, 在请求头的 `Authorization` 中带上了刚才得到的 `token`, 
处理流程同上, 框架允许此次请求, 请求到达 `hello.get` 方法.

### 示意图

              +--------+               +--------------+        +--------------+         +-------------+
              |        |               |              |  None  |              |  guest  |             |
    Deny      |  User  +---------------> decode_token +-------->   get_role   +---------X  hello.get  |
              |        |               |              |        |              |         |             |
              +--------+               +--------------+        +--------------+         +-------------+


              +--------+               +--------------+        +--------------+         +-------------+
              |        |               |              |  None  |              |  guest  |             |
              |  User  +---------------> decode_token +-------->   get_role   +--------->  user.post  |
              |        |               |              |        |              |         |             |
    Login     +---+----+               +--------------+        +--------------+         +------+------+
                  ^                                                                            |
                  |                                +--------------+                            |
                  |          Authorization         |              |          g.token           |
                  +--------------------------------+ encode_token <----------------------------+
                                                   |              |
                                                   +--------------+


              +--------+               +--------------+        +--------------+         +-------------+
              |        | Authorization |              |  token |              |  admin  |             |
     OK       |  User  +---------------> decode_token +-------->   get_role   +--------->  hello.get  |
              |        |               |              |        |              |         |             |
              +--------+               +--------------+        +--------------+         +-------------+


### 分步说明

#### 在 metafile 中设定角色和权限

metafile是一个描述API信息的文件, 通常放在应用的根目录下, 文件名 meta.json.

在Api初始化的时候通过 `Api(metafile="meta.json")` 加载.

    {
        "$roles": {
            "Role": {
                "Resource": ["Action", ...]
            }
        }
    }


请求到来时, 根据 Role, Resource, Action 可以快速确定是否许可此次请求.

**提示**: 

    flask的Development Server不能检测到python代码文件之外变动, 所以修改metafile的内容之后需要手动重启才能生效.


#### 注册 get_role 函数

框架通过URL能解析出Resource, Action, 但是无法知道用户是什么角色, 所以需要你提供一个能返回用户角色的函数.

#### 生成 token

为了能够确认用户的身份, 需要在用户登录成功后生成一个 token, 将 token 通过响应头(`Authorization`)返回给用户.
token 一般会储存用户ID和过期时间, 用户在发送请求时需要将 token 通过请求头发送给服务器.

TokenAuth使用 [json web token](https://github.com/jpadilla/pyjwt) 作为身份验证工具.

**提示**:

     token 会用密钥(app.secret_key)对 token 进行签名, 无法篡改.
     生成 token 前需要先设置 app.secret_key, 或通过 flask 配置.
     token 是未加密的, 不要把敏感信息保存在里面.


身份/权限验证失败会返回:

    {
        "status": 403,
        "error": "PermissionDeny",
        "message": "xxx can't access xxxx"
    }


#### 安全性和设置

对安全性要求不同, 权限管理的实现也会不同, TokenAuth的实现适用于对安全性要求不高的应用.

当收到请求时, 检测到token即将过期, 会主动颁发一个新的token给客户端, 这样能避免token过期
导致中断用户正常使用的问题.但这样也导致token能够被无限被刷新, 有一定的安全隐患.

以下是默认设置:

    {
        "$auth": {
              "algorithm": "HS256",     # token签名算法
              "expiration": 3600,       # token存活时间, 单位为秒
              "header": "Authorization" # 用于传递token的请求/响应头
              "cookie": null            # 用于传递token的cookie名称, 默认不用cookie
              "refresh": true           # 是否主动延长token过期时间
        }
    }


#### 自定义权限管理

`Api.authorize(role)` 方法能根据 `$roles` 和请求URL判断该角色是否有权限调用API, 
利用它可以简化自定义权限管理实现.

以下是基本结构, 具体实现可以参考 [flask_restaction/auth.py](https://github.com/guyskk/flask-restaction):

    class MyAuth:

        def __init__(self, api):
            self.api = api
            self.config = api.meta["$auth"]
            api.before_request(self.before_request)
            api.after_request(self.after_request)

        def before_request(self):
            """Parse request, check permission"""
            # parse role from request
            self.api.authorize(role)

        def after_request(self, rv, status, headers):
            """Modify response"""
            return rv, status, headers


## Add resource

Use `Api.add_resource` to add resource, params of `add_resource` will
be passed to resource's `__init__` method.

URL is the same as resource name, if you want to use another URL, you can
create a new resource like this:

    api.add_resource(type('NewName', (MyResource,), {}))


## API document

![Screenshot](img/docs.png)

There are two ways to config route for API document.

### Flask.route
    
    app.route('/')(api.meta_view)

### Api.add_resource

This way will treat document as a resource, and is easy for permission control.

    # Note: enable token via cookie
    {
        "$auth": {
              "cookie": "Authorization"
        }
    }
    # add_resource
    api.add_resource(type('Docs', (), {'get': api.meta_view}))

Api.meta_view can also response API meta data in JSON format, set request
header `Accept` to `application/json` to do so.


## Blueprint

Api can exist inside blueprint, then all resources will be routed in blueprint.

    from flask import Flask, Blueprint
    from flask_restaction import Api

    app = Flask(__name__)
    bp = Blueprint('api', __name__)
    api = Api(bp)
    api.add_resource(XXX)
    app.register_blueprint(bp)

Note: add_resource should call before register_blueprint, otherwise
add_resource has no effect.


## Event handler

Api provide before_request, after_request, error_handler decorater for register
event handlers.

    @api.before_request
    def before_request():
        # this function will be called before action exec
        # if return value is not None, then use it as response
        return response

    @api.after_request
    def after_request(rv, status, headers):
        # this function is used for process return value of action
        return rv, status, headers

    @api.error_handler
    def error_handler(ex):
        # handle exception raised from before_request and action
        # if return value is not None, then use it as response
        return response


## Custom response format

The dafault response format is JSON, you can add custom response format easily.

    from flask import make_response
    from flask_restaction import exporter

    @exporter('text/html')
    def export_text(data, status, headers):
        return make_response(str(data), status, headers)

The framwork will choose appropriate response format according to the Accept
value in request headers.


## res.js

You can use res.js via open browser console in API document page.

If API's url prefix isn't '/', then you need config **API_URL_PREFIX**.

Example: `http://127.0.0.1:5000/api`

    app.config["API_URL_PREFIX"] = "/api"

See [resjs](resjs.md) for more infomation.


## res.py

res.py's usage is similar as res.js, it use requests for sending HTTP requests.

    >>> from flask_restaction import Res
    >>> help(Res)
