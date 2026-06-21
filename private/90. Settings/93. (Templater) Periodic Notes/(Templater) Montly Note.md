---
tags:
Categories:
  - "[[📚730 Monthly]]"
Indexes:
createdAt: <% tp.date.now("YYYY-MM-DD HH:mm") %>
updatedAt:
---
<%*
let m = moment(tp.file.title, "YYYY-MM");

// # 2026 June
tR += '# ' + m.format('YYYY MMMM') + '\n';

// 2026 / Q2
tR += '[[' + m.format('YYYY') + ']] / ';
tR += '[[' + m.format('YYYY-[Q]Q') + '|' + m.format('[Q]Q') + ']]';
tR += '\n';

// ❮ 2026-05 | 2026-06 | 2026-07 ❯
tR += '❮ [[' + m.clone().subtract(1, 'month').format('YYYY-MM') + ']]';
tR += ' | ' + m.format('YYYY-MM') + ' | ';
tR += '[[' + m.clone().add(1, 'month').format('YYYY-MM') + ']] ❯';
%>

---

### 당월 완료
```tasks
done after <% moment(tp.file.title,"YYYY-MM").startOf('month').subtract(1,'day').format("YYYY-MM-DD") %>
done before <% moment(tp.file.title,"YYYY-MM").endOf('month').add(1,'day').format("YYYY-MM-DD") %>
```
```dataview
TABLE choice(ProjectView_URL, "["+ProjectView+"]("+ProjectView+URL+")","") AS PV, 완료일 as 완료
where 진행상태="완료"
AND date(완료일).year = number(split(this.file.name, "-")[0])
AND date(완료일).month= number(split(this.file.name, "-")[1])
```

### 당월 납기
```tasks
due after <% moment(tp.file.title,"YYYY-MM").startOf('month').subtract(1,'day').format("YYYY-MM-DD") %>
due before <% moment(tp.file.title,"YYYY-MM").endOf('month').add(1,'day').format("YYYY-MM-DD") %>
```
```dataview
TABLE choice(ProjectView_URL, "["+ProjectView+"]("+ProjectView+URL+")","") AS PV, 납기일 as 납기
where 1=1
AND 납기일
AND date(납기일).year = number(split(this.file.name, "-")[0])
AND date(납기일).month = number(split(this.file.name, "-")[1])
```
### Montly 기록
