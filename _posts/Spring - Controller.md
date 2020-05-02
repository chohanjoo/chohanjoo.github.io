## Spring - Controller

### @Controller

### @RestController

Spring MVC Controller에 @ResponseBody가 추가된 것이다.

RestController의 주용도는 Json/Xml 형태로 객체 데이터를 반환하는 것이다.

![restcontroller](C:\Users\USER\Desktop\restcontroller.png)

1. Client는 URI 형식으로 웹 서비스에 요청을 보낸다.
2. Mapping 되는 Handler와 그 Type을 찾는 DispatherServlet이 요청을 인터셉트한다.
3. RestController는 해당 요청을 처리하고 데이터를 반환한다.



### @RequestMapping

컨트롤러에 @RequestMapping 어노테이션을 통해 URL을 매핑한다.

#### 클래스 레벨

@RequestMapping을 클래스에 붙이면 클래스 내 모든 메서드에 최상위 경로를 지정한다.

~~~java
@Controller
@RequestMapping("/test/*")
public class Test {
	@RequestMapping("/add")
	public String Add(Model model){		// test/add
		return "add";
	}
}
~~~

#### HTTP Request Method

~~~java
@Controller
@RequestMapping("/test")
public class Test {
	@RequestMapping("method = RequestMethod.GET")
	public String Add(Model model){		// test/add
		return "add";
	}
}
~~~



### @GetMapping

@RequestMapping ( method = RequestMethod.GET ) 의 축약형



### @ResponseStatus

원래의 response code를 사용자가 지정한 코드로 바꿔서 출력한다.

## Swagger

### @Api

~~~java
@Api(value = "설명")
public class abc { 

}
~~~



### @ApiOperation

~~~~java
@Api(value = "설명")
public class abc { 
	@ApiOperation(value = "설명")
	public void get_method(){
	
	}
}
~~~~



---

출처 : https://mangkyu.tistory.com/49