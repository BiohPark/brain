---
tags:
Categories:
Indexes:
  - "[[🏛 Categories]]"
date_created: 2026-06-15T00:00:00+09:00
date_modified: 2026-06-15T00:00:00+09:00
---

```dataviewjs
// 🏛 Categories.md를 실시간 파싱 → 인터랙티브 트리 렌더링 (중복 없음)

const COLORS = ['#4a90d9', '#27ae60', '#e67e22', '#8e44ad', '#16a085'];
const cur = dv.current().file.path;

// 1. Categories.md 원본 읽기
const file = app.vault.getAbstractFileByPath("🏛 Categories.md");
if (!file) { dv.paragraph("❌ `🏛 Categories.md` 파일을 찾을 수 없습니다."); return; }
const content = await app.vault.cachedRead(file);

// 2. 카테고리별 노트 수 카운트
const countMap = Object.create(null);
for (const page of dv.pages('"private"')) {
  if (!page.Categories) continue;
  const cats = Array.isArray(page.Categories) ? page.Categories : [page.Categories];
  for (const cat of cats) {
    const raw = cat?.path ?? String(cat).replace(/\[\[|\]\]/g, '');
    const key = raw.trim().toLowerCase().replace(/\s+/g, ' ');
    if (key) countMap[key] = (countMap[key] || 0) + 1;
  }
}
const getCount = (name) =>
  countMap[name.trim().toLowerCase().replace(/\s+/g, ' ')] || 0;

// 3. Categories.md 파싱 (## 📖 → - 📚 → 세부)
const sections = [];
let curSec = null, curSub = null;
for (const line of content.split('\n')) {
  const h2 = line.match(/^## \[\[(📖[^\]]+)\]\]\s*(.*)/);
  if (h2) {
    curSec = { linkName: h2[1].trim(), desc: h2[2].trim(), subs: [] };
    sections.push(curSec);
    curSub = null;
    continue;
  }
  const sub = line.match(/^- \[\[(📚[^\]]+)\]\]\s*(.*)/);
  if (sub && curSec) {
    curSub = { linkName: sub[1].trim(), desc: sub[2].trim(), items: [] };
    curSec.subs.push(curSub);
    continue;
  }
  const item = line.match(/^\s{2,}- \[\[([^\]]+)\]\]\s*(.*)/);
  if (item && curSub) {
    curSub.items.push({ name: item[1].trim(), desc: item[2].trim() });
  }
}

// 4. 스타일 주입 (중복 방지)
const STYLE_ID = 'cmm-styles-v2';
if (!document.getElementById(STYLE_ID)) {
  const s = document.createElement('style');
  s.id = STYLE_ID;
  s.textContent = `
    .cmm-wrap { padding: 4px 0; }
    .cmm-sec { margin-bottom: 10px; }
    .cmm-h1 {
      display: flex; align-items: center; gap: 8px;
      padding: 9px 14px; border-radius: 7px;
      cursor: pointer; user-select: none;
      font-weight: 700; font-size: 0.95em;
      transition: opacity .15s;
    }
    .cmm-h1:hover { opacity: .82; }
    .cmm-arrow {
      display: inline-block; width: 14px; text-align: center;
      transition: transform .2s; font-size: .8em; flex-shrink: 0;
    }
    .cmm-h1.open .cmm-arrow { transform: rotate(90deg); }
    .cmm-body { display: none; padding: 4px 0 2px 28px; }
    .cmm-body.open { display: block; }
    .cmm-sub { margin: 3px 0; }
    .cmm-sh {
      display: flex; align-items: center; gap: 6px;
      padding: 5px 10px; border-radius: 6px; cursor: pointer;
      font-size: .88em; background: var(--background-secondary);
      transition: background .12s;
    }
    .cmm-sh:hover { background: var(--background-modifier-hover); }
    .cmm-sh .cmm-arrow { color: var(--text-muted); }
    .cmm-sh.open .cmm-arrow { transform: rotate(90deg); }
    .cmm-badge {
      font-size: .85em; font-weight: 700; padding: 2px 9px; border-radius: 10px;
      margin-left: auto; flex-shrink: 0;
      background: var(--background-modifier-border); color: var(--text-muted);
      border: 1px solid var(--background-modifier-border);
    }
    .cmm-badge.active { color: #fff; border-color: transparent; }
    .cmm-name-link { color: inherit; text-decoration: none; cursor: pointer; }
    .cmm-name-link:hover { text-decoration: underline; }
    .cmm-filter-input {
      width: 100%; box-sizing: border-box; margin-bottom: 12px;
      padding: 8px 12px; font-size: .92em; border-radius: 8px;
      border: 1px solid var(--background-modifier-border);
      background: var(--background-secondary); color: var(--text-normal);
    }
    .cmm-filter-input:focus { outline: none; border-color: var(--interactive-accent); }
    .cmm-sb { display: none; padding: 3px 0 3px 20px; }
    .cmm-sb.open { display: block; }
    .cmm-item {
      font-size: .8em; color: var(--text-muted);
      padding: 2px 6px; margin: 2px 0;
      border-left: 2px solid var(--background-modifier-border);
    }
    .cmm-name { font-weight: 600; }
    .cmm-desc { font-weight: 400; color: var(--text-muted); }
    .cmm-meta { font-size: .8em; color: var(--text-faint); margin-left: auto; flex-shrink: 0; }
  `;
  document.head.appendChild(s);
}

// 5. 렌더링
const wrap = dv.container.createEl('div', { cls: 'cmm-wrap' });

const filterInput = wrap.createEl('input', {
  cls: 'cmm-filter-input',
  attr: { type: 'text', placeholder: '🔍 카테고리 필터링... (입력하면 실시간으로 좁혀집니다)' }
});

function openNote(linkName) {
  app.workspace.openLinkText(linkName, cur, false);
}
function nameLink(parentEl, text, linkName) {
  const a = parentEl.createEl('span', { cls: 'cmm-name cmm-name-link', text });
  a.addEventListener('click', (e) => { e.stopPropagation(); openNote(linkName); });
  return a;
}

sections.forEach((sec, i) => {
  const color = COLORS[i % COLORS.length];
  const secEl = wrap.createEl('div', { cls: 'cmm-sec' });
  sec._searchText = (sec.linkName + ' ' + sec.desc).toLowerCase();

  // 1차 헤더
  const h1 = secEl.createEl('div', { cls: 'cmm-h1 open' });
  h1.style.background = color + '1a';
  h1.style.borderLeft = `4px solid ${color}`;
  h1.createEl('span', { cls: 'cmm-arrow', text: '▶' });
  nameLink(h1, sec.linkName, sec.linkName);
  if (sec.desc) h1.createEl('span', { cls: 'cmm-desc', text: ' — ' + sec.desc });
  h1.createEl('span', { cls: 'cmm-meta', text: sec.subs.length + '개 분류' });

  const body = secEl.createEl('div', { cls: 'cmm-body open' });
  h1.addEventListener('click', () => {
    h1.classList.toggle('open');
    body.classList.toggle('open');
  });
  sec._el = { root: secEl, header: h1, body };

  // 2차 항목
  for (const sub of sec.subs) {
    const cnt = getCount(sub.linkName);
    const subEl = body.createEl('div', { cls: 'cmm-sub' });
    sub._searchText = (sub.linkName + ' ' + sub.desc).toLowerCase();
    const sh = subEl.createEl('div', { cls: 'cmm-sh' });
    sh.style.borderLeft = `3px solid ${color}55`;

    sh.createEl('span', { cls: 'cmm-arrow', text: sub.items.length ? '▶' : '·' });
    nameLink(sh, sub.linkName, sub.linkName);
    if (sub.desc) sh.createEl('span', { cls: 'cmm-desc', text: ' ' + sub.desc });

    const badge = sh.createEl('span', {
      cls: 'cmm-badge' + (cnt > 0 ? ' active' : ''),
      text: cnt > 0 ? cnt + '개' : '—'
    });
    if (cnt > 0) badge.style.background = color;

    // 3차 세부 (있을 때만)
    sub._itemRefs = [];
    let sb = null;
    if (sub.items.length > 0) {
      sb = subEl.createEl('div', { cls: 'cmm-sb' });
      for (const it of sub.items) {
        const itemEl = sb.createEl('div', { cls: 'cmm-item' });
        nameLink(itemEl, it.name, it.name);
        if (it.desc) itemEl.createEl('span', { cls: 'cmm-desc', text: ' ' + it.desc });
        sub._itemRefs.push({ searchText: (it.name + ' ' + it.desc).toLowerCase(), el: itemEl });
      }
      sh.addEventListener('click', () => {
        sh.classList.toggle('open');
        sb.classList.toggle('open');
      });
    }
    sub._el = { root: subEl, header: sh, body: sb };
  }
});

// 6. 실시간 필터
function applyFilter(raw) {
  const q = raw.trim().toLowerCase();
  if (!q) {
    wrap.querySelectorAll('.cmm-sec, .cmm-sub, .cmm-item').forEach(el => { el.style.display = ''; });
    return;
  }
  for (const sec of sections) {
    let secVisible = sec._searchText.includes(q);
    for (const sub of sec.subs) {
      let subVisible = sub._searchText.includes(q);
      for (const item of sub._itemRefs) {
        const m = item.searchText.includes(q);
        item.el.style.display = m ? '' : 'none';
        subVisible = subVisible || m;
      }
      sub._el.root.style.display = subVisible ? '' : 'none';
      if (subVisible && sub._el.body) {
        sub._el.header.classList.add('open');
        sub._el.body.classList.add('open');
      }
      secVisible = secVisible || subVisible;
    }
    sec._el.root.style.display = secVisible ? '' : 'none';
    if (secVisible) {
      sec._el.header.classList.add('open');
      sec._el.body.classList.add('open');
    }
  }
}
filterInput.addEventListener('input', (e) => applyFilter(e.target.value));
```
