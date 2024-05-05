--- 
layout: post 
categories: [Java]
tags: [java]
---

---

면접에서 ***“try-with-resource에대해 설명해주세요”*** 라는 질문을 받은적이 있다.

당시에는 try-with-resource에 대해 이름만 들어봤지 어떻게 사용하는건지 몰랐기에 대답을 못했었다.

그래서 정리해본다.
<br><br><br>



## try with resource?

---
<br>

한줄로 설명하면  ***try에 선언된 객체의 자원을 자동으로 반납해주는 기능*** 이다.

자바7 버전 이후부터 사용할수있다.
<br>

`AutoCloseable`이라는 인터페이스를 구현해서 close라는 메서드를 가진 객체를 try문에서 선언하면 자동으로 리소스를 반납해준다.

<br>

```java
try(AutoCloseable를 구현한 객체 선언;){
	// 예외가 발생할수있는 코드

}catch(Exception e){
  // 적절한 예외 처리
}
```

<br><br><br>

## try-catch-finally VS try-with-resource

---

<br>

기존에 사용하던 try-catch-finally와 비교해보며 왜 try-with-resource를 사용하는지 알아보자.

<br>

### 1. 코드의 가독성

---

<br>

JDBC를 사용하여 데이터베이스에서 member_id를 통해 회원의 정보를 가져오는 코드를 두가지로 나눠서 작성해보았다.

<br>

- try-catch-finally

```java
public Member findById(String memberId) {
    String sql = "SELECT * FROM member WHERE member_id = ?";
    Connection con = null;
    PreparedStatement pstmt = null;
    ResultSet rs = null;

    try {
        con = getConnection();
        pstmt = con.prepareStatement(sql);
        pstmt.setString(1, memberId);
        
        rs = pstmt.executeQuery();
        
        if (rs.next()) {
            Member member = new Member();
            member.setId(rs.getString("member_id"));
            member.setName(rs.getString("name"));
            return member;
        }
    } catch (SQLException e) {
        // 적절한 예외 처리
        e.printStackTrace(); 
    } finally {
        // 리소스 반납 부분
        if (rs != null) {
            try {
                rs.close();
            } catch (SQLException e) {
                e.printStackTrace();
            }
        }
        if (pstmt != null) {
            try {
                pstmt.close();
            } catch (SQLException e) {
                e.printStackTrace();
            }
        }
        if (con != null) {
            try {
                con.close();
            } catch (SQLException e) {
                e.printStackTrace();
            }
        }
    }
    
    return null; // 찾지 못했다면 null 반환
}

```
<br>

리소스 반납을 직접해야하므로 코드가 길어지고 가독성이 떨어진다.

<br><br>

- try-with-resource

```java
public Member findById(String memberId) {
    String sql = "SELECT * FROM member WHERE member_id = ?";

    // try-with-resources 구문을 사용하여, 자원을 자동으로 관리.
    // Connection, PreparedStatement, ResultSet 모두 AutoCloseable 인터페이스를 구현
    try (Connection con = getConnection();
         PreparedStatement pstmt = con.prepareStatement(sql)) {
         
        // 쿼리문 완성
        pstmt.setString(1, memberId);
        
        // 쿼리문 실행
        try (ResultSet rs = pstmt.executeQuery()) {
            if (rs.next()) {
                Member member = new Member();
                member.setId(rs.getString("member_id"));
                member.setName(rs.getString("name"));
                return member;
            }
        }
    } catch (SQLException e) {
        // 적절한 예외 처리
        e.printStackTrace();  
    }

    return null;  // 찾지 못했을 경우 null 반환
}

```

<br>

자동으로 리소스를 반납해주어서 보다 코드가 간결해진다.

Connection, PreparedStatement, ResultSet 모두 AutoCloseable 인터페이스를 구현했기때문에 try-with-resource를 통해 자원을 관리할수있다.

<br><br><br>

### 2. 예외 출력

---
<br>

예외를 처리하는 과정에서 리소를 반납할 때 예외가 발생하면 어떻게 될까?

<br>

`AutoCloseable`를 구현한 `MyResource`클래스이다.

```java
public class MyResource implements AutoCloseable {

    public void doSomeThing() {
        System.out.println("print SomeTing");
        throw new FirstException(); // 예외 던짐
    }
    @Override
    public void close() {
        System.out.println("Close My Resource");
        throw new SecondException(); // 예외 던짐
    }
}
```

<br>

우선 try-catch-finally일 경우를 확인해보자

<br>

```java
public class AppRunner {
    public static void main(String[] args) {
   
        MyResource myResource = new MyResource();
        try {
            myResource.doSomeThing(); // 1.FirstException 발생
        } finally {
            myResource.close(); // 2. SecondException 발생
        }
        
    }
}

```

<br>

![Untitled](/assets/img/24-05-02-img/result(2).png)

<br>

FirstException도 발생했지만  close()에서 발생된 SecondExeption만 화면에 보인다.


<br><br>

그렇다면 try-with-resource는 어떨까?
<br>

```java
public class AppRunner {
    public static void main(String[] args) {
    
        try(MyResource myResource = new MyResource()) {
            myResource.doSomeThing();
            // myResource.close();  try-with-resource가 자동으로 실행시켜주기 떄문에 작성안해도 됨
        }
    }
}
```
<br>

![Untitled](/assets/img/24-05-02-img/result(1).png)

<br>
FirstException과 SecondExeption 모두 화면에 출력된다.

이렇게 예외 처리 과정에서 하나의 예외가 발생하고 처리되는 동안 다른 예외가 발생할 경우 try-catch-finally는 finally에서 발생한 예외를 화면에 보여주고 try-with-resource는 발생한 예외를 모두 보여준다.

예외가 발생할 경우 try-with-resource를 사용하면 보다 코드 디버깅이 더 편리해진다.

<br>

> Suppressed 는 억제된 예외로 억제된 예외는 이미 처리중인 다른 예외가 있을 때, 두 번째 예외가 첫 번째 예외에 '억제되어' 첨부된다는 개념  즉, 원래 발생한 예외가 처리되는 동안에 발생한 예외를 추가로 모아두는 방식
> 

<br><br><br>

## try-with-resource 주의할점

---

1 **`AutoCloseable`을 구현한 객체만 사용하자**
    
     close() 메서드가 try 블록이 종료될때 호출되기 때문이다.

<br>

2 **리소스의 선언순서에 주의하자**
    
    리소스가 순차적으로 반납되기 때문이다.

<br>    

3 **리소스의 재사용에 주의하자**
    
    try-with-resorce를 사용하면 리소스가 자동으로 반납되므로 재사용하려면 새로운 인스턴스를 생성해야한다.
    
<br>

4 **예외 처리의 복잡성**
    
    리소스가 자동으로 반납되면서 리소스 반납 과정에서 발생한 억제된 예외와 메인 예외를 모두  고려하여 예외를 처리해야한다.
