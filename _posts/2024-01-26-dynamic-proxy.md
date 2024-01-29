---
layout: post
title: Dynamic Proxy
subtitle: Dynamic Proxy
image: /assets/img/blog-image.png
optimized_image: /assets/img/blog-image.png
category: JAVA
tags: 
    - java
    - spring
author: whereyedo
---

# 프록시 패턴(Proxy Pattern)

프록시 패턴은 객체지향 프로그래밍 디자인 패턴 중 하나로 기존 대상 원본 객체를 수정 없이 추가 동작 기능들을 추가하고 싶을 때 사용하는 코드 패턴이다.

```java
public interface AInterface {

    void print();
}

public class AImpl implements AInterface {

	@Override
	public void print() {
		System.out.println(this.getClass().getSimpleName());
	}
}

public class AProxy implements AInterface {

	private final AInterface a;

	public AProxy(final AInterface a) {
		this.a = a;
	}

	@Override
	public void print() {
		long start = System.nanoTime();
		System.out.printf("%s start%n", this.getClass().getSimpleName());
		a.print();
		System.out.printf("%s end - resultTime: %f seconds%n", this.getClass().getSimpleName(), (System.nanoTime() - start) / 1_000_000_000.d);
	}
}

public class Main {
	public static void main(String[] args) {
		AInterface a = new AProxy(new AImpl());
		a.print();
	}
}
```

```text
AProxy start
AImpl 
AProxy end - resultTime: 0.000066 seconds
```

위와 같이 프록시 패턴을 사용하면 객체 지향 프로그래밍(OOP) 3요소중 하나인 설계 원칙(SOLID) 중 의존 역전 원칙(DIP)과 개방 폐쇄 원칙(OCP)의 효과를 얻을 수 있다는 장점도 있지만
원본 클래스 수만큼 프록시 클래스도 만들어 주어야 하므로 그로 인해 코드량이 많아지게 되고, 중복 코드도 증가한다는 단점이 존재하기도 한다.
이러한 단점을을 보완하기 위해 컴파일 시점이 아닌 런타임 시점에 프록시 클래스를 만들어주는 동적 프록시(Dynamic Proxy) 기능이 생겨나게 되었다.




