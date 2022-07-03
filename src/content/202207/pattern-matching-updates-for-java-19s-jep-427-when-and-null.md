---
layout: post
title: "Pattern matching updates for Java 19’s JEP 427: when and null"
image: ../img/java_binary_code_gears_programming_coding_development_by_nevarpp_gettyimages-688718788_2400x1600-100795799-large.webp
author: [Sensibile]
date: 2022-07-02T22:00:00.000Z
draft: false
tags:
  - Java Magazine
  - Java
  - Java 19
  - JVM Internals
excerpt: switch에 대한 패턴 일치의 세번째 미리보기는 case 구체화 및 null일 경우의 적절한 처리를 해결합니다.
---

이 문서에서는 세번째 미리보기에서 `switch`에 대한 패턴 일치의 변경사항을 살펴봅니다: case 구체화와 null일 경우의 처리.
이 논의에서는 Java 18의 JEP 420인 두번째 미리보기에서 `switch`에 대한 패턴 일치화 작동하는 방식을 이미 알고 있다고 가정합니다.

Java 19를 대상으로 하는 세번째 미리보기의 변경사항은 [JEP 427](https://openjdk.org/jeps/427)에 설명되어 있습니다. 이들은 핵심적인 세부 사항이며,
향후 플랫폼에서 `switch`에 대한 패턴 일치가 완료되기 전에 변경될 수 있습니다.

큰 그림은 `switch`가 표현력을 얻으며, Java에서 보다 적절한 프로그래밍 구성이 되고 있다는 것입니다. 물론 `switch`는 패턴을 사용하는 중요한 장소이지만 유일한 장소는 아닙니다.

이는 패턴 일치 기능이 `switch`에 대한 보다 친숙한 이해와 이런 추가 기능이 작동하는 방식을 통합하는 것과 같은 더 큰 고려 사항을 처리하기 위해 진화하고 있음을 의미합니다. `switch`에 사용된 패턴의 의미 체계가 다른 곳에서 사용되는 의미 체계와 일치하는 것이 중요합니다.

## Case 구체화

전통적인 `switch` 구문은 변수를 특정 값과 비교하고 일치하는 분기를 선택합니다.

`switch`를 패턴과 사용하면 변수가 유형과 일치합니다. 각 유형은 기본적으로 해당 유형에 유효한 모든 값의 큰 가방으로 볼 수 있습니다. 예를 들어 `Integer` 유형에서는 약 -20억 ~ +20억 사이의 모든 정수가 포함됩니다. `String` 유형은 모든 문자열을 포함합니다.

그러나 항상 모든 유형의 인스턴스를 정확하게 동일하게 처리하고 싶지 않을 수 있습니다. 양의 정수와 음의 정수를 구별하거나 특정 하위 문자열을 포함하거나 포함하지 않는 문자열을 고려할 수 있습니다.

물론 이러한 조건은 `if`로 쉽게 표현할 수 있지만 `switch`의 맥락에서는 `if`는 `if` 앞에 `type` 패턴, `arrow(->)`, 마지막으로 실제로 관심있는 명령문을 필요로 합니다.

예를 들어, *condition, arrow, statements* 대신 다음과 같이 *condition-part-one, arrow, condition-part-two, statements*를 얻을 수 있습니다.

```java
switch (object) {
  case Integer i ->
    if (i >= 0)
      // positive integers
    else
      // negative integers
  case String s ->
    if (s.contains("foo"))
      // strings with "foo"
    else
      // strings without "foo"
  default -> // ...
}
```

이는 *보호된 패턴*가 들어오는 곳이거나, 들어왔던 곳입니다. JEP 427을 보면 `when` 절의 사용을 제안합니다. 둘 다 패턴에 Boolean 조건을 추가하여 왼쪽에 양의 정수와 같이 원하는 경우를 식별한 다음 `arrow` 뒤에 간단한 명령문을 추가할 수 있습니다. 예를 들면 다음과 같습니다.

```java
switch (object) {
  case Integer i ____ i >= 0 -> // positive integers
  case Integer i -> // negative integers
  default -> // ...
}
```

(____ 는 이 문서의 뒷부분에 나오는 항목이 위치합니다.)

JEP 427에서 제안한 `when`절은 다음 두 가지 측면에서 보호된 패턴과 다릅니다.

- 구체화를 소유하는 구성
- 구체화를 표현하는 방법

이름에서 알 수 있듯이 보호된 패턴은 패턴 구문의 일부였으며 매우 강력했습니다. 예를 들어, 중첩 패턴이 도입되면 끝 부분뿐만 아니라 큰 패턴 내부에 Boolean 조건을 추가할 수도 있습니다.

전반적으로 보호된 패턴에는 JDK 팀이 피하고 싶어하는 몇 가지 이상한 경우가 존재합니다. 그래서, 이제 더 이상 구체화를 소유한 패턴이 아닙니다. 이제 `case`는 아래와 같이 구체화를 소유합니다.

```java
switch (shape) {
  // now-obsolete guarded patterns with
  // record patterns from JEP 405
  case Point(int x && x > 0, int y) -> // use positive x, y
  default -> // ...
}

switch (shape) {
  // refinement owned by 'case' can't be "inside" the pattern
  case Point(int x, int y) ____ x > 0 -> // use positive x, y
  default -> // ...
}
```

다른 측면은 구체화를 표현하는 방법의 구문입니다. `&&`를 공평한 용어 사이에서 강력한 묶음 연산자로 보는 것에 익숙할 수 있습니다. 이 표기법은 보호된 패턴이 실제로 패턴의 일부였기 때문에 합리적으로 잘 작동했지만 `case`가 구체화를 소유한 경우에는 덜 작동합니다.

Brian Goetz는 Project Amber의 메일링 리스트에 "`&&`를 패턴의 일부가 아니라 `case`의 일부로 로 상상하는 것이 더 어렵습니다."라고 썼습니다.

따라서 현재 제안은 패턴과 구체화된 Boolean 조건 사이에 새로운 컨텍스트 별 키워드 `when`을 사용하는 것이고, 이것이 앞서 표시된 ____ 자리에 들어가는 것입니다.

```java
switch (object) {
  case Integer i when i >= 0 ->
    // positive integers
  case Integer i ->
    // negative integers
  default -> // ...
}
```

## Null 값

내가 가장 좋아하는 주제는 null입니다! 다시 한번, 그것은 어두운 존재로 아름다움을 더럽힙니다, 특히 패턴 `switch`에서 null을 처리해야 하기 때문입니다.

역사적으로 `switch`는 변수가 null일 때 단순며 `NullPointerException`을 던졌습니다. 그러나 복잡한 유형의 `switch`를 많이 사용할수록 `switch` 전 별도로 해당 상황을 처리하는 더 나은 방법을 찾아야할 필요성이 있습니다.

`switch`에 대한 패턴 일치의 첫번째 미리보기 버전(JEP 427에 의해 변경되지 않음) 이후로 이 특별한 상황에 대한 `case` null을 추가하고 이를 기본값과 결합하는 것이 가능했습니다. 그 `case`가 없으면 어떻게 될까요?

```java
String string = // ???
// JDK 18 and JEP 427
switch (string) {
  case null -> // ...
  case "foo" -> // ...
  case "bar" -> // ...
}
```

JDK 18의 두번째 미리보기에서는 조건이 없는 패턴, 즉 전환된 변수 유형의 가능한 모든 인스턴스와 일치하는 패턴의 존재 여부에 따라 다른 대답입니다.

마지막 `case`가 `case Shape s`인 `Shape` 유형의 변수에 대한 `switch`를 생각해 보십시오. 그 코드는 항상 일치합니다: 이는 `Shape` 유형에 대해 무조건적입니다.

무조건 패턴은 null과도 일치합니다. 따라서 JDK 18에서 변수 `s`는 null일 수 있습니다. 그러나 이는 여러 `NullPointerException` 상황으로 이어질 수 있으며 저는 아래에 표현된 것처럼 다른 모양으로 조용히 null을 처리하는 것을 좋아하지 않았습니다.

```java
Shape shape = // ...
// as previewed in JDK 18
switch (shape) {
  case Point p -> ...
  // unconditional pattern
  //  ~> matches 'null'
  //  ~> 's' can be 'null'
  case Shape s -> ...
}
```

다행히 JEP 427은 이러한 불쾌한 상황을 바꿀 것을 제안합니다. 어떻게? 무조건 패턴은 여전히 null과 일치하지만 `switch`는 그렇게까지 도달하도록 허용하지 않습니다. `case` null이 없으면 `switch`는 패턴을 보지 않고 `NullPointerException`을 던집니다.

```java
Shape shape = // ...
// as proposed by JEP 427:
// no 'case null' ~> NPE
switch (shape) {
  case Point p -> ...
  // unconditional pattern
  //  (still matches 'null')
  case Shape s -> ...
}
```

흥미롭게도 이 최상위 동작은 중첩 패턴으로 확장되지 않습니다.

무조건 내포된 패턴은 여전히 null과 일치하므로 리팩토링 중에 sharp edge가 발생합니다. 이것은 일관성 없지만, 일관되게 null과 일치 하지 않으면 아래와 같이 모든 인스턴스와 일치하는 단일 패턴을 작성할 수 없는 것과 같은 이상한 효과가 있습니다. 

```java
interface Shape { }
record Circle(Point center)
  implements Shape { }

// JEP 427 + JEP 405
Shape shape = // ...
switch (shape) {
  // 'Point center' is unconditional
  // the circle's center 'Point'
  case Circle(Point center) ->
    // 'center' may be 'null'
  case Shape s ->
    // 's' won’t be 'null'
}
```

내가 개인적으로 추구하는 이 소동에 대한 해결책은 null을 완전히 피하는 것입니다. (사실 이미 그렇게 하고 있지만 그건 요점이 아닙니다.)

어쨋든 null이 적법하지 않을 때 `switch`는 그것을 언급할 필요가 없으며, null을 만났을 때 `switch`가 예외를 던졌다면 좋았겠지만 실제로 그렇게 하는 것은 `switch`의 일이 아닙니다.

## 출처

[Oracle Java Magazine - Pattern matching updates for Java 19’s JEP 427: when and null](https://blogs.oracle.com/javamagazine/post/java-lists-view-unmodifiable-immutable)
