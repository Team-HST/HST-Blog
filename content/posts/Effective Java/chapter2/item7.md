---
title: "아이템7 - 다 쓴 객체 참조를 해제해라."
date: "2020-04-17 7"
template: "post"
draft: false
category: "Effective Java"
tags:
  - "Java"
  - "effectiveJava"
description: "2장. 객체 생성과 파괴 - 다 쓴 객체 참조를 해제해라."
---

### 아이템 7 - 다 쓴 객체 참조를 해제하라.

보통은 C, C++처럼 메모리를 직접 관리해야 하는언어와 달리 자바는 가비지 컬렉터를 갖춘 언어로 프로그래머들은 다 쓴 객체들에 대하여 생각하지 않고 메모리 관리에 신경 쓰지 않는다. 절대로 그렇게 되면 안된다.

```java
public class Stack {
    private Object[] elements;
    private int size = 0;
    private static final int DEFAULT_INITIAL_CAPACITY = 16;

    public Stack() {
        elements = new Object[DEFAULT_INITIAL_CAPACITY];
    }

    public void push(Object e) {
        ensureCapacity();
        elements[size++] = e;
    }

    public Object pop() {
        if (size == 0)
            throw new EmptyStackException();
        return elements[--size];
    }

    private void ensureCapacity() {
        if (elements.length == size)
            elements = Arrays.copyOf(elements, 2 * size + 1);
    }
}
```

위 예제를 보면 특별한 문제는 없어 보인다. 별의별 테스트를 수행해도 이상없이 통과할 것이다. 하지만 위에 예제에는 숨은 문제점이 있다. 그것은 바로 **메모리 누수**로 이 스택을 사용하는 프로그램을 오래 실행하다 보면 점차 가비지 컬렉션 활동과 메모리 사용량이 늘어나 결국 성능이 저하될 것이다. 심각한 경우 OutOfMemory를 발생하여 프로그램이 예기치 않게 종료 될 수도 있다.

그렇다면 위 예제에서 메모리 누수는 어디서 일어날까? 문제는 스택이 커졌다가 줄어들었을 때 스택에서 꺼내진 객체들을 가비지 컬렉터가 회수하지 않는다. 프로그램에서 더 이상 사용하지 않더라도 말이다.

결국, 이 스택이 그 객체들의 다 쓴 참조를 여전히 가지고 있고 여기서 다 쓴 참조란 문자 그대로 앞으로 다시 쓰지 않을 참조를 뜻한다. 예제에서 elements 배열의 **활성 영역**밖의 참조들이 모두 여기에 해당된다. 활성 영역이란 인덱스가 size보다 작은 원소들로 구성된다. 따라서 **인덱스가 size보다 큰 참조들을은 다 쓴 참조가 되는 것이다.**

가비지 컬렉션 언어에서는 메모리 누수를 찾기가 아주 까다롭다. 객체 참조 하나를 살려두면 가비지 컬렉터는 그 객체뿐 아니라 그 객체가 참조하는 모든 객체를 회수해가지 못한다. 따라서 단 몇 개의 객체가 매우 많은 객체를 회수되지 못하게 할수 있고 잠재적으로 성능에 악영향을 줄 수 있다. 

그렇다면 위 예제에서 어떻게 해야 메모리 누수가 일어나지 않을까? 방법은 해당 참조를 다 썼을 때 null 처리하면 된다.

```java
public Object pop() {
    if (size == 0) {
        throw new EmptyStackException(); 
    }
    Object result = elements[--size];
    elements[size] = null;
    return result;
}
```

다 쓴 참조를 null 처리하면 다른 이점도 따라온다. 만약 null 처리한 참조를 실수로 사용하려 하면 즉시 NullPointerException을 던지며 종료된다. 프로그램 오류는 가능한 초기에 발견하는 게 좋다.

모든 객체를 일일히 null 처리하는데 혈안이 될 필요는 없지만 **객체 참조를 null 처리하는 일은 예외적인 경우여야 한다.** 다 쓴 참조를 해제하는 가장 좋은 방법은 그 참조를 담은 변수를 유효 범위 밖으로 밀어내는 것이다. 만약 우리가 변수 범위를 최소가 되게 정의했다면 이 일은 자연스럽게 이뤄진다.

그렇다면 null 처리는 언제 해야 할까? Stack 클래스는 왜 메모리 누수에 취약한 걸까? 그 이유는 스택이 자기 메모리를 직접 관리하기 때문이다. 이 스택은 객체가 아니라 객체 참조를 담는 elements 배열로 저장소 풀을 만들어 원소들을 관리한다. 배열의 활성화 영역에 속한 원소들이 사용되고 비활성 영역은 쓰이지 않는다. 따라서 가비지 컬렉터는 이 사실을 알 길이 없다는 데 있다.

**비활성 영역의 객체가 더이상 쓸모 없다는 건 프로그래머만 아는 사실이다. 그러므로 비활성 영역이 되었을 때 null 처리해서 해당 객체를 더는 쓰지 않을 것임을 가비지 컬렉터에게 알려한다.**

**일반적으로 자기 메모리를 직접 관리하는 클래스라면 프로그래머는 항시 메모리 누수에 주의해야 한다.** 원소를 다 사용한 즉시 그 원소가 참조한 객체들을 다 null 처리해줘야 한다.

**캐시 역시 메모리 누수를 일으키는 주범이다.** 객체 참조를 캐시에 넣고 나서, 이 사실을 잊은 채 그 객체를 다 쓴 뒤로도 한참을 그냥 놔두는 일을 자주 접할 수 있다.

#### 캐시 메모리 누수 처리 방법

1. WeakHashMap
2. ThreadPoolExecutor
3. LinkedHashMap