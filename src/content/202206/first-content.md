---
layout: post
title: 테스트
image: ../img/callum-shaw-555357-unsplash.jpg
author: [Sensibile]
date: 2022-06-25T13:42:00.000Z
draft: false
tags:
  - Algorithm
  - Java
excerpt: 카드 놀이를 할 때 손에 쥔 카드를 정렬하는 것과 같은 방법
---

## 테스트

입력: n개 수들의 수열 <$a_1, a_2, ..., a_n$>

출력: $a_1' \leq a_2' \leq ... \leq a_n'$ 을 만족하는 입력 수열의 순열

j 번째 수를 정렬된 배열 nums[1 .. j-1]에 삽입 

```java
public static void insertionSort(int[] nums) {
    for (int j = 1; j < nums.length; ++j) {
        int key = nums[j];
        int i = j -1;
        while (i >= 0 && nums[i] > key) {
            nums[i + 1] = nums[i];
            --i;
        }
        nums[i + 1] = key;
    }
}
```
