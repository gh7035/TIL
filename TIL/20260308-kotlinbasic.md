# kotlin 이란?

Java 와 100% 호환 되면서 Java 보다 더 간결한 문법으로 생산성을 높이고, 널(null) 안정성을 지원하는 언어

## 기본 문법
> 변수를 선언 할 때
- 가변 객체를 선언할 때 -> var
- 불변 객체를 선언할 때 -> val

> 타입 추론

```kotlin
val name = "John" // String 타입으로 추론됨
var age = 30 // Int 타입으로 추론됨

var result: Double // 초기화하지 않을 때는 타입을 명시해야 함
val number: Int = 5 // 명시적으로 Int 타입으로 지정
```
<br></br>

## 스트링 템플릿

'*$*' 표시를 사용해서 변수처럼 사용 가능하다
```kotlin
val name = "Seol"
val age = 20
println("My name is ${name}") //My name is Seol
println("My age is ${age+2}") //My age is 22
```
<br></br>

## 제어문

#### Range Class
배열이 아니라 "Range"라는 새로운 타입. 배열로 구현할때 보다 메모리 사용량이 훨씬 적다.

```kotlin
val numRange : IntRange = 1 .. 5
val charRange : CharRange = 'a'..'z'

// 타입 추론 기능을 사용해서 작동 가능
val numRange = 1..5
val charRange = 'a'..'z'
```


- ```1..5```는 5를 포함하지만, 마지막 숫자를 제외하고 싶을 때는 ```until```을 사용합니다.
```kotlin
1..5 -> 1, 2, 3, 4, 5 (포함)
1 until 5 -> 1, 2, 3, 4 (5 제외)
5 downTo 1 -> 5, 4, 3, 2, 1 (역, 포함)

+ step (int) -> 간격 정의
```

#### 함수 선언

```kotlin
형식:

fun 함수이름 (arg: type) : returnType { }


//예제
fun main() {
    fun addNum (num1:Int, num2:Int):Int{
        return num1+num2
    }
    println(addNum(1, 3));

}
```