### 메서드
* 잘 쓰여진 코드이라면, 한 메서드의 주제는 반드시 하나다.
  * 0개도 2개도 아닌, 무조건 1개

```java
void 서점에_책을_샀다() {
    우빈이는 서점에 가서 보고싶은 책을 골랐다.
    고른 책을 계산대에 가지고 가서 서점 직원에게 건네주고,
    책의 가격에 해당하는 금액을 현금으로 전달하고 책을 얻었다.
}
```  

**메서드의 이름으로 구체적인 내용을 추상화**

```java
void 서점에_책을_샀다() {
    우빈이는 산책을 하다 은행에 가서 현금을 얼마 인출했다.  
    서점에 가는 길에 아이스크림을 하나 사먹었다. 남은 돈으로  
    서점에 가서 보고싶은 책을 골랐다.
}
```  

**추상화된 내용을 보고 구체적인 내용의 유추가 어려움** -> 메서드가 2가지 이상의 일을 하고 있다.

* 생략할 정보와 의미를 부여하고 드러낼 정보를 구분하기
  * 만약 이게 어렵다면 메서드가 2가지 이상의 일을 하고 있을 가능성이 높다.

```java
void 산책하면서_돈쓰기() {
    Money 은행에서_현금_인출();
    Balance 아이스크림_사먹기(Money);
    Book 서점에서_책_구입하기(Balance);
}
```

**더 큰 맥락 안에서 포괄적인 의미를 담기**

`반환타입 메서드명 (파라미터) {}`
* **메서드명**
  * 추상화되니 구체를 유추할 수 있는, 적절한 의미가 담긴 이름
* **파라미터**
  * 파라미터와 연결지어 더 풍부한 의미를 전달할 수도 있다. 
  * 파라미터의 타입, 개수, 순서를 통해 의미를 전달 
  * 파라미터는 외부 세계와 소통하는 창
* **반환타입**
  * 메서드 시그니처에 납득이 가는, 적절한 타입의 반환값 돌려주기 
    * 반환 타입이 boolean인데, 이게 이 메서드에서 무엇을 의미하는거지?  
  * void 대신 충분히 반환할 만한 값이 있는지 고민해보기
    * 반환값이 있다면 테스트도 용이해 진다.

ex) 우리 도메인에 존재하는 개념(도메인 용어) : 매장, 매출  
매장별, 일자별 매출 집게를 하기 위한 어떤 unique key를 만들 때.

shopId="S34", 2024년 6월 1일의 매출을 집계하기 위한 unique key는 "S34_2024-06-01"로 한다고 하자.

```java
import java.time.LocalDate;

public String createDailyShopKey(String shopId, String localDateString) {
    return String.format("%s_%s", shopId, localDateString);
}

public String createDailyShopKey(String shopId, String sellingDate) {
    return String.format("%s_%s", shopId, sellingDate.toString());
}

// S34_2024-06-01
String shopId = "S34";
LocalDate today = LocalDate.of(2024, 6, 1);

// 1
String dailyShopKey = createDailyShopKey(shopId, today.toString());

// 2
String dailyShopKey = createDailyShopKey(shopId, today);
```

* **타입**
  * 1은 어떤 String을 넣어줘야하는지 혼란이 오지만, 2는 LocalDate과 같은 구체화된 타입이 있으면 그 자체로 의미 전달
* **변수명**
  * localDateString와 달리 sellingDate는 명확한 이름

### 추상화
* 하나의 세계 안에서는, 추상화 레벨이 동등해야 한다.

```java
public static void main(String[] args) {
    showGameStartComments();
    initializeGame();
    ...

    if (gameStatus == 1) {
        System.out.println("지뢰를 모두 찾았습니다. GAME CLEAR!");
        break;
    }
    
    ...
}
```
