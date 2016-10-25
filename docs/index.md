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

flask-restaction VS flask-restful:

- Validation and Serialization

    restaction is declarative, simple and clear:

        class Hello:

            def get(self, name):
                """
                Get welcome message

                $input:
                    name?str&escape&default="world": Your name
                $output:
                    message?str: Welcome message
                """

    restaction's serialization is sample as validation, and it can serialize
    any type objects.

    restful use Request Parsing:

        from flask_restful import reqparse

        parser = reqparse.RequestParser()
        parser.add_argument('name', type=str, help='Your name')
        args = parser.parse_args()

    Request Parsing is complicated, and it can't reuse code well.

- Clear URL rules

    restaction's URL rules is clear and consistent, this can reduce the time of
    coding and reading.

- Authorization and Permission control

    restaction provide a flexible permission system, auth is based on json web token, 
    permission is based on config in json, other than decorators.

- Auto generate Javascript SDK and API document

    restaction can generate API document and res.js, res.js is used for call api.

    
## Changelog

2015年9月4日 - 2015年12月
:   项目开始  
    将validater作为一个独立项目  
    自动生成文档和res.js  
    添加身份验证和权限控制  
    重写身份验证和权限控制, 之前的用起来太繁琐  

2016年1月20日 - 2月24日
:   重写 validater, 增强灵活性, 去除一些混乱的语法  
    重构 Api  

    - 将权限从 Api 里面分离
    - 将自动生成工具从 Api 里面分离, 优化 res.js
    - 去除测试工具, 因为 flask 1.0 内置测试工具可以取代这个
    - 将 testing.py 改造成 res.py, 用于调用 API, 功能类似于 res.js

2016年3月 - 5月
:   内部项目使用 flask-restaction 框架, 项目已内测.  
    期间修复一些bug, 做了小的改进和优化, Api基本未变.  

**2016年5月 - 5月12日**
:   完善 res.js, 对代码进行了重构和测试, 支持模块化和标准 Promise.

**2016年7月 - 8月**
:   重写 validater, 形成完善的Schema语法.  
    重构 flask-restaction, 使用YAML格式定义输入输出Schema.  

**2016年9月 - 9月12日**
:   用NodeJS重写res.js, 支持用NodeJS和Python两种方式生成res.js.  
    支持生成HTML格式的API文档.

**2016年10月**
:   重构权限功能, 独立出TokenAuth, 增加灵活性和可拓展性.  
    Validater更名为Validr.
    文档从Sphinx迁移到MKDocs，并完成出英文文档
