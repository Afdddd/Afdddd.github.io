--- 
layout: post 
title:  "[Java]오픈 API 파싱하기(XML)"
categories: [Java]
---

# [Java]오픈 API 파싱하기(XML)



공공데이터 포털을 활용해 특정 버스의 도착 정보를 조회해보았다.

공공데이터 신청부터 원하는 데이터를 파싱하는 것 까지 알아보자


## 오픈API 활용신청

![Untitled](/assets/image/Untitled.png)
우선 공공데이터포탈([https://www.data.go.kr/index.do](https://www.data.go.kr/index.do))에서 데이터 활용신청을 해야한다.

허가는 1~2일 정도 걸렸다.

데이터 허가를 받으면 API인증키를 받을 수 있다.

![Untitled](/assets/image/Untitled1.png)


## 요청변수

![Untitled](/assets/image/Untitled2.png)

오픈API는 get방식 요청하기에  url에  API에서 요구하는 변수들을 넣어 요청하면 된다.

요청 변수들은 “&”로 구분해 값을 넣을 수 있다.

내가 필요한 값들은 인증키, 페이지번호, 한페이지결과수, 정류소ID 이렇게 4개는 필수 값으로 꼭 값을 넘겨 줘야한다.

url 작성은 이런식으로 작성하면 된다.

요청url?serviceKey=”인증키” & pageNo=”pageNo” & numOfRwos= “” & bstopId=””

[http://apis.data.go.kr/6280000/busArrivalService/getAllRouteBusArrivalList?serviceKey=”인증키”&pageNo=1&numOfRows=10&bstopId=165000111](http://apis.data.go.kr/6280000/busArrivalService/getAllRouteBusArrivalList?serviceKey=EeQ8yv6SwhuXSd60XpJqUhtzmlcoYvZuaXMnN0Fw3rMyHD%2FlkxK7CKMA4KCzLfYPz2Pc%2B4XPgsQki831XrQkHg%3D%3D&pageNo=1&numOfRows=10&bstopId=165000111)

작성된 url을 주소창에 입력하면 

![Untitled](/assets/image/Untitled3.png)

이런식으로 xml 형태로 조회한 정보들을 제공한다.

이제 자바 코드로 api를 요청해서 원하는 데이터를 받도록 해보자.

## 구현

### 요청 주소 작성

```java
String key = "EeQ8yv6SwhuXSd60XpJqUhtzmlcoYvZuaXMnN0Fw3rMyHD%2FlkxK7CKMA4KCzLfYPz2Pc%2B4XPgsQki831XrQkHg%3D%3D"; /*인증키*/
StringBuilder urlBuilder = new StringBuilder("http://apis.data.go.kr/6280000/busArrivalService/getAllRouteBusArrivalList"); /*URL*/
urlBuilder.append("?" + URLEncoder.encode("serviceKey","UTF-8") + "="+key); /*Service Key*/
urlBuilder.append("&" + URLEncoder.encode("pageNo","UTF-8") + "=" + URLEncoder.encode("1", "UTF-8")); /*페이지번호*/
urlBuilder.append("&" + URLEncoder.encode("numOfRows","UTF-8") + "=" + URLEncoder.encode("10", "UTF-8")); /*한 페이지 결과 수*/
urlBuilder.append("&" + URLEncoder.encode("bstopId","UTF-8") + "=" + URLEncoder.encode("165000111", "UTF-8")); /*정류소 고유번호*/

URL url = new URL(urlBuilder.toString());
```

StringBuilder로 요청변수를 합쳐 url객체를 생성한다.

### 요청하기

```java
HttpURLConnection conn = (HttpURLConnection) url.openConnection();
conn.setRequestMethod("GET");
conn.setRequestProperty("Content-type", "application/xml");
conn.setRequestProperty("Accept", "*/*;q=0.9");
```

생성한 url 객체로 데이터가 저장된 서버에 request하기 위한HttpURLConnection을 생성한다.

요청메서드와 request header 값을 세팅해준다.

### Document 객체생성

```java
DocumentBuilderFactory dbFactory = DocumentBuilderFactory.newInstance();
DocumentBuilder dBuilder = dbFactory.newDocumentBuilder();
Document doc = dBuilder.parse(conn.getInputStream());
```

Document객체를 생성해준다.

Document는 웹문서 그 자체를 의미하며 웹문서에 존재하는 요소에 접근하고자 할 때 생성한다. 

api가 응답한 결과를 Document객체에 파싱한다.

이제 정규화를 시켜주어야 한다.

```java
doc.getDocumentElement().normalize();
```

doc.getDocumentElement()는 DOM Tree의 Root노드를 가져오는 매서드이다.

Root노드에서 정규화를 해준다.

정규화란 문서의 안정화를 말하는것같다.

### 데이터 꺼내오기

![Untitled](/assets/image/Untitled4.png)

필요한 데이터들의 부모 노드 가져와 list에 담아둔다.

```java
NodeList nList = doc.getElementsByTagName("itemList");
```

태그에서 데이터 하나 가져오는 매서드를 만들어주자.

```java
public static String getTagValue(String tag, Element eElement){
        String result = " ";

        NodeList nList = eElement.getElementsByTagName(tag).item(0).getChildNodes();
        result = nList.item(0).getTextContent();
        return result;
    }
```

이제 for문으로 데이터들을 꺼내 출력해줄수있다.

```java
for(int i=0; i<nList.getLength();i++){
     Node nNode = nList.item(i);

     Element eElement = (Element) nNode;

     System.out.println("ARRIVALESTIMATETIME : "+getTagValue("ARRIVALESTIMATETIME",eElement));
     System.out.println("BSTOPID : "+getTagValue("BSTOPID",eElement));
     System.out.println("BUSID : "+getTagValue("BUSID",eElement));
     System.out.println("BUS_NUM_PLATE : "+getTagValue("BUS_NUM_PLATE",eElement));
     System.out.println("CONGESTION : "+getTagValue("CONGESTION",eElement));
     System.out.println("DIRCD : "+getTagValue("DIRCD",eElement));
	   System.out.println("LASTBUSYN : "+getTagValue("LASTBUSYN",eElement));
     System.out.println("LATEST_STOP_ID : "+getTagValue("LATEST_STOP_ID",eElement));
     System.out.println("LATEST_STOP_NAME : "+getTagValue("LATEST_STOP_NAME",eElement));
     System.out.println("LOW_TP_CD : "+getTagValue("LOW_TP_CD",eElement));
     System.out.println("REMAIND_SEAT : "+getTagValue("REMAIND_SEAT",eElement));
     System.out.println("REST_STOP_COUNT : "+getTagValue("REST_STOP_COUNT",eElement));
     System.out.println("ROUTEID : "+getTagValue("ROUTEID",eElement));
     System.out.println("==============================================");
}
```

### 결과

![Untitled](/assets/image/Untitled5.png)

데이터들이 잘 파싱되었다. 

필요한 데이터만 변수에 담아 파싱한 데이터들을 관리할 수 있게 되었다.

```java
public class apiParse {
    public static void main(String[] args) {
        try {
            String key = "EeQ8yv6SwhuXSd60XpJqUhtzmlcoYvZuaXMnN0Fw3rMyHD%2FlkxK7CKMA4KCzLfYPz2Pc%2B4XPgsQki831XrQkHg%3D%3D"; /*인증키*/
            StringBuilder urlBuilder = new StringBuilder("http://apis.data.go.kr/6280000/busArrivalService/getAllRouteBusArrivalList"); /*URL*/
            urlBuilder.append("?" + URLEncoder.encode("serviceKey","UTF-8") + "="+key); /*Service Key*/
            urlBuilder.append("&" + URLEncoder.encode("pageNo","UTF-8") + "=" + URLEncoder.encode("1", "UTF-8")); /*페이지번호*/
            urlBuilder.append("&" + URLEncoder.encode("numOfRows","UTF-8") + "=" + URLEncoder.encode("10", "UTF-8")); /*한 페이지 결과 수*/
            urlBuilder.append("&" + URLEncoder.encode("bstopId","UTF-8") + "=" + URLEncoder.encode("165000111", "UTF-8")); /*정류소 고유번호*/

            URL url = new URL(urlBuilder.toString());

            HttpURLConnection conn = (HttpURLConnection) url.openConnection();
            conn.setRequestMethod("GET");
            conn.setRequestProperty("Content-type", "application/xml");
            conn.setRequestProperty("Accept", "*/*;q=0.9");
            

            DocumentBuilderFactory dbFactory = DocumentBuilderFactory.newInstance();
            DocumentBuilder dBuilder = dbFactory.newDocumentBuilder();
            Document doc = dBuilder.parse(conn.getInputStream());

            doc.getDocumentElement().normalize();

            NodeList nList = doc.getElementsByTagName("itemList");

            for(int i=0; i<nList.getLength();i++){
                Node nNode = nList.item(i);

                Element eElement = (Element) nNode;

                System.out.println("ARRIVALESTIMATETIME : "+getTagValue("ARRIVALESTIMATETIME",eElement));
                System.out.println("BSTOPID : "+getTagValue("BSTOPID",eElement));
                System.out.println("BUSID : "+getTagValue("BUSID",eElement));
                System.out.println("BUS_NUM_PLATE : "+getTagValue("BUS_NUM_PLATE",eElement));
                System.out.println("CONGESTION : "+getTagValue("CONGESTION",eElement));
                System.out.println("DIRCD : "+getTagValue("DIRCD",eElement));
                System.out.println("LASTBUSYN : "+getTagValue("LASTBUSYN",eElement));
                System.out.println("LATEST_STOP_ID : "+getTagValue("LATEST_STOP_ID",eElement));
                System.out.println("LATEST_STOP_NAME : "+getTagValue("LATEST_STOP_NAME",eElement));
                System.out.println("LOW_TP_CD : "+getTagValue("LOW_TP_CD",eElement));
                System.out.println("REMAIND_SEAT : "+getTagValue("REMAIND_SEAT",eElement));
                System.out.println("REST_STOP_COUNT : "+getTagValue("REST_STOP_COUNT",eElement));
                System.out.println("ROUTEID : "+getTagValue("ROUTEID",eElement));
                System.out.println("==============================================");
            }
        }
        catch (Exception e) {
            e.printStackTrace();
        }
    }

    public static String getTagValue(String tag, String chilTag, Element eElement){ // 하나의 태그에 값이 여러개일 경우 데이터들을 가져오는 메서드
        String result = " ";

        NodeList nList = eElement.getElementsByTagName(tag).item(0).getChildNodes();
        for(int i=0; i<eElement.getElementsByTagName(chilTag).getLength(); i++){

            result += nList.item(i).getChildNodes().item(0).getTextContent()+" ";
        }

        return result;
    }

    public static String getTagValue(String tag, Element eElement){ // 하나의 태그에 한개의 값을 가져오는 메서드
        String result = " ";

        NodeList nList = eElement.getElementsByTagName(tag).item(0).getChildNodes();
        result = nList.item(0).getTextContent();

        return result;
    }

}
```
