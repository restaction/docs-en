# Welcome to Flask-Restaction

A web framwork born to create RESTful API

- Create RESTful API
- Validate request and Serialize response
- Authorization and Permission control
- Auto generate Javascript SDK and API document

Note: Only support Python3.3+

## Install

    pip install flask-restaction

## Examples

[https://github.com/guyskk/flask-restaction/examples](https://github.com/guyskk/flask-restaction/examples)


## Compare

### flask-restful

flask-restaction 相对于 flask-restful 有什么优势，或是什么特性?

- 输入输出校验

    restaction 是声明式的，简单明确:

        class Hello:

            def get(self, name):
                """
                Get welcome message

                $input:
                    name?str&escape&default="world": Your name
                $output:
                    message?str: Welcome message
                """

    restaction 的输出校验和输入校验一样简单，而且可以序列化任意类型的对象。

    restful 中叫做 Request Parsing:

        from flask_restful import reqparse

        parser = reqparse.RequestParser()
        parser.add_argument('name', type=str, help='Your name')
        args = parser.parse_args()

    Request Parsing 很繁琐，不能很好的重用代码。

- 清晰的URL规则

    restaction 的 URL 规则清晰，并始终保持一致，减少了编码和阅读API文档的负担。

- 身份验证及权限控制

    restaction 提供一个灵活的权限系统，身份验证基于 json web token，
    权限验证是通过json配置文件，而不是散布在代码中的装饰器。

- 自动生成文档和res.js

    restaction 可以自动生成文档和 res.js，用 res.js 可以方便的调用 api。

    
## 历程

**2015年9月4日 - 2015年12月**

项目开始

将validater作为一个独立项目

自动生成文档和res.js

添加身份验证和权限控制

重写身份验证和权限控制，之前的用起来太繁琐


**2016年1月20日 - 2月24日**

重写 validater，增强灵活性，去除一些混乱的语法

重构 Api

- 将权限从 Api 里面分离
- 将自动生成工具从 Api 里面分离，优化 res.js
- 去除测试工具，因为 flask 1.0 内置测试工具可以取代这个
- 将 testing.py 改造成 res.py，用于调用 API，功能类似于 res.js

**2016年3月 - 5月**

内部项目使用 flask-restaction 框架，项目已内测。  
期间修复一些bug，做了小的改进和优化，Api基本未变。  

**2016年5月 - 5月12日**

完善 res.js，对代码进行了重构和测试，支持模块化和标准 Promise。

**2016年7月 - 8月**

重写 validater，形成完善的Schema语法。  
重构 flask-restaction，使用YAML格式定义输入输出Schema。

**2016年9月 - 9月12日**

用NodeJS重写res.js，支持用NodeJS和Python两种方式生成res.js。  
支持生成HTML格式的API文档。

**2016年10月**

重构权限功能，独立出TokenAuth，增加灵活性和可拓展性。  
Validater更名为Validr。
