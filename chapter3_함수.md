# ch3. 함수

- 3-1의 코드
  ```java
  //HtmlUtil.java (FitNesse 20070619)
  public static String testableHtml( PageData pageData, boolean includeSuiteSetup ) throws Exception{
    WikiPage wikipage = pageData.getWikiPage();
    StringBuffer buffer = new StringBuffer();
    if( pageData.hasAttribute("Test") ){
      if( includeSuiteSetup ){
        WikiPage suiteSetup =
          PageCrawlerImpl.getInheritedPage( suiteResponder.SUITE_SETUP_NAME, wikiPage );
        if( suiteSetup != null ){
          WikiPagePath pagePath = suiteSetup.getPageCrawler().getFullPath( suiteSetup );
          String pagePathName = PathParser.render( pagePath );
          buffer.append("!include -setup .").append(pagePathName).append("\n");
        }
      }
      WikiPage setup = PageCrawlerImpl.getInheritedPage( "Setup", wikiPage );
      if( setup != null ){
        WikiPagePath setupPath = suiteSetup.getPageCrawler().getFullPath( setup );
        String setupPathName = PathParser.render( setupPath );
        buffer.append("!include -setup .").append(pagePathName).append("\n");
      }
    }
    buffer.append( pageData.getContent() );
    if( pageData.hasAttribute("Test") ){
      WikiPage teardown = PageCrawlerImpl.getInheritedPage( "TearDown", wikiPage );
      if( teardown != null ){
        WikiPagePath tearDownPath = wikiPage.getPageCrawler().getFullPath( teardown );
        String tearDownPathName = PathParser.render( tearDownPath );
        buffer.append("\n").append("!include -teardown .")
          .append(tearDownPathName).append("\n");
      }
      if( includeSuiteSetup ){
        WikiPage suiteTeardown =
          PageCrawlerImpl.getInheritedPage( SuiteResponder.SUITE_TEARDOWN_NAME, wikiPage );
        if( suiteTeardown != null ){
          WikiPagePath pagePath = suiteSetup.getPageCrawler().getFullPath( suiteTeardown );
          String pagePathName = PathParser.render( pagePath );
          buffer.append("!include -teardown .").append(pagePathName).append("\n");
        }
      }
    }
    pageData.setContent( buffer.toString() );
    return pageData.getHtml();
  }
  ```

<br>

### 위의 코드에서의 문제점
  1. 일단 너무 김
  2. 중첩된 if문의 플래그의 용도를 모르겠음
    includeSuiteSetup, 각종 null 체크 하는 것들. <br>
  3. 추상화 수준이 다양함
    getHtml() = 고수준 <br>
    여러 if문들 ~ pagePath 얻는 작업 = 중간 수준 <br>
    buffer에 데이터 추가하는 일 = 저수준 <br>

- 3-2 리팩토링한 버전

  ```java
  public static String renderPageWithSetupsAndTeardowns( PageData pageData, boolean isSuite ) throws Exception{
    boolean isTestPage = pageData.hasAttribute("Test");
    if( isTestPage ){
      WikiPage testPage = PageData.getWikiPage();
      StringBuffer  newPageContent = new StringBuffer();
      includeSetupPages( testPage, newPageContentm isSuite );
      newPageContent.append( pageData.getContent() );
      includeTeardownPages(newPageContent.toString() );
    }
    return pageData.getHtml();
  }
  ```
<br>

### 함수 잘 만드는 법
  - 작게 만들자
    + 중복코드 삭제
      * 단순히 같은 코드뿐이 아닌 비슷한 코드도
    + 인수의 적절한 사용
      * 인수는 적을수록 좋지만, 필요할 때(직사각형의 가로,세로 등)에는 잘 써줘야함
    + Switch, if와 같은 분기문은 추상팩토리를 이용
      * OCP를 잘 지킬 수 있다

<br>

  - 추상화 수준 하나로 만들자
    + 내려가기 규칙
      * 코드가 위에서 아래로 이야기 처럼 읽혀야 좋음
    + 서술적인 이름을 사용
      - testableHtml -> SetUpTeardownIncluder.render, includeSetupAndTeardownPages, includeSetupPages 등등
      - 이름이 길어도 됨
      - 이름의 일관성 유지

<br>

  - 한가지만 하자
    + 명령과 조회를 분리하자
    + 오류코드보다 예외를 사용하자
      * 오류처리도 작업으로 취급되므로
    + 구조적 프로그래밍
      - 리턴은 하나로 하여 리턴되는 값이 명확히 보여야함.
      - 그런데 작은 함수는 오히려 여러 리턴을 갖음으로써 의도를 표현하기 쉬워질 수 있음.

<br>

  - 부수효과를 방지하자
    + 추상화할 때, 특히 조심
      * 추상화 자체가 여러일을 하나로 묶는것이기 때문
    + 한가지 일은 꼭 한가지만
      * 시간적인 결합, 순서 종속성
        -> 플래그 또는 공유되는 객체의 상태를 변경하게 되면, 특정 상태에서만 실행 가능한 함수가 실행이 불가능해 질 수 있음.( ex)db를 닫아버림, 세션을 초기화 시킴)


#### 생각정리
  ~~혼자 생각이어서 맞을지는 모름~~<br>
  함수를 만듦에 있어서 위의 함수 잘 만드는 법에서 중요도가 4 > (2 >= 3) > 1 이라고 생각이 들었습니다.<br>
  사이드이펙트가 생긴다 -> 생각지도 못한 버그 생성 -> 그러므로 제일 중요.<br>
  추상화 수준을 맞추어 한가지만 하는 함수를 만듦 -> 서로 밀접 -> 중요도 비슷하다고 봄.<br>
  추상화를 하는 것 자체가 여러일을 묶는 것이기 때문에 부수효과가 나타날 수 있음. -> 부수효과가 일어나지 않도록 잘 나눌 수 있어야함.<br>
