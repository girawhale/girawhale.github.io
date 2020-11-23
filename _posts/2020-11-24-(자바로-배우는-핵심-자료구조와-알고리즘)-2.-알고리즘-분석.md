---
layout: post
title: (자바로 배우는 핵심 자료구조와 알고리즘) 2. 알고리즘 분석
tags: [Book, Java, 자료구조, ThinkDataStructures, 자바로 배우는 핵심 자료구조와 알고리즘]
color: RGB(81,40,115)
---

# 2. 알고리즘 분석

자바는 List 인터페이스의 구현체로 ArrayList와 LinkedList 클래스를 제공하지만 응용 프로그램마다 빠른 클래스가 다르다

어떤 응용 프로그램에 어느 클래스가 더 좋을지 결정하는 방법 중 하나는 둘 다 시도해 보는 것!

이것을 <u>프로파일링</u>이라 한다. 하지만 문제점이 존재

1. 알고리즘을 비교하려면 사전에 모두 구현해야 한다
2. 결과는 사용하는 컴퓨터의 성능에 의존. 한 알고리즘이 어떤 컴퓨터에서 더 좋을 수 있지만, 다른 알고리즘은 다른 컴퓨터에서 더 좋을 수 있다
3. 결과는 문제 크기나 입력으로 사용하는 데이터에 의존하기도 한다

⇒ **알고리즘 분석**을 사용해 이를 해결할 수 있다.

#### ㅤ

알고리즘 분석은 구현하지 않고 알고리즘을 비교할 수 있게하지만 몇가지 가정이 필요하다

1.  컴퓨터 하드웨어의 세부사항을 다루지 않기 위해 보통 알고리즘을 이루는 더하기, 곱하기, 숫자 비교 등의 기본연산을 식별. <u>각 알고리즘에 필요한 연산 수를 센다</u>
2. 입력 데이터의 세부사항을 다루지 않기 위해 기대하는 입력 데이터에 대한 <u>평균 성능을 비교</u>
   - 가능하지 않을 때는 일반적인 대안으로 최악의 시나리오를 분석
3. 한 알고리즘이 작은 문제에서는 최상의 성능을 보이지만 큰 문제에서는 다른 알고리즘이 더 좋을 수 있다는 가능성 배제❌
   - 보통 <u>큰 문제에 초점</u>을 맞춰야 한다
   - 작은 문제에서는 알고리즘의 차이가 크지 않지만 큰 문제에서 차이가 거대해질 가능성이 존재

이러한 종류의 분석은 간단할 알고리즘 분류에 적합

#### ㅤ

대부분의 간단한 알고리즘은 몇 가지 범주로 분류

#### 상수 시간

실행 시간이 입력 크기에 의존하지 않으면 알고리즘은 <u>상수시간을 따른다</u>고 한다

예시) n개의 배열에서 브래킷 연산([])을 사용하여 요소 중 하나에 접근할 때 이 연산은 배열의 크기와 관계없이 같은 수의 동작을 반복

#### 선형

실행 시간이 입력 크기에 비례하는 알고리즘을 <u>선형</u>이라고 한다

예시) 배열에 있는 요소를 더한다면 n개 요소에 접근하여 n-1번 더하기 연한을 해야함. 연산(접근 + 더하기)의 총 횟수는 2n-1이고, n에 비례

#### 이차

실행 시간이 $$n^2$$에 비례하면 알고리즘은 <u>이차</u>라고한다

예시) 리스트에 있는 어떤 요소가 두번이상 나타나는지 알고 싶다고 가정. 각 요소를 다른 모든 요소와 비교한다

n개의 요소가 있고 각각 n-1개의 다른 요소와 비교하면 총 비교 횟수는 $$n^2-n$$이 되어 n이 커지면서 $$n^2$$에 비례

#### ㅤ
#### ㅤ

### 2.1 선택 정렬

```java
public class SelectionSort {
    // i와 j의 위치에 있는 값을 바꿈
    public static void swapElements(int[] array, int i, int j) {
        int temp = array[i];
        array[i] = array[j];
        array[j] = temp;
    }

    // start로부터 시작하는 최솟값의 위치를 찾아 배열의 마지막 위치까지 간다
    public static int indexLowest(int[] array, int start) {
        int lowIndex = start;
        for (int i = start; i < array.length; i++) {
            if (array[i] < array[lowIndex])
                lowIndex = i;
        }
        return lowIndex;
    }

    // 선택정렬을 사용해 요소 정렬
    public static void selectionSort(int[] array) {
        for (int i = 0; i < array.length; i++) {
            int j = indexLowest(array, i);
            swapElements(array, i, j);
        }
    }
}

```

`swapElements`는 배열에 있는 두 요소의 값을 바꿈

요소를 읽고 쓰는데는 상수 시간이 소요. `swapElements` 메서드에 있는 모든 연산이 상수 시간이므로 전체 메서드는 상수 시간

#### ㅤ

`indexLowest`는 주어진 start에서 시작하여 배열에 있는 가장 작은 요소의 인덱스를 찾는다

반복문을 돌 때마다 배열의 두 요소에 접근하고 한번의 비교연산. 이들은 모두 상수 연산이기 때문에 어떤 것을 세든 중요하지 않다

1. start 인자가 0이면 indexLowest 메서드는 전체 배열을 검색하고 전체 비교 횟수는 배열의 길이인 n
2. start 인자가 1이면 비교 횟수는 n - 1
3. 일반적으로 비교 횟수는 n - start 가 되어 `indexLowest` 메서드는 선형

#### ㅤ

`selectionSort`는 배열을 정렬

0에서 n - 1 까지 반복하므로 n번 반복 실행. 매번 `indexLowest`메서드 호출 뒤 상수 시간 연산인 `swapElements`메서드를 실행

`indexLowest`메서드가 처음 호출 시 n번 비교 연산 수행. 두번 째는 n-1번 수행

총 비교 횟수는 n + n-1 + n-2 + ... + 1 + 0 = $$n(n+1)/2$$이고 $$n^2$$에 비례 

⇒ `selectionSort`는 **이차**라는 것을 의미한다

#### ㅤ

다른 방법으로 같은 결과를 얻기 위해 `indexLowest` 메서드를 중첩 반복문으로 생각 가능

`indexLowest` 메서드를 호출할 때마다 연산 수는 n에 비례, 이를 n번 호추랗면 결과적으로 총 연산 수는 $$n^2$$d에 비례

#### ㅤ
#### ㅤ


### 2.2 빅오 표기법

모든 상수시간 알고리즘은 O(1)이라는 집합에 속한다

따라서 어떤 알고리즘이 **상수 시간**임을 표현하고 싶다면 O(1)에 있다고 말하면 된다

마찬가지로 모든 **선형 알고리즘**은 O(n)에 속하며 **이차 알고리즘**은 O($$n^2$$)에 속한다

이렇게 알고리즘을 분류하는 방식을 **<u>빅오 표기법</u>** 이라 한다

알고리즘을 작성할 때 알고리즘이 어떻게 동작하는지에 관한 일반적인 법칙을 표현하는 간단한 방법 제공

#### ㅤ

예시) 상수 시간 알고리즘에 이어 선형 시간 알고리즘을 수행하면 총 실행 시간은 선형이 된다

`f ∈ O(n) 이고, g ∈ O(1) 이면 f + g ∈ O(n)` 이 된다

#### ㅤ

두개의 선형 연산을 수행하면 합은 여전히 선형이다. `f ∈ O(n) , g ∈ O(n) 이면 f + g ∈ O(n)` 

k가 n에 의존하지 않는 상수인 한 선형 연산을 k번 수행하면 합은 선형이다. `f ∈ O(n)이면 kf ∈ O(n)` 

하지만, 선형 연산을 n번 반복하면 이차가 된다. `f ∈ O(n)이면 nf ∈ O(n^2)` 

일반적으로 n의 가장 큰 지수만 신경 쓰기 때문에 총 연산 횟수가 2n+1이라면 실행시간은 O(n)

#### ㅤ

**증가 차수**는 같은 개념의 다른 이름

증가 차수는 실행시간이 같은 빅오 범주에 해당하는 알고리즘 집합

예) 모든 선형 알고리즘은 실행시간이 O(n)에 있으므로 같은 증가 차수에 속한다

차수는 <u>집단</u>의 의미, 집단을 가리키는 것이지 순서를 말하는 것이 아니다

따라서, <u>선형 알고리즘 차수는 효율적인 알고리즘 집합</u>으로 생각하면 된다

#### ㅤ
#### ㅤ
#### ㅤ

### 2.3 실습

[**Github 소스보기**](https://github.com/girawhale/BooksExample/tree/main/ThinkDataStructures/src/main/java/chapter2)

자바 배열을 사용해 요소를 저장하는 List 인터페이스를 구현

```java
public class MyArrayList<T> implements List<T> {
    int size;                    // 요소의 개수를 추척
    private T[] array;           // 요소를 저장

    public MyArrayList() {
        // 생성자는 초기값이 null이고 10개의 요소를 갖는 배열을 생성, size = 0으로 설정
        array = (T[]) new Object[10]; // 대부분의 시간동안 배열의 크기는 size변수보다 크기때문에 사용하지 않는 슬롯 존재
        size = 0;
    }
}
```

#### ㅤ

`add`

```java
@Override
public boolean add(T element) {
    if (size >= array.length) {
        T[] bigger = (T[]) new Object[array.length * 2];
        System.arraycopy(array, 0, bigger, 0, array.length);
        array = bigger;
    }

    array[size] = element;
    size++;

    return true;
}
```

배열에 여유공간이 없다면 더 큰 배열을 만들어 요소 위에 복사하고 배열의 요소를 저장한 뒤 size를 늘린다

#### ㅤ

`get`

```java
@Override
public T get(int index) {
    if (index < 0 || index >= size) {
        throw new IndexOutOfBoundsException();
    }
    return array[index];
}
```

get 메서드는 인덱스가 범위를 벗어나면 예외를 던지고 아니면 요소를 읽고 반환

#### ㅤ

`set`

```java
@Override
public T set(int index, T element) {
    T old = get(index);
    array[index] = element;

    return old;
}
```

set의 return 값은 이전에 지니고 있던 요소의 값을 반환해야 하므로 get 메서드를 활용해 이전 값을 얻어온 뒤 반환

#### ㅤ

`indexOf`

```java
@Override
public int indexOf(Object target) {
    for (int i = 0; i < size; i++)
        if (equals(target, array[i]))
            return i;

    return -1;
}
```

만약 배열 내에 원하는 값이 존재한다면 해당 값의 인덱스를 반환하고 아니라면 -1을 반환

#### ㅤ

`remove`

```java
@Override
public T remove(int index) {
    T elem = get(index);
    for (int i = index; i < size - 1; i++) {
        array[i] = array[i + 1];
    }
    size--;

    return elem;
}
```

기존 인덱스에 있던 값을 받아온 뒤, 한 칸씩 당기며 빈칸을 요소들을 왼쪽으로 이동하고 삭제된 요소를 반환

#### ㅤ

#### ㅤ

ㅤ

---

### **REFERENCES**

>  자바로 배우는 핵심 자료구조와 알고리즘 (앨런 B. 다우니)

