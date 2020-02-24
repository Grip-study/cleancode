# DI

의존성 주입은 광범위한 역제어 테크닉의 한 형태이다.

어떤 서비스를 호출하려는 클라이언트는 **그 서비스가 어떻게 구성되었는지 알지 못해야 한다**.

클라이언트는 대신 **서비스 제공에 대한 책임을 외부 코드(주입자)로 위임한다.** 클라이언트는 주입자 코드를 호출할 수 없다. 

주입자는 이미 존재하거나 주입자에 의해 구성되었을 서비스를 클라이언트로 주입(전달)한다. 그리고 나서 클라이언트는 서비스를 사용한다.

이는 클라이언트가 주입자와 서비스 구성 방식 또는 사용중인 실제 서비스에 대해 알 필요가 없음을 의미한다. 

클라이언트는 서비스의 사용 방식을 정의하고 있는 서비스의 고유한 인터페이스에 대해서만 알면 된다. 

이것은 "구성"의 책임으로부터 "사용"의 책임을 구분한다.

## IOC(inversion of control : 제어의 역전)

프로그램의 제어 흐름 구조가 뒤바뀌는 것.

일반적인 프로그램 흐름은 사용할 객체를 선택하고 생성하고, 그 객체의 메소드를 호출하는 방법을 사용한다.

각 오브젝트는 프로그램의 흐름을 결정하거나 사용할 오브젝트를 구성하는 작업에 능동적으로 참여한다.

IOC에서는 **자신이 사용할 객체를 스스로 선택하지 않는다.**(물론 생성도 하지 않음.)

모든 제어 권한을 다른대상으로 위임시킴.

예를 들어, 생명주기함수에 코드를 작성하면, 해당 함수는 프레임워크의 제어의 의해 실행이 되므로

제어의 역전 개념이 적용된 대표적인 것이라고 할 수 있다.

## 의존관계

A가 B를 의존하고 있다고 할 때, 의존을 하고 있다는 것은 **B가 변하면 그것이 A에 영향을 미친다는 뜻이다.**

의존관계 주입은 다음과 같은 세 가지 조건을 충족하는 작업을 말한다.

 - 클래스 모델이나 코드에는 런타임 시점의 의존관계가 드러나지 않는다. **즉 인터페이스에만 의존하고 있어야한다.**
 
 - 런타임 시점의 의존관계는 컨테이너나 팩토리 같은 제3의 존재가 결정한다.
 
 - 의존관계는 사용할 객체에 대한 레퍼런스를 외부에서 제공해줌으로써 만들어진다.
 
의존관계의 핵심은 설계지점에는 알지 못했던 두 객체의 관계를 맺도록 도와주는 제3의 존재가 있다는 것이다.

## 의존성얻는 방법.
1. DI
```java
public class UserDao {
    private IConnectionMaker connectionMaker;
    
    public void setConnectionMaker(IConnectionMaker connectionMaker){
        this.connectionMaker = connectionMaker;
    }
    ...
}

public class Main{
    public static void main(String... args){
        UserDao userDao = new UserDao();
        MyConnectionMaker connectionMaker = new MyConnectionMaker();
        
        userDao.setConnectionMaker(connectionMaker);
        ...
    }
}
```

2. DL (Dependency Lookup)

```java
public class UserDao {
    private IConnectionMaker connectionMaker;
    
    public UserDao(){
        AnnotationConfigApplicationContext context = 
                new AnnotationConfigApplicationContext(DaoFactory.class);
        this.connectionMaker = context.getBean("connectionMaker", IConnectionMaker.class)
    }
    ...
}
```

DL또한 DI의 장점을 갖고 있으며 IOC의 원칙에도 잘 들어 맞는다.

하지만 DI쪽이 훨씬 단순하고 깔끔하며, UserDao DB정보를 어떻게 가져와야 하는것에 집중을 해야하는데 DL은 팩토리를 만들고 API를
사용하는 코드가 섞여있어 어색하다. 따라서 대개는 의존관계 주입 방식을 사용하는 편이 낫다.

