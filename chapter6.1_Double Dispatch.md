## 의존 관계
Supplier의 변화가 Client에 영향을 주는 경우 
- Supplier가 Client의 필드
- Supplier가 Client 메소드의 파라미터
- Supplier가 Client의 로컬 변수
- Supplier로 메시지를 보냄

=> 재사용 가능한 객체 지향 설계/개발이 어렵다. 
Client는 재사용이 어렵다
Client는 컴포넌트/서비스가 될 수 없다.

오브젝트 패턴은 런타임시 바뀔 수 있는, (상속 관계보다) 더 동적인 오브젝트 (의존) 관계를 다룬다.
- 생성 관련 패턴 (Creational Pattern) : 객체 인스턴스 생성을 위한 패턴으로, 클라이언트와 그 클라이언트에서 생성해야 할 객체 인스턴스 사이의 연결을 끊어주는 패턴
싱글턴, 팩토리 메소드, 추상 팩토리, 프로토타입, 빌더 패턴
- 행동 관련 패턴 (Behavioral Pattern) : 클래스와 객체들이 상호작용하는 방법 및 역할을 분담하는 방법과 관련된 패턴
스트래티지, 옵저버, 스테이트, 커맨드, 이터레이터, 템플릿 메소드, 인터프리터, 미디에이터, 역할 변경, 메멘토, 비지터
- 구조 관련 패턴 (Structural Pattern) : 클래스 및 객체들을 구성을 통해서 더 큰 구조로 만들 수 있게 해 주는 것과 관련된 패턴
데코레이터, 어댑터, 컴포지트, 퍼사드, 프록시, 브리지, 플라이웨이트


Dynamic method dispatch
Double Dispatch
Visitor Pattern
Visitor Proxy Pattern (Hibernate)

method dispatch
어떤 메소드를 실행할지 결정
# static dispatch
```
class Service {
    void run() {
        System.out.println("run()");
    }
}

public static void main(String[] args) {
    new Service().run();
}
```
컴파일 되는 시점에 컴파일러가 어떤 클래스의 메소드를 수행하는지 알고 있고 바이트 코드도 남는다.

# dynamic dispatch
```
abstract class Service {
    abstract void run();
}

class ServiceImpl extends Service {
    void run() {
        System.out.println("run()");
    }
}

public static void main(String[] args) {
    Service service = new ServiceImpl();
    service.run();
}    
```
어떤 run 메소드를 실행하는지 컴파일 시점에 모름. 추상클래스의 메소드를 호출하는 것만 알고 있음.
런타임 시점에 service에 할당된 객체가 무엇인지 확인하고 메소드를 실행함


# Double Dispatch
Dynamic Dispatch 를 두 번 하는 것


### 1. SNS의 구현체에 따라 로직이 다르지 않은 경우

```
interface Post {
    void postOn(SNS sns);
}
class Text implements Post {
    public void postOn(SNS sns) {
        // text -> sns
    }
}
class Picture implements Post {
    public void postOn(SNS sns) {
        // picture -> sns
    }
}

interface SNS {};
class Facebook implements SNS {

}
class Twitter implements SNS {

}
```
```
    public static void main(String[] args) {
        List<Post> posts = Arrays.asList(new Text(), new Picture());
        List<SNS> sns = Arrays.asList(new Facebook(), new Twitter());

        posts.forEach(p -> sns.forEach(s -> p.postOn(s)));
    }
```

SNS의 구현체에 따라 로직이 달라지는 경우 2번과 같이 코드를 작성



### 2. SNS의 구현체에 따라 로직이 다른 경우 (분기문 사용)
```
interface Post {
    void postOn(SNS sns);
}
class Text implements Post {
    public void postOn(SNS sns) {
        if(sns instanceof Facebook) {
            // text -> facebook
        } else if(sns instanceof Twitter) {
            // text -> twitter
        } else {
            throw new IllegalArgumentException();
        }
    }
}
class Picture implements Post {
    public void postOn(SNS sns) {
         if(sns instanceof Facebook) {
            // picture -> facebook
        } else if(sns instanceof Twitter) {
            // picture -> twitter
        } else {
            throw new IllegalArgumentException();
        }
    }
}

interface SNS {};
class Facebook implements SNS {

}
class Twitter implements SNS {

}
```

- SNS의 새로운 구현체가 생기면 분기문을 추가해야 함
- 만약 실수로 분기문을 추가하지 않으면 의도치 않게 exception 발생


### 3. SNS의 구현체에 따라 로직이 다른 경우 (메소드 오버로딩 사용. static dispatch)
```
interface Post {
    void postOn(SNS sns);
}
class Text implements Post {
    public void postOn(Facebook facebook) {
        // text -> facebook
    }

    public void postOn(Twitter twitter) {
        // text -> twitter
    }
}
class Picture implements Post {
    public void postOn(Facebook facebook) {
        // picture -> facebook
    }

    public void postOn(Twitter twitter) {
        // picture -> twitter
    }
}

interface SNS {};
class Facebook implements SNS {

}
class Twitter implements SNS {

}
```

이전 코드와 달리 메소드 오버로딩을 사용해 분기문을 제거함

하지만 다음 코드 중  s -> p.postOn(s) 에서 컴파일 시점에 에러 발생
```
    public static void main(String[] args) {
        List<Post> posts = Arrays.asList(new Text(), new Picture());
        List<SNS> sns = Arrays.asList(new Facebook(), new Twitter());

        posts.forEach(p -> sns.forEach(s -> p.postOn(s)));
    }
```
- 메소드 오버로딩은 static dispatch 이므로 컴파일 시점에 어떤 클래스의 메소드를 수행할지 알아야 함
- 하지만 s는 SNS라는 interface의 타입이기 때문에 어떤 구현체(Facebook, Twitter 등)의 타입인지 컴파일러가 알 수 없음

오류 발생. 실패 !!!!!!! 



## 결론. Double dispatch 사용
```
interface Post {
    void postOn(SNS sns);
}

class Text implements Post {
    public void postOn(SNS sns) {
        sns.post(this);
    }
}

class Picture implements Post {
    public void postOn(SNS sns) {
        sns.post(this);
    }
}
```
```
interface SNS {
    void post(Text text);
    void post(Picture picture);
}

class Facebook implements SNS {
    public void post(Text text) {
        // text -> facebook
    }
    public void post(Picture picture) {
        // picture -> facebook
    }
}

class Twitter implements SNS {
    public void post(Text text) {
        // text -> twitter
    }
    public void post(Picture picture) {
        // picture -> twitter
    }
}
```
```
    public static void main(String[] args) {
        List<Post> posts = Arrays.asList(new Text(), new Picture());
        List<SNS> sns = Arrays.asList(new Facebook(), new Twitter());

        posts.forEach(p -> sns.forEach(s -> p.postOn(s)));
    }
```
- 한 단계를 더 거친 것 같지만, 분기문을 사용하지 않고 dynamic dispatch를 두 번 사용
- Post 중 어떤 구현체의 postOn 메소드를 실행할지 dynamic dispatch 한 번 사용
- postOn 메소드 내부에서 SNS 중 어떤 구현체의 post 메소드를 실행할지 dynamic dispatch 한 번 사용



=> 새로운 구현체가 생기는 경우에는 다음과 같이 구현체에 대한 코드만 작성하면 됨
```
class Instagram implements SNS {
    public void post(Text text) {
        // text -> instagram
    }
    public void post(Picture picture) {
        // picture -> instagram
    }
}
```
- 구현체를 새로 추가하는 것이 자유로움 => 기존에 의존하던 코드에 직접적으로 영향을 주지 않는다.


=> 비지터 패턴으로 예를 들면, SNS가 visitor 역할을 하고 Post의 메소드 postOn()이 accept() 역할 

## Visitor Pattern

위의 코드로 예를 들면

- Post는 SNS 타입의 어떤 구현체가 들어오는지 관심 없고, postOn() 메소드 (accept) 를 제공
- SNS의 구현체(visistor)의 post() 메소드 (visit) 를 통해 실제 로직을 실행할 수 있음
- SNS의 구현체가 새로 추가되어도 Post에는 영향을 미치지 않음
- Post의 구현체가 추가된다면 ? 어쩔 수 없이 SNS에 Post의 구현체를 처리하는 메소드를 다 추가해 줘야 함. SNS의 구현체에 따라 로직이 다르지 않고 모두 공통 로직을 사용한다면 SNS를 abstract class로 작성해서 사용해도 됨.



## Visitor Proxy Pattern (hibernate)

### polimophic query

```
List<SNS> sns = repository.findSNS();
```
이 list의 요소들을 instanceof로 타입을 체크하면 실패한다. 
- JPA에서 각각의 요소들은 프록시 구조로 반환되기 때문에 타입은 SNS 타입이기 때문.
- 따라서, 이 때는 반드시 visitor 패턴을 사용해야 함.


=> 따라서, 프록시를 visitor로 해서 타입을 체크할 수 있다. 이를 일반화해서 proxy visitor pattern 이라고 함