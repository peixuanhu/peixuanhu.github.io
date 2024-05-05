---
title: Java 8-Date-Time API
categories:
  - [Java, Modern Java]
---

# Java 8 新特性 - Date-Time API

Java 8 在 `java.time` 包下包含一个全新的日期和时间 API。

## 主要类

```java
LocalDateTime.class //日期+时间 format: yyyy-MM-ddTHH:mm:ss.SSS
LocalDate.class //日期 format: yyyy-MM-dd
LocalTime.class //时间 format: HH:mm:ss
```

## 格式化

```java
public void Format(){
    //format yyyy-MM-dd
    LocalDate date = LocalDate.now();
    System.out.println(String.format("date format : %s", date));

    //format HH:mm:ss
    LocalTime time = LocalTime.now().withNano(0);
    System.out.println(String.format("time format : %s", time));

    //format yyyy-MM-dd HH:mm:ss
    LocalDateTime dateTime = LocalDateTime.now();
    DateTimeFormatter dateTimeFormatter = DateTimeFormatter.ofPattern("yyyy-MM-dd HH:mm:ss");
    String dateTimeStr = dateTime.format(dateTimeFormatter);
    System.out.println(String.format("dateTime format : %s", dateTimeStr));
}
```

## 字符串转日期格式

```java
LocalDate date1 = LocalDate.of(2021, 1, 26);
LocalDate date2 = LocalDate.parse("2021-01-26");

LocalDateTime dateTime1 = LocalDateTime.of(2021, 1, 26, 12, 12, 22);
LocalDateTime dateTime2 = LocalDateTime.parse("2021-01-26 12:12:22");

LocalTime time1 = LocalTime.of(12, 12, 22);
LocalTime time2 = LocalTime.parse("12:12:22");
```

**Java 8 之前** 转换都需要借助 `SimpleDateFormat` 类，而**Java 8 之后**只需要 `LocalDate`、`LocalTime`、`LocalDateTime`的 `of` 或 `parse` 方法。

## 日期计算

```java
public void pushWeek(){
     //一周后的日期
     LocalDate localDate = LocalDate.now();
     //方法1
     LocalDate after = localDate.plus(1, ChronoUnit.WEEKS);
     //方法2
     LocalDate after2 = localDate.plusWeeks(1);
     System.out.println("一周后日期：" + after);

     //算两个日期间隔多少天，计算间隔多少年，多少月
     LocalDate date1 = LocalDate.parse("2021-02-26");
     LocalDate date2 = LocalDate.parse("2021-12-23");
     Period period = Period.between(date1, date2);
     System.out.println("date1 到 date2 相隔："
                + period.getYears() + "年"
                + period.getMonths() + "月"
                + period.getDays() + "天");
		 //打印结果是 “date1 到 date2 相隔：0年9月27天”
     //这里period.getDays()得到的天是抛去年月以外的天数，并不是总天数
  
     //如果要获取纯粹的总天数应该用下面的方法
     long day = date2.toEpochDay() - date1.toEpochDay();
     System.out.println(date1 + "和" + date2 + "相差" + day + "天");
     //打印结果：2021-02-26和2021-12-23相差300天
}
```

## 获取指定日期

```java
public void getDayNew() {
    LocalDate today = LocalDate.now();
    //获取当前月第一天：
    LocalDate firstDayOfThisMonth = today.with(TemporalAdjusters.firstDayOfMonth());
    // 取本月最后一天
    LocalDate lastDayOfThisMonth = today.with(TemporalAdjusters.lastDayOfMonth());
    //取下一天：
    LocalDate nextDay = lastDayOfThisMonth.plusDays(1);
    //当年最后一天
    LocalDate lastday = today.with(TemporalAdjusters.lastDayOfYear());
    //2021年最后一个周日，如果用Calendar是不得烦死。
    LocalDate lastMondayOf2021 = LocalDate.parse("2021-12-31").with(TemporalAdjusters.lastInMonth(DayOfWeek.SUNDAY));
}
```

