---
tags:
Categories:
Indexes:
  - "[[🏛 Categories]]"
createdAt: 2026-06-15T00:00:00+09:00
updatedAt: 2026-06-15T00:00:00+09:00
---

```dataviewjs
// 🏛 Categories.md를 실시간 파싱 → 인터랙티브 트리 렌더링 (재귀, 무제한 depth)

const COLORS = ['#4a90d9', '#27ae60', '#e67e22', '#8e44ad', '#16a085'];
const cur = dv.current().file.path;
const TREE_SRC = "🏛 Categories.md";

// 1. Categories.md 원본 읽기
const file = app.vault.getAbstractFileByPath(TREE_SRC);
if (!file) { dv.paragraph("❌ `🏛 Categories.md` 파일을 찾을 수 없습니다."); return; }
const content = await app.vault.cachedRead(file);

// 2. 카테고리별 노트 수 카운트 — Obsidian 링크 리졸버로 양쪽을 동일한 경로로 정규화
//    (카테고리 노트가 실제로는 private/00. Inbox/ 등 하위 폴더에 있어서 단순 텍스트 비교로는
//     대부분 매칭에 실패했었음. getFirstLinkpathDest로 둘 다 실제 파일 경로로 변환해 비교하면
//     폴더 위치·확장자와 무관하게 정확히 매칭된다.)
function resolveKey(linkText, sourcePath) {
  const dest = app.metadataCache.getFirstLinkpathDest(linkText, sourcePath);
  return (dest ? dest.path : linkText).trim().toLowerCase().replace(/\s+/g, ' ');
}

const countMap = Object.create(null);
for (const page of dv.pages('"private"')) {
  if (!page.Categories) continue;
  const cats = Array.isArray(page.Categories) ? page.Categories : [page.Categories];
  for (const cat of cats) {
    let key;
    if (cat?.path) {
      key = cat.path.trim().toLowerCase();
    } else {
      const linkText = String(cat).replace(/\[\[|\]\]/g, '').split('|')[0].trim();
      if (!linkText) continue;
      key = resolveKey(linkText, page.file.path);
    }
    countMap[key] = (countMap[key] || 0) + 1;
  }
}
const getCount = (name) => countMap[resolveKey(name, TREE_SRC)] || 0;

// 3. Categories.md 파싱 — 들여쓰기 기반 재귀 트리 (무제한 depth)
//    depth 0 = 📖(L1, "## [[...]]"), depth 1 = 0칸 들여쓰기 "- [[...]]"(L2),
//    depth 2 = 2칸(L3), depth 3 = 4칸(L4) ... 이후도 동일 규칙
const roots = [];
let stack = [];
for (const line of content.split('\n')) {
  const h1 = line.match(/^## \[\[(📖[^\]]+)\]\]\s*(.*)/);
  if (h1) {
    const node = { linkName: h1[1].trim(), desc: h1[2].trim(), depth: 0, children: [] };
    roots.push(node);
    stack = [node];
    continue;
  }
  const item = line.match(/^( *)- \[\[([^\]]+)\]\]\s*(.*)/);
  if (item && stack.length) {
    const indent = item[1].length;
    const depth = Math.floor(indent / 2) + 1;
    const node = { linkName: item[2].trim(), desc: item[3].trim(), depth, children: [] };
    const parentDepth = Math.min(depth - 1, stack.length - 1);
    stack.length = parentDepth + 1;
    stack[parentDepth].children.push(node);
    stack[parentDepth + 1] = node;
    continue;
  }
}

// 4. 스타일 주입 (중복 방지, v4로 교체)
const STYLE_ID = 'cmm-styles-v4';
document.getElementById('cmm-styles-v2')?.remove();
document.getElementById('cmm-styles-v3')?.remove();
if (!document.getElementById(STYLE_ID)) {
  const s = document.createElement('style');
  s.id = STYLE_ID;
  s.textContent = `
    .cmm-wrap { padding: 4px 0; }
    .cmm-toolbar { display: flex; gap: 8px; margin-bottom: 12px; align-items: center; }
    .cmm-filter-input {
      flex: 1; box-sizing: border-box;
      padding: 8px 12px; font-size: .92em; border-radius: 8px;
      border: 1px solid var(--background-modifier-border);
      background: var(--background-secondary); color: var(--text-normal);
    }
    .cmm-filter-input:focus { outline: none; border-color: var(--interactive-accent); }
    .cmm-toolbtn {
      flex-shrink: 0; white-space: nowrap;
      padding: 7px 12px; font-size: .85em; border-radius: 8px; cursor: pointer;
      border: 1px solid var(--background-modifier-border);
      background: var(--background-secondary); color: var(--text-normal);
      transition: background .12s;
    }
    .cmm-toolbtn:hover { background: var(--background-modifier-hover); }
    .cmm-sec { margin-bottom: 10px; }
    .cmm-row {
      display: flex; align-items: center; gap: 6px;
      cursor: pointer; user-select: none;
      transition: opacity .15s, background .12s;
    }
    .cmm-row.l0 {
      padding: 9px 14px; border-radius: 7px;
      font-weight: 700; font-size: 0.95em;
    }
    .cmm-row.l0:hover { opacity: .82; }
    .cmm-row.l1up {
      padding: 5px 10px; border-radius: 6px;
      font-size: .88em; background: var(--background-secondary);
    }
    .cmm-row.l1up:hover { background: var(--background-modifier-hover); }
    .cmm-arrow {
      display: inline-block; width: 14px; text-align: center;
      transition: transform .2s; font-size: .8em; flex-shrink: 0;
      color: var(--text-muted);
    }
    .cmm-row.open .cmm-arrow { transform: rotate(90deg); }
    .cmm-subtoggle {
      flex-shrink: 0; font-size: .75em; opacity: .45; padding: 0 2px;
      transition: opacity .12s;
    }
    .cmm-subtoggle:hover { opacity: 1; }
    .cmm-body { display: none; padding: 4px 0 2px 22px; }
    .cmm-body.open { display: block; }
    .cmm-name-link { color: inherit; text-decoration: none; cursor: pointer; }
    .cmm-name-link:hover { text-decoration: underline; }
    .cmm-desc { font-weight: 400; color: var(--text-muted); }
    .cmm-right { display: flex; align-items: center; gap: 6px; margin-left: auto; flex-shrink: 0; }
    .cmm-meta { font-size: .8em; color: var(--text-faint); }
    .cmm-badge {
      font-size: .85em; font-weight: 700; padding: 2px 9px; border-radius: 10px;
      background: var(--background-modifier-border); color: var(--text-muted);
      border: 1px solid var(--background-modifier-border);
    }
    .cmm-badge.active { color: #fff; border-color: transparent; }
  `;
  document.head.appendChild(s);
}

// 5. 렌더링 — 재귀
const wrap = dv.container.createEl('div', { cls: 'cmm-wrap' });

const toolbar = wrap.createEl('div', { cls: 'cmm-toolbar' });
const filterInput = toolbar.createEl('input', {
  cls: 'cmm-filter-input',
  attr: { type: 'text', placeholder: '🔍 카테고리 필터링... (입력하면 실시간으로 좁혀집니다)' }
});
const expandAllBtn = toolbar.createEl('button', { cls: 'cmm-toolbtn', text: '⏷ 전체 펼치기' });
const collapseAllBtn = toolbar.createEl('button', { cls: 'cmm-toolbtn', text: '⏶ 전체 접기' });

function openNote(linkName) {
  app.workspace.openLinkText(linkName, cur, false);
}
function nameLink(parentEl, text, linkName) {
  const a = parentEl.createEl('span', { cls: 'cmm-name cmm-name-link', text });
  a.addEventListener('click', (e) => { e.stopPropagation(); openNote(linkName); });
  return a;
}

function subtreeAllOpen(node) {
  if (!node.children.length) return true;
  if (!node._el.row.classList.contains('open')) return false;
  return node.children.every(subtreeAllOpen);
}
function setSubtreeOpen(node, open) {
  if (!node.children.length) return;
  node._el.row.classList.toggle('open', open);
  node._el.body.classList.toggle('open', open);
  node.children.forEach(c => setSubtreeOpen(c, open));
}

const borderAlpha = (depth) => depth <= 1 ? 'ff' : depth === 2 ? '99' : '55';

function renderNode(node, parentEl, color) {
  node._searchText = (node.linkName + ' ' + node.desc).toLowerCase();
  const hasChildren = node.children.length > 0;

  const secEl = parentEl.createEl('div', { cls: 'cmm-sec' });
  const row = secEl.createEl('div', { cls: 'cmm-row' + (node.depth === 0 ? ' l0 open' : ' l1up') });

  if (node.depth === 0) {
    row.style.background = color + '1a';
    row.style.borderLeft = `4px solid ${color}`;
  } else {
    row.style.borderLeft = `3px solid ${color}${borderAlpha(node.depth)}`;
  }

  row.createEl('span', { cls: 'cmm-arrow', text: hasChildren ? '▶' : '·' });
  if (hasChildren) {
    const subBtn = row.createEl('span', {
      cls: 'cmm-subtoggle', text: '⏬',
      attr: { title: '이 카테고리 하위 전체 펼치기/접기' }
    });
    subBtn.addEventListener('click', (e) => {
      e.stopPropagation();
      setSubtreeOpen(node, !subtreeAllOpen(node));
    });
  }
  nameLink(row, node.linkName, node.linkName);
  if (node.desc) {
    row.createEl('span', { cls: 'cmm-desc', text: (node.depth === 0 ? ' — ' : ' ') + node.desc });
  }

  const right = row.createEl('span', { cls: 'cmm-right' });
  if (hasChildren) right.createEl('span', { cls: 'cmm-meta', text: node.children.length + '개 하위' });
  const cnt = getCount(node.linkName);
  const badge = right.createEl('span', {
    cls: 'cmm-badge' + (cnt > 0 ? ' active' : ''),
    text: cnt > 0 ? cnt + '개' : '—'
  });
  if (cnt > 0) badge.style.background = color;

  node._el = { row, body: null };

  if (hasChildren) {
    const body = secEl.createEl('div', { cls: 'cmm-body' + (node.depth === 0 ? ' open' : '') });
    node._el.body = body;
    row.addEventListener('click', () => {
      row.classList.toggle('open');
      body.classList.toggle('open');
    });
    for (const child of node.children) renderNode(child, body, color);
  }
}

roots.forEach((root, i) => renderNode(root, wrap, COLORS[i % COLORS.length]));

expandAllBtn.addEventListener('click', () => roots.forEach(r => setSubtreeOpen(r, true)));
collapseAllBtn.addEventListener('click', () => roots.forEach(r => setSubtreeOpen(r, false)));

// 6. 실시간 필터 — 재귀
function walkVisibility(node, q) {
  const selfMatch = node._searchText.includes(q);
  let childMatch = false;
  for (const c of node.children) if (walkVisibility(c, q)) childMatch = true;
  const visible = selfMatch || childMatch;
  node._el.row.parentElement.style.display = visible ? '' : 'none';
  if (childMatch && node._el.body) {
    node._el.row.classList.add('open');
    node._el.body.classList.add('open');
  }
  return visible;
}
function resetVisibility(node) {
  node._el.row.parentElement.style.display = '';
  node.children.forEach(resetVisibility);
}
function applyFilter(raw) {
  const q = raw.trim().toLowerCase();
  if (!q) { roots.forEach(resetVisibility); return; }
  roots.forEach(r => walkVisibility(r, q));
}
filterInput.addEventListener('input', (e) => applyFilter(e.target.value));
```
