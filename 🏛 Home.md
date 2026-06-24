---
cssclasses:
  - dashboard
tags:
Categories:
  - "[[📚630 KnowledgeManagement]]"
Indexes:
  - "[[🏛 Utils]]"
createdAt: 2026-06-14T00:00:00+09:00
updatedAt: 2026-06-24T23:48:19+09:00
modified: 2026-06-20T13:21:34+09:00
date_modified: 2026-06-24T07:36:11+09:00
date_created: 2026-06-23T21:03:53+09:00
---

```dataviewjs
const pages = dv.pages('"/" AND -"agent"');
const cur = dv.current().file.path;
const total = pages.length;
let links = 0; for (const p of pages) links += (p.file.outlinks ? p.file.outlinks.length : 0);
const orphan = pages.where(p => (!p.Categories || !p.Categories.length) && (!p.Indexes || !p.Indexes.length)).length;
let inboxFolder = 'private/00. Inbox';
try { const ap = JSON.parse(await app.vault.adapter.read('.obsidian/app.json')); if (ap?.newFileFolderPath) inboxFolder = ap.newFileFolderPath; } catch(e){}
const inbox = dv.pages('"' + inboxFolder + '"').length;
const catSet = new Set();
for (const p of pages) if (p.Categories) for (const c of p.Categories) catSet.add(c.path || String(c));
const now = dv.luxon.DateTime.now();
const today = now.setLocale('en').toFormat('yyyy-MM-dd (ccc)');
let calFolder = '10. Calendar Notes';
try { const pn = JSON.parse(await app.vault.adapter.read('.obsidian/plugins/periodic-notes/data.json')); if (pn?.daily?.folder) calFolder = pn.daily.folder; } catch(e){}
const todayPath = calFolder + '/' + now.toFormat('yyyy-MM-dd');
const todayNoteName = now.toFormat('yyyy-MM-dd');
async function openOrCreatePeriodic(type, noteName, fallback) {
  // Periodic Notes 명령을 우선 사용 → 템플릿 기반 열기/생성 보장 (오늘 기준 기간)
  const cmdMap = { daily: 'periodic-notes:open-daily-note', weekly: 'periodic-notes:open-weekly-note', monthly: 'periodic-notes:open-monthly-note', quarterly: 'periodic-notes:open-quarterly-note', yearly: 'periodic-notes:open-yearly-note' };
  const cmdId = cmdMap[type];
  if (cmdId && app.commands.commands?.[cmdId]) { app.commands.executeCommandById(cmdId); return; }
  // 폴백: 플러그인 명령 없을 때만 기존 노트/링크로
  const file = app.metadataCache.getFirstLinkpathDest(noteName, cur);
  if (file) { await app.workspace.getLeaf(false).openFile(file); return; }
  app.workspace.openLinkText(fallback || noteName, cur, false);
}
const _shell = (() => { try { return require('electron').shell; } catch(e) { return null; } })();
const _exec  = (() => { try { return require('child_process').exec; } catch(e) { return null; } })();

// ── 🔧 빠른 설정: 링크/폴더/URL 추가·제거 ────────────────
// type: 'note'    → Obsidian 내부 노트 링크
// type: 'folder'  → 로컬 폴더 (탐색기 + 터미널)
// type: 'url'     → 외부 URL (브라우저)
// type: 'divider' → 구분선
const FAVLINKS = [
  { type: 'note',    label: '🏛 Guides',      href: '🏛 Vault Guides' },
  { type: 'note',    label: '🏛 Tasks',        href: '🏛 Tasks' },
  { type: 'note',    label: '🏛 Categories',   href: '🏛 Categories' },
  { type: 'note',    label: '🏛 Cat.Tree',     href: '🏛 Categories Tree' },
  { type: 'note',    label: '🏛 Utils',        href: '🏛 Utils' },
  { type: 'note',    label: '🏛 Weekly',       href: '🏛 Weekly Notes' },
  { type: 'note',    label: '🏛 Subscription', href: '🏛 Expire•Subscription' },
  { type: 'note',    label: '🏷 Career',       href: '🏷 career' },
  { type: 'note',    label: '🏷 Health',       href: '🏷 health' },
  { type: 'note',    label: '🏷 Invest',       href: '🏷 invest' },
  { type: 'note',    label: '🏷 Manual',       href: '🏷 Manual' },
  { type: 'divider' },
  { type: 'folder',  label: 'mes-agent', path: 'D:\\GithubRepositories==s\\mes-agent' },
  { type: 'folder',  label: 'brain',     path: 'D:\\archive\\obsidian\\brain' },
  { type: 'url',     label: 'mes-agent ↗', url: 'https://github.com/BiohPark/mes-agent' },
];
const favHTML = FAVLINKS.map(f => {
  if (f.type === 'note')    return `<a class="internal-link" data-href="${f.href}">${f.label}</a>`;
  if (f.type === 'divider') return `<div class="dash-favbar-divider"></div>`;
  if (f.type === 'folder')  return `<span class="dash-favbar-chip"><a class="chip-text" data-local-exp="${f.path}">${f.label}</a><a class="chip-term" data-local-term="${f.path}">&gt;_</a></span>`;
  if (f.type === 'url')     return `<span class="dash-favbar-chip"><a class="chip-url" data-url-ext="${f.url}">${f.label}</a></span>`;
  return '';
}).join('');

const root = dv.el('div', '');
root.innerHTML = `
  <div class="dash-header">
    <div><span class="title">🏛 Home</span> &nbsp;<span class="sub">Second Brain</span></div>
    <div class="dash-header-right">
      <button class="dash-capture-btn" data-capture="1">➕ Quick Capture</button>
      <span class="date clickable" data-daily="${todayPath}">📅 ${today}</span>
    </div>
  </div>
  <div class="dash-favbar">
    ${favHTML}
  </div>`;
root.querySelectorAll('a.internal-link').forEach(a => a.onclick = (e) => { e.preventDefault(); app.workspace.openLinkText(a.dataset.href, cur, false); });
root.querySelectorAll('[data-search]').forEach(a => a.onclick = (e) => { e.preventDefault(); const s = app.internalPlugins.getPluginById('global-search'); if (s) s.instance.openGlobalSearch(a.dataset.search); });
root.querySelectorAll('[data-cmd]').forEach(a => a.onclick = (e) => { e.preventDefault(); app.commands.executeCommandById(a.dataset.cmd); });
root.querySelectorAll('[data-daily]').forEach(a => a.onclick = async (e) => { e.preventDefault(); await openOrCreatePeriodic('daily', todayNoteName, a.dataset.daily); });
root.querySelectorAll('[data-local-exp]').forEach(a => a.onclick = (e) => { e.preventDefault(); e.stopPropagation(); _shell ? _shell.openPath(a.dataset.localExp) : new Notice('데스크탑 앱에서만 지원됩니다.'); });
root.querySelectorAll('[data-local-term]').forEach(a => a.onclick = (e) => { e.preventDefault(); e.stopPropagation(); _exec ? _exec(`wt -d "${a.dataset.localTerm}"`) : new Notice('데스크탑 앱에서만 지원됩니다.'); });
root.querySelectorAll('[data-url-ext]').forEach(a => a.onclick = (e) => { e.preventDefault(); e.stopPropagation(); _shell ? _shell.openExternal(a.dataset.urlExt) : new Notice('데스크탑 앱에서만 지원됩니다.'); });
root.querySelectorAll('[data-capture]').forEach(b => b.onclick = async (e) => {
  e.preventDefault();
  // 1순위: Templater "새 노트 생성" 명령 (템플릿 선택 모달)
  const tmplCmds = ['templater-obsidian:create-new-note-from-template', 'templater-obsidian:insert-templater'];
  const avail = app.commands.commands || {};
  const cmd = tmplCmds.find(id => avail[id]);
  if (cmd) { app.commands.executeCommandById(cmd); return; }
  // 폴백: Inbox에 타임스탬프 노트 직접 생성 후 열기
  const ts = now.toFormat('yyyy-MM-dd-HHmmss');
  const path = `${inboxFolder}/${ts}.md`;
  const f = await app.vault.create(path, `---\ncreatedAt: ${now.toISO()}\ntags:\n---\n\n# \n`);
  app.workspace.getLeaf(false).openFile(f);
});
```
# 🟢 운영 (Operate)

```dataviewjs
// 북마크 칩 스트립 (bookmarks.json) — Pulse 쪽으로 이동
const cur = dv.current().file.path;
const _shell = (() => { try { return require('electron').shell; } catch(e) { return null; } })();
const esc = s => String(s ?? '').replace(/[&<>"]/g, c => ({'&':'&amp;','<':'&lt;','>':'&gt;','"':'&quot;'}[c]));
let bkHTML = '';
try {
  const bk = JSON.parse(await app.vault.adapter.read('.obsidian/bookmarks.json'));
  const renderItems = arr => arr.map(item => {
    if (item.type === 'file')   return `<a class="bk-item bk-file" data-href="${esc(item.path)}">📌 ${esc(item.title || item.path.split('/').pop().replace(/\.md$/, ''))}</a>`;
    if (item.type === 'search') return `<a class="bk-item bk-search" data-q="${esc(item.query)}">🔍 ${esc(item.title || item.query)}</a>`;
    if (item.type === 'folder') return `<a class="bk-item bk-folder" data-q='path:"${esc(item.path)}"'>📁 ${esc(item.title || item.path)}</a>`;
    if (item.type === 'url')    return `<a class="bk-item bk-url" data-u="${esc(item.url)}">🌐 ${esc(item.title || item.url)}</a>`;
    if (item.type === 'group')  return `<span class="bk-group-label">${esc(item.title || '')}</span>${renderItems(item.items || [])}`;
    return '';
  }).join('');
  const items = bk.items || [];
  bkHTML = items.length ? renderItems(items) : '<span class="hd-empty">북마크가 없습니다. Obsidian Bookmarks 패널에서 추가하세요.</span>';
} catch(e) { bkHTML = `<span class="hd-empty">북마크 오류: ${esc(e.message)}</span>`; }

const root = dv.el('div', '');
root.innerHTML = `<div class="dash-bk-strip">${bkHTML}</div>`;
root.querySelectorAll('[data-href]').forEach(a => a.onclick = e => { e.preventDefault(); app.workspace.openLinkText(a.dataset.href, cur, false); });
root.querySelectorAll('[data-q]').forEach(a => a.onclick = e => { e.preventDefault(); const s = app.internalPlugins.getPluginById('global-search'); if (s) s.instance.openGlobalSearch(a.dataset.q); });
root.querySelectorAll('[data-u]').forEach(a => a.onclick = e => { e.preventDefault(); _shell ? _shell.openExternal(a.dataset.u) : new Notice('데스크탑 앱에서만 지원됩니다.'); });
```
## Pulse
```dataviewjs
const pages = dv.pages('"/" AND -"agent"');
const cur = dv.current().file.path;
const DT = dv.luxon.DateTime;
const now = DT.now();
async function openOrCreatePeriodic(type, noteName, cur) {
  const file = app.metadataCache.getFirstLinkpathDest(noteName, cur);
  if (file) { await app.workspace.getLeaf(false).openFile(file); return; }
  const cmdMap = { daily: 'periodic-notes:open-daily-note', weekly: 'periodic-notes:open-weekly-note', monthly: 'periodic-notes:open-monthly-note', quarterly: 'periodic-notes:open-quarterly-note', yearly: 'periodic-notes:open-yearly-note' };
  const cmdId = cmdMap[type];
  if (cmdId && app.commands.commands?.[cmdId]) app.commands.executeCommandById(cmdId);
  else app.workspace.openLinkText(noteName, cur, false);
}

// ── Momentum (기존 로직) ──────────────────────────────
function cd(p){ const d=p.createdAt; if(!d) return p.file.ctime; return d.toFormat?d:DT.fromISO(String(d)); }
const wStart = now.startOf('week');
const wPrev  = wStart.minus({weeks:1});
const mStart = now.startOf('month');
const todayK = now.toFormat('yyyy-MM-dd');
const weekNote = now.toFormat("kkkk-'W'WW");
const monthNote = now.toFormat("yyyy-MM");
const todayNote = now.toFormat("yyyy-MM-dd");
const qStart      = now.startOf('quarter');
const yStart      = now.startOf('year');
const quarterNote = now.toFormat("yyyy-'Q'Q");
const yearNote    = now.toFormat("yyyy");
let thisWeek=0,lastWeek=0,thisMonth=0,todayMod=0,thisQuarter=0,thisYear=0;
for(const p of pages){
  const c=cd(p); if(c&&c.isValid){
    if(c>=wStart) thisWeek++; else if(c>=wPrev) lastWeek++;
    if(c>=mStart) thisMonth++;
    if(c>=qStart) thisQuarter++;
    if(c>=yStart) thisYear++;
  }
  const md=p.updatedAt; const mk=md&&md.toFormat?md.toFormat('yyyy-MM-dd'):null;
  if(mk===todayK) todayMod++;
}
const delta=thisWeek-lastWeek;
const dCls=delta>0?'up':delta<0?'down':'flat';
const dArrow=delta>0?`▲+${delta}`:delta<0?`▼${delta}`:'― 0';

// ── Tasks 5분류 (기존 로직) ───────────────────────────
const today=DT.now().startOf('day');
let overdue=0,inprogress=0,recurring=0,backlog=0,done=0;
for(const p of pages){ for(const t of p.file.tasks){
  if(!t.tags || !t.tags.includes('#task')) continue;
  const text=t.text||'';
  if(t.completed){
    const m=text.match(/✅\s*(\d{4}-\d{2}-\d{2})/);
    if(m){ const d=DT.fromISO(m[1]); if(d.isValid && d>=wStart) done++; }
    continue;
  }
  if(/🔁/.test(text)){ recurring++; continue; }
  const due=text.match(/📅\s*(\d{4}-\d{2}-\d{2})/);
  const sched=text.match(/⏳\s*(\d{4}-\d{2}-\d{2})/);
  const start=text.match(/🛫\s*(\d{4}-\d{2}-\d{2})/);
  const dueD=due&&DT.fromISO(due[1]), schedD=sched&&DT.fromISO(sched[1]);
  const isOverdue=(dueD&&dueD.isValid&&dueD<today)||(schedD&&schedD.isValid&&schedD<today);
  if(isOverdue){ overdue++; }
  else if(due||sched||start){ inprogress++; }
  else { backlog++; }
} }

// ── Recently Edited (기존 dataview TABLE과 동일 조건, 10개) ──
const recentArr=[];
for(const p of pages){
  const md=p.updatedAt;
  if(md && md.toFormat && !p.file.folder.includes('30. Secrit Notes')) recentArr.push(p);
}
recentArr.sort((a,b)=>b.updatedAt.toMillis()-a.updatedAt.toMillis());
const recent=recentArr.slice(0,10);

const root=dv.el('div','',{cls:'dash-pulse-strip'});
root.innerHTML=`
  <div class="dash-pulse-seg dash-pulse-momentum">
    <div class="dash-pulse-label">Momentum</div>
    <a class="mom-link" data-periodic="weekly" data-note="${weekNote}">
      <div class="mom-num hero">${thisWeek}<span class="mom-delta ${dCls}">${dArrow}</span></div>
      <div class="mom-label">New This Week</div>
    </a>
    <div class="dash-pulse-mini">
      <a class="mom-link" data-periodic="daily" data-note="${todayNote}"><div class="mom-num mini">${todayMod}</div><div class="mom-label">Edited Today</div></a>
      <a class="mom-link" data-periodic="monthly" data-note="${monthNote}"><div class="mom-num mini">${thisMonth}</div><div class="mom-label">This Month</div></a>
      <a class="mom-link" data-periodic="quarterly" data-note="${quarterNote}"><div class="mom-num mini">${thisQuarter}</div><div class="mom-label">This Quarter</div></a>
      <a class="mom-link" data-periodic="yearly" data-note="${yearNote}"><div class="mom-num mini">${thisYear}</div><div class="mom-label">This Year</div></a>
    </div>
  </div>
  <div class="dash-pulse-seg dash-pulse-tasks">
    <div class="dash-pulse-label">Tasks</div>
    <div class="dash-taskbars">
      <a class="dash-taskpill overdue internal-link" data-href="🏛 Tasks#🔴 지연"><div class="tnum">${overdue}</div><div class="tlabel">지연</div></a>
      <a class="dash-taskpill inprogress internal-link" data-href="🏛 Tasks#🔵 진행중"><div class="tnum">${inprogress}</div><div class="tlabel">진행중</div></a>
      <a class="dash-taskpill recurring internal-link" data-href="🏛 Tasks#🔁 반복"><div class="tnum">${recurring}</div><div class="tlabel">반복</div></a>
      <a class="dash-taskpill backlog internal-link" data-href="🏛 Tasks#⚠️ 미정 — 확인하세요"><div class="tnum">${backlog}</div><div class="tlabel">미정</div></a>
      <a class="dash-taskpill done internal-link" data-href="🏛 Tasks#✅ Done"><div class="tnum">${done}</div><div class="tlabel">Done</div></a>
    </div>
    <a class="card-link internal-link" data-href="🏛 Tasks">🏛 All Tasks →</a>
  </div>
  <div class="dash-pulse-seg dash-pulse-recent">
    <div class="dash-pulse-label">Recent</div>
    <div class="dash-pulse-ticker">${recent.length ? recent.map(p=>`<a class="internal-link" data-href="${p.file.path}">${p.file.name}</a>`).join('<span class="dash-pulse-dot">·</span>') : '<span class="hd-empty">No recent edits</span>'}</div>
    <a class="card-link internal-link" data-href="🔍 최근 수정 노트">See all →</a>
  </div>`;
root.querySelectorAll('[data-periodic]').forEach(a=>a.onclick=async(e)=>{e.preventDefault();await openOrCreatePeriodic(a.dataset.periodic,a.dataset.note,cur);});
root.querySelectorAll('a.internal-link').forEach(a=>a.onclick=(e)=>{e.preventDefault();app.workspace.openLinkText(a.dataset.href,cur,false);});
```
```tasks
not done
sort by priority, due
limit 10
```

## 🚀 Workspace
<!-- 데이터 출처: Obsidian Bookmarks + project-manager 플러그인 (https://github.com/StepanKropachev/obsidian-pm) -->
```dataviewjs
const cur = dv.current().file.path;
const DT = dv.luxon.DateTime;
const today = DT.now().startOf('day');
const _shell = (() => { try { return require('electron').shell; } catch(e) { return null; } })();

// ── 설정: 플러그인 data.json에서 폴더·상태색·임박일수 읽기 (실패 시 폴백) ──
let cfg = {};
try { cfg = JSON.parse(await app.vault.adapter.read('.obsidian/plugins/project-manager/data.json')); } catch(e){}
const folder = cfg.projectsFolder || 'private/20. Projects';
const leadDays = cfg.notificationLeadDays ?? 2;
const SM = {}; (cfg.statuses || []).forEach(s => SM[s.id] = s);
const FALLBACK = {
  'todo':        { label:'To Do',       color:'#8a94a0', complete:false },
  'in-progress': { label:'In Progress', color:'#8b72be', complete:false },
  'blocked':     { label:'Blocked',     color:'#c47070', complete:false },
  'review':      { label:'In Review',   color:'#b8a06b', complete:false },
  'done':        { label:'Done',        color:'#79b58d', complete:true  },
  'cancelled':   { label:'Cancelled',   color:'#767491', complete:true  },
};
const ORDER = ['todo','in-progress','blocked','review','done','cancelled'];
const meta = id => SM[id] || FALLBACK[id] || { label:id||'?', color:'#8a94a0', complete:false };
const parseDue = d => { if(!d) return null; if(d.toFormat) return d.startOf('day'); const x=DT.fromISO(String(d)); return x.isValid ? x.startOf('day') : null; };
const esc = s => String(s ?? '').replace(/[&<>"]/g, c => ({'&':'&amp;','<':'&lt;','>':'&gt;','"':'&quot;'}[c]));

// ── 데이터 수집 ──
const pages = dv.pages(`"${folder}"`);
const projects = pages.where(p => p['pm-project']).array();
const allTasks = pages.where(p => p['pm-task']).array();

const models = projects.map(pr => {
  const tasks = allTasks.filter(t => t.projectId === pr.id);
  const counts = {}; ORDER.forEach(s => counts[s] = 0);
  let overdue = 0, dueSoon = 0, nextDue = null;
  for (const t of tasks) {
    const st = t.status || 'todo';
    counts[st] = (counts[st] || 0) + 1;
    const due = parseDue(t.due);
    if (due && !meta(st).complete) {
      if (due < today) overdue++;
      else if (due <= today.plus({ days: leadDays })) dueSoon++;
      if (!nextDue || due < nextDue) nextDue = due;
    }
  }
  const total = tasks.filter(t => (t.status||'todo') !== 'cancelled').length;
  const done = counts['done'];
  const pct = total ? Math.round(done / total * 100) : 0;
  const top = tasks.filter(t => !t.parentId);
  const isComplete = total > 0 && done === total;
  return { pr, counts, total, done, pct, active: total - done, overdue, dueSoon, nextDue, isComplete, top };
});

const activeM = models.filter(m => !m.isComplete);
const doneM   = models.filter(m => m.isComplete);
activeM.sort((a,b) =>
  b.overdue - a.overdue ||
  (a.nextDue ? a.nextDue.toMillis() : Infinity) - (b.nextDue ? b.nextDue.toMillis() : Infinity) ||
  a.pct - b.pct);

const sumTotal   = models.reduce((s,m) => s + m.total, 0);
const sumDone    = models.reduce((s,m) => s + m.done, 0);
const overallPct = sumTotal ? Math.round(sumDone / sumTotal * 100) : 0;
const sumActive  = models.reduce((s,m) => s + m.active, 0);
const sumOverdue = models.reduce((s,m) => s + m.overdue, 0);
const sumSoon    = models.reduce((s,m) => s + m.dueSoon, 0);

// ── 렌더 헬퍼 ──
const dueBadge = m =>
  m.overdue ? `<span class="dash-due overdue">🔴 지연 ${m.overdue}</span>` :
  m.dueSoon ? `<span class="dash-due soon">🟠 임박 ${m.dueSoon}</span>` :
  m.nextDue ? `<span class="dash-due">📅 ${m.nextDue.toFormat('MM/dd')}</span>` : '';

const segBar = m => {
  const segs = ORDER.filter(s => s !== 'cancelled' && m.counts[s] > 0).map(s => {
    const mt = meta(s);
    return `<span class="dash-seg" style="flex:${m.counts[s]};background:${mt.color}" title="${esc(mt.label)}: ${m.counts[s]}"></span>`;
  }).join('');
  return `<div class="dash-bar-track dash-seg-track">${segs || '<span class="dash-seg" style="flex:1;background:var(--background-modifier-border)"></span>'}</div>`;
};

const pills = m => ORDER.filter(s => m.counts[s] > 0).map(s => {
  const mt = meta(s);
  return `<span class="dash-pill" style="--pc:${mt.color}">${esc(mt.label)} <b>${m.counts[s]}</b></span>`;
}).join('');

const taskRows = m => m.top.map(t => {
  const mt = meta(t.status || 'todo');
  const due = parseDue(t.due);
  const dtxt = due ? `<span class="trow-due">${due.toFormat('MM/dd')}</span>` : '';
  const tic = t.type === 'milestone' ? '◆ ' : '';
  return `<div class="dash-proj-trow" data-task="${esc(t.file.path)}"><span class="trow-dot" style="background:${mt.color}" title="${esc(mt.label)}"></span><span class="trow-title">${tic}${esc(t.title || t.file.name)}</span>${dtxt}</div>`;
}).join('') || '<div class="hd-empty">하위 task 없음</div>';

const card = (m, complete) => {
  const pr = m.pr;
  return `<div class="dash-card dash-proj${complete ? ' is-complete' : ''}" style="--proj-color:${pr.color || 'var(--d-purple)'}">
    <div class="dash-proj-head">
      <span class="dash-proj-dot"></span>
      <a class="card-title dash-proj-title" data-href="${esc(pr.file.path)}">${esc(pr.icon || '📁')} ${esc(pr.title || pr.file.name)}</a>
      ${dueBadge(m)}
    </div>
    <div class="dash-proj-bar">${segBar(m)}<span class="dash-proj-pct">${m.pct}%<small>${m.done}/${m.total}</small></span></div>
    <div class="dash-proj-pills">${pills(m)}</div>
    <details class="dash-proj-details"><summary>task ${m.top.length}개</summary><div class="dash-proj-tasklist">${taskRows(m)}</div></details>
  </div>`;
};

// ── 프로젝트 출력 ──
let projHTML;
if (!projects.length) {
  projHTML = `<div class="hd-empty">프로젝트가 없습니다. <a class="card-link" data-cmd="project-manager:open-projects">📋 보드 열기</a></div>`;
} else {
  projHTML = `
    <div class="dash-proj-actionbar">
      <a class="card-link" data-cmd="project-manager:open-projects">📋 보드 열기</a>
      <a class="card-link" data-cmd="project-manager:open-projects">＋ 새 프로젝트</a>
    </div>
    <div class="dash-quickstats">
      <a class="dash-statchip" data-cmd="project-manager:open-projects"><span class="num">${models.length}</span><span class="label">Projects</span></a>
      <a class="dash-statchip" data-cmd="project-manager:open-projects"><span class="num">${overallPct}%</span><span class="label">Overall</span></a>
      <a class="dash-statchip" data-cmd="project-manager:open-projects"><span class="num">${sumOverdue}</span><span class="label">Overdue</span></a>
      <a class="dash-statchip" data-cmd="project-manager:open-projects"><span class="num">${sumActive}</span><span class="label">Active Tasks</span></a>
      <a class="dash-statchip" data-cmd="project-manager:open-projects"><span class="num">${sumSoon}</span><span class="label">Due ≤${leadDays}d</span></a>
    </div>
    <div class="dash-grid dash-proj-grid">${activeM.map(m => card(m, false)).join('')}</div>
    ${doneM.length ? `<details class="dash-proj-done"><summary>✅ 완료·취소 ${doneM.length}개</summary><div class="dash-grid dash-proj-grid">${doneM.map(m => card(m, true)).join('')}</div></details>` : ''}`;
}

const root = dv.el('div', '');
root.innerHTML = `${projHTML}`;

root.querySelectorAll('[data-href]').forEach(a => a.onclick = e => { e.preventDefault(); app.workspace.openLinkText(a.dataset.href, cur, false); });
root.querySelectorAll('[data-task]').forEach(a => a.onclick = e => { e.preventDefault(); app.workspace.openLinkText(a.dataset.task, cur, false); });
root.querySelectorAll('[data-cmd]').forEach(a => a.onclick = e => { e.preventDefault(); const id = a.dataset.cmd; if (app.commands.commands?.[id]) app.commands.executeCommandById(id); else new Notice('명령을 찾을 수 없습니다: ' + id); });
root.querySelectorAll('[data-q]').forEach(a => a.onclick = e => { e.preventDefault(); const s = app.internalPlugins.getPluginById('global-search'); if (s) s.instance.openGlobalSearch(a.dataset.q); });
root.querySelectorAll('[data-u]').forEach(a => a.onclick = e => { e.preventDefault(); _shell ? _shell.openExternal(a.dataset.u) : new Notice('데스크탑 앱에서만 지원됩니다.'); });
```

# 🔵 회고 (Reflect)

## Activity Heatmap
```dataviewjs
const pages = dv.pages('"/" AND -"agent"');
const cur = dv.current().file.path;
const DT = dv.luxon.DateTime;
let calFolder = '10. Calendar Notes';
try { const pn = JSON.parse(await app.vault.adapter.read('.obsidian/plugins/periodic-notes/data.json')); if (pn?.daily?.folder) calFolder = pn.daily.folder; } catch(e){}
function key(d){ if(!d) return null; if(d.toFormat) return d.toFormat('yyyy-MM-dd'); const x=DT.fromISO(String(d)); return x.isValid?x.toFormat('yyyy-MM-dd'):null; }
const counts={}, byDay={};
for(const p of pages){
  const k = key(p.updatedAt) || key(p.createdAt) || key(p.file.mtime);
  if(!k) continue;
  counts[k]=(counts[k]||0)+1;
  if(!p.file.folder.includes('30. Secrit Notes')) (byDay[k]=byDay[k]||[]).push(p);
}
const weeks=26, days=weeks*7;
const todayD = DT.now().startOf('day');
let max=0; const cells=[];
for(let i=days-1;i>=0;i--){ const d=todayD.minus({days:i}); const c=counts[d.toFormat('yyyy-MM-dd')]||0; if(c>max)max=c; cells.push({d,c}); }
const pal=['#222a4d','#33356e','#4f4bb0','#7b6ee8','#a784ee'];
function color(c){ if(c<=0) return 'var(--background-modifier-border)'; const r=c/Math.max(1,max); return r<.25?pal[0]:r<.5?pal[1]:r<.75?pal[2]:r<1?pal[3]:pal[4]; }

// column-major slot 배열 (앞쪽 pad = 빈칸)
const pad=cells[0].d.weekday%7;          // 0=일 … 6=토
const slots=[]; for(let i=0;i<pad;i++) slots.push(null); for(const cell of cells) slots.push(cell);
const cols=Math.ceil(slots.length/7);

// 상단 월 라벨 (월이 바뀌는 컬럼에만)
const mNames=['Jan','Feb','Mar','Apr','May','Jun','Jul','Aug','Sep','Oct','Nov','Dec'];
let monthHtml=''; let lastM=-1;
for(let c=0;c<cols;c++){
  let m=null; for(let r=0;r<7;r++){ const s=slots[c*7+r]; if(s&&s.d){ m=s.d.month; break; } }
  if(m!==null && m!==lastM){ monthHtml+=`<span class="hm-month" style="grid-column:${c+1}">${mNames[m-1]}</span>`; lastM=m; }
}
// 좌측 요일 라벨 (월·수·금)
const wd=['','Mon','','Wed','','Fri','']; let wdHtml='';
for(let r=0;r<7;r++) wdHtml+=`<span class="hm-wd">${wd[r]}</span>`;
// 셀
let cellHtml='';
for(const s of slots){
  if(!s){ cellHtml+='<span class="dash-hm-cell" style="visibility:hidden"></span>'; continue; }
  const ds=s.d.toFormat('yyyy-MM-dd');
  cellHtml+=`<span class="dash-hm-cell" data-date="${ds}" style="background:${color(s.c)}" aria-label="${ds}: ${s.c}개"></span>`;
}

const root=dv.el('div','',{cls:'dash-heatmap-wrap'});
root.innerHTML=`
  <div class="dash-hm-months">${monthHtml}</div>
  <div class="dash-hm-body">
    <div class="dash-hm-weekdays">${wdHtml}</div>
    <div class="dash-heatmap">${cellHtml}</div>
  </div>
  <div class="dash-hm-legend">Last ${weeks} weeks · click a cell to see that day's notes &nbsp; Less <i style="background:${pal[0]}"></i><i style="background:${pal[1]}"></i><i style="background:${pal[2]}"></i><i style="background:${pal[3]}"></i><i style="background:${pal[4]}"></i> More</div>
  <div class="dash-hm-detail" style="display:none"></div>`;
root.querySelector('.dash-heatmap').style.gridTemplateColumns='repeat('+cols+',1fr)';
root.querySelector('.dash-hm-months').style.gridTemplateColumns='repeat('+cols+',1fr)';
const detail=root.querySelector('.dash-hm-detail');
root.querySelector('.dash-heatmap').addEventListener('click',(e)=>{
  const cell=e.target.closest('.dash-hm-cell'); if(!cell||!cell.dataset.date) return;
  const ds=cell.dataset.date;
  const list=byDay[ds]||[];
  const dailyPath=calFolder+'/'+ds;
  let h=`<div class="hd-head">📅 ${ds} <span class="hd-daily" data-daily="${dailyPath}">open daily note →</span></div>`;
  h += list.length
    ? '<ul>'+list.map(p=>`<li><a class="internal-link" data-href="${p.file.path}">${p.file.name}</a></li>`).join('')+'</ul>'
    : '<div class="hd-empty">No notes created/edited on this day.</div>';
  detail.innerHTML=h; detail.style.display='block';
  detail.querySelectorAll('a.internal-link').forEach(a=>a.onclick=(ev)=>{ev.preventDefault();app.workspace.openLinkText(a.dataset.href,cur,false);});
  detail.querySelectorAll('[data-daily]').forEach(a=>a.onclick=(ev)=>{ev.preventDefault();app.workspace.openLinkText(a.dataset.daily,'',false);});
});
```
```dataviewjs
const pages = dv.pages('"/" AND -"agent"');
const cur = dv.current().file.path;
const total = pages.length;
let links = 0; for (const p of pages) links += (p.file.outlinks ? p.file.outlinks.length : 0);
const orphan = pages.where(p => (!p.Categories || !p.Categories.length) && (!p.Indexes || !p.Indexes.length)).length;
const catSet = new Set();
for (const p of pages) if (p.Categories) for (const c of p.Categories) catSet.add(c.path || String(c));
let inboxFolder = 'private/00. Inbox';
try { const ap = JSON.parse(await app.vault.adapter.read('.obsidian/app.json')); if (ap?.newFileFolderPath) inboxFolder = ap.newFileFolderPath; } catch(e){}
const inbox = dv.pages('"' + inboxFolder + '"').length;

const root = dv.el('div', '');
root.innerHTML = `
  <div class="dash-quickstats">
    <a class="dash-statchip internal-link" data-href="🔍 카테고리별 노트"><span class="num">${total}</span><span class="label">Total Notes</span></a>
    <a class="dash-statchip" data-cmd="graph:open"><span class="num">${links}</span><span class="label">Links</span></a>
    <a class="dash-statchip warn internal-link" data-href="🔍 고립·미분류 노트"><span class="num">${orphan}</span><span class="label">Orphans</span></a>
    <a class="dash-statchip internal-link" data-href="🏛 Categories"><span class="num">${catSet.size}</span><span class="label">Categories</span></a>
    <a class="dash-statchip warn" data-search='path:"${inboxFolder}"'><span class="num">${inbox}</span><span class="label">Inbox</span></a>
  </div>`;
root.querySelectorAll('a.internal-link').forEach(a => a.onclick = (e) => { e.preventDefault(); app.workspace.openLinkText(a.dataset.href, cur, false); });
root.querySelectorAll('[data-search]').forEach(a => a.onclick = (e) => { e.preventDefault(); const s = app.internalPlugins.getPluginById('global-search'); if (s) s.instance.openGlobalSearch(a.dataset.search); });
root.querySelectorAll('[data-cmd]').forEach(a => a.onclick = (e) => { e.preventDefault(); app.commands.executeCommandById(a.dataset.cmd); });
```
## Note Growth

```dataviewjs
const pages = dv.pages('"/" AND -"agent"');
const DT = dv.luxon.DateTime;

// ── 월별 집계 (생성/수정/링크/고립/카테고리/요일) ──────
const monthly={}, mod={}, linksByMonth={}, orphanByMonth={}, catMonthly={}, dailyByMonth={};
const weekdayCnt=[0,0,0,0,0,0,0];   // 월..일
for(const p of pages){
  const cdt = (p.createdAt && p.createdAt.toFormat) ? p.createdAt : (p.file.ctime||null);
  const cm = cdt ? cdt.toFormat('yyyy-MM') : null;
  if(cm){
    monthly[cm]=(monthly[cm]||0)+1;
    linksByMonth[cm]=(linksByMonth[cm]||0)+(p.file.outlinks?p.file.outlinks.length:0);
    if((!p.Categories||!p.Categories.length)&&(!p.Indexes||!p.Indexes.length)) orphanByMonth[cm]=(orphanByMonth[cm]||0)+1;
    if(p.Categories) for(const c of p.Categories){ const nm=(c.path||String(c)).split('/').pop().replace(/\.md$/,''); (catMonthly[nm]=catMonthly[nm]||{}); catMonthly[nm][cm]=(catMonthly[nm][cm]||0)+1; }
    if(cdt.weekday) weekdayCnt[cdt.weekday-1]++;
  }
  const md=(p.updatedAt&&p.updatedAt.toFormat)?p.updatedAt:null;
  const mm=md?md.toFormat('yyyy-MM'):null; if(mm) mod[mm]=(mod[mm]||0)+1;
  if(/^\d{4}-\d{2}-\d{2}$/.test(p.file.name)){ const dm=p.file.name.slice(0,7); dailyByMonth[dm]=(dailyByMonth[dm]||0)+1; }   // 데일리 노트
}
const labels=[...new Set([...Object.keys(monthly),...Object.keys(mod),...Object.keys(dailyByMonth)])].sort();
const newCounts=labels.map(l=>monthly[l]||0);
let run=0; const cum=labels.map(l=>{run+=(monthly[l]||0);return run;});
const modCounts=labels.map(l=>mod[l]||0);
const dailyCounts=labels.map(l=>dailyByMonth[l]||0);
let lrun=0; const cumLinks=labels.map(l=>{lrun+=(linksByMonth[l]||0);return lrun;});
const avgLinks=labels.map((l,i)=>cum[i]?+(cumLinks[i]/cum[i]).toFixed(2):0);
const cohortHealth=labels.map(l=>{const n=monthly[l]||0;return n?Math.round((orphanByMonth[l]||0)/n*100):0;});
const catTotals=Object.entries(catMonthly).map(([nm,mo])=>[nm,Object.values(mo).reduce((a,b)=>a+b,0)]).sort((a,b)=>b[1]-a[1]);
const topCats=catTotals.slice(0,5).map(e=>e[0]);
function catCum(nm){let r=0;return labels.map(l=>{r+=((catMonthly[nm]||{})[l]||0);return r;});}
const topCatCums=topCats.map(catCum);
const etcCum=labels.map((l,i)=>cum[i]-topCatCums.reduce((s,arr)=>s+arr[i],0));
const wdLabels=['Mon','Tue','Wed','Thu','Fri','Sat','Sun'];

// ── 테마 색상 ─────────────────────────────────────────
const css=getComputedStyle(document.body);
const V=(n,f)=>{const v=css.getPropertyValue(n).trim();return v||f;};
// 블루 → 퍼플 세련된 램프 (차트 전용 톤)
const cBlue='#6ea8fe', cPurple='#a784ee', cTeal='#69c8ff', cGreen='#7fe0d4', cAmber='#c3b1ff', cRed='#e08ad0';
const cTxt=V('--text-muted','#9aa5ce'), cGrid=V('--background-modifier-border','#2a2e42');
const palette=['#6ea8fe','#7f9cf5','#8a8df0','#9b86ec','#a784ee','#b88ee8','#69c8ff','#c9b3ff'];
const hex2rgba=(h,a)=>{const m=h.replace('#','');const b=m.length===3?m.split('').map(x=>x+x).join(''):m;const n=parseInt(b,16);return `rgba(${(n>>16)&255},${(n>>8)&255},${n&255},${a})`;};
// 세로 그라데이션 (반짝이는 영역 채우기)
function grad(context, hex, a0, a1){ a0=a0??0.5; a1=a1??0.02; const ch=context.chart, area=ch.chartArea; if(!area) return hex2rgba(hex,a0); const g=ch.ctx.createLinearGradient(0,area.top,0,area.bottom); g.addColorStop(0,hex2rgba(hex,a0)); g.addColorStop(1,hex2rgba(hex,a1)); return g; }

function baseOpts(extra={}){return {
  responsive:true, maintainAspectRatio:false,
  interaction:{mode:'index',intersect:false},
  layout:{padding:{top:4}},
  plugins:{ legend:{ labels:{color:cTxt,boxWidth:10,usePointStyle:true,pointStyle:'circle',padding:14,font:{size:11}} },
            tooltip:{ padding:10, cornerRadius:8, usePointStyle:true, backgroundColor:'rgba(20,22,34,.92)', titleColor:'#fff', bodyColor:'#cbd2ee', borderColor:cGrid, borderWidth:1 } },
  scales:{ x:{ grid:{display:false}, ticks:{color:cTxt,maxTicksLimit:12,autoSkip:true,font:{size:10}}, border:{color:cGrid} } },
  animation:{duration:500,easing:'easeOutQuart'},
  ...extra };}
function yAxis(o){return Object.assign({beginAtZero:true,grid:{color:cGrid,drawTicks:false},ticks:{color:cTxt,font:{size:10},padding:6},border:{display:false}},o);}

const caps={
  all:'New · Edited · Cumulative · Links · Health in one chart — toggle series via the legend',
  cat:'Cumulative growth by field — which areas grew (top 5 + Other)',
  act:'Created vs Edited — writing new, or refining existing',
  pat:'Writing habits by weekday — when you record the most',
};
function specFor(view){
  if(view==='cat'){
    const ds=topCats.map((nm,i)=>({label:nm,data:topCatCums[i],borderColor:palette[i],backgroundColor:(c)=>grad(c,palette[i],.55,.05),fill:true,tension:.35,pointRadius:0,borderWidth:1.5}));
    ds.push({label:'Other',data:etcCum,borderColor:'#5a6488',backgroundColor:(c)=>grad(c,'#8890b8',.28,.03),fill:true,tension:.35,pointRadius:0,borderWidth:1});
    return {type:'line',data:{labels,datasets:ds},options:baseOpts({scales:{x:baseOpts().scales.x,y:yAxis({stacked:true})}})};
  }
  if(view==='act') return {type:'line',data:{labels,datasets:[
      {label:'월 생성',data:newCounts,borderColor:cBlue,backgroundColor:(c)=>grad(c,cBlue,.28,.02),fill:true,tension:.35,pointRadius:0,borderWidth:2.5},
      {label:'월 수정',data:modCounts,borderColor:cPurple,backgroundColor:(c)=>grad(c,cPurple,.22,.02),fill:true,tension:.35,pointRadius:0,borderWidth:2.5}]},
      options:baseOpts({scales:{x:baseOpts().scales.x,y:yAxis({})}})};
  if(view==='pat') return {type:'bar',data:{labels:wdLabels,datasets:[
      {label:'생성 노트',data:weekdayCnt,backgroundColor:(c)=>grad(c,(c.dataIndex>=5?cPurple:cBlue),.95,.45),borderRadius:6,borderSkipped:false}]},
      options:baseOpts({plugins:{legend:{display:false}},scales:{x:{grid:{display:false},ticks:{color:cTxt}},y:yAxis({})}})};
  // all (default) — 전체 시리즈, 일부는 기본 숨김(범례 토글)
  return {type:'bar',data:{labels,datasets:[
      {type:'bar',label:'Daily Notes',data:dailyCounts,borderColor:'#ff922b',backgroundColor:(c)=>grad(c,'#ff922b',.85,.3),borderRadius:4,borderSkipped:false,yAxisID:'yC',order:3},
      {type:'bar',label:'Edited',data:modCounts,borderColor:cTeal,backgroundColor:(c)=>grad(c,cTeal,.75,.25),borderRadius:4,borderSkipped:false,yAxisID:'yC',order:4},
      {type:'bar',label:'New',data:newCounts,borderColor:cBlue,backgroundColor:(c)=>grad(c,cBlue,.95,.4),borderRadius:4,borderSkipped:false,yAxisID:'yC',order:9},
      {type:'line',label:'Cumulative',data:cum,borderColor:cPurple,backgroundColor:(c)=>grad(c,cPurple,.32,.02),fill:true,tension:.35,pointRadius:0,borderWidth:2.5,yAxisID:'yK',order:8},
      {type:'line',label:'Cum. Links',data:cumLinks,fill:true,borderColor:cGreen,backgroundColor:(c)=>grad(c,cGreen,.32,.02),fill:true,tension:.3,pointRadius:0,borderWidth:2,yAxisID:'yK',order:7,hidden:true},
      {type:'line',label:'Avg Links/Note',data:avgLinks,borderColor:'#c9b3ff',backgroundColor:'transparent',tension:.3,pointRadius:0,borderWidth:2,borderDash:[5,3],yAxisID:'yAvg',order:6,hidden:true},
      {type:'line',label:'Unclassified %',data:cohortHealth,borderColor:cRed,backgroundColor:'transparent',tension:.3,pointRadius:0,borderWidth:2,borderDash:[2,2],yAxisID:'yPct',order:5,hidden:true}]},
    options:baseOpts({scales:{ x:baseOpts().scales.x,
      yC:yAxis({position:'left'}),
      yK:yAxis({position:'right',grid:{drawOnChartArea:false}}),
      yAvg:yAxis({position:'right',display:false}),
      yPct:yAxis({position:'right',display:false,min:0,max:100}) }})};
}

const card=dv.el('div','',{cls:'dash-card dash-chart-full'});
card.innerHTML=`<div class="card-head"><span class="card-title">📈 Note Growth</span>
  <span class="dash-seg">
    <button class="dash-seg-btn active" data-view="all">🔀 All</button>
    <button class="dash-seg-btn" data-view="cat">🗂 By Field</button>
    <button class="dash-seg-btn" data-view="act">⚡ Activity</button>
    <button class="dash-seg-btn" data-view="pat">📅 Pattern</button>
  </span></div>
  <div class="dash-chart-cap"></div>`;
const cap=card.querySelector('.dash-chart-cap');
const wrap=card.createDiv({cls:'dash-chart-area'});

async function render(view){
  cap.textContent=caps[view]||'';
  wrap.empty();
  if(window.renderChart){ await window.renderChart(specFor(view), wrap); return; }
  wrap.innerHTML=`<div class="dash-chart-empty">📊<div>Enable the Charts plugin<br>to display graphs</div></div>`;
}
card.querySelectorAll('.dash-seg-btn').forEach(b=>b.onclick=()=>{
  card.querySelectorAll('.dash-seg-btn').forEach(x=>x.classList.remove('active'));
  b.classList.add('active'); render(b.dataset.view);
});
await render('all');
```

## Knowledge Structure

```dataviewjs
const pages = dv.pages('"/" AND -"agent"');
const cur = dv.current().file.path;
function openSearch(q){ const s=app.internalPlugins.getPluginById('global-search'); if(s) s.instance.openGlobalSearch(q); }
function bar(label,val,max,act){
  return `<div class="dash-bar-row${act?' clickable':''}"${act?` data-act="${act.replace(/"/g,'&quot;')}"`:''} title="${label} (${val})">
    <span class="dash-bar-label">${label}</span>
    <span class="dash-bar-track"><span class="dash-bar-fill" style="width:${Math.round(val/Math.max(1,max)*100)}%"></span></span>
    <span class="dash-bar-val">${val}</span></div>`;
}

// ── 데이터: 카테고리 / 태그 / 폴더 → {label,val,act} 정규화 ──
const catMap={};
for(const p of pages) if(p.Categories) for(const c of p.Categories){ const path=c.path||String(c); const name=path.split('/').pop().replace(/\.md$/,''); if(!catMap[name]) catMap[name]={val:0,path}; catMap[name].val++; }
const catItems=Object.entries(catMap).sort((a,b)=>b[1].val-a[1].val).map(([name,o])=>({label:name,val:o.val,act:'o|'+o.path}));

const tagCounts={};
for(const p of pages){ const t=p.file.etags; if(t) for(const tag of t) tagCounts[tag]=(tagCounts[tag]||0)+1; }
const tagItems=Object.entries(tagCounts).sort((a,b)=>b[1]-a[1]).map(([t,v])=>({label:t,val:v,act:'s|tag:'+t}));

const folderCounts={};
for(const p of pages){ const parts=p.file.folder.split('/'); const k=parts.length>1?parts[1]:'(루트)'; folderCounts[k]=(folderCounts[k]||0)+1; }
const folderItems=Object.entries(folderCounts).sort((a,b)=>b[1]-a[1]).map(([k,v])=>({label:k,val:v,act:k==='(루트)'?null:'s|path:"'+k+'"'}));

// ── 건강도 (고정 카드) ────────────────────────────────
const total=pages.length;
const orphan=pages.where(p=>(!p.Categories||!p.Categories.length)&&(!p.Indexes||!p.Indexes.length)).length;
const noCat=pages.where(p=>!p.Categories||!p.Categories.length).length;
const noTag=pages.where(p=>{const t=p.file.etags;return !t||!t.length;}).length;
const pct=n=>Math.round(n/Math.max(1,total)*100);
function hCls(p){ return p<=10?'good':p<=25?'warn':'bad'; }
function hColor(p){ return p<=10?'var(--d-green)':p<=25?'var(--d-amber)':'var(--d-red)'; }
function hRow(label,val,href){ const p=pct(val), c=hColor(p);
  return `<div class="dash-health-row"><span class="h-label">${label}</span><div class="h-val-wrap"><div class="h-mini-bar"><div class="h-mini-fill" style="width:${Math.min(p,100)}%;background:${c}"></div></div><a class="internal-link" data-href="${href}"><span class="h-num ${hCls(p)}">${val} <small>(${p}%)</small></span></a></div></div>`;
}
const healthCard=`<div class="dash-card"><div class="card-head"><span class="card-title">🩺 Vault Health</span></div><div class="dash-health">${hRow('🔗 Orphans (no category & index)',orphan,'🔍 고립·미분류 노트')}${hRow('📁 Uncategorized (no Categories)',noCat,'🔍 고립·미분류 노트')}${hRow('🏷 Untagged (no tags)',noTag,'🔍 태그별 노트')}</div></div>`;

// ── 테마 색상 팔레트 ──────────────────────────────────
const css=getComputedStyle(document.body);
const V=(n,f)=>{const v=css.getPropertyValue(n).trim();return v||f;};
const cTxt=V('--text-muted','#888');
const cardBg=V('--background-secondary','#1e2030');
const palette=['#6ea8fe','#7c9cf5','#8a8df0','#9b86ec','#a784ee','#b88ee8','#69c8ff','#c9b3ff','#8fb3ff','#b8a0f0'];
function doAct(act){ if(!act) return; const i=act.indexOf('|'); const t=act.slice(0,i), v=act.slice(i+1); if(t==='s') openSearch(v); else if(t==='o') app.workspace.openLinkText(v,cur,false); }

// ── 분포 카드 정의 + 헬퍼 ─────────────────────────────
const cards=[
  {icon:'📚',title:'Categories',link:'🔍 카테고리별 노트',linkText:'All →',items:catItems},
  {icon:'🏷',title:'Tags',link:'🔍 태그별 노트',linkText:'Add tags →',items:tagItems},
  {icon:'🗂',title:'Folders',link:null,linkText:'',items:folderItems},
];
function listHtml(items){ const max=Math.max(1,...items.map(i=>i.val),1); return items.length? items.map(i=>bar(i.label,i.val,max,i.act)).join('') : '<div class="hd-empty">No data</div>'; }
function donutData(items){ const top=items.slice(0,8); const rest=items.slice(8).reduce((s,i)=>s+i.val,0); const labels=top.map(i=>i.label); const vals=top.map(i=>i.val); const acts=top.map(i=>i.act); if(rest>0){labels.push('Other');vals.push(rest);acts.push(null);} return {labels,vals,acts}; }

// ── 렌더 + 리스트/도넛 토글 ───────────────────────────
const canDonut=!!window.renderChart;
const legendPos=window.innerWidth<540?'bottom':'right';
const root=dv.el('div','',{cls:'dash-struct'});
let gridHtml='';
cards.forEach((c,i)=>{ gridHtml+=`<div class="dash-card"><div class="card-head"><span class="card-title">${c.icon} ${c.title} <small class="card-count">${c.items.length}</small></span>${c.link?`<a class="card-link internal-link" data-href="${c.link}">${c.linkText}</a>`:''}</div><div class="dash-struct-body" data-idx="${i}"></div></div>`; });
gridHtml+=healthCard;
root.innerHTML=`<div class="dash-struct-bar"><span class="dash-seg"><button class="dash-seg-btn active" data-mode="list">📋 List</button>${canDonut?'<button class="dash-seg-btn" data-mode="donut">🍩 Donut</button>':''}</span></div><div class="dash-grid">${gridHtml}</div>`;
root.querySelectorAll('a.internal-link').forEach(a=>a.onclick=(e)=>{e.preventDefault();app.workspace.openLinkText(a.dataset.href,cur,false);});

async function renderMode(mode){
  for(const bodyEl of root.querySelectorAll('.dash-struct-body')){
    const c=cards[+bodyEl.dataset.idx]; bodyEl.innerHTML='';
    if(mode==='donut' && window.renderChart){
      bodyEl.className='dash-struct-body dash-donut-area';
      const d=donutData(c.items);
      if(!d.vals.length){ bodyEl.innerHTML='<div class="hd-empty">데이터 없음</div>'; continue; }
      const colors=d.labels.map((_,k)=>palette[k%palette.length]);
      await window.renderChart({type:'doughnut',data:{labels:d.labels,datasets:[{data:d.vals,backgroundColor:colors,borderColor:cardBg,borderWidth:2,hoverOffset:7,hoverBorderColor:cardBg}]},options:{responsive:true,maintainAspectRatio:false,cutout:'62%',radius:'92%',onClick:(evt,els)=>{ if(els&&els.length) doAct(d.acts[els[0].index]); },plugins:{legend:{position:legendPos,labels:{color:cTxt,boxWidth:9,usePointStyle:true,pointStyle:'circle',padding:10,font:{size:10}}},tooltip:{padding:9,cornerRadius:8,usePointStyle:true,backgroundColor:'rgba(20,22,34,.92)',titleColor:'#fff',bodyColor:'#cbd2ee'}}}}, bodyEl);
    } else {
      bodyEl.className='dash-struct-body dash-bars dash-bars-scroll';
      bodyEl.innerHTML=listHtml(c.items);
    }
  }
  root.querySelectorAll('.dash-bar-row.clickable').forEach(r=>r.onclick=()=>{ const act=r.dataset.act; if(!act) return; const i=act.indexOf('|'); const type=act.slice(0,i), val=act.slice(i+1); if(type==='s') openSearch(val); else if(type==='o') app.workspace.openLinkText(val,cur,false); });
}
root.querySelectorAll('.dash-seg-btn').forEach(b=>b.onclick=()=>{ root.querySelectorAll('.dash-seg-btn').forEach(x=>x.classList.remove('active')); b.classList.add('active'); renderMode(b.dataset.mode); });
await renderMode('list');
```

## Discover

```dataviewjs
const cur=dv.current().file.path;
const pages=dv.pages('"/" AND -"agent"').where(p=>!p.file.folder.includes('30. Secrit Notes') && p.file.name!=='🏛 Home');
const r=pages[Math.floor(Math.random()*pages.length)];
const root=dv.el('div','',{cls:'dash-discovery'});
if(r){
  root.innerHTML=`🎲 Random note → <a class="internal-link" data-href="${r.file.path}">${r.file.name}</a> &nbsp;·&nbsp; 🔗 <a class="internal-link" data-href="🔍 고립·미분류 노트">Notes needing cleanup</a>`;
  root.querySelectorAll('a.internal-link').forEach(a=>a.onclick=(e)=>{e.preventDefault();app.workspace.openLinkText(a.dataset.href,cur,false);});
}
```
