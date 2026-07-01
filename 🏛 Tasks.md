---
tags:
Categories:
  - "[[📚710 Daily]]"
  - "[[📚720 Weekly]]"
Indexes:
  - "[[🏛 Weekly Notes]]"
  - "[[🏛 Tasks Kanban]]"
created: 2025-03-21 23:10:21
updated: 2025-11-26T20:43:17+09:00
---
## 🔴 지연
```tasks
not done
is not recurring
tags include #task
(due before today) OR (scheduled before today)
```

## 🔵 진행중
### 오늘 마감
```tasks
not done
is not recurring
(due on today) OR (scheduled on today)
```

### 내일 마감
```tasks
not done
is not recurring
(due on tomorrow) OR (scheduled on tomorrow)
```

### 향후 일정
```tasks
not done
is not recurring
((has start date) AND (no due date) AND (no scheduled date)) OR (due after tomorrow) OR (scheduled after tomorrow)
```

## 💬 오늘 회의·작업
```tasks
not done
is not recurring
tags regex matches /회의|작업/
```

## 🔁 반복
```tasks
is recurring
not done
```

## ⚠️ 미정 — 확인하세요
```tasks
no due date
no scheduled date
no start date
is not recurring
not done
```

## ✅ Done
### 오늘 완료
```tasks
done today
```

### 이번주 완료
```tasks
done this week
done before today
```