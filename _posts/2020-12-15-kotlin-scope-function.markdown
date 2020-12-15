---
layout: post
title:  "[Kotlin] 코틀린 표준 스코프 함수 정리"
date:  2020-12-15 23:59:59 +0900
categories: Kotlin
tags: [Kotlin]
---

# 스코프 함수란?

코틀린 표준 라이브러리에는 <u>Object Context 내에서 코드 블록을 실행하는 것이 목적</u>인 여러 함수가 있다.
제공된 람다 식을 사용하여 객체에 이러한 함수들을 호출하면 임시적으로 스코프(범위)가 설정된다.
이 범위에서는 해당 객체의 이름 없이 접근할 수 있다.

총 5가지의 함수가 존재한다.

`let`  `run`  `with`  `apply`  `also`

이 5개의 함수는 기본적으로 같은 일을 한다.

- **객체에 붙어 있는 코드 블록을 실행**

그렇다면 이 5개의 함수의 차이점은 무엇일까?

- **해당 객체를 어떻게 코드 블록 내에서 사용 가능하게 만드는가?**
- **전체 표현식의 결과가 무엇인가?**

5가지 함수의 정의는 아래와 같다.

```kotlin

inline fun <T, R> T.run(block: T.() -> R): R {
    return block()
}

inline fun <T, R> with(receiver: T, block: T.() -> R): R {
    return receiver.block()
}

inline fun <T> T.apply(block: T.() -> Unit): T {
    block()
    return this
}

inline fun <T> T.also(block: (T) -> Unit): T {
    block(this)
    return this
}

inline fun <T, R> T.let(block: (T) -> R): R {
    return block(this)
}

```

![Kotlin Scope Function Table](https://miro.medium.com/max/700/1*Qt5rTtOpAeGhxfuw7IOsRA.png)

그럼, 다 비슷한데 어떤 상황일 때 어떤 함수를 사용해야하는가?

각각 함수를 사용해야하는 경우를 살펴보자.

<br>

# 1. run

- `run` 함수는 확장 함수이기 때문에 context object를 receiver(**this**)로 이용할 수 있다.
- `run` 함수는 반환 결과가 람다의 결과이다.
- `run` 함수는 객체의 초기화와 리턴 값의 계산을 람다가 포함할 때 유용하다.
- 확장함수이기 때문에 safe call(`.?`)을 붙여 non-null 일 때에만 실행할 수 있다.
- 주로 어떤 값을 계산할 필요가 있거나 여러 개의 지역변수 범위를 제한할 때 사용한다.
    
```kotlin
data class Member(val name: String, var age: Int)

val member = Member("Wangi", 26)
val nextYearHisAge = member.run {
    ++age               // this.age
}

println(nextYearHisAge) // 27
```

위의 `run`과 다른 형태의 `run`도 존재한다.

```kotlin
inline fun <R> run(block:() -> R): R{
    return block()
}
```

이 `run`은 확장 함수도 아니고, 블록에 입력값도 없다. 따라서 객체를 전달받아서 속성을 변경하는 형식에 사용되지 않는다.

이 함수는 단지 어떤 객체를 생성하여 명령문을 블록 안에 적음으로써 가독성을 높이는 역할을 한다.

```kotlin
val member = run {
    val name = "Wangi"
    val age = 26
    Member(name, age)
}
```

<br>

# 2. with

- `with` 함수는 확장 함수가 아니기 때문에 context object를 **argument**로 전달한다. 그러나, 람다의 내부에는 확장함수로 적용되어서 `this`로 사용가능하다.
- `with` 함수는 반환 결과가 람다의 결과이다.
- `with` 함수는 수신 객체는 non-nullable이고, 결과가 필요하지 않은 경우에 유용하다.
        
```kotlin
val member = Member("Wangi", 26)
with(member) {
    println("This member name is $name") // this.name
    println("This member age is $age")   // this.age
}
```

<br>

# 3. apply

- `apply` 함수는 확장 함수이기 때문에 context object를 receiver(**this**)로 이용할 수 있다.
- `apply` 함수는 반환 결과가 객체 자신이다. `Builder` 패턴과 동일한 용도로 사용된다.
- `apply` 함수는 객체의 프로퍼티 만을 사용하는 경우가 많으며, 대표적인 사례는 객체의 초기화이다.

```kotlin
val member = Member("Wangi").apply{
    age = 26                // this.age
}

println(member)             // Member(name=Wangi, age=26)
```

<br>

# 4. also

- `also` 함수는 확장 함수이기 때문에 context object를 receiver(**this**)로 전달한다. 그러나, 코드 블럭 내에서 this를 파라미터로 입력하기 때문에 `it`을 사용해 프로퍼티에 접근할 수 있다.
- `also` 함수는 반환 결과가 객체 자신이다. `Builder` 패턴과 동일한 용도로 사용된다.
- `also` 함수는 객체의 속성을 전혀 사용하지 않거나 변경하지 않고 사용하는 경우에 유용하다. 예를 들면, 객체의 데이터 유효성을 확인하거나, 디버그, 로깅 등의 부가적인 목적으로 사용할 때에 적합하다.
    
```kotlin
class Membership(member: Member) {
    val member = member.also {
        requireNotNull(it.age)
        println(it.name)
    }
}
```

`also`를 사용하지 않는 동일한 코드는 아래와 같다.
```kotlin
class Membership(val member: Member) {
    init {
        requireNotNull(member.age)
        println(member.name)
    }
}
```

<br>

# 5. let

- `let` 함수는 확장 함수이기 때문에 context object를 receiver(**this**)로 전달한다. 그러나, 코드 블럭 내에서 this를 파라미터로 입력하기 때문에 `it`을 사용해 프로퍼티에 접근할 수 있다.
- `let` 함수는 반환 결과가 람다의 결과이다.
- `let` 함수는 **지정된 값이 `null`이 아닌 경우에 코드를 실행해야 하는 경우**, Nullable 객체를 다른 Nullable 객체로 변환하는 경우, 단일 지역 변수의 범위를 제한하는 경우에 유용하다.

```kotlin
getMember()?.let {
                    // null이 아닐때만 실행
    println(it)     // it: member
}

val length = str?.let {
    println("this str is not null")
    it.length
} ?: 0

// str이 "Wangi"일 경우 length = 5
// str이 null일 경우 length = 0
```

<br>

위의 스코프 함수들은 새로운 기술이 아니라, 코드들을 더욱 간결하고 가독성 좋게 하자는 의도로 사용한다.
그러므로 각 함수의 용도와 컨벤션을 잘 정하여 사용하는 것이 중요할 것 같다.

마지막으로, 코틀린 도큐먼트에서 제공하는 *Scope Function Selection Guide*를 보면서 이 포스팅을 마무리 하겠다.

![Kotlin Scope Function Guide](https://user-images.githubusercontent.com/43199318/102169642-f982ff00-3ed5-11eb-8b4b-c70c603e8979.png)

<br>


<br>
------------------------------------------------------------------------------
참고: 
- [https://kotlinlang.org/docs/reference/scope-functions.html](https://kotlinlang.org/docs/reference/scope-functions.html)
- [https://medium.com/@limgyumin/%EC%BD%94%ED%8B%80%EB%A6%B0-%EC%9D%98-apply-with-let-also-run-%EC%9D%80-%EC%96%B8%EC%A0%9C-%EC%82%AC%EC%9A%A9%ED%95%98%EB%8A%94%EA%B0%80-4a517292df29](https://medium.com/@limgyumin/%EC%BD%94%ED%8B%80%EB%A6%B0-%EC%9D%98-apply-with-let-also-run-%EC%9D%80-%EC%96%B8%EC%A0%9C-%EC%82%AC%EC%9A%A9%ED%95%98%EB%8A%94%EA%B0%80-4a517292df29)
- [https://blog.yena.io/studynote/2020/04/15/Kotlin-Scope-Functions.html](https://blog.yena.io/studynote/2020/04/15/Kotlin-Scope-Functions.html)
