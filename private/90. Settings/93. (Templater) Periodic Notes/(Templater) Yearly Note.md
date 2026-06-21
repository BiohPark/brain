---
tags:
Categories:
  - "[[📚760 Yearly]]"
Indexes:
createdAt: <% tp.date.now("YYYY-MM-DD HH:mm") %>
updatedAt:
---
<%*
let m = moment(tp.file.title, "YYYY");

// # 2026
tR += '# ' + m.format('YYYY') + '\n';

// ❮ 2025 | 2026 | 2027 ❯
tR += '❮ [[' + m.clone().subtract(1, 'year').format('YYYY') + ']]';
tR += ' | ' + m.format('YYYY') + ' | ';
tR += '[[' + m.clone().add(1, 'year').format('YYYY') + ']] ❯';
tR += '\n';

// Q1 - Q2 - Q3 - Q4
let yStart = m.clone().startOf('year');
for (let i = 0; i < 4; i++) {
    let q = yStart.clone().add(i, 'quarter');
    tR += '[[' + q.format('YYYY-[Q]Q') + '|' + q.format('[Q]Q') + ']]';
    if (i < 3) tR += ' - ';
}
%>

---
### 당해 완료
```tasks
done after <% moment(tp.file.title,"YYYY").startOf('year').subtract(1,'day').format("YYYY-MM-DD") %>
done before <% moment(tp.file.title,"YYYY").endOf('year').add(1,'day').format("YYYY-MM-DD") %>
```
```dataview
TABLE choice(ProjectView_URL, "["+ProjectView+"]("+ProjectView+URL+")","") AS PV, 완료일 as 완료
where 진행상태="완료"
AND date(완료일).year = number(split(this.file.name, "-")[0])
```

### 당해 납기
```tasks
due after <% moment(tp.file.title,"YYYY").startOf('year').subtract(1,'day').format("YYYY-MM-DD") %>
due before <% moment(tp.file.title,"YYYY").endOf('year').add(1,'day').format("YYYY-MM-DD") %>
```
```dataview
TABLE choice(ProjectView_URL, "["+ProjectView+"]("+ProjectView+URL+")","") AS PV, 납기일 as 납기
where 1=1
AND 납기일
AND date(납기일).year = number(split(this.file.name, "-")[0])
```
### Yearly 기록
