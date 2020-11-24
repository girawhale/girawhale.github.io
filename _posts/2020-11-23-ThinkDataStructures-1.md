---
layout: post
title: (자바로 배우는 핵심 자료구조와 알고리즘) 1. 인터페이스
tags: [Book, Java, 자료구조, ThinkDataStructures, 자바로 배우는 핵심 자료구조와 알고리즘]
color: RGB(81,40,115)
---

# 1. 인터페이스

### 1.1 리스트가 두 종류인 이유?

자바는 List 인터페이스에 ArrayList와 LinkedList 구현체를 제공

상황에 따라 속도와 메모리 사용량에 차이가 존재한다   

#### ㅤ

### 1.2 자바 Interface

자바 interface란 **메서드의 집합**을 의미

interface를 구현하는 클래스는 이러한 메서드를 제공해야 한다   

#### ㅤ

예시)

``` java
// java.lang.Comparable
public interface Comparable<T>{
    public int compareTo(T o);
}
```

타입 파라미터인 T를 사용해 Comparable이라는 제네릭 타입을 정의

이 interface를 구현하기위해

- T 타입의 명시
- T 타입의 객체를 인자로 받아 int를 반환하는 compareTo() 메서드를 제공해야 함

#### ㅤ

```java
// java.lang.Integer
public final class Integer extends Number implements Comparable<Integer>{
    public int compareTo(Integer anotherInteger){
        int thisVal = this.value;
        int anotherVal = anotherInteger.value;
        return (thisVal < anotherVal ? -1 : (thisVal == anotherVal ? 0 : 1));
	}
}
```

이 클래스는 Number 클래스를 확장

Number 클래스의 메서드와 인스턴스 변수를 상속하고 Comparable<Integer> 인터페이스를 구현, 따라서 Integer 객체를 인자로 받아 int를 반환하는 compareTo 메소드 제공

클래스가 interface를 구현한다고 선언시 컴파일러는 interface가 정의한 모든 메서드를 제공하는지 확인

#### ㅤ

### 1.3 List interface

JCF(Java Collection Framework)는 List라는 interface를 정의하고 ArrayList와 LinkedList 구현 클래스 제공

interface에는 List가 된다는 의미가 무엇인지를 정의

이 interface를 구현하는 클래스는 `add`, `get`, `remove` 등의 메서드를 포함한 특정 메서드 집합을 제공해야 한다

List로 동작하는 메서드는 ArrayList와 LinkedList, 또는 List를 구현하는 어떤 객체와도 동작 가능

#### ㅤ

```java
public class ListClientExample {
	private List list;

	public ListClientExample() {
		list = new LinkedList();
	}

	public List getList() {
		return list;
	}

	public static void main(String[] args) {
		ListClientExample lce = new ListClientExample();
		List list = lce.getList();
		System.out.println(list);
	}
}
```

`ListClientExample`은 유용한 동작을 하지는 않는다

클래스는 새로운 `LinkedList`객체를 만들어 리스트를 초기화 -> `getList`메서드는 게터 메서드로 List 객체에 대한 참조를 반환

중요한 내용은 필요한 경우가 아니라면 LinkedList나 ArrayList같은 구현 클래스를 사용하지 않고 <u>List 인터페이스를 사용</u>한다는 점	

인스턴스 변수는 List 인터페이스로 선언하고 `getList` 메소드도 List형을 반환하지만 구체적인 클래스 언급❌

만약 ArrayList 클래스를 사용하고자 한다면 생성자만 바꾸면 된다!

#### ㅤ

이러한 스타일을 **인터페이스 기반 프로그래밍(interface-based programming)** 또는 **인터페이스 프로그래밍**이라 한다(자바의 interface가 아닌 일반적인 개념의 인터페이스)

라이브러리를 사용할 때 코드는 오직 List와 같은 인터페이스만 의존하고 ArrayList와 같은 클래스와 같은 특정 구현에 의존해서는 안된다

이러한 방식으로 구현하면 나중에 변경되어도 인터페이스를 사용하는 코드는 그대로 사용 가능

#### ㅤ

### 1.4 실습

[**Github 소스보기**](https://github.com/girawhale/BooksExample/tree/main/ThinkDataStructures/src/main/java/chapter1)

```java
// ListClientExampleTest.java
public class ListClientExampleTest {
    @Test
    public void testListClientExample() {
        ListClientExample lce = new ListClientExample();
        List list = lce.getList();
        assertThat(list, instanceOf(ArrayList.class));
    }
} // ERROR
```

만약 `ListClientExample` 클래스에서 LinkedList 클래스를 ArrayList 클래스로 변경한 뒤 실행한다면 **성공**

객체 생성부분만 변경하고 List가 있는 선언 부분을 변경하지 않았다

만약 선언부를 변경한다면? 과다지정(overspecified)!

이후 다시 인터페이스로 돌아갈 경우 더 많은 코드의 수정을 발생

#### ㅤ

**만약 ArrayList 객체를 List 인터페이스로 교체한다면?**

인터페이스 인스턴스가 생성되지 않는다!

#### ㅤ

#### ㅤ

#### ㅤ

---

### **REFERENCES**

>  자바로 배우는 핵심 자료구조와 알고리즘 (앨런 B. 다우니)
