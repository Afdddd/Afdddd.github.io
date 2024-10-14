--- 
layout: post 
categories: [Algorithm]
tags: [Algorithm,Hash]
---



# Hash


Hash란  (**Key + Value)**를 한쌍으로 하고 데이터를 효율적으로 저장하고 빠르게 찾기위한 자료구조이다.

일반 배열과 Hash의 차이를 알아보고 HashMap의 원리에 대해 설명하겠다.

<br>

### 배열에서 데이터 찾기

일반 배열에서 데이터를 찾기 위해선 선형 구조로 모든 배열의 값을 조회해서 해당 값을 찾는 구조이다.

<br>

❓선형 구조란? **자료를 구성하는 원소들을 하나씩 순차적으로 나열시킨 형태**

<br>

```java
String[] arr = {"김","이", "박"};

for(int i=0; i<arr.length; i++){
	if(arr[i].equals("김")){
		return i;
	}
}
```

이렇게 길이가 작은 배열이라면 값을 찾는데 오래걸리지 않지만, 만약 배열의 길이가 100,000이상이라면 for문을 100,000이상 반복해야 한다.

배열의 길이가 길어질수록 값을 찾는데 시간이 오래 걸릴 것이다.

<br>

![Untitled](/assets/img/hash/hash_1.png)

입력값이 증가함에 따라 시간 또한 같은 비율로 증가하는것을 **선형 복잡도(linear complexity)**라 한다.

<br>
<br>

### Hash에서 데이터 찾기

```java
HashMap hash = new HashMap<String,Integer>();
hash.put("김",1);
hash.put("이",2);
hash.put("박",3);
return hash.get("김");
```

Hash는 put(key, value)을 통해 값을 저장하고 get(key)를 통해 값을 가져온다.

일반 배열의 특정 인덱스 값을 지정해 데이터를 찾는 방식과 같다.

<br>

```java
hash.get(”김”) == arr[0];
```

Hash는 데이터를 찾을 때 모든 값을 조회하지않고 key값을 통해 value 값을 꺼내온다. Hash에 아무리 많은 데이터가 있어도 특정 Key값만 주어지면 즉시 값을 꺼내 올 수있다.

<br>

![Untitled](/assets/img/hash/hash_2.png)

이렇게 입력값이 증가하더라도 시간이 늘어나지 않는것을 **일정한** **복잡도(constant complexity)**라한다. O(상수)

Hash가 어떻게 값을 빨리 불러오고 저장하는지 알아보자

<br>
<br>

### HashMap의 동작원리

HashMap은 배열(Bucket)과 해시 함수(Hash Function)로 구현되어있다.
<br>

![Untitled](/assets/img/hash/hash_3.png)
<br>

❓Hash Fucnction이란?

임의의 데이터를 정수로 변환하는 함수, 여기서 리턴된 정수를 Hash라 한다. Object의 `hashCode()`를 사용한다.

<br>

버킷(배열)의 크기를 capacity(용량)라 부른다. 해시 함수를 통해 반환된 정수 Hash를 이 capacity로 모듈러 연산을 수행한다. 모듈러 연산의 결과값으로 버킷의 인덱스에 접근한다.

배열의 Index를 사용해 다이렉트로 값을 꺼내 오기 때문에 조회 성능이 뛰어나다.

capacity의 3/4 이상 데이터가 존재하면 capacity의 사이즈를 늘려준다. 2배씩 늘어난다.

<br>

❓모듈러 연산이란?

% 나머지 연산을 말한다.

<br>

❓왜 모듈러 연산을 수행할까?

버킷(배열)은 한정적이다. 자바는 HashMap을 생성할때 버킷의 초기 크기를 16으로 설정한다.

즉 배열의 인덱스가 16을 넘어서면 안된다. 따라서 모듈러 연산을 하면 무조건 16보다 작은 수가 나오기 때문에 올바르게 인덱스값을 지정할수있도록 도와준다.

<br>
<br>

이제 실제로 내부에서 어떻게 값을 저장하고 꺼내오는지 보자

사진에서 버킷의 크기는 16이다.

**put**

![Untitled](/assets/img/hash/hash_4.png)

1. put()으로 키와 값을 저장한다.
2. key값을 Hash Function에 넣어 Hash값 35를 리턴받는다.
3. 리턴받은 Hash(35)를 버킷의 capacity(16)로 모듈러 연산을 한다. 35%16 = 3
4. 연산된 값 3을 버킷의 인덱스 3번에 접근해 값을 저장한다.


<br>

**get**

![Untitled](/assets/img/hash/hash_5.png)

1. get(key)으로 값을 꺼내오려 한다.
2. key값을 Hash Function에 넣어 Hash값 35를 리턴받는다.
3. 리턴받은 Hash를 버킷의 capacity로 모듈러 연산을 한다.
4. 연산된 값 3을 인덱스의 3번 버킷에 접근해 값을 꺼내온다.


<br>

여기서 발생할 수 있는 문제를 생각해보자

모듈러 연산을 통해 나온 연산값이 중복값이 나오거나 Hahs값이 중복이면 어떻게 될까?

이 문제를 해시 충돌이라 한다.(Hash Collision)

<br>

✅해시충돌이 발생할수있는 경우

- key는 다른데 Hash가 같을 때
- key도 Hash도 다른데 모듈러 연산을 통해 받은 인덱스가 같을 떄

<br>

![Untitled](/assets/img/hash/hash_6.png)

“홍길동” 이라는 키를 이용해 값을 꺼내오려고한다. 해시 함수를 이용해 해시값을 구하고 모듈러 연산을 수행 해 인덱스 값(3)을 구한다.  하지만 3번 방에는 “홍길동”의 값이 아닌 이전에 저장한 “김인엽”의 값이 들어가 있다. 이럴경우 “김인엽”의 값을 꺼낼까?

<br>

사실 버킷에는 해당 값에 대한 키값도 같이 들어가있다. 뿐만 아니라 해시 함수를 통해 얻은 Hash값도 함께 들어가 있다. 즉 버킷에는 3개의 값이 저장되는것이다.(키,값,Hash)

<br>

❓Hash값이 들어있는 이유? 해쉬함수를 한번만 수행하고 저장해두면 버킷의 사이즈를 늘려주는 과정에서 다시 연산을 하지 않아도 된다.

<br>

![Untitled](/assets/img/hash/hash_7.png)

다시 한번 보자. “홍길동” 이라는 키로 값을 찾으려 할때 이미 해당 인덱스에는 “김인엽”이라는 키에대한 값들이 저장되어있다. 이럴 경우 “홍길동”과 버킷에 저장된 “김인엽”을  `equals()` 로 서로 같은 키인지 비교해 맞으면 값을 반환하고 틀리면 해당 키에 대한 값이 없는걸로 판단되어 탐색을 마친다. 결국 “홍길동”에 대한 값이 없으므로 연산은 종료된다. 값을 꺼낼때 발생할 수 있는 Hash Collison을 알아 보았다.

<br>

이번에는 저장할때 발생하는 Hash Collision에 대해 알아 보자.

<br>

![Untitled](/assets/img/hash/hash_8.png)

정상적으로  인덱스 값을 구해서 버킷에 저장하려 했지만 해당 인덱스는 이미 “김인엽”이라는 키의 값이 저장되어있다. 이 경우에는 어떻게 될까?

<br>

바로 해당 인덱스에 liked list를 생성해 값을 저장한다. 이 방식을 **Separate Chining**이라 한다.

![Untitled](/assets/img/hash/hash_9.png)

<br>

계속 해시 출동이 발생하면 엔트리를 계속해서 생성해 저장한다.

![Untitled](/assets/img/hash/hash_10.png)

<br>

값을 꺼낼때는 linked list를 순회 하며 `equasl()` 를 실행해 같은 키의 값을 찾아 반환한다.

![Untitled](/assets/img/hash/hash_11.png)

<br>

❓엔트리란? HashMap에서 키-값 쌍을 저장하는 단위

<br>
<br>

### **HashMap에서의 equasl()와 hashCode()**

<br>

위에 설명한 HashMap은 두가지를 성립해야한다.
<br>

1. 두 객체가 `equals()` 메서드에 의해 동일하다고 판명되면, 두 객체의 `hashCode()` 메서드는 동일한 해시 코드를 반환해야 한다.
2. `hashCode()` 메서드는 항상 동일한 객체에 대해 동일한 해시 코드를 반환해야 한다.

<br>

하지만 Java에서 `equals()`는 객체의 메모리 주소값을 비교한다. 따라서 물리적으로 같은지가 아니라 논리적으로 같은 객체임을 확인하려면 재정의(Overrride)를 해줘야한다.

<br>

만약 hashCode()를 재정의 하지않으면 어떻게 될까?

![Untitled](/assets/img/hash/hash_12.png)

3번 인덱스에 저장된 “김인엽” 키와 `new` 키워드를 붙여 새로 생성한 “김인엽”이라는 키는 서로 다른 메모리 주소를 가진 다른 객체이다. 따라서 `hashCode()` 함수를 실행시키면 다른 Hash가 나오기 때문에 맞지 않는 인덱스에서 값을 찾을것이다. 결국에는 값을 찾지 못해 연산을 종료시킬것이다.

<br>

그럼 만약 equasl()를 재정의하지 않으면 어떻게 될까?

![Untitled](/assets/img/hash/hash_13.png)

해시 충돌이 일어나 같은 키값을 가진 엔트리를 찾을수 없을 것이다. 재정의 하지 않은 equals()는 객체 주소를 비교하니까 `“김철수”.equlas(”김철수”)`는 false를 출력해 값을 찾지 못하고 연산은 종료될것이다.

<br>
<br>

### Hash 알고리즘 구현해보기

Key와 Value를 갖는 Node 클래스

```java
public class Node {
    String key;
    String value;

    public Node(String key, String value) {
        this.key = key;
        this.value = value;
    }

    public String getKey() {
        return key;
    }

    public void setValue(String value) {
        this.value = value;
    }

    public String getValue() {
        return value;
    }
}
```

```java
import java.util.LinkedList;

public class HashTable {
    // 데이터를 저장할 리스트
    LinkedList<Node>[] data;

    // 테이블 생성 시 크기 지정
    public HashTable(int size) {
        this.data = new LinkedList[size];
    }

    /**
     * @param key
     * @return ASCII 코드로 변환된 HashCode 반환
     */
    int getHashCode(String key){
        int hashcode = 0;
        // ASCII 코드로 변환
        for(char c : key.toCharArray()){
            hashcode += c;
        }
        return hashcode;
    }

    /**
     * @param hashcode
     * @return 배열의 Index값 할당
     */
    int convertToIndex(int hashcode){
        return hashcode % data.length;
    }

    /**
     *
     * @param list
     * @param key
     * @return index로 배열의 방을 찾은 이후 방에 여러개의 Node를 가지고 있을 경우 list에서 key로 해당 노드 찾는 함수
     */
    Node searchKey(LinkedList<Node> list, String key){
        if(list==null) return null;
        for(Node node : list){
            if(node.getKey().equals(key)){
                return node;
            }
        }
        return null;
    }

    void put(String key, String value){
        int hashCode = getHashCode(key);
        int index = convertToIndex(hashCode);
        LinkedList<Node> list = data[index]; // index에 해당하는 리스트 가져오기
        if (list == null) { // 해당 리스트가 null이라면
            list = new LinkedList<Node>();  // 새로운 list를 만든다.
            data[index] = list; // 해당 index에 list를 넣어준다.
        }
        Node node = searchKey(list,key); // list에서 node 찾기
        if (node == null) { // 만약 해당 node가 없다면 새로 생성해 넣어준다.
            list.addLast(new Node(key,value));
        }else{ // 기존의 value에 덮어 씌우기
            node.setValue(value);
        }
    }

    String get(String key){
        int hashCode = getHashCode(key);
        int index = convertToIndex(hashCode);
        LinkedList<Node> list = data[index];

        Node node = searchKey(list, key);
        if(node == null){
            return "Not Found";
        }
        return node.getValue();
    }

}
```

```java
import java.util.Hashtable;

public class main {
    public static void main(String[] args) {
        Hashtable hashtable = new Hashtable(3);
        hashtable.put("kim", "25살 남");
        hashtable.put("park", "22살 여");
        hashtable.put("lee", "20살 남");
				hashtable.put("lee", "30살 여");
        System.out.println(hashtable.get("kim"));
        System.out.println(hashtable.get("park"));
        System.out.println(hashtable.get("lee"));
    }
}
```

![Untitled](/assets/img/hash/hash_14.png)
