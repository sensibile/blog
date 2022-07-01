---
layout: post
title: "Quiz yourself: Manipulating Java lists—and views of lists"
image: ../img/java_binary_code_gears_programming_coding_development_by_nevarpp_gettyimages-688718788_2400x1600-100795799-large.webp
author: [Sensibile]
date: 2022-07-01T18:00:00.000Z
draft: false
tags:
  - Java Magazine
  - Java
excerpt: List가 수정 불가능할까요? 가능할까요? 리스트의 출력은 무엇일까요?
---

다음의 코드 조각이 주어졌습니다.

```java
String[] arr = new String[] {"A1", "A2"};

List<String> ls = new ArrayList<>();
ls.add("L1");ls.add("L2");

List<String> la = Arrays.asList(arr);
List<String> lf = List.of(arr);
List<String> lc = List.copyOf(ls);
List<String> lu = Collections.unmodifiableList(ls);

arr[1]="A3";
ls.set(1, "L3");

// line n1

System.out.println("la=" + la);
System.out.println("lf=" + lf);
System.out.println("lc=" + lc);
System.out.println("lu=" + lu);
```

다음 중 옳은 것은? 3개를 골라보세요.

A. 출력에 `la=[A1, A2]` 가 포함되어 있다.

B. 출력에 `lf=[A1, A2]` 가 포함되어 있다.

C. 출력에 `lc=[L1, L3]` 가 포함되어 있다.

D. 출력에 `lu=[L1, L3]` 가 포함되어 있다.

E. line n1에 `lf.set(1, "A3")`가 들어가면, 출력에 `lf=[A1, A3]` 가 포함되어 있다.

F. line n1에 `la.set(1, "A3")`가 들어가면, 출력에 `la=[A1, A3]` 가 포함되어 있다.

### 풀이

이 퀴즈 질문은 배열에서 제공된 데이터를 반영하는 List(bridging array 및 list라고 알려진)를 만들고, List의 분리된 복사본과 List의 View들을 만드는 다양한 접근 방식을 탐구합니다.

질문의 코드는 값 A1 및 A2를 포함하는 배열을 생성한 다음 값 L1 및 L2를 포함하는 ArrayList를 생성합니다. 그런 다음 4개의 추가 List를 초기화하는 데 사용됩니다.

다음으로 코드는 정적 팩토리 메서드인 Arrays.asList(array)를 사용하여 배열에서 초기화된 List를 만듭니다. 이 메서드는 배열에 대한 View 역할을 하는 List를 만듭니다. View가 된다는 것은 List의 메소드가 배열에 저장된 데이터와 직접 상호 작용한다는 것을 의미합니다. get 작업은 배열에서 값을 반환하고 set 작업은 배열의 값을 변경합니다. 또한 배열 값이 배열에서 직접 변경되면 뷰에서 후속 가져오기 작업을 통해 표시되는 데이터에 이러한 변경 사항이 반영됩니다.

물론 Java 배열의 길이는 생성 후에 변경할 수 없으며 결과적으로 Arrays.asList에 의해 생성된 List의 길이도 변경할 수 없습니다. 새 요소를 추가할 수 없으며 요소를 제거할 수 없습니다.

List la가 배열 arr에 대한 View인 경우 할당 arr[1]="A3"로 List la의 내용도 변경됩니다. 따라서 la를 인쇄하는 행의 결과는 la=[A1, A3]이 됩니다. 그러므로 옵션 A는 올바르지 않습니다.

배열의 View인 List의 길이를 변경하지 않으려면 List.set 메서드를 사용하여 요소를 변경할 수 있습니다. 이러한 변경 사항은 View와 기본 배열 모두에서 볼 수 있습니다. 옵션 F는 line n1에서 la.set(1, "A3")를 실행하도록 코드 추가를 제안합니다. 이 코드는 값을 변경하진 않지만, 문제 없이 컴파일 및 실행됩니다. 출력에 이미 텍스트 la=[A1, A3]이 포함되어 있으므로 F는 정답입니다.

두 번째 List는 List.of(array) 팩토리 메서드를 사용하여 배열에서 생성됩니다. 12개의 오버로드된 List.of 팩토리 각각은 수정할 수 없는 List를 생성합니다. 수정 불가능이라는 용어는 List 자체의 구조를 변경할 수 없음을 의미합니다. 이것은 List의 요소가 변경 가능한 경우(Java에서 매우 일반적임) 요소 자체의 내용을 변경할 수 있다는 점에서 진정한 불변의 List와 다릅니다. 그러나 다른 요소를 참조하도록 목록을 변경할 수는 없습니다. 또한 요소를 추가하거나 제거할 수 없습니다.

결과적으로 List의 요소 자체가 변경할 수 없는 경우(이 경우 문자열도 마찬가지임) 완전히 변경할 수 없는 List가 됩니다.

List.of() 팩토리는 배열 요소를 생성하는 List의 새 구조로 복사합니다. 이렇게 하면 새 List가 초기화된 배열과 독립적이 됩니다. 이를 바탕으로 소스 배열의 수정 arr[1]="A3"는 List 상태에 영향을 주지 않습니다.

List lf의 내용을 인쇄하는 행은 lf=[A1, A2]를 출력합니다. 따라서 B는 정답입니다.

위에서 언급했듯이 List.of 팩토리 메소드의 제품은 수정할 수 없습니다. 이러한 목록은 단순히 변경하려는 시도를 무시하는 것이 아니라 수정하려는 시도가 있으면 예외가 발생합니다. 옵션 E는 목록에 set 메소드 호출을 추가하는 것을 제안합니다. 이러한 작업으로 인해 코드에서 예외가 발생합니다. 때문에 옵션 E는 올바르지 않습니다.

세 번째 목록인 lc는 List.copyOf(collection) 팩토리 메소드를 사용하여 생성됩니다. 이 메서드는 또한 팩토리가 호출될 때 초기화 컬렉션의 상태를 반영하는 수정 불가능한 목록을 반환합니다. 이것은 lc 초기화 후에 실행되는 코드 ls.set(1, "L3")이 lc List에 영향을 미치지 않을 것임을 알려줍니다. 따라서 옵션 C는 올바르지 않습니다.

List.copyOf 메소드는 Java 10에 추가되었으며 그 동작은 인수가 이미 수정할 수 없는 List인지 여부에 따라 다릅니다. 만약 수정할 수 없는 List라면 즉시 반환됩니다. 그러나 List가 변경 가능한 경우 수정할 수 없는 복사본이 생성됩니다. 이렇게 하면 원본이 수정 가능한 경우 불필요하게 중복을 만들지 않고 필요할 때 코드에서 수정할 수 없는 복사본을 만들 수 있습니다.

네 번째 목록은 Collections.unmodifiableList(list) 팩토리를 사용하여 생성됩니다. 이 메서드는 수정할 수 없는 View를 만듭니다. 수정할 수 없는 View는 View 유형이므로 이에 대한 get 작업은 기본 List(즉, unmodifiableList 팩토리 메서드에 대한 인수인 List)의 데이터를 반환합니다. 기본 목록이 변경되면 뷰에서 get을 호출하여 보고된 결과가 변경됩니다.

그러나 수정할 수 없는 View 자체에서 메서드를 호출하여 데이터를 변경하려는 모든 시도는 실패합니다. 특히, 수정할 수 없는 View에 대한 이러한 모든 작업(예: 설정, 추가, 제거 또는 지우기)은 UnsupportedOperationException을 발생시킵니다. 이 질문에서 기본 목록은 [L1, L3]을 포함하도록 변경되었으므로 출력에는 lu=[L1, L3]이 포함됩니다. 이것은 옵션 D가 정답임을 의미합니다.

### 결론
B, D, F가 정답

### 출처

[Oracle Java Magazine - Quiz yourself: Manipulating Java lists—and views of lists](https://blogs.oracle.com/javamagazine/post/java-lists-view-unmodifiable-immutable)
