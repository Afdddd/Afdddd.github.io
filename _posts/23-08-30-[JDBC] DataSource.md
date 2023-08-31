--- 
layout: post 
categories: [JDBC]
tags: [java,jdbc,datasource]
---

# [JDBC] DataSource
<br>
<br>
## DataSource란?

---
<br>
커넥션을 획득하는 방법을 추상화하는 인터페이스.
<br>
```java
public interface DataSource {
	Connection getConncetion() throws SQLException;
}
```
<br>
이전에는 Connection을 획득할 때 는 JDBC의 DriverManager 기술만을 의존해왔다.
커넥션풀을 사용하는 방법으로 변경하려면 애플리케이션 코드도 함께 변경해야한다.
그래서 커넥션을 획득하는 방법을 추상화 시켰다. 이것을 **DataSource**라 한다.

<br>
![23-08-30](/assets/img/23-08-30.png)
<br>
## DriverManager VS DataSource

---

### DriverManager

```java
@Test
    void driverManager() throws SQLException {
        Connection con1 = DriverManager.getConnection(URL, USERNAME, PASSWORD);
        Connection con2 = DriverManager.getConnection(URL, USERNAME, PASSWORD);
        log.info("connection ={} class= {}",con1,con1.getClass());
        log.info("connection ={} class= {}",con2,con2.getClass());
    }
```
<br>
> [결과]<br>
INFO hello.jdbc.Connection.ConnectionTest - connection =conn0: url=jdbc:h2:tcp://localhost/~/test2 user=SA class= class org.h2.jdbc.JdbcConnection
INFO hello.jdbc.Connection.ConnectionTest - connection =conn1: url=jdbc:h2:tcp://localhost/~/test2 user=SA class= class org.h2.jdbc.JdbcConnection
> 
<br>
<br>
### DataSource

```java
@Test
    void dataSourceDriverManager() throws SQLException {
        DriverManagerDataSource dataSource = new DriverManagerDataSource(URL, USERNAME, PASSWORD);
        useDataSource(dataSource);
    }

    private void useDataSource(DataSource dataSource) throws SQLException {
        Connection con1 = dataSource.getConnection();
        Connection con2 = dataSource.getConnection();
        log.info("connection ={} class= {}",con1,con1.getClass());
        log.info("connection ={} class= {}",con2,con2.getClass());
    }
```
<br>
`DriverManagerDataSource` 는 spring에서 `DriverManager` 도 `DataSource` 를 통해서 사용할 수 있도록 구현한 클래스이다.
<br>
> [결과] <br>
DEBUG org.springframework.jdbc.datasource.DriverManagerDataSource - Creating new JDBC DriverManager Connection to [jdbc:h2:tcp://localhost/~/test2]
DEBUG org.springframework.jdbc.datasource.DriverManagerDataSource - Creating new JDBC DriverManager Connection to [jdbc:h2:tcp://localhost/~/test2]
INFO hello.jdbc.Connection.ConnectionTest - connection =conn0: url=jdbc:h2:tcp://localhost/~/test2 user=SA class= class org.h2.jdbc.JdbcConnection
INFO hello.jdbc.Connection.ConnectionTest - connection =conn1: url=jdbc:h2:tcp://localhost/~/test2 user=SA class= class org.h2.jdbc.JdbcConnection
> 
<br>
`DriverManager`는 커넥션을 획득할 때 마다 URL , USERNAME , PASSWORD 같은 파라미터를 계속 전달해야 한다. 
반면에 `DataSource`를 사용하는 방식은 처음 객체를 생성할 때만 필요한 파리미터를 넘겨두고, 커넥션을 획득할 때는 단순히 `dataSource.getConnection()`만 호출하면 된다.

<br>
<br>
## 설정과 사용의 분리

---

**설정** : DataSource를 만들고  URL , USERNAME , PASSWORD 같은 부분을입력하는 것.
설정과 관련된 속성들을 한곳에 모아두면 향후에 변경에 더욱 유연하게 대처할 수 있다.

**분리** : 설정은 신경쓰지 않고 `DataSource`의 `getConnection()`을 사용하는것.
<br>
필요한 데이터를 `DataSource` 가 만들어지는 시점에 미리 다 넣어두게 되면, `DataSource` 를 사용하는 곳에서는 `dataSource.getConnection()` 만 호출하면 되므로, URL , USERNAME , PASSWORD 같은 속성들에 의존하지 않아도 된다. 그냥 `DataSource`만 주입받아서 `getConnection()` 만 호출하면 된다.
