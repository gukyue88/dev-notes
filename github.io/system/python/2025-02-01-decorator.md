---
layout: single
title: "[python] decorator"
categories: system/python
published: false
---

#### 결론 코드

```python
import functools

def decorator(func):
	@functools.wraps(func)
	def wrapper(*args, **kwargs):
		print("wrapper")
		return func(*args, **kwargs)
	return wrapper
```

## 개념

#### 일급 함수란? (First-class function)

함수를 다른 일반적인 "값"처럼 다룰 수 있는 특성 == 함수가 일급 객체임

#### 일급이란? (First-class)

다른 일반적인 요소들과 동등하게 다루어지는 대상 앞에 붙임

- 함수의 실제 매개변수가 될 수 있다.
- 함수의 반환 값이 될 수 있다.
- 할당 명령문의 대상이 될 수 있다.
- 동일 비교의 대상이 될 수 있다.

> 왜? 일급이라는 이름이 사용되었나? 예전에 현실사회에서 First-class Citizen 라는 용어를 사용했는데, 일급 시민은 다른 시민들과 동등하게 모든 권리와 특권을 누리는 사람을 의미했음.

- 보통 일급이 되는 대상을 First-class Citizen, First-class type, First-class object 등으로 부른다.

#### closure

first-class function 개념을 이용해 scope에 묶인 변수를 바인딩(하나를 다른 것으로 매핑)하기 위한 일종의 기술

- 클로저는 함수를 저장한 레코드(자료구조)
- 스코프의 인수들은 클로저가 만들어질 때 정의

```python
def startAt(x):
	def incrementBy(y):
		return x + y
	return incrementBy

closure1 = startAt(1) # closure1 = lambda y: 1 + y
closure2 = startAt(2) # closure2 = lambda y: 2 + y
```

## 데코레이터

기존의 코드에 여러가지 기능을 추가하는 파이썬 구문?

데코레이터는 closure를 만드는 함수의 argument에 함수가 넘어간다.

```python
def decorator_function(original_function):
	def wrapper_function():
		print("wrapper_function")
		return original_function()
	return wrapper_function

def say_hello():
	print("Hello")

say_hello = decorator_function(say_hello) # lambda : print("wrapper_function"); say_hello()
```

#### "@" 심볼 활용

```python
@decorator_function
def say_hello():
	print("Hello")
```

#### original_function에 인자가 있는 경우를 고려

```python
def decorator(original):
	def wrapper(*args, **kwargs):
		print("wrapper")
		return original(*args, **kwargs)
	return wrapper
```

(**init**, **call**)을 활용하여 클래스 형태로 데코레이터를 작성할 수 있지만, 함수형태로 널리 쓰임

#### wraps

데코레이터에 의해 함수가 감싸지면서 오류 추적 및 문서화에 문제가 생김
wraps는 그런 문제를 해결해줌

```python
import functools

def decorator(original):
	@functools.wraps(original)
	def wrapper(*args, **kwargs):
		print("wrapper")
		return original(*args, **kwargs)
	return wrapper
```
