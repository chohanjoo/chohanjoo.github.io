---
layout: post
title: Sanic - Request Data
description: >
  엔드 포인트가 HTTP 요청을 수신하면 라우트 함수에 요청 오브젝트가 전달됩니다. Request 객체의 속성으로 다음 변수에 액세스 할 수 있습니다.
author: Joo
noindex: true
---

# Sanic - Request Data

엔드 포인트가 HTTP 요청을 수신하면 라우트 함수에 요청 오브젝트가 전달됩니다.

Request 객체의 속성으로 다음 변수에 액세스 할 수 있습니다.

### json - JSON body

~~~python
from sanic.response import json

@app.route("/json")
def post_json(request):
    return json({ "received": True, "message": request.json })
~~~

### args (dict)

쿼리 문자열은 ?key1 = value1 & key2 = value2와 유사한 URL 섹션입니다. 해당 URL을 구문 분석해야하는 경우 args 사전은 { 'key1': [ 'value1'], 'key2': [ 'value2']}와 같습니다. 요청의 query_string 변수는 구문 분석되지 않은 문자열 값을 보유합니다. 속성이 기본 구문 분석 전략을 제공하고 있습니다. 

~~~python
from sanic.response import json

@app.route("/query_string")
def query_string(request):
    return json({ "parsed": True, "args": request.args, "url": request.url, "query_string": request.query_string })
~~~



### query_args (list)

대부분의 경우 덜 압축 된 형태로 url 인수에 액세스해야합니다. query_args는 (키, 값) 튜플의 목록입니다. 속성이 기본 구문 분석 전략을 제공하고 있습니다. 

동일한 이전 URL queryset? key1 = value1 & key2 = value2의 경우 query_args 목록은 [( 'key1', 'value1'), ( 'key2', 'value2')]와 같습니다. 

? key1 = value1 & key2 = value2 & key1 = value3과 같은 키를 가진 여러 매개 변수의 경우 query_args 목록은 [( 'key1', 'value1'), ( 'key2', 'value2'), ( 'key1 ','value3 ')].

~~~python
from sanic import Sanic
from sanic.response import json

app = Sanic(__name__)


@app.route("/test_request_args")
async def test_request_args(request):
    return json({
        "parsed": True,
        "url": request.url,
        "query_string": request.query_string,
        "args": request.args,
        "raw_args": request.raw_args,
        "query_args": request.query_args,
    })

if __name__ == '__main__':
    app.run(host="0.0.0.0", port=8000)
~~~

Output

~~~
http://0.0.0.0:8000/test_request_args?key1=value1&key2=value2&key1=value3
~~~

~~~json
{
  "parsed": true,
  "url": "http://0.0.0.0:8000/test_request_args?key1=value1&key2=value2&key1=value3",
  "query_string": "key1=value1&key2=value2&key1=value3",
  "args": {
    "key1": [
      "value1",
      "value3"
    ],
    "key2": [
      "value2"
    ]
  },
  "raw_args": {
    "key1": "value1",
    "key2": "value2"
  },
  "query_args": [
    [
      "key1",
      "value1"
    ],
    [
      "key2",
      "value2"
    ],
    [
      "key1",
      "value3"
    ]
  ]
}
~~~

### files (dictionary of File objects)

### form (dict) - Posted form variables

~~~python
from sanic.response import json

@app.route("/form")
def post_json(request):
    return json({ "received": True, "form_data": request.form, "test": request.form.get('test') })
~~~

### body (bytes) - Posted raw body

~~~python
from sanic.response import text

@app.route("/users", methods=["POST","GET"])
def create_user(request):
    return text("You are trying to create a user with the following POST: %s" % request.body)
~~~



## Accessing values using get and getlist

사전을 반환하는 요청 속성은 실제로 RequestParameters라는 dict의 하위 클래스를 반환합니다. 이 객체를 사용할 때의 주요 차이점은 get과 getlist 메소드의 차이점입니다.

- get (key, default = None)은 주어진 키의 값이 목록 인 경우 첫 번째 항목 만 반환된다는 점을 제외하고는 정상적으로 작동합니다.

- getlist (key, default = None)은 정상적으로 작동하여 전체 목록을 반환합니다.

~~~python
from sanic.request import RequestParameters

args = RequestParameters()
args['titles'] = ['Post 1', 'Post 2']

args.get('titles') # => 'Post 1'

args.getlist('titles') # => ['Post 1', 'Post 2']
~~~



## Accessing the handler name with the request.endpoint attribute

request.endpoint 속성은 핸들러 이름을 갔습니다. 예를 들어 아래 경로는 "hello"를 반환합니다.

~~~python
from sanic.response import text
from sanic import Sanic

app = Sanic()

@app.get("/")
def hello(request):
    return text(request.endpoint)
~~~

blueprint으로 마침표로 구분하여 둘 다 포함합니다. 예를 들어 아래 경로는 foo.bar를 반환합니다.

~~~python
from sanic import Sanic
from sanic import Blueprint
from sanic.response import text


app = Sanic(__name__)
blueprint = Blueprint('foo')

@blueprint.get('/')
async def bar(request):
    return text(request.endpoint)

app.blueprint(blueprint)

app.run(host="0.0.0.0", port=8000, debug=True)
~~~

