# ch4. 주석

> 우리는 **실패**를 만회하기 위해 주석을 사용한다.  
> 주석 없이는 자신을 표현할 방법을 찾지 못해 할 수 없이 주석을 사용한다.
>
> 잘 달린 주석은 그 어떤 정보보다 유용할 수 있지만, 경솔하고 근거 없는 주석은 코드를 이해하기 어렵게  
> 만든다. 오래되고 조잡한 주석은 거짓과 잘못된 정보를 퍼뜨릴 수 있다.
>
> **주석이 필요한 상황에 처하면 한번 더 생각해보자. 코드로 의도를 표현할 방법이 정말 없는지.**

<br>

### 주석은 나쁜 코드를 보완하지 못한다.

주석을 추가하는 이유는 코드 품질이 나쁘기 때문이다.

표현력이 풍부하고 깔끔하며 주석이 거의 없는 코드가, 복잡하고 어수선하며 주석이 많이 달린 코드보다 훨씬 좋다.

코드를 짬 => 엉망이고 알아보기 어려움 => 모듈이 지저분하다는 사실을 자각

~~=> 주석을 달아놔야겠다고 결심~~      **=> 코드를 정리해야한다!**

<br>

### 코드로 의도를 표현하라.

몇 초만 더 생각하여 주석의 설명을 함수로 만들어 표현해도 충분할 수 있다.

```java
// Bad
/// 직원에게 복지 혜택을 받을 자격이 있는지 검사한다.
if (employee.flags && HOURLY_FLAG) && (employee.age > 65) {	}
  
// Better
if employee.isEligibleForFullBenefits() {	}
```



<br>

### 좋은 주석

- 법적인 주석

  ```java
  // 저작권, 소유권 정보를 소스파일 첫머리에 기재
  // Copyright (C) 2003, 2004, 2005 by Object Montor, Inc. All right reserved.
  // GNU General Public License
  ```

- 정보 제공 주석

  ```javascript
  // 테스트 중인 Responder 인스턴스를 반환
  protected abstract Responder responderInstance();
  
  // Better
  func responderBeingTested() 
  ```

  하지만 가능하다면 함수 이름 또는 클래스를 만들어 정보를 담는 편이 더 좋다.

- 의도 설명 주석

  저자의 의도를, 왜 이렇게 짰는지 드러내준다. (가장 옳은 방법이 아닐지라도)

  ```java
  // 스레드를 대량 생성하는 방법으로 어떻게든 경쟁 조건을 만들려 시도한다.
  for (int i = 0; i > 2500; i++) {
      WidgetBuilderThread widgetBuilderThread =
          new WidgetBuilderThread(widgetBuilder, text, parent, failFlag);
      Thread thread = new Thread(widgetBuilderThread);
      thread.start();
  }
  ```

- 결과를 경고하는 주석

  ```java
  // 특정 테스트 케이스를 꺼야하는 이유 설명
  // 여유 시간이 충분하지 않다면 실행하지 마십시오.
  public void _testWithReallyBigFile() {	}
  
  
  public static SimpleDataFormat makeStandardHttpDateFormat() {
    // SimpleDataFormat은 스레드에 안전하지 못하다.
    // 따라서 각 인스턴스를 독립적으로 생성해야 한다.
    SimpleDateFormat df = new SimpleDateFormat("EEE, dd MMM yyyy HH:mm");
    df.setTimeZone(TimeZone.getTimeZone("GMT"));
    return df;
  }
  // => 다른 개발자가 정적 초기화 함수를 사용하려다 해당 주석을 보고 실수를 면할 수 있다.
  ```

- TODO 주석

  ```java
  // TODO-MdM 현재 필요하지 않다.
  // 체크아웃 모델을 도입하면 함수가 필요 없다.
  protected VersionInfo makeVersion() throws Exception {
      return null;
  }
  ```

  누군가에게 문제를 봐달라는 요청, 더 좋은 이름을 떠올려달라는 부탁, 앞으로 발생할 이벤트에 맞춰 코드를 고치라는

  주의 등에 적절히 사용

  But. 남발되지않도록 주기적으로 TODO주석을 점검해 최신화 시켜 놓는것이 바람직하다.

- 중요성 강조 주석

  대수롭지 않다고 여겨질 무언가의 중요성을 강조

  ```java
  String listItemContent = match.group(3).trim();
  // 여기서 trim은 정말 중요하다. trim 함수는 문자열에서 시작 공백을 제거한다.
  // 문자열에 시작 공백이 있으면 다른 문자열로 인식되기 때문이다.
  new ListItemWidget(this, listItemContent, this.level + 1);
  return buildList(text.substring(match.end()));
  ```

- 공개 API에서는 Javadocs 사용 추천

<br>

### 나쁜 주석

대다수의 주석이 이 범주에 속한다. 허술한 코드를 지탱하거나, 엉성한 코드를 변명하거나, 미숙한 결정을 합리화하는 등

**개발자의 주절주절 독백...**에서 크게 벗어나지못한다.

- 주절거리는 주석

  주석의 답을 알기 위해 다른 코드, 다른 모듈까지 다 뒤져봐야하는 주석은 소통이 안되는 주석이다.

  주석을 달기로 결정했다면 충분한 시간을 들여 최고의 주석을 달도록 노력해야한다.

- 같은 이야기를 중복하는 주석

- 오해할 여지가 있는 주석

- 있으나 마나 한 주석

  개발자가 주석을 무시하게 되는 습관을 만든다. 코드를 읽으며 자동으로 주석을 건너뛰게 만들 수 있다.

- 함수나 변수로 표현할 수 있다면 주석을 달지마라

  ```java
  // 전역 목록 <smodule>에 속하는 모듈이 우리가 속한 하위 시스템에 의존하는가?
  if (module.getDependSubsystems().contains(subSysMod.getSubSystem())) { }
    
  // Better
  ArrayList moduleDependencies = smodule.getDependSubSystems();
  String ourSubSystem = subSysMod.getSubSystem();
  if (moduleDependees.contains(ourSubSystem)) { }
  ```

- 전역 정보

  시스템의 전반적인 정보를 기술하지마라. 해당 시스템의 코드가 변해도 아래 주석이 변하리라는 보장이 없다.

  ```java
  /**
   * 적합성 테스트가 동작하는 포트: 기본값은 <b>8082</b>.
   *
   * @param fitnessePort
   */
  public void setFitnessePort(int fitnessePort) {
      this.fitnewssePort = fitnessePort;
  }
  ```

- 너무 많은 정보

- 모호한 정보, 관계