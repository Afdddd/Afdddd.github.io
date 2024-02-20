--- 
layout: post 
categories: [Algorithm]
tags: [Algorithm,Hash]
---


# Hash

Hash란  (**Key + Value)**를 한쌍으로 하고 데이터를 효율적으로 저장하고 빠르게 찾기위한 자료구조이다.

이번 포스트에선 일반 배열과 Hash의 차이를 알아보고 간단한 예제로 Hash 알고리즘을 구현한다.

### 배열에서 데이터 찾기

일반 배열에서 데이터를 찾기 위해선 선형 구조로 모든 배열의 값을 조회해서 해당 값을 찾는 구조이다.

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

![Untitled](Hash%20f689f09cd5a6410e86194a706e3dabc2/Untitled.png)

입력값이 증가함에 따라 시간 또한 같은 비율로 증가하는것을 **선형 복잡도(linear complexity)**라 한다.

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

```java
hash.get(”김”) == arr[0];
```

Hash는 데이터를 찾을 때 모든 값을 조회하지않고 key값을 통해 value 값을 꺼내온다. Hash에 아무리 많은 데이터가 있어도 특정 Key값만 주어지면 즉시 값을 꺼내 올 수있다.

![Untitled](Hash%20f689f09cd5a6410e86194a706e3dabc2/Untitled%201.png)

이렇게 입력값이 증가하더라도 시간이 늘어나지 않는것을 일정한 **복잡도(constant complexity)**라한다.

Hash가 어떻게 값을 빨리 불러오는지 알아보자

### HashTable

![Untitled](Hash%20f689f09cd5a6410e86194a706e3dabc2/Untitled%202.png)

HashTable이라는 알고리즘을 통해 값을 저장하고 꺼내온다.

```
Hash Function(key) --> HashCode --> Index --> Value
```

1. 검색하고자 하는 key를 Hash 함수 실행한다.
2. Hash 함수를 통해  HashCode를 반환받는다.
3. 해당 HashCode를 배열의 Index로 환산한다.
4. 해당 Index의 값에 value를 찾아온다.

즉 HashCode가 배열의 Index로 사용되는 것이기 때문에 다이렉트로 값을 꺼내올 수 있는 것이다.

```java
arr[key]
```

만약에 중복된 HashCode를 받아 하나의 방안에 여러개의 value가 들어갈 경우가 생긴다. 이런 경를 Collision 충돌이라 한다.

결국 방에 맞는 value 값을 꺼내려면 방안에  모든 value를 순회해 알맞는 값을 꺼내와야 하기 때문에 Hash에서도 선형 복잡도가 발생할 수 있다.

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

![Untitled](Hash%20f689f09cd5a6410e86194a706e3dabc2/Untitled%203.png)
