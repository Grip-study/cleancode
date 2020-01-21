# ch.8 경계

시스템에 들어가는 모든 소프트웨어를 직접개발하는 경우는 드뭄.

오픈소스 or 사내 컴포넌트 이용.

이 외부 코드들을 우리코드에 깔끔하게 통합해야만 한다.

## 외부코드 사용하기

인터페이스 제공자는 최대한 많은곳에 사용될 수 있게 만들려함.

인터페이스 사용자는 자신의 요구에 맞는 인터페이스를 바람.

이로 인해 시스템 경계에서 문제가 생실 소지가 많음.

java.util.Map이 제공하는 메서드들

> clear() - void - Map 
>
> put(Object key, Object value) Object - Map 
>
> containtsKey(Object key) boolean - Map 
>
> equals(Object o) boolean - Map 
>
> get(Object key) Object - Map 
>
> entrySet() Set - Map 
>
> ....

나는 Map에 clear()가 사용되지 않기를 바랬는데 Map 인스턴스를 다른곳에다 던져줬는데 그곳에서 clear를 해버리는 일이 발생 할 수 있음.

Map에 데이터를 저장할 때, 특정 객체유형을 제한하지 않는다.

```java
Map sensors = new HashMap();
  ...
Sensor s = (Sensor)sensors.get(sensorId);
```

데이터를 꺼낼 때, 직접 캐스팅을 하게 됨. 

즉, Map이 반환하는 객체를 올바른 유형으로 변환할 책임이 사용자에게 있음.

<br>

```java
Map<String, Sensor> sensors = new HashMap();
  ...
Sensor s = sensors.get(sensorId);
```

제네릭을 사용하면 코드 가독성과 강제 캐스팅으로 인한 Exception을 피할 수 있게 됨.

하지만 이 방법도 사용자에게 필요하지 않은 기능까지 제공한다는 문제를 해결하지 못함.

그리고 많은곳에서 Map<String, Sensor> 인스턴스를 여기저기로 넘기고 인터페이스가 변경되게 되면 수정할 코드가 상당히 많아지게 됨.

<br>

```java
public class Sensors {
  private Map sensors = new HashMap();
  
  public Sensor getById(String id){
    return (Sensor) sensors.get(id);
  }
  ...
}
```

Map을 Sensors 안에 집어 놓고, 원하는 기능을 수행하는 메서드를 만들어 밖으로 노출시켜 위의 문제를 해결할 수 있음.

또한 Map 인터페이스가 바뀌더라도 Sensors 클래스만 변경하면 되기 때문에 좋음.

<br>

## 경계 살피고 익히기

외부 코드를 사용하면 적은 시간에 더 많은 기능 출시하기 쉬움.

하지만 외부 코드를 익히기는 어려움.

**학습테스트**를 통해 API의 변경에 따른 디버깅과 학습을 동시에 하자!

### ~~log4j 알아서 익히기~~

~~단위 테스트 기대합니다~~

### 학습 테스트는 공짜 이상이다

API를 배우는 시간을 학습테스트에 투자하면 배우면서 API 변경에 따른 버그 색출 가능!

<br>

## 아직 존재하지 않는 코드를 사용하기

API나 컴포넌트가 개발되지 않았지만, 대충 내가 해야할 일은 뭔지는 알고 있음.

-> API에 대응하는 인터페이스를 정의하고 구현을 해주면 됨.

### 어뎁터패턴

```java
public interface 실제공자{    // API로 부터 얻을 것으로 예상되는 것
  public 짬짜면 get짬짜면(); 
  public 라면 get라면(); 
}

public class Main{
  실제공자 m실 = TODO() // or 주입
  
  public void main(String[] args){
    짬짜면 짬짜면 = m실.get짬짜면();
    라면 라면 = m실.get라면();
  }
}

// 오랜시간 후 

public interface 찐제공자{    //근데 진짜로 준거는 요거
  public 짬뽕 get짬뽕();
  public 짜장 get짜장();
  public 라면 get싱거운라면();
}

//얘가 어뎁터의 역할을 수행함.
public class 찐to실 implements 실제공자{
  private 찐제공자 m찐 = 주입받음;
  
  public 짬짜면 get짬짜면(){
    짬뽕 v짬뽕 = m찐.get짬뽕();
    짜장 v짜장 = m찐.get짬뽕();
    
    짬짜면 v짬짜면 = new 짬짜면();
    v짬짜면.add짜장(v짜장.get짜장소스());
    v짬짜면.add짬뽕(v짜장.get짬뽕소스());
    ...
    
    return v짬짜면
  }
  
  public 라면 get라면(){
    라면 v라면 = m찐.싱거운라면();
    v라면.add소금(10);
    return v라면;
  }
}
```

찐제공자가 나중에 제공되었지만 main메서드의 내용이 바뀔 필요가 없으며,

추가로 다른 제공자가 나타나도, 그에따른 실제공자 구현체(어뎁터를) 만들어 주어 주입해주기만 하면 됨.

------

어뎁터 패턴 : https://jusungpark.tistory.com/22
