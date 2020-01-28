# 단위 테스트


### TDD 법칙 세가지
1. 실패하는 단위 테스트를 작성할 때까지 실제 코드를 작성하지 않는다.
2. 컴파일은 실패하지 않으면서 실행이 실패하는 정도로만 단위 테스트를 작성한다.
3. 현재 실패하는 테스트를 통과할 정도로만 실제 코드를 작성한다.



## 깨끗한 테스트 코드 유지하기 


<b> 테스트케이스가 없으면 ? </b> <br>
수정한 코드가 제대로 돌아가는지 확인할 방법이 없음. <br>
검증할 수 없기 때문에 코드는 결함율이 높아지고, 변경을 주저하게 되고, 코드를 정리하지 않게 됨 

<b> 지저분한 테스트 코드라도 대충 빠르게 테스트케이스를 만들면 ? </b> <br>
실제 코드가 변하면 테스트 코드도 변해야 함. <br>
하지만 지저분한 테스트 코드는 관리하기 어렵기 때문에 테스트케이스 만들기를 포기하게 됨. <br> <br>

###  <b> => 테스트는 유연성, 유지보수성, 재사용성을 제공한다. </b>
- 테스트케이스가 없으면 모든 변경이 잠정적인 버그
- 테스트케이스가 있으면 코드를 변경, 개선 하더라도 테스트할 수 있기 때문에 믿고 바꿀 수 있다. 
<br>

### 테스트케이스가 있다면 ? 
- 기존 테스트케이스가 모두 통과한 상태에서 코드를 변경한 후에 테스트케이스를 실행했을 때 실패한다면
- => 변경한 코드 내에 버그가 있음
- => 실패한 테스트케이스의 목적에 맞게 버그를 쉽게 찾을 수 있음
<br> <br>


## 깨끗한 테스트 코드
###  <b>  => 가독성 </b>

1. Build. 테스트 자료를 만들고
2. Operate. 테스트 자료를 조작하고
3. Check. 조작한 결과가 올바른지 확인

<b> 도메인에 특화된 테스트 언어 </b> <br>
시스템 조작 API를 사용하는 대신 API 위에다 함수와 유틸리티를 구현한 후, 그 함수와 유틸리티를 사용 <br>
```
public void testBefore() {
    // build
    crawler.addPage(root, PathParser.parse("pageOne"));
    crawler.addPage(root, PathParser.parse("pageOne.ChildOne));
    crawler.addPage(root, PathParser.parse("pageTwo"));

    // operate
    request.setResource("root");
    request.addInput("type", "pages");
    Responder responder = new SerializedPageResponder();
    SimpleResponse response = (SimpleResponse) responser.makeResponse(new FitNesseContext(root), request);
    String xml = response.getContent();

    // check
    assertEquals("test/xml", response.getContentType());
    assertSubstring("<name>PageOne</name>", xml);
    assertSubstring("<name>PageTwo</name>", xml);
    assertSubstring("<name>ChildOne</name>", xml);
}
```

```
public void testAfter() {
    // build
    makePages("pageOne", "pageOne.child", "pageTwo");

    // operate
    submitRequest("root", "type:pages");

    // check
    assertResponseIsXML();
    assertResponseContains("<name>PageOne</name>", "<name>PageTwo</name>", "<name>ChildOne</name>");
}
```
이렇게 구현한 함수와 유틸리티가 테스트 코드에서 사용하는 특수 API가 된다. <br>
이런 테스트 API는 테스트 코드를 계속 리팩토링 하면서 진화된 API 이다.

<b> 이중 표준 </b>
- 테스트 API 코드에 적용하는 표준은 실제 코드에 적용하는 표준과 확실히 다르다.
- 단순하고, 간결하고, 표현력이 풍부해야 하지만, 실제 코드만큼 효율적일 필요는 없다.


## 테스트 당 assert 하나 
<b> given-when-then 패턴 </b>
```
public test1() {
    // given
    givenPages("pageOne", "pageOne.childOne", "pageTwo");

    // when
    whenRequestIsIssued("root", "type:pages");

    // then
    thenResponseShouldBeXML();
}

public test1() {
    // given
    givenPages("pageOne", "pageOne.childOne", "pageTwo");

    // when
    whenRequestIsIssued("root", "type:pages");

    // then
    thenResponseShouldContain("<name>PageOne</name>", "<name>PageTwo</name>", "<name>ChildOne</name>");
}
```
- Given. 테스트 수행하기 이전의 상태 (test를 위한 사전 조건)
- When. 사용자가 지정하는 동작
- Then. 지정된 동작으로 인해 예상되는 변경 사항 <br>

=> 문제. 테스트 코드를 읽기는 쉬워졌지만, 테스트를 분리하면 중복되는 코드들이 많아진다. 

=> 해결1. Template Method 패턴
given-when-then 패턴의 given/when 부분을 부모 클래스에 두고 then 부분을 자식 클래스를 두어 중복 제거 <br>

=> 해결 2. @Before 함수, @Test 함수
@Before 함수에 given/when을, @Test에 then 부분을 넣어 중복 제거

=> 하지만, 완벽하게 테스트 하나 당 assert 하나는 위와 같은 중복 문제 혹은 중복을 제거하기 위한 수고가 더 발생할 수 있음.


### <b> => 테스트 당 개념 하나 </b>
따라서, 테스트 함수마다 하나의 개념만을 테스트하도록 한다. 
<br><br><br>

## F.I.R.S.T. 원칙
깨끗한 테스트가 따르는 다섯가지 규칙

### 1. Fast 빠르게 
테스트가 빨라야 자주 돌릴 수 있다. 자주 돌리지 않으면 초반에 문제를 찾아내 고치지 못하고 코드를 마음껏 정리할 수 없다.

### 2. Independent 독립적으로
각 테스트는 서로 의존하면 안된다. 한 테스트가 다음 테스트가 실행할 환경을 준비해서는 안된다. 
각 테스트는 독립적으로, 어떤 순서로 실행해도 통과할 수 있어야 한다. <br>
테스트가 서로에게 의존하면 하나가 실패할 때 나머지도 잇달아 실패하기 때문에 원인을 진단하기 어렵다.

### 3. Repeatable 반복 가능하게
테스트는 어떤 환경에서도 반복 가능해야 한다. <br>
테스트가 돌아가지 않는 환경이 하나라도 있다면 테스트가 실패한 다른 이유가 생긴다.

### 4. Self-Validating 자가 검증하는
테스트는 부울 값으로 결과를 내야 한다. (성공 또는 실패) <br>
통과 여부를 알기 위해 로그 파일을 읽게 만들어서는 안된다.

### 5. Timely 적시에
테스트는 적시에 작성한다. 단위테스트는 테스트하려는 실제 코드를 구현하기 직전에 구현한다. 



## 결론
테스트 코드는 중요한데 간단하고 간결하게 작성해라 ~~~~