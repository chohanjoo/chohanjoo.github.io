---
title: "Spring Test Code - Mock, Given-When-Then 패턴"
description: Spring Test Code - Mock, Given-When-Then 패턴
categories:
 - springboot
 - spring
tags:
- spring
- springboot
---

# Spring Test Code - Mock, Given-When-Then 패턴

## Given - When - Then

다음은 Give-When-Then pattern 의 예제이다. < [URL](https://martinfowler.com/bliki/GivenWhenThen.html)>

~~~
# example

Feature: User trades stocks
  Scenario: User requests a sell before close of trading
    Given I have 100 shares of MSFT stock
       And I have 150 shares of APPL stock
       And the time is before close of trading

    When I ask to sell 20 shares of MSFT stock
     
     Then I should have 80 shares of MSFT stock
      And I should have 150 shares of APPL stock
      And a sell order for 20 shares of MSFT stock should have been executed
~~~



즉, `[준비 - 실행 - 검증]`

다음 코드는 해당 [블로그](https://brunch.co.kr/@springboot/292) 에서 가져온 것이다.

( 커피의 가격을 할인해주는 메서드를 테스트한다. 
1000원짜리 아메리카노는 -100원 할인해서 900원을 리턴하는지 검증한다. )

~~~java
@Test
public void When_Get_Discount_Expect_Minus_100() {
    
    //given
    String name = "americano";
    int defaultPrice = 1000;
    int expectedPrice = 900;
    when(coffeeRepository.findOne(name))
        .thenReturn(Coffee.builder().name(name).isMilk(false)....)
        
    //when
    int actualPrice = coffeeService.getDiscountedPrice(name);
    
    //Then
    assertEquals(expectedPrice, actualPrice);
}
~~~



### Given

테스트를 준비하는 과정이다.

테스트에 사용하는 변수, 입력 값 등을 정의, Mock 객체 정의하는 구문 등이 Given에 포함된다.

> *Mockito - when*
>
> ## 아래의 모든 내용은 해당 [블로그](https://jdm.kr/blog/222) 에서 가져왔다.
>
> `Mockito` 란 단위 테스트를 위한 Java mocking framework 이다.
>
> **mock()**
> mock 객체를 만들어서 반환한다.
>
> ~~~java
> // Person 객체
> public class Person {
> 	private String name;
>     
>     public String getName() { return name; }
> }
> 
> // 목 객체 생성
> @Test
> public void test(){
>     Person ps = mock(Person.class);
>     assertTrue(p != null)
> }
> ~~~
>
> **@Mock **
>
> ~~~java
> @Mock
> Person ps;
> 
> @Test
> public void test(){
> 	MockitoAnnotations.initMocks(this);	// Mockito 어노테이션이 선언된 변수들은 객체를 생성
> 	assertTrue(p != null);
> }
> ~~~
>
> **when()**
>
> 목에 특정 조건을 지정한다.
>
> ~~~java
> @Test
> public void test(){
> 	Person ps = mock(Person.class);
> 	when(ps.getName()).thenReturn("ABC");
> 	assertTrue("ABC".equals(ps.getName()));
> }
> ~~~
>
> 



### When

실제로 액션을 하는 테스트를 실행하는 과정이다.



### Then

테스트를 검증하는 과정이다. 예상한 값, 실제 실행을 통해서 나온 값을 검증한다.





---

출처

https://brunch.co.kr/@springboot/292

https://jdm.kr/blog/222

https://nesoy.github.io/articles/2018-09/Mockito