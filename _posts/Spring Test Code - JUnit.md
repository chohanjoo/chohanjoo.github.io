# Spring Test Code - JUnit



## 아래 글은 해당 [블로그](https://nesoy.github.io/articles/2017-02/JUnit) 에서 가져왔다.



## JUnit Function Flow

1> setUp() : 테스트 대상 클래스의 실행전에 가장 먼저 setUp()을 실행한다.

​	ex) 네트워크 연결, DB 연결

2> tearDown() :  가장 마지막에 수행된다. setUp()의 반대 개념

** Test Case를 진행할때마다 반복적으로 실행

​	ex) setup -> TestA -> tearDown -> setup....



## JUnit Annotation

### @Test

~~~java
// 해당 Method는 Test 대상 메소드임을 의미

public class Example {
    @Test
    public void method() {
        
    }
}
~~~

### @Before

~~~java
// 해당 테스트가 진행이 시작되기 전에 작업할 내용을 호출

public class Example {
    List empty;
    @Before
    public void init(){
        empty = new ArrayList();
    }
}
~~~



-----

출처

https://nesoy.github.io/articles/2017-02/JUnit