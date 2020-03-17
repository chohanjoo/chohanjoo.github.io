---
title: "Django AWS에서 배포하기"
description: Django AWS에서 배포
categories:
 - project
 - aws
tags:
- project
- aws
---

# Django AWS에서 배포하기



## 1. ssh로 AWS 접속

~~~shell
$ ssh -i /path/my-key-pair.pem ubuntu@public_ip
~~~



## 2. git clone

~~~shell
$ git clone git@...
~~~



## 3. Nginx를 이용한 포트 설정

Nginx는 서버 앞단에서 request를 가장 먼저 처리하는 웹 서버이다.
Nginx를 이용하여 기본 포트인 80으로 들어와도 웹 사이트에 접근할 수 있도록 한다.

~~~shell
$ sudo apt-get install nginx
~~~



### 설정파일 생성

~~~shell
$ vim /etc/nginx/sites-enable/django-website

server {
		listen 80; 
		server_name 123.123.123.123;	// public ip 
		location / { 
				proxy_pass http://127.0.0.1:8000/; 
		} 
		location /static/ { 
				alias /home/ubuntu/django-website/ypc/staticfiles; 
			} 
}

~~~



### 설정파일 저장 후 nginx 재실행

~~~shell
$ sudo nginx -t

$ sudo service nginx restart
~~~



## 4. uWSGI로 django server 실행

### uWSGI

WSGI는 장고와 웹서버를 연결해주는 Python 프레임워크이다.

웹서버가 직접적으로 Python으로 된 장고와 통신할 수 없기 때문에 그 사이에서 WSGI Server (middleware) 가 실행되어 웹서버와 장고를 연결해준다.

웹서버가 전달받은 사용자의 요청을 WSGI Server에서 처리하여 Django로 넘겨주고, 다시 Django가 넘겨준 응답을  WSGI Server가 받아서 웹 서버에 전달한다.

~~~~shell
$ pip3 install uwsgi
~~~~



Django의 프로젝트 폴더 안에 있는 프로젝트명의 폴더를 보면, wsgi.py 파일이 들어 있을 것이다.
이것을 uWSGI로 서비스 하려면 다음과 같이 하면 된다.
내 프로젝트명은 lily 이다.

프로젝트의 전체 경로는 /home/ubuntu/lily 이며 wsgi.py 파일은 /home/ubuntu/lily/wsgi.py 에 있다.

모듈의 경로는 lily/wsgi.py를 넣는 것이 아니라 html.wsgi 로 넣는다. 

~~~
uwsgi \
--http :[포트번호] \
--home [virtualenv 경로] \
--chdir [장고프로젝트폴더 경로] \
-w [wsgi 모듈명].wsgi
~~~



~~~shell
$ uwsgi --http :8000 --chdir /home/ubuntu/lily/ -w lily.wsgi
~~~



여기까지 설정을 하면 public ip로 접속했을 때 django server로 연결된다.

이제 구입한 도메인과 연결하자

## public ip와 도메인 연결

도메인은 hosting.kr 에서 구입하였다. ( co.kr - 1년 구독 9800원 )

AWS 인스턴스와 구입한 도메인은 AWS Route53을 이용한다.

> AWS Route53을 이용한 도메인 연결 : 
>
> https://wingsnote.com/57
>
> https://tech.cloud.nongshim.co.kr/2018/10/16/초보자를-위한-aws-웹구축-8-무료-도메인으로-route-53-등록-및-elb/



** hosting.kr 의 기본 도메인

~~~
ns1.hosting.co.kr
ns2.hosting.co.kr
~~~



---

출처

https://dailyheumsi.tistory.com/19

https://crystalcube.co.kr/205

https://twpower.github.io/41-connect-nginx-uwsgi-django

https://nachwon.github.io/django-deploy-2-wsgi/

https://brownbears.tistory.com/174