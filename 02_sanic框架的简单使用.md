# 1. sanic简单使用

### 1.1 创建第一个 sanic 代码

```python
from sanic import Sanic
from sanic.response import text

app = Sanic(__name__)

@app.route("/")
async def test(request):
    return text('Hello world!')

app.run(host="0.0.0.0", port=8000, debug=True)
# python main.py
```

## 1.2 路由(routing)

```python
@app.route('/')
def index():
    return text('Index Page')

@app.route('/hello')
def hello():
    return text('Hello World')
```

### 1.3 路径传参变量规则

```python
from sanic.response import text

@app.route('/tag/<tag>')
async def tag_handler(request, tag):
    return text('Tag - {}'.format(tag))

@app.route('/number/<integer_arg:int>')
async def integer_handler(request, integer_arg):
    return text('Integer - {}'.format(integer_arg))

@app.route('/number/<number_arg:number>')
async def number_handler(request, number_arg):
    return text('Number - {}'.format(number_arg))

@app.route('/person/<name:[A-z]>')
async def person_handler(request, name):
    return text('Person - {}'.format(name))

@app.route('/folder/<folder_id:[A-z0-9]{0,4}>')
async def folder_handler(request, folder_id):
    return text('Folder - {}'.format(folder_id))
```

### 1.4 HTTP请求类型

默认情况下, 定义的URL只支持GET请求, @app.route装饰器提供了一个可选参数methods,这个参数允许传入所有HTTP方法

```python
from sanic.response import text

@app.route('/post', methods=['POST'])
async def post_handler(request):
    return text('POST request - {}'.format(request.json))

@app.route('/get', methods=['GET'])
async def get_handler(request):
    return text('GET request - {}'.format(request.args))
```

也可简写为:

```python
from sanic.response import text

@app.post('/post')
async def post_handler(request):
    return text('POST request - {}'.format(request.json))

@app.get('/get')
async def get_handler(request):
    return text('GET request - {}'.format(request.args))
```

### 1.5 add_route方法

除了`@app.route`装饰器，Sanic 还提供了`add_route`方法。

`@app.route 只是包装了 add_route方法。`

```python
from sanic.response import text

# Define the handler functions
async def handler1(request):
    return text('OK')

async def handler2(request, name):
    return text('Folder - {}'.format(name))

async def person_handler2(request, name):
    return text('Person - {}'.format(name))

# Add each handler function as a route
app.add_route(handler1, '/test')
app.add_route(handler2, '/folder/<name>')
app.add_route(person_handler2, '/person/<name:[A-z]>', methods=['GET'])
```

### 1.6 URL构建

url_for() 函数就是用于构建指定函数的URL的。它把函数名称作为第一个参数，其余参数对应URL中的变量，例如：

```python
@app.route('/')
async def index(request):
    # generate a URL for the endpoint `post_handler`
    url = app.url_for('post_handler', post_id=5)
    # the URL is `/posts/5`, redirect to it
    return redirect(url)


@app.route('/posts/<post_id>')
async def post_handler(request, post_id):
    return text('Post - {}'.format(post_id))
```

未定义变量会作为URL的查询参数：

```python
url = app.url_for('post_handler', post_id=5, arg_one='one', arg_two='two')
# /posts/5?arg_one=one&arg_two=two

# 支持多值参数
url = app.url_for('post_handler', post_id=5, arg_one=['one', 'two'])
# /posts/5?arg_one=one&arg_one=two
```

### 1.7 使用蓝图(Blueprint)

- 把一个应用分解为一套蓝图。大型应用的理想方案：一个项目可以实例化一个 应用，初始化多个扩展，并注册许多蓝图。
- 在一个应用的 URL 前缀和（或）子域上注册一个蓝图。 URL 前缀和（或）子域的参数 成为蓝图中所有视图的通用视图参数（缺省情况下）。
- 使用不同的 URL 规则在应用中多次注册蓝图。
- 通过蓝图提供模板过滤器、静态文件、模板和其他工具。蓝图不必执行应用或视图 函数。

blueprint示例:

```python
from sanic import Sanic
from sanic.response import json
from sanic import Blueprint

bp = Blueprint('my_blueprint')

@bp.route('/')
async def bp_root(request):
    return json({'my': 'blueprint'})

app = Sanic(__name__)
#注册蓝图
app.blueprint(bp)

app.run(host='0.0.0.0', port=8000, debug=True)
```

### 1.8 使用蓝图注册全局中间件

```python
@bp.middleware
async def print_on_request(request):
    print("I am a spy")

@bp.middleware('request')
async def halt_request(request):
    return text('I halted the request')

@bp.middleware('response')
async def halt_response(request, response):
    return text('I halted the response')
```

### 1.9 使用蓝图处理异常

```python
@bp.exception(NotFound)
def ignore_404s(request, exception):
    return text("Yep, I totally found the page: {}".format(request.url))
```

### 1.10 使用蓝图处理静态文件

```python
#第一个参数指向当前的Python包
#第二个参数是静态文件的目录
bp.static('/folder/to/serve', '/web/path')
```

### 1.11 使用url_for

如果要创建页面链接,可以和通常一样使用url_for()函数, 只是要把蓝图名称作为端点的前缀,并且用一个(.)来分割

```python
@blueprint_v1.route('/')
async def root(request):
    url = app.url_for('v1.post_handler', post_id=5) # --> '/v1/post/5'
    return redirect(url)


@blueprint_v1.route('/post/<post_id>')
async def post_handler(request, post_id):
    return text('Post {} in Blueprint V1'.format(post_id))
```

