# [c++] pair



## pair 클래스란?

- 두 객체를 하나의 객체로 취급 할 수 있게 묶어주는 클래스
- STL에서 데이터 쌍을 표현할 때 사용
- <utility> 헤더에 존재

## pair 클래스 생김새

- template <class T1, class T2> struct pair;

- template <typename T1, typename T2> struct pair;

  T1 : first

  T2 : second

## 사용법

- pair<[type1], [type2]> p	=> 사용할 데이터 타입1, 2를 넣고 그 타입의 pair 클래스인 p를 만든다.
- p.first           ->        p의 첫번째 인자를 반환
- p.second      ->       p의 두번째 인자를 반환
- make_pair( 변수1, 변수 )         ->         변수1과 변수2가 들어긴 pair를 만든다.
- operator로 (==, !=, <, >, <=, >=)가 정의 되어있어서, 사용이 가능
- sort 알고리즘에 의해 정렬이 가능

**대소 비교 or sort : 첫번째 인자 기준, 첫번째가 같으면 두번째 인자로 판단**



예제)

~~~

~~~



----

참고

https://blockdmask.tistory.com/64