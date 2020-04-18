---
title: "아이템8 - finalizer와 cleaner를 사용을 피하라."
date: "2020-04-17 08"
template: "post"
draft: false
category: "Effective Java"
tags:
  - "Java"
  - "effectiveJava"
description: "2장. 객체 생성과 파괴 - finalizer와 cleaner를 사용을 피하라."
---

### 아이템 8 - finalizer와 cleaner 사용을 피하라

#### 객체 소멸자의 문제점
- finalize는 객체가 소멸될 때 호출되는 메소드로 JVM에 의해 실행된다.
- 하지만 이 메소드는 실행된다는 보장이 없고 예측불가한 상황을 만들수 있는 가능성이 높다.
- 즉, 분명 쓰임새가 없진 않지만 일반적으로는 **사용하지 말아야한다.**
- 특히 DB Lock 해제 등을 finalize에 해두면 정말 큰일나는거다.
- `System.gc()`나 `System.runFinalization` 메소드는 finalizer 호출 가능성을 높여주지만 여전히 호출된다고 보장하지 않는다.

#### 성능에 문제
- finalizer는 가비지 컬렉터의 효율을 떨어트린다.

#### 정합성이 깨진 객체가 생성됨
- 직렬화 과정에서 예외가 발생하면 악의적읜 finalizer가 호출될 수도 있다.
- 자신의 정적 필드에 자신을 할당하여 영원히 GC되지 않는 객체가 될 수도 있다.
- 이런 공격을 방지하려면 텅 빈 finalize메소드를 만들고 final로 선언하면 된다. (애초에 final 클래스는 상관 X)

#### 파일이나 스레드 종료 처리 대안
- AutoCloseale를 사용하면 자원 종료에 대한 문제를 finalizer나 cleaner없이 처리해줄수있다.

#### finalizer 혹은 cleaner 사용처
- 일반적으로 자원 회수에 대한 안전망 역할(하지만 성능 손해가 크기때문에 반드시 그래야하는지 고민 필요)
- 네이티브 메소드를 사용하는 객체(네이티브 피어)에서 사용하는 자원 회수 처리