---
layout: post
title:  "[STL] Vector"
date:   2019-04-30 17:40:00
author: Hanjoo Cho
categories: Algorithm
tags:    Vector STL
cover:  "/assets/instacode.png"
---
# [STL] Vector

## vector ?

- 임의 접근 반복자를 지원하는 배열 기반 컨테이너
- 원소가 하나의 메모리 블록에 연속하게 저장
- 메모리 할당 크기를 알 수 있게 capacity() 함수, 한번에 메모리를 할당할 수 있는 reserve()함수 제공
- 원소가 연속하게 저장되므로 [] 연산자 또는 at으로 읽기에 빠름
- 앞쪽이 막혀 있는 형태로 앞쪽에는 원소를 추가/제거 할 수 없으며 뒤쪽에만 추가/제거 할 수 있다.



## 템블릿 형식

template<typename T, typename Allocator= allocator<T>> class vector

## 생성자

- vector v : v는 빈 컨테이너
- vector v(n) : v는 기본값으로 초기화된 n개의 원소를 갖는다.
- vector v(v2) : v는 v2 컨네이너의 복사본

## 멤버함수

- v.assign(n, x) : v에 x값으로 n개의 원소를 할당
- v.at(i) : v의 i번째 원소를 참조
- v.back() : v의 마지막 원소를 참조
- v.capacity() : v에 할당된 공간의 크기
- v.clear() : v의 모든 원소를 제거
- v.empty() : v가 비었는지 조사
- q = v.erase(p) : p가 가리키는 원소를 제거, q는 다음 원소를 가리킴
- v.front() : v의 첫번째 원소를 참조
- q = v.insert(p,x) : p가 가리키는 위치에 x값을 삽입, q는 삽입한 원소를 가리킴
- v.push_back(x) : v의 끝에 x를 추가
- v.pop_back() : v의 마지막 원소 제거
- v.swap(v2) : v와 v2를 swap한다.
- v.size() : v의 원소 갯수

## 연산자

- v1 == v2 : v1과 v2의 모든 원소가 같은가? (bool)
- v[i] : v의 i번째 원소를 참조

## 멤버형식

- size_type : 첨자(index)나 원소의 개수 등의 형식

~~~c++
#include <iostream>
#include <vector>
using namespace std;

int main(){

    vector<int> v;
    v.reserve(8);        // 벡터 메모리 공간 8 예약 할당

    v.push_back(10);
    v.push_back(20);
    v.push_back(30);
    v.push_back(40);
    v.push_back(50);

    for (vector<int>::size_type i = 0; i < v.size(); ++i)
        cout << v[i] << endl;
    cout << endl;

    cout << "size : " << v.size()                // 벡터 원소 개수
        << " capacity : " << v.capacity()            // 벡터의 할당된 메모리의 크기
        << "  max_size : " << v.max_size() << endl;    // 최대 저장 가능한 원소 개수


    cout << endl << "-- resize(10) -- " << endl;
    
    v.resize(10);            // 벡터의 원소 갯수를 10개로 확장, 추가된 공간은 디폴트 0으로 채워진다.
    
    for (vector<int>::size_type i = 0; i < v.size(); ++i)   // 벡터의 size 타입으로 i를 지정한다 (unsigned int) 
        cout << v[i] << endl;
    cout << endl;

    cout << "size : " << v.size()
        << " capacity : " << v.capacity()
        << "  max_size : " << v.max_size() << endl;


    cout << endl << "-- resize(3) -- " << endl;

    v.resize(3);            // 벡터의 원소 갯수를 3개로 축소, capacity는 변화 없다.

    for (vector<int>::size_type i = 0; i < v.size(); ++i)
        cout << v[i] << endl;
    cout << endl;

    cout << "size : " << v.size()                
        << " capacity : " << v.capacity()            
        << "  max_size : " << v.max_size() << endl;


    cout << endl << "-- vector clear() -- " << endl;

    v.clear();    // 벡터 비운다    capacity(메모리) 는 그대로 남아있다.

    if (v.empty()){    // 벡터에 원소가 없는지 확인한다.
        cout << "벡터에 원소가 없다." << endl;
    }

    cout << "size : " << v.size()
        << " capacity : " << v.capacity()
        << "  max_size : " << v.max_size() << endl;

    return 0;
}

~~~



---

출처

https://hyeonstorage.tistory.com/324
