---
layout: post
title: Sanic - Configuration
description: >
  Sanic은 응용 프로그램 객체의 config 속성에서 구성을 유지합니다.
author: Joo
noindex: true
---

# Sanic - Configuration

Sanic은 응용 프로그램 객체의 config 속성에서 구성을 유지합니다.

### dot - notation

~~~python
app = Sanic('myapp')
app.config.DB_NAME = 'appdb'
app.config.DB_USER = 'appuser'
~~~



### dictionary

~~~python
db_settings = {
    'DB_HOST': 'localhost',
    'DB_NAME': 'appdb',
    'DB_USER': 'appuser'
}
app.config.update(db_settings)
~~~



Configuration 을 로딩하기 위해 다양한 방법이 존재한다.

### From Environment Variables

SANIC_ 접두어로 정의 된 모든 변수는 sanic 구성에 적용됩니다. 예를 들어, SANIC_REQUEST_TIMEOUT 설정은 애플리케이션에 의해 자동으로 로드되어 REQUEST_TIMEOUT 구성 변수에 공급됩니다. Sanic에 다른 접두사를 전달할 수 있습니다.

~~~python
app = Sanic(load_env = 'MYAPP_')
~~~

그런 다음 위 변수는 MYAPP_REQUEST_TIMEOUT입니다. 환경 변수에서 로딩을 비활성화하려면 대신 False로 설정할 수 있습니다.

~~~python
app = Sanic(load_env=False)
~~~



### From an Object

만약 configuration 값이 많고 default 값이면 module에 넣는 것이 도움이 된다.

~~~python
import myapp.default_settings

app = Sanic('myapp')
app.config.from_object(myapp.default_settings)
~~~



### From a File

from_pyfile (/ path / to / config_file)을 사용하여 파일에서 Configuration을 로드 할 수 있습니다. 그러나 이를 위해서는 프로그램이 구성 파일의 경로를 알아야합니다. 따라서 환경 변수에서 구성 파일의 위치를 지정하고 구성 파일을 찾는 데 Sanic에 지시 할 수 있습니다.

~~~python
app = Sanic('myapp')
app.config.from_envvar('MYAPP_SETTINGS')
~~~

~~~
$ MYAPP_SETTINGS=/path/to/config_file python3 myapp.py
~~~



Configuration 파일은 대문자 변수 만 구성에 추가됩니다. 가장 일반적으로 구성은 간단한 키 값 쌍으로 구성됩니다.

~~~python
# config_file
DB_HOST = 'localhost'
DB_NAME = 'appdb'
DB_USER = 'appuser'
~~~

