# ch2. 의미있는 이름

### 이름을 잘 지어야하는 이유?

> 1. 코드는 작성되는 것보다 훨씬 더 많이 읽혀진다. 코드 작성의 편의성보다는 코드의 가독성에 더 중점을 두어야한다.
> 2. 좋은 이름을 지으려면 시간이 좀 더 걸리지만, 좋은 이름으로 절약되는 시간이 훨씬 더 많아진다.
> 3. 추가적인 주석이나 설명 없이 코드에 담긴 정보만으로 의미와 문맥을 이해할 수 있다.
> 4. 그릇된 이름는 프로그래머에게 잘못된 정보나 혼란을 야기시킬 수 있다.

<br>

### 그렇다면, 좋은 이름이란?

- 의도가 명확한 코드

  ```java
  // 좋지 않은 예
  public List<int[]> getThem() {
      List<int[]> list1 = new ArrayList<int[]>();
      for (int[] x : theList) {
          if (x[0] == 4) {
              list1.add(x);
          }
      }
      return list1;
  }
  ```

  ```java
  // 좋은 예
  public List<Cell> getFlaggedCells() {
      List<Cell> flaggedCells = new ArrayList<Cell>();
      for (Cell cell : gameBoard) {
          if (cell.isFlagged()) {
              flaggedCells.add(cell);
          }
      }
      return flaggedCells;
  }
  ```

  => 코드의 단순성 < **코드의 함축성** : 코드 맥락이 코드 자체에 명시적으로 드러나도록.

- 검색하기 쉬운 이름

  => 짧은 이름보다는 긴 이름이 좋다.  e.g. WORK_DAYS_PER_WEEK

- 의미있는 맥락을 추가하라

  "firstName, lastName, street, houseNumber, city, state, zip code"  => 주소와 관련된 변수들

  => **addrFirstName, addrsLastName, addrState** 와 같이쓰면 맥락이 더 분명해짐.

- 불필요한 맥락은 없애라

  ```Swift
  // 좋지 않은 예
  struct Centipede {
  	let centipedesHead: Int
    let centipedesBody: Int
    let centipedesTail: Int
  }
  
  // 좋은 예
  struct Centipede {
  	let head: Int
    let body: Int
    let tail: Int
  }
  ```

  => Centipede 라는 구조체 내부 변수들이기때문에 불필요한 이름은 생략

- 기발한 이름은 피해라

  농담이나 재치가 섞인 이름 또는 특정 문화에만 국한된 이름은 X => **명료하고 정확한 이름**

  > 똑똑한 프로그래머는 자신의 정신적 능력을 과시하고 싶어하지만,
  >
  > 전문가 프로그래머는 명료함이 최고라는 사실을 이해한다. 
  >
  > 그리고 자신의 능력을 좋은 방향으로 사용해 남들이 이해하는 코드를 내놓는다.

- 한 개념에 한 단어를 사용하라

  add라는 단어를 add(a, b) : a와 b를 더하는 함수로 사용했다면,

  tempArray.add(element) 처럼 추가해주는 개념에서의 사용은 insert나 append를 사용하여

  기존 add를 쓰던 방식과 달라지지않도록 한다.





출처

https://dalkomit.tistory.com/155



추가

https://soojin.ro/blog/english-for-developers-swift

https://soojin.ro/blog/naming-boolean-variables

https://brunch.co.kr/@goodvc78/12