---
tags:
Categories:
  - "[[📚760 Yearly]]"
Indexes:
created: <% tp.date.now("YYYY-MM-DD HH:mm") %>
updated:
---
<%*
let m = moment(tp.file.title, "YYYY");
tR += '# ' + m.format('YYYY') + '\n';
tR += '❮ [[' + m.clone().subtract(1, 'years').format('YYYY') + ']] | ' + m.format('YYYY') + ' | [[' + m.clone().add(1, 'years').format('YYYY') + ']] ❯\n';
tR += '[[' + m.clone().startOf('year').format('YYYY-[Q]1|[Q]1') + ']] - [[' + m.clone().startOf('year').format('YYYY-[Q]2|[Q]2') + ']] - [[' + m.clone().startOf('year').format('YYYY-[Q]3|[Q]3') + ']] - [[' + m.clone().startOf('year').format('YYYY-[Q]4|[Q]4') + ']]\n';
%>

---
### 당해 완료
```tasks
done after <% moment(tp.file.title, "YYYY").startOf('year').subtract(1, 'days').format('YYYY-MM-DD') %>
done before <% moment(tp.file.title, "YYYY").endOf('year').add(1, 'days').format('YYYY-MM-DD') %>
```
```dataview
TABLE choice(ProjectView_URL, "["+ProjectView+"]("+ProjectView+URL+")","") AS PV, 완료일 as 완료
where 진행상태="완료"
AND date(완료일).year = number(split(this.file.name, "-")[0])
```

### 당해 납기
```tasks
due after <% moment(tp.file.title, "YYYY").startOf('year').subtract(1, 'days').format('YYYY-MM-DD') %>
due before <% moment(tp.file.title, "YYYY").endOf('year').add(1, 'days').format('YYYY-MM-DD') %>
```
```dataview
TABLE choice(ProjectView_URL, "["+ProjectView+"]("+ProjectView+URL+")","") AS PV, 납기일 as 납기
where 1=1
AND 납기일
AND date(납기일).year = number(split(this.file.name, "-")[0])
```
### Yearly 기록

