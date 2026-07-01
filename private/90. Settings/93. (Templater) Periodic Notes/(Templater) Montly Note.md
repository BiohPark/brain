---
tags:
Categories:
  - "[[📚730 Monthly]]"
Indexes:
created: <% tp.date.now("YYYY-MM-DD HH:mm") %>
updated:
---
<%*
let m = moment(tp.file.title, "YYYY-MM");
tR += '# ' + m.format('YYYY-MM') + '\n';
tR += '[[' + m.format('YYYY') + ']] / [[' + m.format('YYYY-[Q]Q|[Q]Q') + ']]\n';
tR += '❮ [[' + m.clone().subtract(1, 'months').format('YYYY-MM') + ']] | ' + m.format('YYYY-MM') + ' | [[' + m.clone().add(1, 'months').format('YYYY-MM') + ']] ❯\n';
%>

---

### 당월 완료
```tasks
done after <% moment(tp.file.title, "YYYY-MM").endOf('month').subtract(1, 'months').format('YYYY-MM-DD') %>
done before <% moment(tp.file.title, "YYYY-MM").startOf('month').add(1, 'months').format('YYYY-MM-DD') %>
```
```dataview
TABLE choice(ProjectView_URL, "["+ProjectView+"]("+ProjectView+URL+")","") AS PV, 완료일 as 완료
where 진행상태="완료"
AND date(완료일).year = number(split(this.file.name, "-")[0])
AND date(완료일).month= number(split(this.file.name, "-")[1])
```

### 당월 납기
```tasks
due after <% moment(tp.file.title, "YYYY-MM").endOf('month').subtract(1, 'months').format('YYYY-MM-DD') %>
due before <% moment(tp.file.title, "YYYY-MM").startOf('month').add(1, 'months').format('YYYY-MM-DD') %>
```
```dataview
TABLE choice(ProjectView_URL, "["+ProjectView+"]("+ProjectView+URL+")","") AS PV, 납기일 as 납기
where 1=1
AND 납기일
AND date(납기일).year = number(split(this.file.name, "-")[0])
AND date(납기일).month = number(split(this.file.name, "-")[1])
```
### Montly 기록

