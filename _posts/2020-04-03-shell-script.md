---
title: "[sns project] Shell Script 문법 정리"
description: 프로젝트를 진행하는 동안 사용했던 shell script 문법을 정리한다.
categories:
 - project
 - shell script
tags:
- project
- shell script
---

# Shell Script



## output 제거

~~~shell
curl --silent --output /dev/null http://example.com
~~~



## Sleep

쉘 스크립트를 사용하다보면 스크립트 중간에 그 다음 로직으로 넘어가기 전에 특정 시간동안 멈추고 싶을 때 사용

~~~shell
sleep 5 # waits 5 second
sleep 5m # waits 5 minutes
~~~



## 무한 루프

~~~shell
#!/bin/bash

while :
do
     echo "Hello World!"
     sleep 1
done
~~~



## JSON Parsing

jq 에 대한 내용은 해당 [블로그](https://www.44bits.io/ko/post/cli_json_processor_jq_basic_syntax) 를 참고하였다.

jq는 JSON 포맷의 데이터를 다루는 커맨드라인 유틸리티이다.

### jq install

~~~shell
$ brew install jq
~~~



### how to use

~~~shell
$ echo '{"foo" : "bar"}' | jq '.'

{
	"foo" : "bar"
}
~~~

#### 오브젝트 속성 필터

오브젝트의 특정한 속성을 가져오려면 점(.) 뒤에 이름을 붙여주면 된다.

~~~shell
$ echo '{"foo": "bar", "hoge": "piyo"}' | jq '.foo'

>> "bar"
~~~





### option

`--raw-output, -r` : jq를 사용해 JSON의 문자열이 출력되는 경우 쌍따옴표로 감싸진 문자열이 출력된다. 이 옵션을 사용하는 경우 쌍따음표 없이 문자열만 출력한다. 객체나 배열의 경우 그대로 JSON 형식으로 출력된다.