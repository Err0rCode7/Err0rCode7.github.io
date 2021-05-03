---
bg: "Review.jpg"
layout: post
title:  "Spring과 Design Pattern - 2"
crawlertitle: "Spring과 Design Pattern"
summary: "Spring과 Design Pattern - 2"
date: 2021-05-03 17:37:00 +0900
category: Spring
author: Err0rCode7
---

#### 스프링이 사랑한 디자인 패턴 - 2
##### [자바 객체지향의 원리](https://wikibook.co.kr/java-oop-for-spring/) 책을 읽고 내용을 정리하는 글입니다.

## 템플릿 메서드 패턴(Template Method Pattern)

강아지와 고양이를 키운다고 상상하고 같이 재미있는 시간을 보내는 세계를 프로그램으로 표현한다면 다음과 같은 2개의 클래스가 필요할 것이다.

```java
public class Dog {
  public void playWithOwner() {
    System.out.println("귀염둥이 이리온...");
    System.out.println("멍! 멍!");
    System.out.println("꼬리 살랑 살랑~");
    System.out.println("잘했어");
  }
}

public class Cat {
  public void playWithOwner() {
    System.out.println("귀염둥이 이리온...");
    System.out.println("야옹~ 야옹~");
    System.out.println("꼬리 살랑 살랑~");
    System.out.println("잘했어");
  }
}
```

강아지는 멍멍, 고양이는 야옹야옹 하는 부분 이외에는 모두 같은 것을 볼 수 있다. 코드를 보고 있으면 객체 지향의 4대 특성 가운데 상속을 통해 동일한 부분(중복)은 상위 클래스로 달라지는 부분만 하위 클래스로 분할하고 싶은 객체 지향 설계에 대한 욕구가 자극될 것이다.

이에 따라 코드를 개선해보자.

```java
public abstract class Animal {
  // 템플릿 메서드
  public void playWithOwner() {
    System.out.println("귀엽둥이 이리온...");
    play();
    runSomething();
    System.out.println("잘했어");
  }
  // 추상 메서드
  abstract void play();
  // Hook(갈고리) 메서드
  void runSomething() {
    System.out.println("꼬리 살랑 살랑 ~");
  }
}
```

```java
public class Dog extends Animal {
  @Override
  // 추상 메서드 오버라이딩
  void play() {
    System.out.println("멍! 멍!");
  }
  
  @Override
  // Hook(갈고리) 메서드 오버라이딩
  void runSomething() {
    System.out.println("멍~ 멍~ 꼬리 살랑 살랑~");
  }
}
```

```java
public class Cat extends Animal {
  @Override
  // 추상 메서드 오버라이딩
  void play() {
    System.out.println("야옹! 야옹!");
  }
  
  @Override
  // Hook(갈고리) 메서드 오버라이딩
  void runSomething() {
    System.out.println("야옹~ 야옹~ 꼬리 살랑 살랑~");
  }
}
```

```java
public class Driver {
  public static void main(String[] args) {
    Animal bolt = new Dog();
    Animal kitty = new Cat();
    
    bolt.playWithOwner();
    
    System.out.println();
    System.out.println();
    
    kitty.playWithOwner();
  }
}
```

상위 클래스인 `Animal`에는 템플릿(견본)을 제공하는 `playWithOwner()` 메서드와 하위 클래스에게 구현을 강제하는 `play()` 추상 메서드, 하위 클래스가 선택적으로 오버라이딩할 수 있는 `runSomething()` 메서드가 있다. 하위 클래스인 `Dog`과 `Cat`은 상위 클래스인 `Animal`에서 구현을 강제하고 있는 `play()` 추상 메서드를 반드시 구현해야 한다. `runSomgthing()` 메서드는 선택적으로 오버라이딩할 수 있다.

이처럼 상위 클래스에 공통 로직을 수행하는 템플릿 메서드와 하위 클래스에 오버라이딩을 강제하는 추상 메서드 또는 선택적으로 오버라이딩할 수 있는 훅(Hook) 메서드를 두는 패턴을 템플릿 메서드 패턴이라고 한다.

> Template Method Pattern 이란,
>
> 상위 클래스의 견본 메서드에서 하위 클래스가 오버라이딩한 메서드를 호출하는 패턴

## 팩터리 메서드 패턴(Factory Method Pattern)

팩터리는 공장을 의미힌다. 공장은 물건을 생산하는데 객체 지향에서 팩터리는 객체를 생성한다. 결국 팩터리 메서드는 객체를 생성 반환하는 메서드를 말한다. 여기에 패턴이 붙으면 하위 클래스에서 팩터리 메서드를 오버라이딩해서 객체를 반환하게 하는 것을 의미한다.

위에 템플릿 메서드 패턴에서의 예제를 더 이어나가보자. 강아지와 고양이가 각자 가지고 놀고 싶어하는 장난감을 가져오는 모습을 상상해 보자. 이를 코드로 구현해보자.

```java
public abstract class Animal {
  // 추상 팩터리 메서드
  abstract AnimalToy getToy();
}
```

```java
public class Dog extends Animal {
  // 추상 팩터리 메서드 오버라이딩
  @Override
  AnimalToy getToy() {
    return new DogToy();
  }
}

// 팩터리 메서드가 생성할 객체
public class DogToy extends AnimalToy {
  @Override
  public void identify() {
    System.out.println("나는 테니스공! 강아지의 친구!");
  }
}

public class Cat extends Animal {
  // 추상 팩터리 메서드 오버라이딩
  @Override
	AnimalToy getToy() {
    return new CatToy();
  }
}

// 팩터리 메서드가 생성할 객체
public class CatToy extends AnimalToy {
  @Override
  public void identify() {
    System.out.println("나는 캣타워! 고양이의 친구!");
  }
}
```

```java
public class Driver {
  public static void main(String[] args) {
    Animal bolt = new Dog();
    Animal kitty = new Cat();
    
    // 팩터리 메서드가 반환하는 객체들
    AnimalToy boltBall = bolt.getToy();
    AnimalToy kittyTower = kitty.getToy();
    
    // 팩터리 메서드가 반환한 객체들을 사용
    boltBall.identify();
    kittyTower.identify();
  }
}
```

> Factory Method Pattern 이란,
>
> 오버라이드된 메서드가 객체를 반환하는 패턴

## 전략 패턴(Strategy Pattern)

디자인 패턴의 꽃이라고 하는 전략 패턴이다. 전략 패턴을 구성하는 세 요소는 꼭 기억해 둬야한다.

- 전략 메서드를 가진 전략 객체
- 전략 객체를 사용하는 컨텍스트(전략 객체의 사용자/소비자)
- 전략 객체를 생성해 컨텍스트에 주입하는 클라이언트(제3자, 전략 객체의 공급자)

클라이언트는 다양한 전략 중 하나를 선택해 생성한 후 컨텍스트에 주입한다.

군인이 있다고 상상해보자. 그리고 그 군인이 사용할 무기가 있다고 하자. 보급 장교가 무기를 군인에게 지급해 주면 군인은 주어진 무기에 따라 전투를 수행하게 된다. 이 이야기를 전략 패턴에 따라 구분해 보면 무기는 전략이 되고, 군인튼 컨텍스트, 보급 장교는 제 3자, 즉 클라이언트가 된다. 이를 자바 코드로 구현해보자.

```java
public interface Strategy {
  public abstract void runStratety();
}
```

```java
public class StrategyGun implements Strategy {
  @Override
  public void runStrategy() {
    System.out.println("탕, 타당, 타다당");
  }
}

public class StrategySword implements Strategy {
  @Override
  public void runStrategy() {
    System.out.println("챙.. 채쟁챙 챙챙");
  }
}

public class StrategyBow implements Strategy {
  @Override
  public void runStrategy() {
    System.out.println("슝.. 쐐액.. 쉑, 최종 병기");
  }
}
```

```java
public class Soldier {
  void runContext(Strategy strategy) {
    System.out.println("전투 시작");
    strategy.runStrategy();
    System.out.println("전투 종료");
  }
}
```

```java
public class Client {
  public static void main(String[] args) {
    Strategy strategy = null;
    Soldier rambo = new Soldier();
    
    // 총을 람보에게 전달해서 전투를 수행하게 한다.
    strategy = new StrategyGun();
    rambo.runContext(strategy);
    
    System.out.println("");
    
    // 검을 람보에게 전달해서 전투를 수행하게 한다.
    strategy = new StrategySword();
    rambo.runContext(strategy);
    
    System.out.println("");
    
    // 활을 람보에게 전달해서 전투를 수행하게 한다.
    strategy = new StrategyBow();
    rambo.runContext(strategy);
  }
}
```

위 코드처럼 전략을 다양하게 변경하면서 컨텍스트를 실행할 수 있다. 전략 패턴은 디자인 패턴의 꽃이라고 할 정도로 다양한 곳에서 다양한 문제 상황의 해결책으로 사용된다.

혹시나 템플릿(견본) 메서드 패턴과 유사하다는 느낌이 든다면, 제대로 본 것이다. 같은 문제의 해결책으로 상속을 이용하는 템플릿 메서드 패턴과 객체 주입을 통한 전략 패턴 중에서 선택/적용할 수 있다.

단일 상속만이 가능한 자바 언어에서는 상속이라는 제한이 있는 템플릿 메서드 패턴보다는 전략 패턴이 더 많이 활용된다.

전략 패턴 또한 전략 인터페이스를 두어 OCP, DIP이 적용된 것을 짐작할 수 있다.

> Strategy Pattern 이란,
>
> 클라이언트가 전략을 생성해 전략을 실행할 컨텍스트에 주입하는 패턴

## 템플릿 콜백 패턴(Template Callback Pattern - 견본/회신 패턴)

템플릿 콜백 패턴은 전략 패턴의 변형으로, **스프링의 3대 프로그래밍 모델 중 하나인 DI(의존성 주입)에서 사용하는 특별한 형태의 전략 패턴이다.**

템플릿 콜백 패턴은 전략 패턴과 모든 것이 동일한데, 전략을 익명 내부 클래스로 정의해서 사용한다는 특징이 있다. 앞에서 살펴본 전략 패턴 코드를 템플릿 콜백 패턴으로 바꿔보자.

익명 내부 클래스를 사용하기 때문에 StrategyGun, StrategySword, StrategyBow는 필요 없다.

```java
public interface Strategy {
  public abstract void runStratety();
}
```

```java
public class Soldier {
  void runContext(Strategy strategy) {
    System.out.println("전투 시작");
    strategy.runStrategy();
    System.out.println("전투 종료");
  }
}
```

```java
public class Client {
  public static void main(String[] args) {
    Strategy strategy = null;
    Soldier rambo = new Soldier();
    
    rambo.runContext(new Strategy() {
      @Override
      public void runStrategy() {
        System.out.println("탕, 타당, 타다당");
      }
    });
    
    System.out.println("");

    rambo.runContext(new Strategy() {
      @Override
      public void runStrategy() {
        System.out.println("챙.. 채쟁챙 챙챙");
      }
    });
    
    System.out.println("");
    
    rambo.runContext(new Strategy() {
      @Override
      public void runStrategy() {
        System.out.println("도끼! 도도독끼!");
      }
    });
  }
}
```

활 대신 도끼를 쓴다는 것 외에 실행 결과는 기존 전략 패턴과 차이가 없다. 그런데 코드를 보면 많은 부분에서 중복된 코드가 보인다. 리팩터링을 해보자.

```java
public interface Strategy {
  public abstract void runStratety();
}
```

```java
public class Soldier {
  void runContext(String weaponSound) {
    System.out.println("전투 시작");
    executeWeapon(weaponSound).runStrategy();
    System.out.println("전투 종료");
  }
  
  private Strategy executeWeapon(final String weaponSound) {
    return new Strategy() {
      @Override
      public void runStrategy() {
		    System.out.println(weaponSound);
      }
    };
  }
}
```

전략을 생성하는 코드가 컨텍스트, 즉 군인 내부로 들어왔다.

```java
public class Client {
  public static void main(String[] args) {
    Strategy strategy = null;
    Soldier rambo = new Soldier();
    
    rambo.runContext("탕, 타당, 타다당");
    
    System.out.println("");

    rambo.runContext("챙.. 채쟁챙 챙챙");
    
        System.out.println("");
    
    rambo.runContext("도끼! 도도독끼!");
  }
}
```

중복되는 부분을 컨텍스트로 이관하여 코드가 상당히 깔끔해졌다.

**스프링은 이런 형식으로 리팩터링된 템플릿 콜백 패턴을 DI에 적극 활용하고 있다.** 따라서 스프링을 이해하고 활용하기 위해서는 전략 패턴과 템플릿 콜백 패턴, 리팩터리된 템플릿 콜백 패턴을 잘 기억해 두자.

>Template Callback Pattern 이란,
>
>전략을 익명 내부 클래스로 구현한 전략 패턴

다양한 디자인 패턴들을 살펴보았다. 스프링에서 사용한 디자인 패턴과 객체 지향 코드를 작성하기 위한 레시피가 어떤 것들이 있었는 지 확실히 기억해두자.