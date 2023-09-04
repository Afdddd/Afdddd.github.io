--- 
layout: post 
categories: [JDBC]
tags: [java,connection-pool,jdbc]
---

<br><br>

## 데이터베이스 커넥션 획득 과정

---
<br>

![Untitled](/assets/img/23-08-31(1).png)

<br>
1. 애플리케이션 로직은 DB드라이버를 통해 커넥션을 조회한다.
2. DB드라이버는 DB와 TCP/IP 커넥션을 연결한다. 이과정에서 3way handShake같은 TCP/IP같은 연결을 위한 네트워크 동작이 이루어짐.
3. 연결된 커넥션에 ID,PW와 부가 정보 전달
4. DB는 넘어온 ID,PW로 내부인증을 하고 내부에 DB 세션을 생성한다.
5. DB는 커넥션 생성완료 되었다는 응답을 보낸다.
6. DB드라이버는 커넥션 객체를 생성해 클라이언트에 반환한다.
<br>
<br>
## Connection Pool이란?

---
<br>
> 커넥션이 필요할때마다 매번 이러한 복잡하고 많은 시간이 걸리는 과정을 반복해야한다.
이러한 문제를 해결하기위해 커넥션을 미리 생성해두는 방법을 **커넥션풀**이라한다.


<br>

![Untitled](/assets/img/23-08-31(2).png)

<br>

- 애플리케이션을 시작하는 시점에 커넥션 풀은 필요한 만큼 커넥션을 미리 확보해서 풀에 보관한다.  보통 얼마나 보관할 지는 서비스의 특징과 서버 스펙에 따라 다르지만 기본값은 보통 10개이다.
- 커넥션 풀을 통해 이미 생성되어 있는 커넥션을 객체 참조로 그냥 가져다 쓰기만 하면 된다.
- 커넥션을 모두 사용하고 나면 이제는 커넥션을 종료하는 것이 아니라, 다음에 다시 사용할 수 있도록 해당 커넥션을 그대로 커넥션 풀에 반환하면 된다. 여기서 주의할 점은 커넥션을 종료하는 것이 아니라 커넥션이 살아있는 상태로 커넥션 풀에 반환해야 한다는 것이다.
- 대표적인 커넥션 풀 오픈소스는 commons-dbcp2 , tomcat-jdbc pool , **HikariCP** 등이 있다
- 스프링부트 2.0부터는 기본적으로 **HikariCP**를 제공한다.

<br>
<br>
### HikariCp 연결
---

<br>

```java
@Test
    void dataSourceConnectionPool() throws SQLException, InterruptedException {
        //커넥션 풀링
        HikariDataSource dataSource = new HikariDataSource();
        dataSource.setJdbcUrl(URL);
        dataSource.setUsername(USERNAME);
        dataSource.setPassword(PASSWORD);
        dataSource.setPoolName("MyPool");
        dataSource.setMaximumPoolSize(10);

        useDataSource(dataSource);
        Thread.sleep(1000); 
    }
}
```
<br>

`HikariDataSource`는 `DataSource`인터페이스를 구현하고 있다.

<br>

풀의 이름은 “MyPool”, 최대 사이즈는 10으로 설정했다.

<br>

커넥션 풀에서 커넥션을 생성하는 작업은 애플리케이션 실행 속도에 영향을 주지 않기 위해 별도의 쓰레드에서 작동한다. 별도의 쓰레드에서 동작하기 때문에 테스트가 먼저 종료되어 버린다. 
예제처럼 Thread.sleep 을 통해 대기 시간을 주어야 쓰레드 풀에 커넥션이 생성되는 로그를 확인할 수 있다.

<br>

> [결과]
> 
> 
> INFO hello.jdbc.Connection.ConnectionTest - connection =HikariProxyConnection@864221358 wrappingconn0:url=jdbc:h2:tcp://localhost/~/test2user=SAclass=classcom.zaxxer.hikari.pool.HikariProxyConnection // conn0연결
> 
> INFO hello.jdbc.Connection.ConnectionTest - connection =HikariProxyConnection@864221358 wrappingconn1:url=jdbc:h2:tcp://localhost/~/test2user=SAclass=classcom.zaxxer.hikari.pool.HikariProxyConnection // conn1연결
> 
> [MyPool connection adder] DEBUG com.zaxxer.hikari.pool.HikariPool - MyPool - Added connection conn0: url=jdbc:h2:tcp://localhost/~/test2 user=SA
> [MyPool connection adder] DEBUG com.zaxxer.hikari.pool.HikariPool - MyPool - Added connection conn1: url=jdbc:h2:tcp://localhost/~/test2 user=SA
> [MyPool connection adder] DEBUG com.zaxxer.hikari.pool.HikariPool - MyPool - Added connection conn2: url=jdbc:h2:tcp://localhost/~/test2 user=SA
> [MyPool connection adder] DEBUG com.zaxxer.hikari.pool.HikariPool - MyPool - Added connection conn3: url=jdbc:h2:tcp://localhost/~/test2 user=SA
> [MyPool connection adder] DEBUG com.zaxxer.hikari.pool.HikariPool - MyPool - Added connection conn4: url=jdbc:h2:tcp://localhost/~/test2 user=SA
> [MyPool connection adder] DEBUG com.zaxxer.hikari.pool.HikariPool - MyPool - Added connection conn5: url=jdbc:h2:tcp://localhost/~/test2 user=SA
> [MyPool connection adder] DEBUG com.zaxxer.hikari.pool.HikariPool - MyPool - Added connection conn6: url=jdbc:h2:tcp://localhost/~/test2 user=SA
> [MyPool connection adder] DEBUG com.zaxxer.hikari.pool.HikariPool - MyPool - Added connection conn7: url=jdbc:h2:tcp://localhost/~/test2 user=SA
> [MyPool connection adder] DEBUG com.zaxxer.hikari.pool.HikariPool - MyPool - Added connection conn8: url=jdbc:h2:tcp://localhost/~/test2 user=SA
> [MyPool connection adder] DEBUG com.zaxxer.hikari.pool.HikariPool - MyPool - Added connection conn9: url=jdbc:h2:tcp://localhost/~/test2 user=SA
> 
> //커넥션 10개 생성
> [MyPool connection adder] DEBUG com.zaxxer.hikari.pool.HikariPool - MyPool - After adding stats (total=10, active=2, idle=8, waiting=0)
> BUILD SUCCESSFUL in 4s

<br>
<br>

### **MyPool connection adder**

---

별도의 쓰레드 사용해서 커넥션 풀에 커넥션을 채우고 있는 것을 확인할 수 있다. 이 쓰레드는 커넥션 풀에 커넥션을 최대 풀 수( 10 )까지 채운다.

<br>

그렇다면 왜 별도의 쓰레드를 사용해서 커넥션 풀에 커넥션을 채우는 것일까?

<br>

커넥션 풀에 커넥션을 채우는 것은 상대적으로 오래 걸리는 일이다.
애플리케이션을 실행할 때 커넥션 풀을 채울 때 까지 마냥 대기하고 있다면 애플리케이션 실행 시간이 늦어진다. 
따라서 이렇게 별도의 쓰레드를 사용해서 커넥션 풀을 채워야 애플리케이션 실행 시간에 영향을 주지 않는다.

<br>

> [MyPool connection adder] DEBUG com.zaxxer.hikari.pool.HikariPool - MyPool - After adding stats (total=10, active=2, idle=8, waiting=0)

<br>

커넥션 풀에서 커넥션을 획득하고 그 결과를 출력했다. 
여기서는 커넥션 풀에서 커넥션을 2개 획득하고 반환하지는 않았다. 

<br>

따라서 풀에 있는 10개의 커넥션 중에 2개를 가지고 있는 상태이다.
그래서 마지막 로그를 보면 사용중인 커넥션 `active=2` , 풀에서 대기 상태인 커넥션 `idle=8` 을 확인할 수 있다.
