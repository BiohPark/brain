---
tags:
Categories:
  - "[[📚740 Quaterly]]"
Indexes:
created: <% tp.date.now("YYYY-MM-DD HH:mm") %>
updated:
---
<%*
let m = moment(tp.file.title, "YYYY-[Q]Q");
tR += '# ' + m.format('YYYY-[Q]Q') + '\n';
tR += '[[' + m.format('YYYY') + ']]\n';
tR += '❮ [[' + m.clone().subtract(1, 'quarters').format('YYYY-[Q]Q') + ']] | ' + m.format('YYYY-[Q]Q') + ' | [[' + m.clone().add(1, 'quarters').format('YYYY-[Q]Q') + ']] ❯\n';
tR += '[[' + m.clone().startOf('quarter').format('YYYY-MM|MMMM') + ']] - [[' + m.clone().startOf('quarter').add(1, 'months').format('YYYY-MM|MMMM') + ']] - [[' + m.clone().startOf('quarter').add(2, 'months').format('YYYY-MM|MMMM') + ']]\n';
%>

---

### 당분기 완료
```tasks
done after <% moment(tp.file.title, "YYYY-[Q]Q").startOf('quarter').subtract(1, 'days').format('YYYY-MM-DD') %>
done before <% moment(tp.file.title, "YYYY-[Q]Q").endOf('quarter').add(1, 'days').format('YYYY-MM-DD') %>
```
```dataview
TABLE choice(ProjectView_URL, "["+ProjectView+"]("+ProjectView+URL+")","") AS PV, 완료일 as 완료
where 진행상태="완료"
AND date(완료일).year = number(split(this.file.name, "-")[0])
AND dateformat(완료일, "q") = string(split(this.file.name, "-Q")[1])
```

### 당분기 납기
```tasks
due after <% moment(tp.file.title, "YYYY-[Q]Q").startOf('quarter').subtract(1, 'days').format('YYYY-MM-DD') %>
due before <% moment(tp.file.title, "YYYY-[Q]Q").endOf('quarter').add(1, 'days').format('YYYY-MM-DD') %>
```
```dataview
TABLE choice(ProjectView_URL, "["+ProjectView+"]("+ProjectView+URL+")","") AS PV, 납기일 as 납기
where 1=1
AND 납기일
AND date(납기일).year = number(split(this.file.name, "-")[0])
AND dateformat(납기일, "q") = string(split(this.file.name, "-Q")[1])
```
### Quarterly 기록

