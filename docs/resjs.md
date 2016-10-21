# resjs

res.js is a AJAX lib used for call API in a easy way.

## Generate res.js

Use cli provided by flask-restaction or https://www.npmjs.com/package/resjs
can generate res.js, they has the same usage and generate the same code.

Usage:

    usage: resjs [-h] [-d DEST] [-p PREFIX] [-n] [-m] url

    generate res.js for browser or nodejs

    positional arguments:
      url                   url of api meta

    optional arguments:
      -h, --help                  show this help message and exit
      -d DEST, --dest DEST        dest path to save res.js
      -p PREFIX, --prefix PREFIX  url prefix of generated res.js
      -n, --node                  generate res.js for nodejs, default for browser
      -m, --min                   minimize generated res.js, default not minimize

Example:

    resjs http://127.0.0.1:5000 -d static/res.js


## Usage of res.js

res.js use [SuperAgent][SuperAgent] to send HTTP requests, and use [babel-plugin-transform-runtime](https://babeljs.io/docs/plugins/transform-runtime)
to polyfill Promise.

It will add  auth token(Authorization) to request headers, and save
auth token(Authorization) from response headers to localStorage of browser.

Usage:

    //by module loader(UMD)
    var res = require('./res.js');

    //by script tag
    <script type="text/javascript" src="/static/res.js"></script>


### res.ajax

res.ajax is [SuperAgent][SuperAgent].


### res.resource.action

Usage:

    res.resource.action({
        // some data
    }).then(function(value){
        // success
    }).catch(function(error){
        // error
    })

Example, call Hello API:

    // Hello World
    res.hello.get({
        name: 'kk'
    }).then(function(value){
        console.log(value.message);
    }).catch(function(error){
        // error
    })


### res.xxxToken

    // clear token saved in localStorage
    res.clearToken()
    // get token saved in localStorage
    var token = res.getToken()
    // set token saved in localStorage
    res.setToken(token)


### res.config

    //set url prefix, used for specify backend server
    res.config.urlPrefix = 'http://127.0.0.1:5000'
    //set auth header, note that use lowercase letters
    res.config.authHeader = 'authorization'


[SuperAgent]: http://visionmedia.github.io/superagent/
