# 6.  객체와 자료 구조

생성일: 2022년 2월 3일 오전 12:16

## 1. 자료 추상화

```java
// 1. 구체적인 Point Class 
public class Point {
    public double x;
    public double y;
}

// 2. 추상적인 Point Class 
public interface Point {
    double getX();
    double getY();
    void setCartesian(double x, double y); //직교좌표
    double getR();
    double getTheta();
    void setPolar(double r, double theta); //극좌표
}
```

- 1, 2번 모두 2차원 점을 표현한다. 1번 클래스는 구현을 외부로 노출하고 2번 클래스는 구현을 완전히 숨긴다.
- 1번은 확실히 직교 좌표계를 사용한다. 또한 개별적으로 좌표값을 읽고 설정하게 강제한다. ⇒ (구현을 노출)  변수를 private으로 선언하더라도 각 값마다 조회(getter)함수와 설정(setter)함수를 제공한다면 구현을 외부로 노출하는 셈이다.
- 2번은 점이 직교좌표계를 사용하는지, 극좌표계를 사용하는지 알 길이 없지만 ⇒ (구현을 숨김), 인터페이스는 자료구조를 명백하게 표현한다.  ( setCartesian, setPolar )
- 사실 2번은 자료 구조 이상을 표현한다. 클래스 메서드가 접근 정책을 강제한다. 좌표를 읽을 때는 각 값을 개별적으로 읽어야하지만, 좌표를 설정할 때는 두 값을 한꺼번에 설정해야 한다.

- 변수 사이에 함수라는 계층을 넣는다고 구현이 저절로 감춰지는 것은 아니다. 구현을 감추려면 **추상화**가 필요하다. **추상 인터페이스를 제공해 사용자가 구현을 모른 채 자료의 핵심을 조작할 수 있어야 진정한 의미의 클래스**다.

```java
// 3. 구체적인 Vehicle 클래스 
public interface Vehicle {
    double getFuelTankCapacityInGallons();
    double getGallonsOfGasoline(); 
}

// 4. 추상적인 Vehicle 클래스
public interface Vehicle {
    double getPercentFuelRemaining();
}
```

- 3번은 자동차 연료 상태를 구체적인 숫자 값으로 알려주지만, 4번은 자동차 연료 상태를 백분율이라는 추상적인 개념으로 알려준다. 3번은 두 함수가 변수값을 읽어 반환할 뿐이라는 사실이 거의 확실하다. 4번은 정보가 어디서 오는지 전혀 드러나지 않는다.
- 1번보다는 2번이, 3번보다는 4번이 더 좋다.  자료를 세세하게 공개하기보다는 추상적인 개념으로 표현하는 편이 좋다.

## 2. 자료/객체 비대칭

- **객체**는 추상화 뒤로 자료를 숨긴 채 자료를 다루는 함수만 공개한다.
- **자료 구조**는 자료를 그대로 공개하며 별다른 함수는 제공하지 않는다.
- 두 정의는 본질적으로 상반되며 두 개념은 사실상 정반대다. 사소한 차이로 보이지만 그 차이가 미치는 영향은 굉장한데  아래의 예를 들 수 있다.

```java
// 5. 절차적인 도형
public class Square {
    public Point topLeft;
    public double side;
}

public class Circle {
    public Point center;
    public double radius;
}

public class Geometry {
    public double area(Object shape) throws NoSuchShapeException{
        if(shape instanceof Square) {
            ...
        }
        else if(shape instanceof Circle) {
            ...
        }
        else{
            throw new NoSuchShapeException();
        }
    }
}
```

- 각 도형 클래스는 간단한 자료 구조다. 아무 메서드도 제공하지 않는다. 도형이 동작하는 방식은 Geometry 클래스에서 구현한다.
- ⇒  Geometry 클래스에 구현된 함수는 절차지향 방식. (cf. 절차지향은 순차적인 실행에 초점이 되어있고, 데이터를 중심으로 함수를 구현)
- 장점 : Geometry에 둘레 길이를 구하는 perimeter()함수를 추가하고 싶다면 ? 각 도형 클래스는 아무 영향을 받지 않는다. (도형 클래스에 의존하는 다른 클래스도 마찬가지)
- 단점 : 새 도형을 추가하고 싶다면 ? Geometry 에 속한 함수를 모두 고쳐야한다.

```java
// 6. 다형적인 도형 
public class Square implements Shape {
    private Point topLeft;
    private double side;

    public double area() {
        return side * side;
    }
}

public class Circle implements Shape {
    private Point center;
    private double radius;
    public final double PI = 3.14;

    public double area() {
        return PI * radius * radius;
    }
}
```

- 객체 지향적인 도형 클래스다. area()는 다형메서드다.
- 장점 : Geometry 클래스는 필요없고, 그러므로 새 도형 클래스를 추가해도 기존 함수에 아무런 영향을 미치지 않는다.
- 단점 : 도형 클래스에 새로운 함수를 추가하면 모든 도형 클래스를 고쳐야 한다.

### 결론

- 예제 5와 6은 상호 보완적인 특질이 있다. 그래서 객체와 자료 구조는 근본적으로 양분된다.
- 객체 지향 코드에서 어려운 변경은 절차적인 코드에서 쉽고, 절차 지적인 코드에서 어려운 변경은 객체 지향에서 쉽다.
- 새로운 함수가 아니라 새로운 자료 타입이 필요한 경우에는 클래스와 객체 지향 기법이 가장 적합하다.
- 새로운 자료 타입이 아니라 새로운 함수가 필요한 경우에는 절차적인 코드와 자료 구조가 더 적합하다.
- 분별 있는 프로그래머는 모든 것이 객체라는 생각이 미신임을 잘 안다. 때로는 단순한 자료 구조와 절차적인 코드가 가장 적합한 상황도 있다.

## 3. 디미터 법칙

- 잘 알려진 휴리스틱(heuristic)으로, **모듈은 자신이 조작하는 객체의 속사정을 몰라야 한다**는 법칙.
- 디미터의 법칙은 클래스 C의 메서드 f는 아래 객체의 메서드만 호출해야한다고 주장한다. 하지만 위 객체에서  허용된 메서드가 반환하는 객체의 메서드는 호출하면 안된다. (낯선 사람은 경계하고 친구랑만 놀라는 의미)
    - 클래스 C
    - f가 생성한 객체
    - f 인수로 넘어온 객체
    - C 인스턴스 변수에 저장된 객체
- 좋은 참조 예제

[[OOP] 디미터의 법칙(Law of Demeter)](https://mangkyu.tistory.com/147)

### 기차 충돌

- 아래와 같은 코드를 기차 충돌(train wreck)이라 부른다. 일반적으로 조잡하다 여겨지는 방식이므로 피하는 편이 좋다.

```java
final String outputDir = ctxt.getOptions().getScratchDir().getAbsolutePath();
```

- 위 코드는 다음과 같이 나누는 편이 좋다.

```java
Options opts = ctxt.getOptions();
File scratchDir = opts.getScratchDir();
final String outputDir = scratchDir.getAbsolutePath();
```

- 위 코드 형태로 구현된 함수는 ctxt 객체가 Options를 포함하며, Options가 ScratchDir을 포함하며, ScratchDir이 AbsolutePath를 포함한다는 사실을 안다. 함수 하나가 아는 지식이 너무 많다.
- 위의 두 예제는 ctxt, Options, ScratchDir이 객체인지 아니면 자료 구조인지에 따라 디미터의 법칙 위반여부를 알 수 있다.
- 객체라면 내부 구조를 숨겨야 하므로 디미터의 법칙을 위반한다.
- 자료 구조라면 내부 구조를 노출하므로 디미터 법칙이 적용되지 않는다.
- 위 예제는 조회함수(get)를 사용하는 바람에 혼란을 일으키는데, 코드를 다음과 같이 구현했다면 디미터 법칙을 거론할 필요가 없다.

```java
final String outputDir = = ctxt.options.scratchDir.absolutePath;
```

- 자료 구조는 무조건 함수 없이 공개 변수만 포함하고, 객체는 비공개 변수와 공개 함수를 포함하는 것이 좋다. 하지만 단순한 자료 구조에도 조회/설정(getter/setter) 함수를 정의하라 요구하는 프레임워크와 표준 (ex. bean)이 존재한다.

### 잡종 구조

- 때때로 절반은 객체, 절반은 자료 구조인 잡종 구조가 나온다. 중요한 기능을 수행하는 함수도 있고, 공개 변수나 공개 조회/설정(get/set)함수도 있다.
- 잡종 구조는 새로운 함수는 물론이고 새로운 자료 구조도 추가하기 어렵다.  양쪽 세상에서 단점만 모아놓은 구조인 잡종 구조는 되도록 피하자.

### 구조체 감추기

- ctxt, options, scratchDir이 객체라면 ? 기차 충돌 방식을 피해야한다. 객체라면 내부 구조를 감춰야하니까.
- 그럼 임시 디렉터리의 절대 경로는 어떻게 얻어야 좋을까 ?

```java
// ctxt 객체에 공개해야 하는 메서드가 너무 많아진다.
ctxt.getAbsolutePathOfScratchDirectoryOption(); 

// getScratchDirectoryOption()이 객체가 아니라 자료 구조를 반환한다 가정. 
ctxt.getScratchDirectoryOption().getAbsolutePath();
```

- 위 두 방법도 썩 내키지 않는다.
- ctxt가 객체라면 **뭔가를 하라고** 말해야지 속을 드러내라고 말하면 안된다.
- 임시 디렉터리의 절대 경로를 얻으려는 이유가 임시 파일을 생성하기 위한 목적이라면, ctxt 객체에 임시 파일을 생성하라고 시키는 것이 낫다.

```java
BufferedOutputStream bos = ctxt.createScratchFileStream(classFileName);
```

- ctxt는 내부 구조를 드러내지 않으며, 모듈에서 해당 함수는 자신이 몰라야 하는 여러 객체를 탐색할 필요가 없다. 따라서 디미터 법칙을 위반하지 않는다.

## 4. 자료 전달 객체

- 자료 구조체의 전형적인 형태는 공개 변수만 있고 함수가 없는 클래스다. 이런 자료 구조체를 때로는 자료 전달 객체(DTO)라 한다.
- 흔히 DTO(Data Transfer Object) 는 데이터베이스에 저장된 가공되지 않은 정보를 애플리케이션 코드에서 사용할 객체로 변환하는 일련의 단계에서 가장 처음으로 사용하는 구조체.
- 조금 더 일반적인 형태는 Bean 구조다. Bean은 비공개(private) 변수를 getter/setter 함수로 조작한다. 일종의 사이비 캡슐화로 별다른 이익을 제공하지는 않는다.
- cf. DTO와 VO 그리고 Entity의 차이

[DTO와 VO 그리고 Entity의 차이](https://youngjinmo.github.io/2021/04/dto-vo-entity/)

### 활성 레코드

- DTO의 특수한 형태. 공개 변수가 있거나 비공개 변수에 getter/setter 함수가 있는 자료 구조지만, 대게 save, find와 같은 탐색 함수도 제공한다. 활성 레코드는 데이터베이스 테이블이나 다른 소스에서 자료를 직접 변환한 결과다.
- 활성 레코드에 비지니스 규칙 메서드를 추가해 이런 자료 구조를 객체로 취급하는 개발자가 흔한데, 이는 바람직하지 않다. 자료 구조도 아니고 객체도 아닌 잡종 구조가 나오기 때문.
- 활성 레코드는 자료 구조로 취급한다. 비지니스 규칙을 담으면서 내부 자료를 숨기는 객체는 따로 생성한다. (여기서 내부 자료는 활성 레코드의 인스턴스일 가능성이 높다.)

## 결론

- 객체는 동작을 공개하고 자료를 숨긴다. 그래서 기존 동작을 변경하지 않으면서 새 객체 타입을 추가하기는 쉬운 반면, 기존 객체에 새 동작을 추가하기는 어렵다.
- 자료구조는 별다른 동작 없이 자료를 노출한다. 그래서 기존 자료 구조에 새 동작을 추가하기는 쉬우나, 기존 함수에 새 자료 구조를 추가하기는 어렵다.
- 어떤 시스템을 구현할 때, 새로운 자료 타입을 추가하는 유연성이 필요하면 객체가 더 적합하다.
- 반대로 새로운 동작을 추가하는 유연성이 필요하면 자료 구조와 절차적인 코드가 더 적합하다.
- 우수한 소프트웨어 개발자는 편견없이 이 사실을 이해해 직면한 문제에 최적인 해결책을 선택한다.