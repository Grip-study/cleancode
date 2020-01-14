# 오류 처리

### 오류 코드보다 예외를 사용하라

- 기존 오류 처리 코드

: 호출한 즉시 오류를 확인해야 하기 때문에 호출자 코드가 복잡해짐

```java
public class DeviceController {
	...
	public void sendShutDown() {
		DeviceHandle handle = getHandle(DEV1);
		// 디바이스 상태를 점검한댜.
		if (handle != DeviceHandle.INVALID) {
			// 레코드 필드에 디바이스 상태를 저장한다.
			retrieveDeviceRecord(handle);
			// 디바이스가 일시정지 상태가 아니라면 종료한다.
			if (record.getStatus() != DEVICE_SUSPENDED) {
				pauseDevice(handle);
				clearDeviceWorkQueue(handle);
				closeDevice(handle);
			} else {
				logger.log("Device suspended. Unable to shut down");
			}
		} else {
			logger.log("Invalid handle for: " + DEV1.toString());
		}
	}
	...
}
```

- 오류 발견시 예외를 던지는 코드

  ( 예외란: 프로그램 실행 중에 정상적인 프로그램의 흐름에 어긋나는 이벤트 )

  논리와 오류 처리 코드를 분리하여 더 깔끔해짐

```java
public class DeviceController {
	...
	public void sendShutDown() {
		try {
			tryToShutDown();
		} catch (DeviceShutDownError e) {
			logger.log(e);
		}
	}

	private void tryToShutDown() throws DeviceShutDownError {
		DeviceHandle handle = getHandle(DEV1);
		DeviceRecord record = retrieveDeviceRecord(handle);
		pauseDevice(handle); 
		clearDeviceWorkQueue(handle); 
		closeDevice(handle);
	}

	private DeviceHandle getHandle(DeviceID id) {
		...
		throw new DeviceShutDownError("Invalid handle for: " + id.toString());
		...
	}
	...
}
```



### Try-Catch-Finally 문부터 작성하라

```java
1. 파일이 없으면 예외를 던지는 단위 테스트
@Test(expected = StorageException.class)
public void retrieveSectionShouldThrowOnInvalidFileName() {
   sectionStore.retrieveSection("invalid - file");
}

2. 단위테스트에 맞춰 코드 구현
public List<RecordedGrip> retrieveSection(String sectionName) {
// 실제로 구현할 때까지 비어 있는 더미를 반환한다.
  return new ArrayList<RecordedGrip>();
}

3.파일 접근 시도 코드 추가
public List<RecordedGrip> retrieveSection(String sectionName) {
   try {
       FileInputStream stream = new FileInputStream(sectionName);
   } catch (Exception e) {
       throw new StorageException("retrieval error", e);
   }
   return new ArrayList<RecordedGrip>();
}

4. Exception의 범위를 FileNotFoundException로 범위를 좁혀 개선
public List<RecordedGrip> retrieveSection(String sectionName) {
 try {
     FileInputStream stream = new FileInputStream(sectionName);
     stream.close();
 } catch (FileNotFoundException e) {
     throw new StorageException("retrieval error", e);
 }
 return new ArrayList<RecordedGrip>();
}
```



### 미확인(unchecked) 예외를 사용하라

확인된 예외는 OCP를 위반한다.

Ex. 대규모 시스템에서 호출이 일어나는 방식 

=> 최상위 함수가 아래 함수를 호출(단계를 내려갈수록 호출하는 함수 증가) 

=> 최하위 함수를 변경해 새로운 오류를 던져야하는 상황이 추가됨 

=> 확인된 오류를 던진다면, 변경한 함수를 호출하는 함수 모두가 catch블록으로 새 예외처리해주거나, 선언부에 throw절을 추가해야함 

=> 결과적으로 최하위부터 최상위 단계까지 연쇄적인 수정이 모두 일어남. 모든 함수가 최하위 함수에서 던지는 예외를 알아야하므로 캡슐화가 깨짐.

**따라서, 확인된 예외 뿐만아니라 미확인 예외도 확인해주어야한다.**

**확인된 예외도 유용하고, 아주 중요한 라이브러리를 작성한다면 모든 예외를 잡아줘야하지만, 일반적인 상황에서는 위에 언급된 의존성이라는 비용이 아주 크다.**

```
checked 예외: 컴파일 단계에서 확인되며 반드시 처리해야 하는 예외
IOException
SQLException

Unchecked 예외: 실행 단계에서 확인되며 명시적인 처리를 강제하지는 않는 예외
NullPointerException
IllegalArgumentException
IndexOutOfBoundException
SystemException
```



### 예외에 의미를 제공하라

발생 원인과 위치를 찾기 쉽도록 호출 스택뿐만아니라, 오류메세지에 정보를 담아 함께 던져라.

(실패한 코드, 연산, 실패 유형 등등..)



### 호출자를 고려해 예외클래스를 정의하라

가장 중요한 것은 '오류를 잡아내는 방법' 이다.

```java
// Bad
// 외부 라이브러리가 던질 예외를 모두 잡아 낸다. 
// 같은 에러 잡아 내는 코드가 많다. 
// 변경하기 어렵다.

ACMEPort port = new ACMEPort(12);
try {
  port.open();
} catch (DeviceResponseException e) {
  reportPortError(e);
  logger.log("Device response exception", e);
} catch (ATM1212UnlockedException e) {
  reportPortError(e);
  logger.log("Unlock exception", e);
} catch (GMXError e) {
  reportPortError(e);
  logger.log("Device response exception");
} finally {
  ...
}

// ============================================================

// Good
// 호출하는 라이브러리 API를 감싸면서 예외 유형 하나를 반환.
// Wrapper클래스 덕분에 의존성이 크게 감소.

LocalPort port = new LocalPort(12);
try {
  port.open();
} catch (PortDeviceFailure e) {
  reportError(e);
  logger.log(e.getMessage(), e);
} finally {
  ...
}

public class LocalPort {
  private ACMEPort innerPort;
  public LocalPort(int portNumber) {
    innerPort = new ACMEPort(portNumber);
  }

  public void open() {
    try {
      innerPort.open();
    } catch (DeviceResponseException e) {
      throw new PortDeviceFailure(e);
    } catch (ATM1212UnlockedException e) {
      throw new PortDeviceFailure(e);
    } catch (GMXError e) {
      throw new PortDeviceFailure(e);
    }
  }
  ...
}
```

LocalPort클래스 와 같이 ACMEPort를 감싸는 클래스는 매우 유용하다.

외부 API를 감싸면 외부 라이브러리와 프로그램 사이에 의존성이 크게 줄어 외부 API의 설계방식에 크게 발목 잡히지 않는다.

또한 라이브러리 교체. 변경에도 유용하고, 외부 API호출 대신 테스트 코드를 넣어 테스트하기도 더 쉬워진다.



### 정상 흐름을 정의하라

```java
// Bad
// 식비비용 조회를 실패하면 일일 기본 식비를 총계에 더한다.
try { 
  MealExpenses expenses = expenseReportDAO.getMeals(employee.getID());
  m_total += expenses.getTotal(); 
} catch(MealExpensesNotFound e) {
   m_total += getMealPerDiem(); 
}

// ============================================================

// Good
// 특수 상황을 처리할 필요가 없어 더 코드는 간결해진다.
MealExpenses expenses = expenseReportDAO.getMeals(employee.getID());
m_total += expenses.getTotal();

public class PerDiemMealExpenses implements MealExpenses {
  public int getTotal() {
    // 기본값으로 일일 기본 식비를 반환
  }
}
```





### null을 반환하지마라

null 반환은 일거리를 늘릴뿐만아니라 호출자에게 문제를 떠넘기는 행위.

null을 반환하기보다는 **예외**를 던지거나 **특수 사례 객체**를 반환하는 것이 좋다

```java
// BAD
// 호출자가 항상 null을 체크해야함
// NullPointerException 발생 위험이 생기고, null확인이 너무많아짐.
public void registerItem(Item item) { 
  if (item != null) {
    ItemRegistry registry = peristentStore.getItemRegistry();
    if (registry != null) {
      Item existing = registry.getItem(item.getID());
      if (existing.getBillingPeriod().hasRetailOwner()) {
        existing.register(item);
      }
    }
  }
}

// Not good
List<Employee> employees = getEmployees();
if (employees != null) {
  for(Employee e : employees) {
    totalPay += e.getPay();
  }
}


// Good
List<Employee> employees = getEmployees();
for(Employee e : employees) {
  totalPay += e.getPay();
}

public List<Employee> getEmployees() {
  if( .. there are no employees .. )
    return Collections.emptyList();
  }
}

```



### null을 전달하지마라

일반적으로 대다수의 프로그래밍 언어들은 파라미터로 들어온 null에 대해 적절한 방법을 제공하지 못한다. (???? 그런가?)

```java
// Bad
// Parameter에 null값이 들어오면 NullPointerException 발생.
public class MetricsCalculator { 
  public double xProjection(Point p1, Point p2) { 
    return (p2.x – p1.x) * 1.5; 
  } 
}

// Better
public class MetricsCalculator { 
  public double xProjection(Point p1, Point p2) { 
    if(p1 == null || p2 == null){
      throw InvalidArgumentException("Invalid argument for MetricsCalculator.xProjection"); 
    } 
    return (p2.x – p1.x) * 1.5; 
  } 
}

// Good
// assert문 사용
public class MetricsCalculator { 
  public double xProjection(Point p1, Point p2) { 
    assert p1 != null : "p1 should not be null";
    assert p2 != null : "p2 should not be null";
    return (p2.x – p1.x) * 1.5; 
  } 
}
```





추가 정보 및 출처

[http://amazingguni.github.io/blog/2016/05/Clean-Code-7-%EC%98%A4%EB%A5%98-%EC%B2%98%EB%A6%AC](http://amazingguni.github.io/blog/2016/05/Clean-Code-7-오류-처리)

https://nesoy.github.io/articles/2018-02/CleanCode-ErrorHandle