## DAO, DTO, mybatis

### mybatis

mybatis를 한마디로 SQL, 프로시저 등에 이름을 지어준다고 생각하자. 

mybatis는 DB 연결 설정, 쿼리, 결과 처리 등 각각의 부분을 분리하여 처리하기 쉽도록 도와주는 프레임워크다.

아래 코드는 쿼리부분을 mapper를 통해 분리한 것이다.

이렇게 되면 데이터를 처리하는 부분은 해당 쿼리의 이름을 호출하게 되고, 
쿼리를 수정하더라도 호출하는 부분은 바뀌지 않기 때문에 개발이 한층 편리해진다.

~~~xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE mapper
  PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
  "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="cafe">
    
    <select id="selectCustomer" resultType="java.lang.String">
        select                 
        age, product, price
        from
        Customer_Table
        where
        product = '아메리카노'
        and age > 19    
    </select>
    
</mapper>

~~~

--> DAO가 DB에서 데이터를 가져오고 
	가져온 데이터는 Service에서 처리하고
	Controller를 통해서 적절한 View에 전달된다.

~~~java
package com.first.begin.sample.dao;
 
public interface SampleDao {
 
    public String selectSampleData() throws Exception;
    
}

~~~

~~~java
package com.first.begin.sample.dao;
 
import org.mybatis.spring.SqlSessionTemplate;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Repository;
 
@Repository("sampleDao")
public class SampleDaoImpl implements SampleDao {
 
    @Autowired
    protected SqlSessionTemplate sqlSession;	// DB와의 연결을 열고 닫아주는 역할을 간편하게 대신함
    
    @Override
    public String selectSampleData() throws Exception {
        return sqlSession.selectOne("spl.selectDisposableTable");
        	// sqlSession을 통해 mapper로 만든 쿼리의 이름을 호출해서 데이터를 가져옴
    }
 
}
~~~



### DAO

Data Access Object의 줄임말 즉, Database의 data에 access하는 트랜잭션 객체이다.

DB를 사용해 데이터를 조회하거나 조작하는 기능을 담당하는 것들을 DAO라고 부른다.

Domain logic (비즈니스 로직이나 DB와 관련없는 코드들)을 persistence mechanism과 분리하기 위해 사용한다.

** persistence layer : Database에 data를 CRUD 하는 계층



사용자는 자신이 필요한 Interface를 DAO에게 던지고 DAO는 이 interface를 구현한 객체를 사용자에게 편리하게 사용 할 수 있도록 반환해준다,

DB에 대한 접근을 DAO가 담당하도록 하여 데이터베이스 액세스를 DAO에서만 하게 되면 다수의 원격호출을 통한 오버헤드를 VO나 DTO를 통해 줄일 수 있고 다수의 DB 호출문제를 해결할 수 있다.

Spring에서 DAO는 @Repository annotation으로 정의한다.



> 코드
>
> ~~~java
> # TestListDAO.java
> 
> public interface TestListDAO {
> 	public List<TestListResponseDTO> getTestListdao();
> }
> ~~~
>
> ~~~java
> # TestListDAOImpl.java
> 
> @Repository("testListDAO")
> public class TestListDAOImpl implements TestListDAO {
> 	
> 	@Override
> 	public List<TestListResponseDTO> getTestListdao(){
> 		List<TestListResponseDTO> testlist = new ArrayList<TestListResponseDTO>();
> 	
> 		testList.add(...)
> 		// Database 접근
> 		// 보통 SqlSession을 bean으로 만들어서 @Autowired로 멤버변수에 묶은 뒤 사용
> 		
> 		return testList;
> 	}
> }
> ~~~



### DTO

Data Transfer Object의 줄임말이다. VO (Value Object)라고도 표현한다.
계층간 데이터 교환을 위한 Java Beans 이다.

이 객체는 데이터베이스 레코드의 데이터를 매핑하기 위한 데이터 객체를 말한다. DTO는 보통 로직을 가지고 있지 않고 data와 그 data에 접근을 위한 getter, setter만 가지고 있다.



정리하면,
DTO는 Database에서 Data를 얻어 Service나 Controller 등으로 보낼 때 사용하는 객체를 말한다. 
위 코드에서도 DAO 가 Database로부터 Data를 얻은 뒤 List에 담아서 보내주고 있다.

> 코드
>
> ~~~java
> public class TestListResponseDTO {
>     private String testStr;
>     
>     public String getTestStr() {
>         return testStr;
>     }
> 
>     public void setTestStr(String testStr) {
>         this.testStr = testStr;
>     }
> }
> ~~~



### Service

Service는 비지니스 로직이 들어가는 부분이다. 

Controller 가 Request를 받으면 적절한 Service에 전달하고, 전달 받은 Service는 비즈니스 로직을 처리한다.
DAO로 데이터베이스를 접근하고,  DTO로 데이터를 전달받은 다음, 적절한 처리를 해 반환한다.



> 코드
>
> ~~~java
> public interface TestListService {
> 	public List<TestListResponseDTO> getTestListService();
> }
> ~~~
>
> ~~~java
> @Service("TestListService")
> public class TestListServiceImpl implements TestListService {
> 	@Autowired
> 	TestListDAOImpl testListDAO;
> 	
> 	@Override
> 	public List<TestListResponseDTO> getTestListService() {
> 		List<TestListResponseDTO> testList = testrListDAO.getTestListdao();
> 		
> 		return testList;
> 	}
> }
> ~~~



### Controller

> 코드
>
> ~~~java
> @Controller
> public class MainController {
>     @Autowired
>     TestListServiceImpl testListService;
> 
>     @RequestMapping(value = "/", method = RequestMethod.GET)
>     public ModelAndView home(ModelAndView mv) {
>         List<TestListResponseDTO> testList = testListService.getTestListService();
> 
>         for(int i = 0; i < testList.size(); i++) {
>             System.out.println("testStr: " + testList.get(i).getTestStr());
>         }
> 
>         mv.addObject("testList", testList);
>         mv.setViewName("main/mainview");
> 
>         return mv;
>     }
> }
> ~~~
>
> 

---

출처 

https://lazymankook.tistory.com/30

https://genesis8.tistory.com/214

https://jayviii.tistory.com/20?category=996149