---
layout: post
title: Sanic 시작하기
description: >
  Sanic은 Python 3.6 이상의 웹 서버 및 웹 프레임워크로, 빠르게 실행되도록 작성되었다. Python 3.5에 추가된 async/await 구문을 사용할 수 있어 코드를 차단하지 않고 빠르게 사용할 수 있다.
author: Joo
noindex: true
---

# Sanic

Sanic은 Python 3.6 이상의 웹 서버 및 웹 프레임워크로, 빠르게 실행되도록 작성되었다. Python 3.5에 추가된 async/await 구문을 사용할 수 있어 코드를 차단하지 않고 빠르게 사용할 수 있다.

이 프로젝트의 목표는 구축, 확장, 궁극적으로는 확장하기 쉬운 고성능 HTTP 서버를 가동하고 실행할 수 있는 간단한 방법을 제공하는 것이다.

## Getting Started

### Install Sanic

~~~
$ pip3 install sanic
~~~



### Create a file called main.py

~~~python
from sanic import Sanic
from sanic.response import json

app = Sanic()

@app.route("/")
async def test(request):
    return json({"hello": "world"})

if __name__ == "__main__":
    app.run(host="0.0.0.0", port=8000)
~~~

### Run the server

~~~
$ python3 main.py
~~~

### Check your browser

~~~
http://0.0.0.0:8000
~~~

