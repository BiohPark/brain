---
tags:
Categories:
  - "[[📚710 Daily]]"
Indexes:
createdAt: <% tp.date.now("YYYY-MM-DD HH:mm") %>
updatedAt:
---
<%*
let today = moment(tp.file.title);

// # 2023/01/01 - Sunday
tR += '# ' + today.format('YYYY/MM/DD - dddd') + '\n';

// 2023 / Q1 / January / Week 1
tR += '[[' + today.format('YYYY') + ']] / ';
tR += '[[' + today.format('YYYY-[Q]Q') + '|' + today.format('[Q]Q') + ']] / ';
tR += '[[' + today.format('YYYY-MM') + '|' + today.format('MMMM') + ']] / ';
tR += '[[' + today.format('gggg-[W]ww') + '|' + today.format('[Week] w') + ']]';
tR += '\n';

// ❮ 2022-12-31 | 2023-01-01 | 2023-01-02 ❯
tR += '❮ [[' + today.subtract(1, 'days').format('YYYY-MM-DD') + ']]';
tR += ' | ' + today.add(1, 'days').format('YYYY-MM-DD') + ' | ';
tR += '[[' + today.add(1, 'days').format('YYYY-MM-DD') + ']] ❯';
today.subtract(1, 'days');
%>

---
## 업무사항
### <% tp.file.title %> 완료 tasks
```tasks
done <% tp.file.title.slice(0,10) %>
```

---
## <% tp.file.title %> 회고
- 배운 것
- 감사한 것
- 일기

---

