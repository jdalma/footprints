<!-- TOC -->

- [**JAVA8에 새로운 날짜와 시간 API가 생긴 이유**](#java8%EC%97%90-%EC%83%88%EB%A1%9C%EC%9A%B4-%EB%82%A0%EC%A7%9C%EC%99%80-%EC%8B%9C%EA%B0%84-api%EA%B0%80-%EC%83%9D%EA%B8%B4-%EC%9D%B4%EC%9C%A0)
    - [**Date와 Time API**](#date%EC%99%80-time-api)
        - [지금 이 순간을 기계 시간으로 표현 하는 방법](#%EC%A7%80%EA%B8%88-%EC%9D%B4-%EC%88%9C%EA%B0%84%EC%9D%84-%EA%B8%B0%EA%B3%84-%EC%8B%9C%EA%B0%84%EC%9C%BC%EB%A1%9C-%ED%91%9C%ED%98%84-%ED%95%98%EB%8A%94-%EB%B0%A9%EB%B2%95)
        - [인류용 일시를 표현하는 방법](#%EC%9D%B8%EB%A5%98%EC%9A%A9-%EC%9D%BC%EC%8B%9C%EB%A5%BC-%ED%91%9C%ED%98%84%ED%95%98%EB%8A%94-%EB%B0%A9%EB%B2%95)
        - [기간을 표현하는 방법](#%EA%B8%B0%EA%B0%84%EC%9D%84-%ED%91%9C%ED%98%84%ED%95%98%EB%8A%94-%EB%B0%A9%EB%B2%95)
        - [포맷팅](#%ED%8F%AC%EB%A7%B7%ED%8C%85)
            - [미리 정의된 포맷](#%EB%AF%B8%EB%A6%AC-%EC%A0%95%EC%9D%98%EB%90%9C-%ED%8F%AC%EB%A7%B7)
        - [파싱](#%ED%8C%8C%EC%8B%B1)
        - [레거시 API 지원](#%EB%A0%88%EA%B1%B0%EC%8B%9C-api-%EC%A7%80%EC%9B%90)
- [**참고**](#%EC%B0%B8%EA%B3%A0)

<!-- /TOC -->

# **JAVA8에 새로운 날짜와 시간 API가 생긴 이유**

**그전까지 사용하던 java.util.Date클래스는 [mutable](https://www.edureka.co/blog/java-mutable-and-immutable-objects/)하기 때문에 쓰레드에 안전하지 않다.**  
**[java.util.Date 클래스를 피해야하는 이유](https://codeblog.jonskeet.uk/2017/04/23/all-about-java-util-date/)**

## **Date와 Time API**

-   **기계용 시간 과 인류용 시간으로 나눌 수 있다.**
-   **기계용 시간은 EPOCK (1970년 1월 1일 0시 0분 0초)부터 현재까지의 타임스탬프를 표현한다.**
-   **인류용 시간은 우리가 흔히 사용하는 연,월,일,시,분,초 등을 표현한다.**
-   **타임스탬프는 Instant를 사용한다.**
-   **특정 날짜(LocalDate) , 시간(LocalTime) , 일시(LocalDateTime)를 사용할 수 있다.**
-   **기간을 표현할 때는 Duration(시간 기반)과 Period(날짜 기반)을 사용할 수 있다.**
-   **DateTimeFormatter를 사용해서 일시를 특정한 문자열로 포매팅할 수 있다.**
-   <span style="color:red; font-weight:bold">Immutable하고 새로운 인스턴스가 만들어진다.</span>

### 지금 이 순간을 기계 시간으로 표현 하는 방법

```java
Instant instant = Instant.now();
System.out.println(instant);
// 현재 시간은 2021-01-14 22:08분
// 출력 - 2021-01-14T13:08:28.794934700Z
// 기준 시간이 UTC , GMT 이기 때문에 시간이 다르다.

ZoneId zone = ZoneId.systemDefault();
System.out.println(zone);
// 출력 - Asia/Seoul

ZonedDateTime zonedDateTime = instant.atZone(zone);
System.out.println(zonedDateTime);
// 출력 - 2021-01-14T22:11:49.055336800+09:00[Asia/Seoul]
```

### 인류용 일시를 표현하는 방법

```java
LocalDateTime now = LocalDateTime.now();
System.out.println(now);
// 현재 시간은 20210-01-14 22:16
// 출력 - 2021-01-14T22:16:06.431744200

LocalDateTime birth =
        LocalDateTime.of(1993 , Month.APRIL , 30 , 0 , 0 , 0);
System.out.println(birth);
// 출력 - 1993-04-30T00:00

ZonedDateTime nowInKorea = ZonedDateTime.now(ZoneId.of("Asia/Seoul"));
System.out.println(nowInKorea);
// 출력 - 2021-01-14T22:18:52.191717500+09:00[Asia/Seoul]

Instant beforeInstant = Instant.now();
ZonedDateTime zonedDateTime1 = beforeInstant.atZone(ZoneId.of("Asia/Seoul"));
Instant afterInstant = zonedDateTime1.toInstant();
// ZonedDateTime 과 Instant는 서로 변환이 가능하다.
```

### 기간을 표현하는 방법

```java
LocalDate today = LocalDate.now(); // 2021-01-14
LocalDate dateOfBirth = LocalDate.of(1993 , Month.APRIL , 30);
Period period = Period.between(dateOfBirth , today);
System.out.println(period.getYears());  // 27
System.out.println(period.getMonths()); // 8
System.out.println(period.getDays());   // 15

Period period1 = dateOfBirth.until(today);
System.out.println(period1.get(ChronoUnit.DAYS));   // 15


Instant now2 = Instant.now();
Instant plus = now2.plus(10, ChronoUnit.DAYS);
Duration diff = Duration.between(now2 , plus);
System.out.println(diff);   // PT240H
```

### 포맷팅

```java
LocalDateTime dateTime = LocalDateTime.now();
DateTimeFormatter formatter = DateTimeFormatter.ofPattern("yyyy-MM-dd");
System.out.println(now.format(formatter)); // 2021-01-14
```

#### 미리 정의된 포맷
![](../../../assets/images/java/java8/date-time/1.png)

### 파싱

```java
DateTimeFormatter parseFormatter = DateTimeFormatter.ofPattern("yyyy/MM/dd");
LocalDate localDate = LocalDate.parse("2021/01/14", parseFormatter);
System.out.println(localDate);
```

### 레거시 API 지원
✅**기존 Date , GregorianCalendar 변환 가능**
{: .fh-default .fs-4 }

```java
Date date = new Date();
// Instant <- Date
Instant instant1 = date.toInstant();
// Date <- Instant
Date newDate = Date.from(instant1);

GregorianCalendar gregorianCalendar = new GregorianCalendar();

// Instant <- GregorianCalendar
Instant toInstant = gregorianCalendar.toInstant();

// ZonedDateTime <- GregorianCalendar
ZonedDateTime toZonedDateTime = gregorianCalendar.toInstant()
        .atZone(ZoneId.systemDefault());

// LocalDateTime <- GregorianCalendar
LocalDateTime toLocalDateTime = gregorianCalendar.toInstant()
        .atZone(ZoneId.systemDefault()).toLocalDateTime();

// GregorianCalendar <- toZonedDateTime
GregorianCalendar from = GregorianCalendar.from(toZonedDateTime);
```

# **참고**

[https://docs.oracle.com/javase/tutorial/datetime/overview/index.html](https://docs.oracle.com/javase/tutorial/datetime/overview/index.html)
{: .fh-default .fs-4 }
[https://docs.oracle.com/javase/tutorial/datetime/iso/overview.html](https://docs.oracle.com/javase/tutorial/datetime/iso/overview.html)
{: .fh-default .fs-4 }
