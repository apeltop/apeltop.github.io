---
title: 캘린더에 보이는 날짜 구하기
author: sunshine@ptokos.com
categories: [javascript]
---

아래 화면과 같이 캘린더를 보여줄 필요가 있었다. 
2023년 12월이라면 11월 26일 ~ 1월 6일까지 보여주면 된다.

![캘린더 예시](/assets/img/js/calendar-calc-1.png)

간단히 로직을 구성해보자

1. 현재 달의 1일부터 마지막일을 배열로 만든다.
2. 배열[0] 가 일요일이 나올 때까지 요일을 추가한다.
3. 배열[-1] 토요일이 나올 때까지 요일을 추가한다.

### 만들기
#### 라이브러리
[date-fns](https://date-fns.org) 를 사용한다.

```bash
yarn add date-fns
```

#### 타입
각 요일에 대한 정보는 MonthDay 라는 타입으로 만들어서 사용한다. 
```typescript
type MonthDay = {
    date: string;
    isCurrentMonth: boolean;
    isToday: boolean;
}
```

#### 함수 만들기
calcYearMonth 함수를 만들어서 해당 년도와 월을 인자로 받아서 해당 월에서 보여줄 날짜를 구한다.

```typescript
import {format, getDaysInMonth} from "date-fns";

export const calcYearMonth = (year: number, month: number) => {
    const currentDate = new Date(year, month);
    const daysInMonth = getDaysInMonth(currentDate);

    const dates = Array.from({length: daysInMonth}, (_, i) => {
        return new Date(currentDate.getFullYear(), currentDate.getMonth(), i + 1);
    });


    while (dates[0].getDay() !== 0) {
        dates.unshift(new Date(dates[0].getFullYear(), dates[0].getMonth(), dates[0].getDate() - 1));
    }

    while (dates[dates.length - 1].getDay() !== 6) {
        dates.push(new Date(dates[dates.length - 1].getFullYear(), dates[dates.length - 1].getMonth(), dates[dates.length - 1].getDate() + 1));
    }

    return dates.map<MonthDay>(date => {
        const isCurrentMonth = date.getMonth() === currentDate.getMonth();
        const isToday = format(date, "yyyy-MM-dd") === getTodayDateFormatted();
        return {date: format(date, "yyyy-MM-dd"), isCurrentMonth, isToday};
    });
}

export const getTodayDateFormatted = () => {
    return format(new Date(), "yyyy-MM-dd");
}
```

#### 테스트 케이스 추가
2023년 12월 테스트 코드 마지막 expect 부분을 보면 의문이 들 수 있다.
`expect(calcYearMonth(2023, 12 - 1)).toEqual(dates);`

12월에 `-1` 을 추가하는 이유는 `Date` 객체의 월은 0부터 시작하기 때문이다.

```typescript
import {describe, expect, test} from "@jest/globals";
import {calcYearMonth, getTodayDateFormatted} from "@/utils/date";

describe('calcYearMonth', () => {
    test('2023-12', () => {
        const dates = [
            {"date":"2023-11-26","isCurrentMonth":false,"isToday":false},
            {"date":"2023-11-27","isCurrentMonth":false,"isToday":false},
            {"date":"2023-11-28","isCurrentMonth":false,"isToday":false},
            {"date":"2023-11-29","isCurrentMonth":false,"isToday":false},
            {"date":"2023-11-30","isCurrentMonth":false,"isToday":false},
            {"date":"2023-12-01","isCurrentMonth":true,"isToday":false},
            {"date":"2023-12-02","isCurrentMonth":true,"isToday":false},
            {"date":"2023-12-03","isCurrentMonth":true,"isToday":false},
            {"date":"2023-12-04","isCurrentMonth":true,"isToday":false},
            {"date":"2023-12-05","isCurrentMonth":true,"isToday":false},
            {"date":"2023-12-06","isCurrentMonth":true,"isToday":false},
            {"date":"2023-12-07","isCurrentMonth":true,"isToday":false},
            {"date":"2023-12-08","isCurrentMonth":true,"isToday":false},
            {"date":"2023-12-09","isCurrentMonth":true,"isToday":false},
            {"date":"2023-12-10","isCurrentMonth":true,"isToday":false},
            {"date":"2023-12-11","isCurrentMonth":true,"isToday":false},
            {"date":"2023-12-12","isCurrentMonth":true,"isToday":false},
            {"date":"2023-12-13","isCurrentMonth":true,"isToday":false},
            {"date":"2023-12-14","isCurrentMonth":true,"isToday":false},
            {"date":"2023-12-15","isCurrentMonth":true,"isToday":false},
            {"date":"2023-12-16","isCurrentMonth":true,"isToday":false},
            {"date":"2023-12-17","isCurrentMonth":true,"isToday":false},
            {"date":"2023-12-18","isCurrentMonth":true,"isToday":false},
            {"date":"2023-12-19","isCurrentMonth":true,"isToday":false},
            {"date":"2023-12-20","isCurrentMonth":true,"isToday":false},
            {"date":"2023-12-21","isCurrentMonth":true,"isToday":false},
            {"date":"2023-12-22","isCurrentMonth":true,"isToday":false},
            {"date":"2023-12-23","isCurrentMonth":true,"isToday":false},
            {"date":"2023-12-24","isCurrentMonth":true,"isToday":false},
            {"date":"2023-12-25","isCurrentMonth":true,"isToday":false},
            {"date":"2023-12-26","isCurrentMonth":true,"isToday":false},
            {"date":"2023-12-27","isCurrentMonth":true,"isToday":false},
            {"date":"2023-12-28","isCurrentMonth":true,"isToday":false},
            {"date":"2023-12-29","isCurrentMonth":true,"isToday":false},
            {"date":"2023-12-30","isCurrentMonth":true,"isToday":false},
            {"date":"2023-12-31","isCurrentMonth":true,"isToday":false},
            {"date":"2024-01-01","isCurrentMonth":false,"isToday":false},
            {"date":"2024-01-02","isCurrentMonth":false,"isToday":false},
            {"date":"2024-01-03","isCurrentMonth":false,"isToday":false},
            {"date":"2024-01-04","isCurrentMonth":false,"isToday":false},
            {"date":"2024-01-05","isCurrentMonth":false,"isToday":false},
            {"date":"2024-01-06","isCurrentMonth":false,"isToday":false}
        ];

        const todayIndex = dates.findIndex(date => date.date === getTodayDateFormatted());
        todayIndex !== -1 && (dates[todayIndex]["isToday"] = true);

        expect(calcYearMonth(2023, 12 - 1)).toEqual(dates);
    });

    test('2024-01', () => {
        const dates = [
            {"date": "2023-12-31", "isCurrentMonth": false, "isToday": false},
            {"date": "2024-01-01", "isCurrentMonth": true, "isToday": false},
            {"date": "2024-01-02", "isCurrentMonth": true, "isToday": false},
            {"date": "2024-01-03", "isCurrentMonth": true, "isToday": false},
            {"date": "2024-01-04", "isCurrentMonth": true, "isToday": false},
            {"date": "2024-01-05", "isCurrentMonth": true, "isToday": false},
            {"date": "2024-01-06", "isCurrentMonth": true, "isToday": false},
            {"date": "2024-01-07", "isCurrentMonth": true, "isToday": false},
            {"date": "2024-01-08", "isCurrentMonth": true, "isToday": false},
            {"date": "2024-01-09", "isCurrentMonth": true, "isToday": false},
            {"date": "2024-01-10", "isCurrentMonth": true, "isToday": false},
            {"date": "2024-01-11", "isCurrentMonth": true, "isToday": false},
            {"date": "2024-01-12", "isCurrentMonth": true, "isToday": false},
            {"date": "2024-01-13", "isCurrentMonth": true, "isToday": false},
            {"date": "2024-01-14", "isCurrentMonth": true, "isToday": false},
            {"date": "2024-01-15", "isCurrentMonth": true, "isToday": false},
            {"date": "2024-01-16", "isCurrentMonth": true, "isToday": false},
            {"date": "2024-01-17", "isCurrentMonth": true, "isToday": false},
            {"date": "2024-01-18", "isCurrentMonth": true, "isToday": false},
            {"date": "2024-01-19", "isCurrentMonth": true, "isToday": false},
            {"date": "2024-01-20", "isCurrentMonth": true, "isToday": false},
            {"date": "2024-01-21", "isCurrentMonth": true, "isToday": false},
            {"date": "2024-01-22", "isCurrentMonth": true, "isToday": false},
            {"date": "2024-01-23", "isCurrentMonth": true, "isToday": false},
            {"date": "2024-01-24", "isCurrentMonth": true, "isToday": false},
            {"date": "2024-01-25", "isCurrentMonth": true, "isToday": false},
            {"date": "2024-01-26", "isCurrentMonth": true, "isToday": false},
            {"date": "2024-01-27", "isCurrentMonth": true, "isToday": false},
            {"date": "2024-01-28", "isCurrentMonth": true, "isToday": false},
            {"date": "2024-01-29", "isCurrentMonth": true, "isToday": false},
            {"date": "2024-01-30", "isCurrentMonth": true, "isToday": false},
            {"date": "2024-01-31", "isCurrentMonth": true, "isToday": false},
            {"date": "2024-02-01", "isCurrentMonth": false, "isToday": false},
            {"date": "2024-02-02", "isCurrentMonth": false, "isToday": false},
            {"date": "2024-02-03", "isCurrentMonth": false, "isToday": false}
        ];

        const todayIndex = dates.findIndex(date => date.date === getTodayDateFormatted());
        todayIndex !== -1 && (dates[todayIndex]["isToday"] = true);

        expect(calcYearMonth(2024, 1 - 1)).toEqual(dates);
    });
});
```




