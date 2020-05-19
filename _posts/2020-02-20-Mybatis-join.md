---
title: "[sns project] Mybatis join"
description: mybatis join query를 연습한다.
categories:
 - project
 - spring
tags:
- project
- sns
- spring
---

# Mybatis

Database에서 select 절을 이용해 데이터를 조회할 대 mybatisdml resultMap을 이용하면 사용자가 정의한 Model 객체로 값을 직접 받아올 수 있다. 

그렇다면 Model 객체안에 멤버면수로 또 다른 Model 객체가 있다면 해당 객체에는 값을 어떻게 넣을까?



## Join (1:N)

문제 : 두 테이블을 조인 하는 과정 가운데 N개가 나오는 것이 아니라 1개만 나왔다.

상황 :

### Dto

~~~java
public class AAADto {
	private int a_id_1;
    private String a_2;
    
    private ArrayList<BBBDto> bbbDto;
}
~~~

~~~java
public class BBBDto {
    private int b_id_1;
    private String b_id_2;
    private String b_3;
}
~~~



### DB Table

####  AAADto

| Column_Name |   Datatype    |  PK   |  NN   |
| :---------: | :-----------: | :---: | :---: |
|   a_id_1    |      INT      | check | check |
|     a_2     | VARCHAR(2048) |       | check |

#### BBBDto

| Column_Name |   Datatype    |  PK   |  NN   |
| :---------: | :-----------: | :---: | :---: |
|   b_id_1    |      INT      | check | check |
|   b_id_2    | VARCHAR(100)  | check | check |
|     b_3     | VARCHAR(2048) | check | check |



### Mapper

~~~xml
<resultMap id="aaaListMap" type="AAADto">
	<id property="a_id_1" column="a_id_1"/>
    <result column="a_2" property="a_2"/>
    <collection property="bbbDto" resultMap="bbbListMap" ofType="java.util.ArrayList" />
</resultMap>

<resultMap id="bbbListMap" type="BBBDto">
	<id property="b_id_1" column="b_id_1"/>
    <result column="b_3" property="b_3"/>
</resultMap>

<select id="joinList" resultMap="aaaListMap">
	select
    	a.a_id_1,
    	a.a_2,
    	b.b_id_2,
    	b.b_3
    from aaa a
    left join bbb b
    on a.a_id_1 = b.b_id_1
    where a.a_2 = #{value} 
</select>
~~~

여기서 `collection` 과 `association` 의 차이는 _has one_ 관계 일때는 `association` , _has many_ 관계 일때는 `collection` 을 사용한다는 것을 알고가자 [참고](http://noveloper.github.io/blog/spring/2015/05/31/mybatis-assocation-collection.html)



해결 : 

다양한 블로그를 참조했으나 해결이 되지 않았다.

[blog1](https://noritersand.github.io/mybatis/2013/11/28/mybatis-1-n-관계-데이터-처리-data-concatenation/)

[blog2](https://gubok.tistory.com/400)

[blog3](https://queserasera.tistory.com/15)

[blog4](https://ssssssu12.tistory.com/4)

[blog5](https://ssodang.tistory.com/entry/MyBatis-두테이블-정보를-한개의-모델로-Join-쿼리로-받기)

이분들과 차이점은 두번째 테이블 (BBBDto) 의 primary 키가 2개였던 것 같다.

### Mapper

~~~xml
<resultMap id="aaaListMap" type="AAADto">
	<id property="a_id_1" column="a_id_1"/>
    <result column="a_2" column="a_2"/>
    <collection property="bbbDto" resultMap="BBBDto" orType="java.util.ArrayList" />
</resultMap>

<resultMap id="bbbListMap" type="BBBDto">
	<id property="b_id_1" column="b_id_1"/>
    <id property="b_id_2" column="b_id_2"/>
    <result column="b_3" property="b_3"/>
</resultMap>

<select id="joinList" resultMap="aaaListMap">
	select
    	a.a_id_1,
    	a.a_2,
    	b.b_id_2,
    	b.b_3
    from aaa a
    left join bbb b
    on a.a_id_1 = b.b_id_1
    where a.a_2 = #{value} 
</select>
~~~

문제가 됬던 bbbListMap에서 id값을 primary 키와 동일하게 2개 줬더니 예상대로 문제가 해결되었다.



결과 :

~~~json
[
  {
    "a_id_1": 70,
    "a_2": "test",
    "bbbDto": [
      {
        "b_id_1": 70,
        "b_id_2": "HAND",
        "b_3": "Good",
      },
      {
        "b_id_1": 70,
        "b_id_2": "KR62431",
        "b_3": "Good",
      },
      {
        "b_id_1": 70,
        "b_id_2": "REACT",
        "b_3": "Good"
      },
      {
        "b_id_1": 70,
        "b_id_2": "SINGAPORE",
        "b_3": "Good"
      }
    ]
  }
]
~~~



해결해야 할 문제 : select 에 포함되지 않은 내용도 나온다. 