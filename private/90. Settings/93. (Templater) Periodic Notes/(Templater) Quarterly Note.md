---
tags:
Categories:
  - "[[📚740 Quaterly]]"
Indexes:
createdAt: <% tp.date.now("YYYY-MM-DD HH:mm") %>
updatedAt:
---
<%*
let parts = tp.file.title.split("-Q");
let m = moment([parseInt(parts[0], 10)]).quarter(parseInt(parts[1], 10)).startOf('quarter');

// # 2026 Q2
tR += '# ' + m.format('YYYY [Q]Q') + '\n';

// 2026
tR += '[[' + m.format('YYYY') + ']]';
tR += '\n';

// ❮ 2026-Q1 | 2026-Q2 | 2026-Q3 ❯
tR += '❮ [[' + m.clone().subtract(1, 'quarter').format('YYYY-[Q]Q') + ']]';
tR += ' | ' + m.format('YYYY-[Q]Q') + ' | ';
tR += '[[' + m.clone().add(1, 'quarter').format('YYYY-[Q]Q') + ']] ❯';
tR += '\n';

// April - May - June
let qStart = m.clone().startOf('quarter');
for (let i = 0; i < 3; i++) {
    let mo = qStart.clone().add(i, 'month');
    tR += '[[' + mo.format('YYYY-MM') + '|' + mo.format('MMMM') + ']]';
    if (i < 2) tR += ' - ';
}
%>

---

### 당분기 완료
```tasks
done after <% moment([+tp.file.title.split("-Q")[0]]).quarter(+tp.file.title.split("-Q")[1]).startOf('quarter').subtract(1,'day').format("YYYY-MM-DD") %>
done before <% moment([+tp.file.title.split("-Q")[0]]).quarter(+tp.file.title.split("-Q")[1]).endOf('quarter').add(1,'day').format("YYYY-MM-DD") %>
```
```dataview
TABLE choice(ProjectView_URL, "["+ProjectView+"]("+ProjectView+URL+")","") AS PV, 완료일 as 완료
where 진행상태="완료"
AND date(완료일).year = number(split(this.file.name, "-")[0])
AND dateformat(완료일, "q") = string(split(this.file.name, "-Q")[1])
```

### 당분기 납기
```tasks
due after <% moment([+tp.file.title.split("-Q")[0]]).quarter(+tp.file.title.split("-Q")[1]).startOf('quarter').subtract(1,'day').format("YYYY-MM-DD") %>
due before <% moment([+tp.file.title.split("-Q")[0]]).quarter(+tp.file.title.split("-Q")[1]).endOf('quarter').add(1,'day').format("YYYY-MM-DD") %>
```
```dataview
TABLE choice(ProjectView_URL, "["+ProjectView+"]("+ProjectView+URL+")","") AS PV, 납기일 as 납기
where 1=1
AND 납기일
AND date(납기일).year = number(split(this.file.name, "-")[0])
AND dateformat(납기일, "q") = string(split(this.file.name, "-Q")[1])
```
### Quarterly 기록
