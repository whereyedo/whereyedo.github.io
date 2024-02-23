---
layout: post
title: Proxy Pattern
subtitle: Proxy Pattern
image: /assets/img/java.png
optimized_image: /assets/img/java.png
category: JAVA
tags: 
    - java
    - spring
author: whereyedo
---

## 프록시 패턴

프록시 패턴(Proxy Pattern)은 객체지향 프로그래밍 디자인 패턴 중 하나로, **로직의 흐름을 제어하기 위해 실제 객체의 프록시(대리자) 객체를 두어 대신 처리하게 하는 방법**이다.
프록시 객체는 실제 객체에 대한 인터페이스를 제공하여 클라이언트가 실제 객체에 직접 접근하는 것을 대신하거나 보완하는 역할을 한다.

## 프록시 패턴의 주요 목적
1. **보안 향상 (Security Enhancement)**: 프록시를 사용하여 실제 객체에 직접 접근하는 것을 제어하고, 특정 권한이나 제약을 추가하여 보안을 향상시킬 수 있다.
2. **지연 로딩 (Lazy Loading)**: 실제 객체의 생성 및 초기화에 비용이 많이 들 때, 프록시는 객체가 실제로 필요한 시점에 생성될 수 있도록 하는 지연 로딩을 구현할 수 있다.
3. **원격 프록시 (Remote Proxy)**: 객체가 원격 서버에 존재하는 경우, 클라이언트는 로컬에 있는 프록시를 통해 원격 객체에 접근할 수 있다.
4. **캐싱 (Caching)**: 프록시를 사용하여 이전에 수행한 작업의 결과를 캐싱하고, 동일한 요청이 들어올 때 이전 결과를 반환함으로써 성능을 향상시킬 수 있다.

## 프록시 패턴의 구성 요소
- **Subject**: 실제 객체와 프록시 객체가 공유하는 인터페이스
- **Real Subject**: 프록시 패턴을 적용할 실제 객체
- **Proxy**: 실제 주체의 대리자로 동작하는 객체

## 프록시 패턴의 종류

### 기본 프록시 (Normal Proxy)
```java
public interface Subject {
    void request();
}

public class RealSubject implements Subject {
	@Override
	public void request() {
		System.out.println("RealSubject: Handling request.");
	}
}

public class Proxy implements Subject {
	private RealSubject realSubject;

	public Proxy() {
		// 프록시가 실제 객체에 대한 접근을 제어하기 때문에 실제 객체를 생성하거나 관리할 수 있다.
		// 이 예제에서는 프록시 내에서 실제 객체를 생성
		this.realSubject = new RealSubject();
	}

	@Override
	public void request() {
		// 프록시가 실제 객체에 대한 접근을 제어하고 추가적인 작업을 수행할 수 있다.
		System.out.println("Proxy: Pre-processing");

		// 실제 객체의 메서드를 호출
		realSubject.request();

		System.out.println("Proxy: Post-processing");
	}
}

public class Client {
	public static void main(String[] args) {
		// 클라이언트는 프록시를 통해 실제 객체에 접근
		Subject proxy = new Proxy();
		proxy.request();
	}
}
```

### 가상 프록시 (Virtual Proxy)
- 객체의 생성 및 초기화에 비용이 많이 들고, 이를 실제로 사용하는 시점이 불분명한 경우 사용된다.
- 객체가 실제로 필요할때 생성되도록 지연시키는 방식
```java
// 이미지 표시를 위한 인터페이스
public interface Image {
    void display();
}

// 실제 이미지 표시를 담당하는 클래스
public class RealImage implements Image {
	private String filename;

	public RealImage(String filename) {
		this.filename = filename;
		loadImageFromDisk();
	}

	private void loadImageFromDisk() {
		System.out.println("Loading image from disk: " + filename);
	}

	@Override
	public void display() {
		System.out.println("Displaying image: " + filename);
	}
}

// 이미지 표시를 지연시키는 프록시 클래스
public class VirtualImageProxy implements Image {
	private RealImage realImage;
	private String filename;

	public VirtualImageProxy(String filename) {
		this.filename = filename;
	}

	@Override
	public void display() {
		if (realImage == null) {
			// 이미지가 필요한 시점에서 실제 이미지를 생성
			realImage = new RealImage(filename);
		}
		realImage.display();
	}
}

public class Client {
	public static void main(String[] args) {
		// 클라이언트는 프록시를 통해 이미지에 접근
		Image image = new VirtualImageProxy("sample.jpg");

		// 이미지가 실제로 필요한 시점에서 로딩이 발생
		image.display();
	}
}
```

### 보호 프록시 (Protection Proxy)
- 객체에 대한 접근을 제한하고 보안 검사를 수행해야 하는 경우 사용된다.
```java
// 파일을 읽기 위한 인터페이스
public interface FileReader {
    void readFile(String fileName);
}

// 실제 파일 읽기를 담당하는 클래스
public class RealFileReader implements FileReader {
    @Override
    public void readFile(String fileName) {
        System.out.println("Reading file: " + fileName);
    }
}

// 파일에 대한 접근을 제어하는 프록시 클래스
public class FileReaderProxy implements FileReader {
    private RealFileReader realFileReader;
    private String allowedUser;

    public FileReaderProxy(String allowedUser) {
        this.allowedUser = allowedUser;
        realFileReader = new RealFileReader();
    }

    @Override
    public void readFile(String fileName) {
        if (checkAccess()) {
            realFileReader.readFile(fileName);
        } else {
            System.out.println("Access denied. You don't have permission to read the file.");
        }
    }

    private boolean checkAccess() {
        // 여기에서 사용자의 권한을 확인하고 허용 여부를 반환
        String currentUser = getCurrentUser(); // 사용자 확인을 위한 메서드
        return allowedUser.equals(currentUser);
    }

    private String getCurrentUser() {
        // 현재 사용자를 확인하는 메서드 
        return "admin"; // 임의의 사용자를 admin으로 가정
    }
}

public class Client {
    public static void main(String[] args) {
	    // 등록된 사용자가 보호 프록시를 통해 파일을 읽으려고 할 때
        FileReader fileReader = new FileReaderProxy("admin");
        fileReader.readFile("example.txt"); // 접근 허용

        // 등록되지 않은 사용자가 보호 프록시를 통해 파일을 읽으려고 할 때
        FileReader unauthorizedReader = new FileReaderProxy("user");
        unauthorizedReader.readFile("example.txt"); // 접근 거부
    }
}
```

### 캐싱 프록시 (Caching Proxy)
- 객체의 생성 및 초기화에 비용이 많이 들고, 동일한 요청이 반복되는 경우에 사용된다.
- 매번 작업을 다시 수행하지 않고 캐싱된 결과를 반환하는 방식
```java
public interface Calculator {
    int calculate(int num);
}

// 계산 작업을 수행하는 원본 클래스
public class RealCalculator implements Calculator {
    @Override
    public int calculate(int num) {
        return num * num;
    }
}

// 계산 작업을 캐싱하는 프록시 클래스
public class CachingCalculatorProxy implements Calculator {
    private RealCalculator realCalculator;
    private Map<Integer, Integer> cache;

    public CachingCalculatorProxy() {
        realCalculator = new RealCalculator();
        cache = new HashMap<>();
    }

    @Override
    public int calculate(int num) {
        if (cache.containsKey(num)) {
            System.out.println("Result from cache: " + cache.get(num));
            return cache.get(num);
        } else {
            int result = realCalculator.calculate(num);
            cache.put(num, result);
            System.out.println("Result calculated and cached: " + result);
            return result;
        }
    }
}

public class Client {
    public static void main(String[] args) {
        Calculator calculator = new CachingCalculatorProxy();

        // 첫 번째 호출 (캐싱)
        System.out.println("Result: " + calculator.calculate(5));

        // 두 번째 호출 (캐시된 결과 사용)
        System.out.println("Result: " + calculator.calculate(5));

        // 다른 값을 넣어 호출
        System.out.println("Result: " + calculator.calculate(10));
    }
}
```

이 외에도 다양한 프록시 유형이 있을 수 있으며, 사용 사례 및 요구 사항에 따라 적절한 프록시를 선택해 사용하면 된다.

이렇게 프록시 패턴을 사용하면 객체 지향 프로그래밍(OOP) 3요소중 하나인 설계 원칙(SOLID) 중 의존 역전 원칙(DIP)과 개방 폐쇄 원칙(OCP)의 효과를 얻을 수 있다는 장점도 있다.

하지만 원본 클래스 수만큼 프록시 클래스도 만들어 주어야 하므로 그로 인해 코드량이 많아지게 되고, 중복 코드도 증가한다는 단점이 존재하기도 한다.

이러한 단점을을 보완하기 위해 컴파일 시점이 아닌 런타임 시점에 프록시 클래스를 만들어주는 동적 프록시(Dynamic Proxy) 기능이 있다.




