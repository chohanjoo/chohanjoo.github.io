---
layout: post
title: Sanic - Examples
description: >
  애플리케이션 개발을 빠르게 시작할 수 있도록 도와주는 간단한 예제 코드 모음입니다. 
author: Joo
noindex: true
---

# Sanic - Examples

### Simple Apps

~~~python
from sanic import Sanic
from sanic import response as res

app = Sanic(__name__)


@app.route("/")
async def test(req):
    return res.text("I\'m a teapot", status=418)


if __name__ == '__main__':
    app.run(host="0.0.0.0", port=8000)
~~~

~~~python
from sanic import Sanic
from sanic import response

app = Sanic(__name__)


@app.route("/")
async def test(request):
    return response.json({"test": True})


if __name__ == '__main__':
    app.run(host="0.0.0.0", port=8000)
~~~



### Simple App with `Sanic Views`

`sanic.viewes.HTTPMethodView`를 사용하는 간단한 메커니즘과 뷰에 대한 사용자 정의 비동기 동작을 제공하도록 확장하는 방법을 보여줍니다.

~~~python
from sanic import Sanic
from sanic.views import HTTPMethodView
from sanic.response import text

app = Sanic('some_name')


class SimpleView(HTTPMethodView):

    def get(self, request):
        return text('I am get method')

    def post(self, request):
        return text('I am post method')

    def put(self, request):
        return text('I am put method')

    def patch(self, request):
        return text('I am patch method')

    def delete(self, request):
        return text('I am delete method')


class SimpleAsyncView(HTTPMethodView):

    async def get(self, request):
        return text('I am async get method')

    async def post(self, request):
        return text('I am async post method')

    async def put(self, request):
        return text('I am async put method')


app.add_route(SimpleView.as_view(), '/')
app.add_route(SimpleAsyncView.as_view(), '/async')

if __name__ == '__main__':
    app.run(host="0.0.0.0", port=8000, debug=True)
~~~



### URL Redirect

~~~python
from sanic import Sanic
from sanic import response

app = Sanic(__name__)

    
@app.route('/')
def handle_request(request):
    return response.redirect('/redirect')


@app.route('/redirect')
async def test(request):
    return response.json({"Redirected": True})


if __name__ == '__main__':
    app.run(host="0.0.0.0", port=8000)
~~~



### Named URL redirection

`Sanic`은 고유 한 URL 이름을 인수로 사용하여 할당 된 실제 경로를 반환하는 `url_for`라는 도우미 메서드를 통해 요청을 리디렉션하는 사용하기 쉬운 방법을 제공합니다. 이를 통해 응용 프로그램의 다른 섹션간에 사용자를 리디렉션하는 데 필요한 노력을 단순화 할 수 있습니다.

```python
from sanic import Sanic
from sanic import response

app = Sanic(__name__)


@app.route('/')
async def index(request):
    # generate a URL for the endpoint `post_handler`
    url = app.url_for('post_handler', post_id=5)
    # the URL is `/posts/5`, redirect to it
    return response.redirect(url)


@app.route('/posts/<post_id>')
async def post_handler(request, post_id):
    return response.text('Post - {}'.format(post_id))
    
if __name__ == '__main__':
    app.run(host="0.0.0.0", port=8000, debug=True)
```

