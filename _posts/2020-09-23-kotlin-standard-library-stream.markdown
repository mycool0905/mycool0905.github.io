---
layout: post
title:  "[Kotlin] 코틀린 표준 스트림 함수 정리"
date:  2020-09-23 23:59:59 +0900
categories: Kotlin
tags: [Kotlin]
---

# 1. 변환

- map()
    + 컬렉션 내 인자를 다른 값 혹은 타입으로 변환할 때 사용한다.
    
```kotlin
fun main(args: Array<String>) {
    val cities = listOf("Seoul", "Tokyo", "London")

    cities.map { city -> city.toUpperCase() }
            .forEach { println(it) }
    // SEOUL TOKYO LONDON

    cities.map { city -> city.length }
            .forEach { println(it) }
    // 5 5 6
}
```

<br>

- mapIndexed()
    + 컬렉션 내 포함된 인자의 인덱스 값을 변환 함수 내에서 사용할 때 사용한다.
    
```kotlin
fun main(args: Array<String>) {
    val numbers = 0..5
    numbers.mapIndexed { idx, number -> idx * number }
            .forEach { println(it) }
    // 0 1 4 9 16 25
}
```

<br>

- mapNotNull()
    + 컬렉션 내 인자를 변환함과 동시에, 변환한 결과가 null 값인 경우 이를 무시한다.
    
```kotlin
fun main(args: Array<String>) {
    val cities = listOf("Seoul", "Tokyo", "London")

    cities.mapNotNull { city -> if (city.length <= 5) city else null }
            .forEach{ println(it) }
    // Seoul Tokyo
}
```

<br>

- flatMap()
    + flatMap()은 map() 함수와 비슷하지만 하나의 인자에서 여러 개의 인자로 매핑이 필요할 때 사용한다.

```kotlin
fun main(args: Array<String>){
    val numbers = 1..5

    numbers.flatMap{ number -> number..number+1}
            .forEach { println(it) }
    // 1 2 2 3 3 4 4 5 5 6
}
```

<br>

- groupBy()
    + 컬렉션 내 인자들을 지정한 기준에 따라 분류하여 맵 형태로 결과를 반환한다.
    
```kotlin
fun main(args: Array<String>) {
    val cities = listOf("Seoul", "Tokyo", "London")

    cities.groupBy { city -> if (city.length <= 5) "A" else "B" }
            .forEach { (key, cities) -> println("$key $cities") }
    // A [Seoul Tokyo] B [London]
}
```

<br>

# 2. 필터

- filter()
    + 컬렉션 내 인자들 중 주어진 조건과 일치하는 인자만 걸러주는 역할
    
```kotlin
fun main(args: Array<String>) {
    val cities = listOf("Seoul", "Tokyo", "London")

    cities.filter { city -> city.length <= 5 }
            .forEach { println(it) }
    // Seoul Tokyo
}
```

<br>

- take()
    + take(): 컬렉션 내 인자들 중 앞에서 take 함수의 인자로 받은 개수만큼만을 인자로 갖는 리스트를 반환한다.
    + takeLast(): take() 함수와 반대로 뒤에서부터 적용해 반환한다.
    + takeWhile(): 첫 번째 인자부터 시작하여 주어진 조건을 만족하는 인자까지를 포함하는 리스트를 반환한다.
    + takeLastWhile(): takeWhile() 함수와 반대로 뒤에서부터 적용해 반환한다.
    
```kotlin
fun main(args: Array<String>) {
    val cities = listOf("Seoul", "Tokyo", "Beijing", "NYC", "London", "Singapore")

    cities.take(2)
            .forEach { println(it) }
    // Seoul Tokyo

    cities.takeLast(3)
            .forEach { println(it) }
    // NYC London Singapore

    cities.takeWhile { city -> city.length <= 5 }
            .forEach { println(it) }
    // Seoul Tokyo

    cities.takeLastWhile { city -> city.length > 5 }
            .forEach { println(it) }
    // London Singapore
}
```

<br>

- drop()
    + take() 함수의 반대 역할을 하며, 조건을 만족하는 항목을 컬렉션에서 제외한 결과를 반환한다.
    
```kotlin
fun main(args: Array<String>) {
    val cities = listOf("Seoul", "Tokyo", "Beijing", "NYC", "London", "Singapore")

    cities.drop(2)
            .forEach { println(it) }
    // Beijing NYC London Singapore

    cities.dropLast(3)
            .forEach { println(it) }
    // Seoul Tokyo Beijing

    cities.dropWhile { city -> city.length <= 5 }
            .forEach { println(it) }
    // Beijing NYC London Singapore

    cities.dropLastWhile { city -> city.length > 5 }
            .forEach { println(it) }
    // Seoul Tokyo Beijing NYC
}
```

<br>

- first(), last()
    + 컬렉션 내 첫 번째 인자를 반환한다. 단순히 리스트 내에서 첫 번째에 위치하는 인자를 반환하는 것뿐 아니라, 특정 조건을 만족하는 첫 번째 인자를 반환하도록 구성하는 것도 가능하다.
    + 조건을 만족하는 인자가 없는 경우엔 NoSuchElementException 예외를 발생시키며, firstOrNull() 함수를 사용하면 null 값을 반환하도록 할 수 있다.
    
```kotlin
fun main(args: Array<String>) {
    val cities = listOf("Seoul", "Tokyo", "Beijing", "NYC", "London", "Singapore")

    println(cities.first())
    // Seoul

    println(cities.last())
    // Singapore

    println(cities.first { it.length > 5 })
    // Beijing

    println(cities.firstOrNull { it.length > 10 })
    // null

}
```

<br>

- distinct()
    + 컬렉션 내에 포함된 항목 중 중복된 항목을 걸러낸 결과를 반환한다. 항목의 중복 여부는 equals()로 판단하며, distinctBy() 함수를 사용하면 비교에 사용할 키 값을 직접 설정할 수 있다.
    
```kotlin
fun main(args: Array<String>) {
    val cities = listOf("Seoul", "Tokyo", "London", "Seoul", "Tokyo")

    cities.distinct()
            .forEach { println(it) }
    // Seoul, Tokyo, London

    cities.distinctBy { it.length }
            .forEach { println(it) }
    // Seoul, London
}
```

<br>

# 3. 조합 및 합계

- zip()
    + 컬렉션 내의 자료들을 조합하여 새로운 자료를 만들 때 사용한다. 두 컬렉션 간 자료의 개수가 달라도 사용할 수 있으며, 이 경우에 반환되는 수는 컬렉션의 수 중 더 적은 쪽을 따라간다.
    + 기본 값으로는 조합된 결과를 Pair로 만들며, 원하는 경우 조합 규칙을 직접 정의할 수 있다.
    
```kotlin
fun main(args: Array<String>) {
    val cityCodes = listOf("SEO", "TOK", "MTV", "NYC")
    val cityNames = listOf("Seoul", "Tokyo", "Mountain View")

    // Pair 형태로 출력
    cityCodes.zip(cityNames)
            .forEach { println(it) }
    // (SEO, Seoul) (TOK, Tokyo) (MTV, Mountain View)

    // 사용자가 직접 정의
    cityCodes.zip(cityNames) { code, name -> "<${code} ${name}>" }
            .forEach { println(it) }
    // <SEO, Seoul> <TOK, Tokyo> <MTV, Mountain View>
}
```

<br>

- joinToString()
    + 컬렉션 내 자료를 문자열 형태로 변환함과 동시에, 이를 조합하여 하나의 문자열로 생성한다.
    + 몇 가지 인자를 함께 전달하면 자신이 원하는 형태로 출력 문자열을 구성하는 것도 가능하다.
    
```kotlin
fun main(args: Array<String>){
    val cities = listOf("Seoul", "Tokyo", "London", "NYC", "Singapore")

    println(cities.joinToString())
    //Seoul, Tokyo, London, NYC, Singapore

    println(cities.joinToString(separator = " & "))
    // Seoul & Tokyo & London & NYC & Singapore
}
```

<br>

- count()
    + 컬렉션 내 포함된 자료의 개수를 반환하며, 별도의 조건식을 추가하면 해당 조건을 만족하는 자료의 개수를 반환할 수 있다.
    
```kotlin
fun main(args: Array<String>) {
    val cities = listOf("Seoul", "Tokyo", "Beijing", "NYC", "London", "Singapore")

    println(cities.count())
    // 6

    println(cities.count { it.length <= 5 })
    // 3
}
```

<br>

- reduce()
    + 컬렉션 내 자료들을 모두 합쳐 하나의 값으로 만들어주는 역할을 한다.
    
```kotlin
fun main(args: Array<String>) {
    val numbers = 1..5
    val strings = listOf("a", "b", "c")

    println(numbers.reduce { acc, s -> acc + s })
    // 15

    println(strings.reduce { acc, str -> acc + str })
    // abc
}
```

<br>

- fold()
    + reduce() 함수와 거의 동일한 역할을 하지만, 초깃값을 지정할 수 있다.
    
```kotlin
fun main(args: Array<String>) {
    val numbers = 1..5
    val strings = listOf("a", "b", "c")

    println(numbers.fold(20) { acc, s -> acc + s })
    // 35

    println(strings.fold("this is ") { acc, str -> acc + str })
    // this is abc
}
```

<br>

# 4. 기타

- any()
    + 컬렉션 내 단 하나의 자료라도 존재하면 true를, 그렇지 않으면 false를 반환한다.
    + 인자로 조건식을 전달할 경우, 해당 조건식을 만족하는 자료의 유무 여부를 반환한다.
    
```kotlin
fun main(args: Array<String>) {
    val cities = listOf("Seoul", "Tokyo", "Beijing", "NYC", "London", "Singapore")

    println(cities.any())
    // true

    println(cities.any { it.length >= 5 })
    // true
}
```

<br>

- none()
    + any() 함수와 반대 작업을 수행하며, 컬렉션이 비어있는지 여부를 반환한다.
    + any() 함수와 마찬가지로, 조건식을 전달하여 해당 조건식에 만족하는 자료의 유무를 판단할 수 있다.
    
```kotlin
fun main(args: Array<String>) {
    val cities = listOf("Seoul", "Tokyo", "Beijing", "NYC", "London", "Singapore")

    println(cities.none())
    // false

    println(cities.none { it.length >= 10 })
    // true
}
```

<br>


<br>
------------------------------------------------------------------------------
참고: [https://leveloper.tistory.com/134?category=761696](https://leveloper.tistory.com/134?category=761696)