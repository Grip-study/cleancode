# ch.6 객체와 자료 구조

## 자료구조 :

자료를 그대로 공개하며 별다른 함수는 제공하지 않는다.
 
자료구조를 다루는 클래스의 함수를 추가하기 쉽다.

새로운 자료구조를 추가하기 어렵다.
 
```java
  public class Square{
    public Point topLeft;
    public double side;
  }
  
  public class Rectangle{
    public Point topLeft;
    public double height;
    public double width;
  }
  
  public class Geometry{
    // 새로운 자료구조가 생성될 때 마다, 모든 메서드에 해당 자료구조의
    // if문을 추가시켜줘야 하기 때문에 자료구조를 추가하기 어려움.
    public double area(Object Shape){
      if(shape instanceof  Square){
        Square s = (Square)shape;
        return s.side * s.side;
      } else if(shape instanceof Rectangle){\
        Rectangle r = (Rectangle)shape;
        return r.height * r.width;
      }
    }
  }
```

## 객체 : 

추상화 뒤로 자료를 숨긴 채 자료를 다루는 함수만 공개한다.

기존함수를 변경하지 않으면서 새 클래스를 추가하기 쉽다.

새로운 함수를 추가하기 어렵다.

```java
  public interface Shape{
    // 새로운 메서드가 생성 될 때 마다 Shape을 상속받은 클래스들마다
    // 메서드를 추가 시켜줘야 하기 때문에 메서드 추가 어려움.
    double area();
  }
  
  public class Square implements Shape{
    private Point topLeft;
    private double side;
    
    public double area(){
      return side * side;
    }
  }
  
  public class Rectangle implements Shape{
    private Point topLeft;
    private double height;
    private double width;
    
    public double area(){
      return height * width;
    }
  }
```

### 클래스를 만들 때

- 새로운 자료 타입이 더 늘어날 경우 -> 객체지향기법이 적합

- 새로운 함수가 더 늘어날 경우-> 절차적인 코드와 자료구조가 좀 더 적합.

- 둘 다 늘어날 경우 -> ~~퇴사~~

<br>

## 디미터 법칙

모듈은 자신이 조작하는 객체의 속사정을 몰라야 한다.

클래스 C의 메서드f 는 다음과 같은 메서드만 호출해야 한다.

> 1. C의 메서드
> 2. f내에서 생성한 객체의 메서드
> 3. f의 인수로 넘오온 객체의 메서드
> 4. C의 인스턴스 변수의 메서드

```java
  public class C{
    private Point p = new Point(1,2);
    
    public void hi(){
      ...
    }
    
    public void f(Rectangle r){
      Square s = new Square(p, 3);
      hi();                           // 1번
      double sArea = s.area();        // 2번
      double rArea = r.area();        // 3번
      String pString = p.toString();  // 4번
    }
  }
```

<br>

### 기차충돌 (train wreck)
```java
  final String 곡물 = m집.get냉장고().get곡식칸().get곡물();
  ... //집에서 물도 꺼내고 밥통도 꺼내고 적당히 밥하기.
```
위의 코드는 다음과 같이 변경하는게 좋다.
```java
  //객체인 경우
  냉장고 m냉장고 = m집.get냉장고();
  곡식칸 m곡식칸 = m냉장고.get곡식칸();
  final String 곡물 = m곡식칸.get곡물();
  
  //자료구조인 경우
  final String 곡물 = m집.m냉장고.m곡식칸.m곡물;
```
그런데 둘다 디미터 법칙을 피해감..

<br>

### 잡종구조 

어떤것이 더 좋은 것인지 혼란이 와서, 객체와 자료구조를 섞은 구조가 나오는데

이건 두개의 단점만 모아놓은 구조가 됨.

<br>

### 구조체 감추기
 ```java
 final String 곡물 = m밥집.냉장고안에_곡식칸에_곡물();
 ```
이렇게 하면 추상화가 된 것 처럼 보일 수 있겠지만, 객체 속에서의 구현되어 있는 

내용을 이름으로 설명하고 있기 때문에 기존에 한거랑 별 차이가 없음.

객체니까 차라리 그 객체에게 **맞는 일**을 주어 원하는 결과값을 얻자
 ```java
 //원래 하려던 일은 밥 만들기.
 
 public class 집{
  private 냉장고 m냉장고 = new 냉장고();
  private 밥통 m밥통 = new 밥통();
  ...
  
  public 밥 밥_만들기(첨가재료 첨가재료){
    ...// 대충 냉장고에서 곡물 꺼내서 첨가재료와 밥통에 넣고 밥만들기
  }
  
  class Main(){
    public static void main(String... args){
      집 m집 = new 집();
      밥 m밥 = m집.밥_만들기(new 첨가재료(""));
    }
  }
 }
 ```

<br>

### 자료 전달 객체 (DTO) && 활성레코드

자료구조다. 비즈니스 규칙 메서드 추가하지 말 것.

===

Double-dispatch : https://multifrontgarden.tistory.com/133

visitor패턴 : https://kunoo.tistory.com/entry/%ED%96%89%EC%9C%84-%ED%8C%A8%ED%84%B4-Visitor-pattern-%EB%B9%84%EC%A7%80%ED%84%B0-%ED%8C%A8%ED%84%B4
