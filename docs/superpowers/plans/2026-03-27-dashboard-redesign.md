# Note.com Analytics Dashboard Redesign Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Rebuild the Briefing tab with consulting-style chart storytelling, computed action titles, grey-out/colour-in visual language, right-side observation panels, trend arrows on KPIs, a sticky article key bar, and proper dual-timezone (JST/GMT) axis labels. Fix all 41 identified UX issues.

**Architecture:** Single-file HTML/JS app (`app/index.html`). Rewrite from scratch, preserving the existing IndexedDB layer, data parsing, and extraction scripts. All rendering functions rebuilt to follow the new visual language. Chart.js for charts, no build step.

**Tech Stack:** HTML, CSS, vanilla JS, Chart.js 4.x, chartjs-plugin-annotation 3.x, chartjs-adapter-date-fns 3.x, IndexedDB.

**Spec:** `docs/superpowers/specs/2026-03-27-dashboard-redesign.md`

---

## File Structure

This is a single-file app. The rewrite produces one file:

- **Rewrite:** `app/index.html` (the entire app)

The file is organised into clearly commented sections:
1. `<style>` block: all CSS
2. `<body>` HTML structure: header, tabs, briefing tab skeleton, data tab, dictionary tab, article key bar
3. `<script>` block, in order:
   - Constants: colour palette, i18n strings, scaling thresholds, seed data
   - i18n functions
   - IndexedDB layer (preserved from current)
   - Data parsing (preserved from current)
   - Import logic (preserved from current)
   - `computeMetrics()` (extended with peak zone, action titles, trend comparison)
   - Render functions: one per component (KPIs, article key, each chart, status bar, data tab, dictionary)
   - Event handlers (tabs, toggle, file upload, drag-drop, clipboard, export)
   - Init and main

Because this is a full rewrite of a 2800-line file, tasks are organised by **functional layer**, not line-by-line edits. Each task produces a working (if incomplete) app that can be opened in a browser.

---

### Task 1: Foundation — HTML skeleton, CSS, constants, IndexedDB, parsing

**Files:**
- Rewrite: `app/index.html`

This task creates the full file shell with all HTML structure, all CSS, all constants, the complete IndexedDB layer, all data parsing, and the import/export logic. Rendering functions are stubs that output placeholder text. The app loads, tabs work, data can be imported, but charts are not rendered yet.

- [ ] **Step 1: Create the complete HTML document structure**

Write the full `<!DOCTYPE html>` through `</html>` with:

**Head:** charset, viewport, title "Note.com Analytics", CDN links for Chart.js, annotation plugin, date-fns adapter. Full `<style>` block.

**CSS must include all of the following (reference the spec for exact values):**

```
Root variables:
  --bg: #fafafa
  --surface: #ffffff
  --border: #e5e7eb
  --ink: #1a1a2e
  --ink-light: #4b5563
  --ink-muted: #9ca3af
  --accent: #1e40af        (darker blue — bars in focus)
  --accent-border: #1e3a8a (border/emphasis)
  --accent-zone: #dbeafe   (peak zone background — lighter blue)
  --grey-bar: #d1d5db      (context bars)
  --green: #059669          (follows)
  --red: #dc2626            (publish markers)
  --purple: #7c3aed
  --radius: 8px
  --shadow: 0 1px 3px rgba(0,0,0,0.08)

Components to style:
  .app-header (sticky top, flex, space-between)
  .app-title (16px, 700)
  .header-controls (flex, gap)
  .lang-toggle (button, small)
  .status-bar (11px, muted)
  .tab-bar (flex, border-bottom)
  .tab-btn (13px, 500, border-bottom indicator)
  .tab-content (hidden by default, .active = display:block, max-width:1080px, margin:0 auto, padding:24px)

  .kpi-grid (display:grid, grid-template-columns:repeat(6,1fr), gap:0, border-bottom)
  .kpi-item (text-align:center, padding:8px 4px, border-left on siblings)
  .kpi-value (22px, 700, colour: var(--ink) — no colour coding)
  .kpi-label (10px, 600, uppercase, letter-spacing, muted)
  .kpi-sub (10px, muted)
  .kpi-trend (display:inline, font-size:12px, colour: var(--ink-muted), margin-left:4px)

  .section-heading (13px, 700, uppercase, letter-spacing:0.08em, colour:#9ca3af, margin:32px 0 12px)
  .chart-section (display:grid, grid-template-columns:1fr 220px, gap:0, border:1px solid var(--border), border-radius:var(--radius), overflow:hidden, margin-bottom:24px)
  .chart-main (padding:20px 24px 12px)
  .chart-obs (border-left:1px solid var(--border), padding:20px 18px, display:flex, flex-direction:column, justify-content:center)
  .obs-box (background:#f3f4f6, border-radius:6px, padding:12px 14px, font-size:12px, colour:#374151, line-height:1.7)
  .obs-extra (font-size:11px, colour:#6b7280, margin-top:10px)
  .action-title (15px, 700, colour:var(--ink), margin-bottom:14px)

  .chart-legend (flex, gap:14px, font-size:11px, colour:#6b7280, margin-top:6px, padding-top:8px, border-top:1px solid #f3f4f6)
  .legend-item (flex, align-items:center, gap:4px)
  .legend-dot (10px square, border-radius:2px)
  .legend-tick (width:0, height:10px, border-left:2px solid var(--red))
  .legend-tri (width:0, height:0, border-left:5px solid transparent, border-right:5px solid transparent, border-bottom:10px solid var(--green))

  .axis-row (display:flex)
  .axis-row span (flex:1, text-align:center)
  .axis-jst span (10px, weight 500, colour:var(--ink))
  .axis-gmt span (9px, colour:var(--ink-muted))
  .tz-label (9px, 600, colour:#9ca3af, flex:none, width:24px, text-align:right, margin-right:2px)

  .article-key-bar (position:fixed, bottom:0, left:0, right:0, background:var(--surface), border-top:1px solid var(--border), padding:6px 16px, z-index:100, font-size:10px, display:flex, align-items:center, gap:8px, cursor:pointer, transition:all 0.2s)
  .article-key-bar.hidden (display:none)
  .article-key-bar .key-dot (width:8px, height:8px, border-radius:50%, flex-shrink:0)
  .article-key-bar .key-label (font-weight:600, colour:var(--ink-light))
  .article-key-panel (position:fixed, bottom:0, left:0, right:0, background:var(--surface), border-top:1px solid var(--border), box-shadow:0 -4px 16px rgba(0,0,0,0.1), z-index:101, max-height:50vh, overflow-y:auto, padding:16px 24px, display:none)
  .article-key-panel.open (display:block)
  .article-key-search (width:100%, padding:6px 10px, font-size:12px, border:1px solid var(--border), border-radius:4px, margin-bottom:8px)
  .article-key-grid (display:grid, grid-template-columns:repeat(auto-fill,minmax(280px,1fr)), gap:4px 24px)
  .ak-item (display:flex, align-items:baseline, gap:8px, padding:4px 0, font-size:12px)
  .ak-num (font-weight:700, font-size:11px, min-width:24px)
  .ak-title (colour:var(--ink), flex:1, overflow:hidden, text-overflow:ellipsis, white-space:nowrap)
  .ak-stat (colour:var(--ink-muted), font-size:11px, white-space:nowrap)

  Import zone, data table, dictionary styles (carry over from current, no changes)

  @media (max-width:900px): chart-section to single column, chart-obs border-top instead of border-left
  @media (max-width:768px): kpi-grid to repeat(3,1fr), tab-content padding:16px
```

**Body HTML structure:**

```html
<header class="app-header">
  <div class="app-title">Note.com Analytics</div>
  <div class="header-controls">
    <div class="status-bar" id="statusBar"></div>
    <button class="lang-toggle" id="langToggle" onclick="toggleLang()">EN</button>
  </div>
</header>

<nav class="tab-bar">
  <button class="tab-btn active" data-tab="briefing" onclick="switchTab('briefing')">Briefing</button>
  <button class="tab-btn" data-tab="data" onclick="switchTab('data')">Data</button>
  <button class="tab-btn" data-tab="dict" onclick="switchTab('dict')">Dictionary</button>
</nav>

<div class="tab-content active" id="tab-briefing">
  <div class="kpi-grid" id="kpiGrid"></div>

  <div class="section-heading" id="heading1"></div>
  <div class="chart-section">
    <div class="chart-main" id="section1Main"></div>
    <div class="chart-obs" id="section1Obs"></div>
  </div>

  <div class="section-heading" id="heading2"></div>
  <div class="chart-section">
    <div class="chart-main" id="section2Main"></div>
    <div class="chart-obs" id="section2Obs"></div>
  </div>

  <div class="section-heading" id="heading3"></div>
  <div class="chart-section">
    <div class="chart-main" id="section3Main"></div>
    <div class="chart-obs" id="section3Obs"></div>
  </div>

  <div class="section-heading" id="heading4"></div>
  <div class="chart-section">
    <div class="chart-main" id="section4Main"></div>
    <div class="chart-obs" id="section4Obs"></div>
  </div>
</div>

<div class="tab-content" id="tab-data">
  <!-- Import zone, events table, import history — same structure as current -->
</div>

<div class="tab-content" id="tab-dict">
  <!-- Definitions, extraction scripts, limitations — same structure as current -->
</div>

<!-- Article key bar (fixed bottom) -->
<div class="article-key-bar" id="articleKeyBar" onclick="toggleArticleKeyPanel()"></div>
<div class="article-key-panel" id="articleKeyPanel"></div>
```

- [ ] **Step 2: Add the complete `<script>` block with constants and i18n**

Start the script block. Include:

**Colour constants:**
```javascript
const COLOURS = {
  accent: '#1e40af',
  accentBorder: '#1e3a8a',
  accentZone: '#dbeafe',
  grey: '#d1d5db',
  green: '#059669',
  red: '#dc2626',
  purple: '#7c3aed',
  ink: '#1a1a2e',
  inkLight: '#4b5563',
  inkMuted: '#9ca3af',
};
const ARTICLE_COLOURS = ['#2563eb', '#7c3aed', '#059669', '#d97706', '#dc2626', '#0891b2'];
```

**Scaling thresholds:**
```javascript
const THRESHOLDS = {
  publishTickMax: 10,
  timelineScrollAt: 15,
  timelineMaxHeight: 600,
  articleKeyInlineMax: 15,
  articleKeySearchAt: 10,
  journeyScrollAt: 15,
  journeyAggregateAt: 50,
  peopleAggregateAt: 100,
  tagMax: 10,
};
```

**i18n strings:** Carry over the full `STRINGS` object from the current file. Update all English strings to match the spec (remove em dashes, fix wording, use "GMT" not "UK", use "total readers" in status, etc.). Update all Japanese strings to match. Add new keys:
- `empty_chart`: 'Import data to see this chart' / 'データをインポートすると表示されます'
- `empty_status`: 'No data yet. Go to the Data tab to import.' / 'データなし。データタブからインポートしてください。'
- `trend_up`: '↑' (just the arrow character)
- `trend_down`: '↓'
- `article_key_title`: 'Article Key' / '記事一覧'
- `showing_top`: 'showing top {n} of {total}' / '上位{n}件（{total}件中）'
- Update `status_events` to status format: '{n} articles, {m} total readers' / '{n}記事, {m}人の読者'
- Section headings: 'heading_1' through 'heading_4' with the spec text

**Tag translations:** Carry over `TAG_EN` map.

**i18n functions:** `t(key)`, `toggleLang()`, `applyI18n()` — same logic as current.

- [ ] **Step 3: Add IndexedDB layer, parsing, and import logic**

Copy from the current file verbatim (these work correctly):
- `openDB()`, `dbPut()`, `dbGetAll()`, `dbClear()`
- `escapeHtml()`, `makeEventId()`
- `parseNotifications()`, `parseArticles()`
- `importData()` — add one change: after import, save current metrics to `metrics_snapshot` store before recomputing:

```javascript
async function importData(raw) {
  // ... existing import logic (unchanged) ...

  // Save pre-import metrics for trend comparison
  const oldEvents = await dbGetAll('events');
  const oldArticles = await dbGetAll('articles');
  if (oldEvents.length > 0) {
    const oldMetrics = computeMetrics(oldEvents, oldArticles);
    await dbPut('metrics_snapshot', {
      id: 'latest',
      articles: oldMetrics.articles.length,
      avgViews: oldMetrics.totalViews / (oldMetrics.articles.length || 1),
      avgLikes: oldMetrics.totalLikesReported / (oldMetrics.articles.length || 1),
      likeRate: oldMetrics.totalViews > 0 ? oldMetrics.totalLikesReported / oldMetrics.totalViews * 100 : 0,
      followRate: oldMetrics.uniqueUsers > 0 ? oldMetrics.totalFollows / oldMetrics.uniqueUsers * 100 : 0,
      returnRate: oldMetrics.uniqueUsers > 0 ? oldMetrics.repeatLikers.length / oldMetrics.uniqueUsers * 100 : 0,
      timestamp: new Date().toISOString(),
    });
  }

  // ... rest of import (unchanged) ...
}
```

Add `metrics_snapshot` object store to `openDB()`:
```javascript
if (!d.objectStoreNames.contains('metrics_snapshot')) {
  d.createObjectStore('metrics_snapshot', { keyPath: 'id' });
}
```
Increment `DB_VERSION` to 2.

- [ ] **Step 4: Add stub rendering functions and init**

Add placeholder render functions that output minimal text so the app loads:

```javascript
let currentMetrics = null;

function renderStatusBar(m) {
  const bar = document.getElementById('statusBar');
  if (!m || m.events.length === 0) {
    bar.textContent = t('empty_status');
    return;
  }
  bar.textContent = m.articles.length + ' articles, ' + m.uniqueUsers + ' total readers';
}

function renderKPIs(m) {
  document.getElementById('kpiGrid').textContent = 'KPIs: ' + m.articles.length + ' articles (rendering in Task 2)';
}

function renderArticleKeyBar(m) {
  const bar = document.getElementById('articleKeyBar');
  if (!m || m.articles.length === 0) { bar.classList.add('hidden'); return; }
  bar.classList.remove('hidden');
  bar.textContent = 'Article Key: ' + m.articles.length + ' articles (rendering in Task 3)';
}

function renderSection1(m) { document.getElementById('section1Main').textContent = 'Timeline chart (Task 4)'; }
function renderSection2(m) { document.getElementById('section2Main').textContent = 'Activity + DoW charts (Task 5)'; }
function renderSection3(m) { document.getElementById('section3Main').textContent = 'Article perf + Tags charts (Task 6)'; }
function renderSection4(m) { document.getElementById('section4Main').textContent = 'Journeys chart (Task 7)'; }

// Section headings
function renderHeadings() {
  const isJa = currentLang === 'ja';
  document.getElementById('heading1').textContent = isJa ? '1. エンゲージメントの推移' : '1. How engagement built over time';
  document.getElementById('heading2').textContent = isJa ? '2. いつ読者が反応したか' : '2. When readers respond';
  document.getElementById('heading3').textContent = isJa ? '3. どの記事が読者を引き付けたか' : '3. Which articles attract and retain readers';
  document.getElementById('heading4').textContent = isJa ? '4. フォロワーがどのようにフォローに至ったか' : '4. How followers discovered and converted';
}

// Render all
function renderAll() {
  if (!currentMetrics) return;
  applyI18n();
  renderStatusBar(currentMetrics);
  renderKPIs(currentMetrics);
  renderArticleKeyBar(currentMetrics);
  renderHeadings();
  renderSection1(currentMetrics);
  renderSection2(currentMetrics);
  renderSection3(currentMetrics);
  renderSection4(currentMetrics);
  renderDataTab(currentMetrics);
  renderDictionary();
}
```

Add `switchTab()`, seed data, `loadAndRender()`, `seedIfEmpty()`, `init()` — carried over from current file. Update `switchTab` to show/hide article key bar on Briefing tab only.

Add the Data tab render functions (`renderEventsTable`, `renderImportHistory`, `processImport`, `handleFileUpload`, `pasteFromClipboard`, `exportData`) and Dictionary tab render function (`renderDictionary`) — carried over from current.

Add the date-fns adapter loader and `init()` call at the end.

- [ ] **Step 5: Add computeMetrics with peak zone and action title generation**

Carry over the existing `computeMetrics()` function. Add these new computed properties to its return object:

```javascript
// Peak zone: hours above 75th percentile
const allHourCounts = hourLikes.map((l, i) => l + hourFollows[i]);
const sortedCounts = [...allHourCounts].filter(c => c > 0).sort((a, b) => a - b);
const p75 = sortedCounts.length > 0 ? sortedCounts[Math.floor(sortedCounts.length * 0.75)] : 0;
const peakHoursSet = [];
for (let h = 0; h < 24; h++) {
  if (allHourCounts[h] >= p75 && allHourCounts[h] > 0) peakHoursSet.push(h);
}
// Contiguous peak range
let peakStart = peakHoursSet.length > 0 ? peakHoursSet[0] : null;
let peakEnd = peakHoursSet.length > 0 ? peakHoursSet[peakHoursSet.length - 1] : null;

// Action title generators
const actionTitles = {
  timeline: generateTimelineTitle(sortedArticles, likes, follows),
  activity: generateActivityTitle(peakStart, peakEnd),
  dow: generateDowTitle(DOW_NAMES, dowLikes),
  articlePerf: generateArticlePerfTitle(articleMetrics),
  tags: generateTagsTitle(tagData),
  journeys: generateJourneysTitle(journeys),
};
```

Add the title generator functions before `computeMetrics`:

```javascript
function generateTimelineTitle(articles, likes, follows) {
  if (articles.length === 0) return '';
  const isJa = currentLang === 'ja';
  // Check if most engagement arrives within 48h of publish
  let within48h = 0;
  let total = likes.length;
  for (const art of articles) {
    const pubTime = new Date(art.published_date_jst).getTime();
    const artLikes = likes.filter(e => e.article_url === art.article_url);
    within48h += artLikes.filter(e => {
      const likeTime = new Date(e.timestamp_jst).getTime();
      return (likeTime - pubTime) <= 48 * 60 * 60 * 1000;
    }).length;
  }
  const pct48 = total > 0 ? Math.round(within48h / total * 100) : 0;
  if (pct48 >= 60) {
    return isJa
      ? 'いいねの' + pct48 + '%は公開後48時間以内に発生'
      : pct48 + '% of likes arrived within 48 hours of publish';
  }
  return isJa
    ? articles.length + '記事に対して' + likes.length + '件のいいねと' + follows.length + '件のフォロー'
    : likes.length + ' likes and ' + follows.length + ' follows across ' + articles.length + ' articles';
}

function generateActivityTitle(peakStart, peakEnd) {
  const isJa = currentLang === 'ja';
  if (peakStart === null) return isJa ? 'まだ十分なデータがありません' : 'Not enough data yet';
  const s = String(peakStart).padStart(2, '0');
  const e = String(peakEnd).padStart(2, '0');
  return isJa
    ? 'いいねの多くはJST ' + s + ':00から' + e + ':00の間に集中'
    : 'Most likes arrive between ' + s + ':00 and ' + e + ':00 JST';
}

function generateDowTitle(DOW_NAMES, dowLikes) {
  const isJa = currentLang === 'ja';
  const maxVal = Math.max(...dowLikes);
  if (maxVal === 0) return isJa ? 'まだ十分なデータがありません' : 'Not enough data yet';
  const topDays = DOW_NAMES.filter((_, i) => dowLikes[i] === maxVal);
  const dayStr = topDays.join(isJa ? 'と' : ' and ');
  return isJa
    ? dayStr + 'に最もエンゲージメントが集中'
    : dayStr + (topDays.length > 1 ? ' see' : ' sees') + ' the most engagement';
}

function generateArticlePerfTitle(articleMetrics) {
  const isJa = currentLang === 'ja';
  const withViews = articleMetrics.filter(a => a.views > 0);
  if (withViews.length === 0) return isJa ? 'まだ十分なデータがありません' : 'Not enough data yet';
  const best = withViews.reduce((a, b) => (b.likes_total / b.views) > (a.likes_total / a.views) ? b : a);
  const idx = articleMetrics.indexOf(best);
  const rate = (best.likes_total / best.views * 100).toFixed(0);
  return isJa
    ? 'A' + (idx + 1) + 'のスキ率が' + rate + '%で最も高い'
    : 'A' + (idx + 1) + ' has the highest like rate at ' + rate + '%';
}

function generateTagsTitle(tagData) {
  const isJa = currentLang === 'ja';
  if (tagData.length === 0) return isJa ? 'まだ十分なデータがありません' : 'Not enough data yet';
  const top = tagData[0];
  const tagName = currentLang === 'en' && TAG_EN[top.tag] ? TAG_EN[top.tag] : top.tag;
  return isJa
    ? '「' + top.tag + '」タグが最もいいねを集めている'
    : 'Articles tagged "' + tagName + '" attract the most likes';
}

function generateJourneysTitle(journeys) {
  const isJa = currentLang === 'ja';
  if (journeys.length === 0) return isJa ? 'まだフォロワーデータがありません' : 'No follower data yet';
  const likedFirst = journeys.filter(j => j.likes.length > 0).length;
  return isJa
    ? likedFirst + '人のフォロワーがフォロー前にいいねした'
    : likedFirst + (likedFirst === 1 ? ' follower' : ' followers') + ' liked before they followed';
}
```

Add peak zone and action titles to the return object of `computeMetrics`.

- [ ] **Step 6: Verify the foundation loads in browser**

Open `app/index.html` in a browser. Verify:
- Header shows "Note.com Analytics"
- Three tabs work (Briefing, Data, Dictionary)
- Briefing tab shows KPI placeholder, 4 section headings, 4 placeholder chart sections
- Data tab allows JSON import (use existing seed data or paste test JSON)
- After import, status bar updates to show article/reader count
- Language toggle switches between EN and JP
- No console errors

- [ ] **Step 7: Commit**

```bash
git add app/index.html
git commit -m "Rebuild index.html foundation: skeleton, CSS, IndexedDB, parsing, stubs"
```

---

### Task 2: KPI Grid with trend arrows

**Files:**
- Modify: `app/index.html` (replace `renderKPIs` stub)

- [ ] **Step 1: Implement renderKPIs**

Replace the `renderKPIs` stub with the full implementation:

```javascript
async function renderKPIs(m) {
  const el = document.getElementById('kpiGrid');
  const isJa = currentLang === 'ja';
  const artCount = m.articles.length || 1;
  const totalViews = m.totalViews;
  const totalLikes = m.totalLikesReported;
  const likeRate = totalViews > 0 ? (totalLikes / totalViews * 100) : 0;
  const followRate = m.uniqueUsers > 0 ? (m.totalFollows / m.uniqueUsers * 100) : 0;
  const returnPct = m.uniqueUsers > 0 ? (m.repeatLikers.length / m.uniqueUsers * 100) : 0;

  // Load previous snapshot for trend arrows
  let prev = null;
  try { const all = await dbGetAll('metrics_snapshot'); prev = all.find(s => s.id === 'latest'); } catch(e) {}

  function trend(current, prevVal) {
    if (prev === null || prevVal === undefined || prevVal === null) return '';
    if (current > prevVal) return ' <span class="kpi-trend">\u2191</span>';
    if (current < prevVal) return ' <span class="kpi-trend">\u2193</span>';
    return '';
  }

  const kpis = [
    {
      value: m.articles.length,
      label: isJa ? '記事数' : 'Articles Published',
      sub: m.daySpan > 0 ? (isJa ? m.daySpan + '日間に公開' : 'over ' + m.daySpan + ' days') : '',
      trend: trend(m.articles.length, prev?.articles),
    },
    {
      value: Math.round(totalViews / artCount),
      label: isJa ? '記事あたり閲覧数' : 'Avg. Views per Article',
      sub: totalViews.toLocaleString() + ' ' + (isJa ? '合計閲覧' : 'total views'),
      trend: trend(totalViews / artCount, prev?.avgViews),
    },
    {
      value: (totalLikes / artCount).toFixed(1),
      label: isJa ? '記事あたりいいね' : 'Avg. Likes per Article',
      sub: totalLikes.toLocaleString() + ' ' + (isJa ? '合計いいね' : 'total likes'),
      trend: trend(totalLikes / artCount, prev?.avgLikes),
    },
    {
      value: likeRate.toFixed(1) + '%',
      label: isJa ? 'スキ率' : 'Like Rate',
      sub: isJa ? '閲覧者のうちいいねした割合' : '% of viewers who liked',
      trend: trend(likeRate, prev?.likeRate),
    },
    {
      value: followRate.toFixed(0) + '%',
      label: isJa ? 'フォロー率' : 'Follow Rate',
      sub: m.totalFollows + (isJa ? '人がフォロー（いいねした' + m.uniqueUsers + '人中）' : ' followed out of ' + m.uniqueUsers + ' who liked'),
      trend: trend(followRate, prev?.followRate),
    },
    {
      value: returnPct.toFixed(0) + '%',
      label: isJa ? '再訪率' : 'Return Rate',
      sub: m.repeatLikers.length + (isJa ? '人が2件以上の記事にいいね（' + m.uniqueUsers + '人中）' : ' came back to like another article (of ' + m.uniqueUsers + ')'),
      trend: trend(returnPct, prev?.returnRate),
    },
  ];

  let html = '';
  for (const kpi of kpis) {
    html += '<div class="kpi-item">';
    html += '<div class="kpi-value">' + kpi.value + kpi.trend + '</div>';
    html += '<div class="kpi-label">' + kpi.label + '</div>';
    if (kpi.sub) html += '<div class="kpi-sub">' + kpi.sub + '</div>';
    html += '</div>';
  }

  el.innerHTML = html;
}
```

Note: `renderKPIs` is now async (reads from metrics_snapshot). Update the call in `renderAll` to not await it (fire and forget is fine, the DOM update is fast).

- [ ] **Step 2: Handle empty state**

Add to the top of `renderKPIs`:
```javascript
if (!m || m.articles.length === 0) {
  el.innerHTML = Array(6).fill('<div class="kpi-item"><div class="kpi-value">--</div><div class="kpi-label">&nbsp;</div></div>').join('');
  return;
}
```

- [ ] **Step 3: Verify in browser**

Open the app, import data. Verify:
- 6 KPIs show in a row with correct values
- No colour coding on any number (all same colour)
- Sub-labels are in plain language matching the spec
- Language toggle updates all KPI labels and sub-labels
- On first import: no trend arrows (no previous snapshot)
- On second import (import same data again): arrows appear (all showing no change or slight change)

- [ ] **Step 4: Commit**

```bash
git add app/index.html
git commit -m "Add KPI grid with trend arrows and empty states"
```

---

### Task 3: Article key sticky bottom bar

**Files:**
- Modify: `app/index.html` (replace `renderArticleKeyBar` stub, add panel logic)

- [ ] **Step 1: Implement the article key bar and expandable panel**

Replace the stub with:

```javascript
function toggleArticleKeyPanel() {
  document.getElementById('articleKeyPanel').classList.toggle('open');
}

function renderArticleKeyBar(m) {
  const bar = document.getElementById('articleKeyBar');
  const panel = document.getElementById('articleKeyPanel');
  const isJa = currentLang === 'ja';

  if (!m || m.articles.length === 0) {
    bar.classList.add('hidden');
    panel.classList.remove('open');
    return;
  }
  bar.classList.remove('hidden');

  // Collapsed bar: colour dots + A-numbers
  let barHtml = '<span class="key-label">' + (isJa ? '記事一覧' : 'Article Key') + '</span>';
  const maxInline = Math.min(m.articles.length, THRESHOLDS.articleKeyInlineMax);
  for (let i = 0; i < maxInline; i++) {
    barHtml += '<span class="key-dot" style="background:' + getArticleColour(i) + ';" title="A' + (i + 1) + '"></span>';
    barHtml += '<span style="font-size:10px;color:var(--ink-light);margin-right:4px;">A' + (i + 1) + '</span>';
  }
  if (m.articles.length > THRESHOLDS.articleKeyInlineMax) {
    barHtml += '<span style="font-size:10px;color:var(--ink-muted);">+' + (m.articles.length - maxInline) + ' more</span>';
  }
  bar.innerHTML = barHtml;

  // Expanded panel
  let panelHtml = '';
  if (m.articles.length >= THRESHOLDS.articleKeySearchAt) {
    panelHtml += '<input class="article-key-search" placeholder="' + (isJa ? '記事を検索...' : 'Search articles...') + '" oninput="filterArticleKey(this.value)">';
  }
  panelHtml += '<div class="article-key-grid" id="akGrid">';
  for (let i = 0; i < m.articles.length; i++) {
    const art = m.articles[i];
    const title = isJa ? (art.title_jp || art.title_en) : (art.title_en || art.title_jp);
    const views = art.views || 0;
    const likes = art.likes_total || 0;
    const rate = views > 0 ? (likes / views * 100).toFixed(0) + '%' : '';
    panelHtml += '<div class="ak-item" data-search="' + escapeHtml(title).toLowerCase() + '">';
    panelHtml += '<span class="ak-num" style="color:' + getArticleColour(i) + '">A' + (i + 1) + '</span>';
    panelHtml += '<span class="ak-title" title="' + escapeHtml(title) + '">' + escapeHtml(shortTitle(title, 36)) + '</span>';
    panelHtml += '<span class="ak-stat">' + views + (isJa ? '閲覧' : ' views') + (rate ? ', ' + rate + (isJa ? 'スキ率' : ' like rate') : '') + '</span>';
    panelHtml += '</div>';
  }
  panelHtml += '</div>';
  panel.innerHTML = panelHtml;
}

function filterArticleKey(query) {
  const q = query.toLowerCase();
  document.querySelectorAll('#akGrid .ak-item').forEach(el => {
    el.style.display = el.dataset.search.includes(q) ? '' : 'none';
  });
}

function getArticleColour(idx) { return ARTICLE_COLOURS[idx % ARTICLE_COLOURS.length]; }
```

Update `switchTab` to show/hide the article key bar:
```javascript
function switchTab(tabId) {
  document.querySelectorAll('.tab-btn').forEach(b => b.classList.toggle('active', b.dataset.tab === tabId));
  document.querySelectorAll('.tab-content').forEach(c => c.classList.toggle('active', c.id === 'tab-' + tabId));
  const bar = document.getElementById('articleKeyBar');
  const panel = document.getElementById('articleKeyPanel');
  if (tabId !== 'briefing') {
    bar.classList.add('hidden');
    panel.classList.remove('open');
  } else if (currentMetrics && currentMetrics.articles.length > 0) {
    bar.classList.remove('hidden');
  }
}
```

- [ ] **Step 2: Verify in browser**

- Briefing tab: bottom bar visible with colour dots and A-numbers
- Click bar: panel expands upward with full article details
- Click again: panel closes
- Switch to Data tab: bar hidden
- Switch back to Briefing: bar reappears
- With seed data (5-6 articles): no search box, all articles visible
- Language toggle updates the bar label and article stats

- [ ] **Step 3: Commit**

```bash
git add app/index.html
git commit -m "Add sticky article key bar with expandable panel and search"
```

---

### Task 4: Section 1 — Engagement Timeline chart

**Files:**
- Modify: `app/index.html` (replace `renderSection1` stub)

- [ ] **Step 1: Implement renderSection1 with timeline chart and observation panel**

Replace the stub. This is the largest single render function. Key elements:

- Action title from `m.actionTitles.timeline`
- Scatter chart: x = time, y = article row index. All likes as one dataset in `COLOURS.accent`. Follows as triangles in `COLOURS.green`.
- Publish date markers: dashed grey vertical lines via annotation plugin
- Y-axis ticks: "A1", "A2", ... "Follow" labels
- Toggle button: "Show People" / "Show Timeline"
- Observation panel (right side): grey box with computed facts about engagement timing
- Tooltip z-index set to 9999
- Chart height: adaptive based on article count with scroll at >15

```javascript
let chartTimeline = null;
let timelineMode = 'timeline';

function renderSection1(m) {
  const main = document.getElementById('section1Main');
  const obs = document.getElementById('section1Obs');
  const isJa = currentLang === 'ja';

  if (m.articles.length === 0) {
    main.innerHTML = '<div class="action-title">' + t('empty_chart') + '</div>';
    obs.innerHTML = '';
    return;
  }

  // Build main HTML
  let html = '<div class="action-title">' + m.actionTitles.timeline + '</div>';
  html += '<div style="display:flex;justify-content:flex-end;margin-bottom:8px;">';
  html += '<button class="tab-btn" style="font-size:11px;padding:4px 10px;" onclick="toggleTimelineMode()">';
  html += '<span id="timelineToggleLabel">' + (isJa ? '人別に表示' : 'Show People') + '</span></button></div>';
  html += '<div id="timelineChartWrap" style="position:relative;"><canvas id="chartTimeline"></canvas></div>';
  html += '<div id="timelineAlt"></div>';

  // Legend
  html += '<div class="chart-legend">';
  html += '<div class="legend-item"><div class="legend-dot" style="background:' + COLOURS.accent + ';"></div> ' + (isJa ? 'いいね' : 'Like') + '</div>';
  html += '<div class="legend-item"><div class="legend-tri"></div> ' + (isJa ? 'フォロー' : 'Follow') + '</div>';
  html += '<div class="legend-item" style="border-left:1.5px dashed ' + COLOURS.inkMuted + ';height:10px;width:0;margin:0 2px;"></div>';
  html += '<span style="color:var(--ink-muted);font-size:11px;">' + (isJa ? '記事公開日' : 'Article publish date') + '</span>';
  html += '</div>';

  main.innerHTML = html;

  // Observation panel
  const likes = m.events.filter(e => e.kind === 'like');
  const follows = m.events.filter(e => e.kind === 'follow');
  let within48h = 0;
  for (const art of m.articles) {
    const pubTime = new Date(art.published_date_jst).getTime();
    within48h += likes.filter(e => e.article_url === art.article_url && (new Date(e.timestamp_jst).getTime() - pubTime) <= 48 * 3600000).length;
  }
  const pct48 = likes.length > 0 ? Math.round(within48h / likes.length * 100) : 0;

  let obsHtml = '<div class="obs-box">';
  obsHtml += '<p><strong>' + pct48 + '%</strong> ' + (isJa ? 'のいいねは公開後48時間以内に発生。' : 'of likes arrived within 48 hours of publish.') + '</p>';
  if (follows.length > 0) {
    const followersWhoLikedFirst = m.journeys.filter(j => j.likes.length > 0).length;
    obsHtml += '<p><strong>' + followersWhoLikedFirst + ' of ' + follows.length + '</strong> ' + (isJa ? 'フォロワーがフォロー前にいいねした。' : 'followers liked before they followed.') + '</p>';
  }
  obsHtml += '</div>';
  obs.innerHTML = obsHtml;

  // Render the chart
  if (timelineMode === 'timeline') {
    renderTimelineChart(m);
  } else {
    renderPeopleView(m);
  }
}
```

Add `renderTimelineChart(m)` as a separate function handling the Chart.js scatter chart construction with annotation plugin for publish date lines. Use the patterns from the current file but with:
- Uniform `COLOURS.accent` for all like dots
- `COLOURS.green` triangles for follows
- Dashed `COLOURS.inkMuted` vertical lines for publish dates (not per-article colours)
- Chart.js tooltip `z` option or CSS `.chartjs-tooltip { z-index: 9999 !important; }` to ensure tooltips render above peak zones
- Adaptive height: `Math.max(280, Math.min(THRESHOLDS.timelineMaxHeight, (m.articles.length + 2) * (m.articles.length > THRESHOLDS.timelineScrollAt ? 30 : 50)))` + overflow-y:auto on container

Add `renderPeopleView(m)` — the card grid for followers/repeat likers (carried over from current `renderTimelineByUser` but cleaner card layout).

Add `toggleTimelineMode()` function.

- [ ] **Step 2: Verify in browser**

- Chart renders with blue dots and green triangles
- Dashed vertical lines at publish dates
- Y-axis shows A1, A2, ... Follow
- Hover on any dot shows tooltip with both timezones and count, above all other elements
- Toggle button switches between timeline scatter and people card view
- Observation panel shows 48h stat and follower-liked-first stat
- Action title is a computed sentence
- Language toggle updates everything

- [ ] **Step 3: Commit**

```bash
git add app/index.html
git commit -m "Add engagement timeline chart with observation panel and people toggle"
```

---

### Task 5: Section 2 — Activity by Hour + Day of Week charts

**Files:**
- Modify: `app/index.html` (replace `renderSection2` stub)

- [ ] **Step 1: Implement renderSection2**

This section has two charts side by side (within the left column) plus the right observation panel.

Key elements for Activity by Hour:
- 24 bars: `COLOURS.grey` for non-peak, `COLOURS.accent` for peak zone hours
- Peak zone background shading via annotation plugin `box` type with `backgroundColor: COLOURS.accentZone`
- "Peak" label via annotation positioned at top of the shaded region with adequate padding
- Publish time ticks: use `THRESHOLDS.publishTickMax` to decide between individual ticks and density strip
- Dual axis: JST primary (10px, weight 500, `COLOURS.ink`), GMT secondary (9px, `COLOURS.inkMuted`). Each rendered as HTML divs below the Chart.js canvas, not as Chart.js axis labels (for precise control over two-row layout).
- Tooltip z-index above peak zone

Key elements for Day of Week:
- 7 stacked bars: likes in `COLOURS.accent`, follows in `COLOURS.green`
- Published-day markers as a subtle third bar layer in `COLOURS.red + '30'`
- Chart.js legend disabled; use HTML legend matching the visual language

Observation panel:
- How many articles published before/during/after peak zone
- Which day has most engagement

```javascript
let chartActivity = null;
let chartDow = null;

function renderSection2(m) {
  const main = document.getElementById('section2Main');
  const obs = document.getElementById('section2Obs');
  const isJa = currentLang === 'ja';

  if (m.events.length === 0) {
    main.innerHTML = '<div class="action-title">' + t('empty_chart') + '</div>';
    obs.innerHTML = '';
    return;
  }

  let html = '';

  // Activity by hour
  html += '<div class="action-title">' + m.actionTitles.activity + '</div>';
  html += '<div style="position:relative;"><canvas id="chartActivity"></canvas></div>';
  // Dual timezone axis (HTML, not Chart.js)
  html += '<div class="axis-row axis-jst"><span class="tz-label">JST</span>';
  for (let h = 0; h < 24; h++) html += '<span>' + h + '</span>';
  html += '</div>';
  html += '<div class="axis-row axis-gmt"><span class="tz-label">GMT</span>';
  for (let h = 0; h < 24; h++) html += '<span>' + jstToGmt(h) + '</span>';
  html += '</div>';
  // Legend for activity chart
  html += '<div class="chart-legend">';
  html += '<div class="legend-item"><div class="legend-dot" style="background:' + COLOURS.accent + ';"></div> ' + (isJa ? 'ピーク時間帯' : 'Peak hours') + '</div>';
  html += '<div class="legend-item"><div class="legend-dot" style="background:' + COLOURS.grey + ';"></div> ' + (isJa ? 'その他' : 'Other hours') + '</div>';
  html += '<div class="legend-item"><div class="legend-tick"></div> ' + (isJa ? '記事公開時刻' : 'Article publish time') + '</div>';
  html += '</div>';

  // Day of week
  html += '<div style="margin-top:24px;">';
  html += '<div class="action-title">' + m.actionTitles.dow + '</div>';
  html += '<div><canvas id="chartDow"></canvas></div>';
  html += '<div class="chart-legend">';
  html += '<div class="legend-item"><div class="legend-dot" style="background:' + COLOURS.accent + ';"></div> ' + (isJa ? 'いいね' : 'Likes') + '</div>';
  html += '<div class="legend-item"><div class="legend-dot" style="background:' + COLOURS.green + ';"></div> ' + (isJa ? 'フォロー' : 'Follows') + '</div>';
  html += '<div class="legend-item"><div class="legend-dot" style="background:' + COLOURS.red + '30;"></div> ' + (isJa ? '記事公開日' : 'Day article was published') + '</div>';
  html += '</div>';
  html += '</div>';

  main.innerHTML = html;

  // Render activity chart
  renderActivityChart(m);

  // Render day of week chart
  renderDowChart(m);

  // Observation panel
  const peakStart = m.peakStart;
  const peakEnd = m.peakEnd;
  let beforePeak = 0, duringPeak = 0, afterPeak = 0;
  if (peakStart !== null) {
    for (const ph of m.publishHours) {
      if (ph.hour < peakStart) beforePeak++;
      else if (ph.hour <= peakEnd) duringPeak++;
      else afterPeak++;
    }
  }
  const totalPeakLikes = m.hourLikes.slice(peakStart || 0, (peakEnd || 0) + 1).reduce((a, b) => a + b, 0);
  const totalLikes = m.hourLikes.reduce((a, b) => a + b, 0);
  const peakPct = totalLikes > 0 ? Math.round(totalPeakLikes / totalLikes * 100) : 0;

  const bestDowIdx = m.dowLikes.indexOf(Math.max(...m.dowLikes));

  let obsHtml = '<div class="obs-box">';
  if (peakStart !== null) {
    obsHtml += '<p><strong>' + beforePeak + ' of ' + m.articles.length + '</strong> ' + (isJa ? '記事がピーク前に公開。' : 'articles were published before the peak zone.') + '</p>';
    obsHtml += '<p><strong>' + duringPeak + ' of ' + m.articles.length + '</strong> ' + (isJa ? '記事がピーク中に公開。' : 'were published during it.') + '</p>';
  }
  obsHtml += '</div>';
  obsHtml += '<div class="obs-extra">' + peakPct + '% ' + (isJa ? 'のいいねがピーク時間帯に集中。' : 'of all likes arrived during peak hours.') + '</div>';
  obsHtml += '<div class="obs-extra" style="margin-top:6px;">' + (isJa ? '最も活発な曜日: ' : 'Most active day: ') + '<strong>' + m.DOW_NAMES[bestDowIdx] + '</strong></div>';
  obs.innerHTML = obsHtml;
}
```

Add `jstToGmt(h)` utility function:
```javascript
function jstToGmt(jstHour) { return (jstHour - 9 + 24) % 24; }
```

Add `renderActivityChart(m)` and `renderDowChart(m)` as separate functions. The activity chart must:
- Disable Chart.js x-axis labels (the HTML axis rows handle this)
- Use the annotation plugin for peak zone background box
- Use the annotation plugin for publish time vertical lines (or render as HTML ticks below canvas)

- [ ] **Step 2: Verify in browser**

- Activity chart shows 24 bars, grey outside peak, darker blue inside
- Peak zone has lighter blue background shading
- "Peak" label visible inside shading, not hitting ceiling
- JST axis directly under bars, GMT directly under JST
- Hover tooltip shows both timezones, renders above peak zone
- Publish ticks visible on axis
- Day of week chart shows stacked bars
- Observation panel shows before/during/after peak stats
- Action titles are computed sentences

- [ ] **Step 3: Commit**

```bash
git add app/index.html
git commit -m "Add activity by hour and day of week charts with dual timezone axis"
```

---

### Task 6: Section 3 — Article Performance + Tag Reach charts

**Files:**
- Modify: `app/index.html` (replace `renderSection3` stub)

- [ ] **Step 1: Implement renderSection3**

Two charts in the left column. Article performance uses per-article colours from `ARTICLE_COLOURS`. Tag chart uses purple/blue split.

Article Performance chart:
- Grouped bar: views as `COLOURS.accent + '30'` (light), likes as `COLOURS.accent` (solid)
- Line overlay for like rate on secondary Y-axis in `COLOURS.red`
- X-axis labels: A1, A2, ... (short). Full titles in tooltip.
- Left Y: "Views / Likes". Right Y: "Like Rate (%)".
- Chart.js legend at bottom, small, using pointStyle.

Tag Reach chart:
- Horizontal stacked bar. Top 10 tags.
- Repeat readers: `COLOURS.purple + '90'`. One-time: `COLOURS.accent + '50'`.
- If more than 10 tags, show "(showing top 10 of X)" text.
- Y-axis labels: tag names (translated in EN mode).

Observation panel:
- Best and worst like rate articles
- Which tag has repeat readers

```javascript
let chartArticlePerf = null;
let chartTags = null;

function renderSection3(m) {
  const main = document.getElementById('section3Main');
  const obs = document.getElementById('section3Obs');
  const isJa = currentLang === 'ja';

  if (m.articles.length === 0) {
    main.innerHTML = '<div class="action-title">' + t('empty_chart') + '</div>';
    obs.innerHTML = '';
    return;
  }

  let html = '';

  // Article performance
  html += '<div class="action-title">' + m.actionTitles.articlePerf + '</div>';
  html += '<div><canvas id="chartArticlePerf"></canvas></div>';
  html += '<div class="chart-legend">';
  html += '<div class="legend-item"><div class="legend-dot" style="background:' + COLOURS.accent + '30;border:1px solid ' + COLOURS.accent + ';"></div> ' + (isJa ? '閲覧数' : 'Views') + '</div>';
  html += '<div class="legend-item"><div class="legend-dot" style="background:' + COLOURS.accent + ';"></div> ' + (isJa ? 'いいね' : 'Likes') + '</div>';
  html += '<div class="legend-item"><div class="legend-dot" style="background:' + COLOURS.red + ';border-radius:50%;width:8px;height:8px;"></div> ' + (isJa ? 'スキ率 (%)' : 'Like Rate (%)') + '</div>';
  html += '</div>';

  // Tags
  html += '<div style="margin-top:24px;">';
  html += '<div class="action-title">' + m.actionTitles.tags + '</div>';
  html += '<div><canvas id="chartTags"></canvas></div>';
  const totalTags = m.tagData.length;
  let tagNote = '';
  if (totalTags > THRESHOLDS.tagMax) {
    tagNote = '<div style="font-size:11px;color:var(--ink-muted);margin-top:4px;">' +
      t('showing_top').replace('{n}', THRESHOLDS.tagMax).replace('{total}', totalTags) + '</div>';
  }
  html += tagNote;
  html += '<div class="chart-legend">';
  html += '<div class="legend-item"><div class="legend-dot" style="background:' + COLOURS.purple + '90;"></div> ' + (isJa ? '複数記事にいいねした読者' : 'Readers who liked multiple articles') + '</div>';
  html += '<div class="legend-item"><div class="legend-dot" style="background:' + COLOURS.accent + '50;"></div> ' + (isJa ? '1記事のみの読者' : 'Readers who liked one article only') + '</div>';
  html += '</div>';
  html += '</div>';

  main.innerHTML = html;

  renderArticlePerfChart(m);
  renderTagsChart(m);

  // Observation
  const withViews = m.articleMetrics.filter(a => a.views > 0);
  let obsHtml = '<div class="obs-box">';
  if (withViews.length >= 2) {
    const sorted = [...withViews].sort((a, b) => (b.likes_total / b.views) - (a.likes_total / a.views));
    const best = sorted[0];
    const worst = sorted[sorted.length - 1];
    const bestIdx = m.articleMetrics.indexOf(best);
    const worstIdx = m.articleMetrics.indexOf(worst);
    const bestRate = (best.likes_total / best.views * 100).toFixed(0);
    const worstRate = (worst.likes_total / worst.views * 100).toFixed(0);
    obsHtml += '<p>' + (isJa ? 'スキ率の範囲: ' : 'Like rates range from ') + '<strong>' + worstRate + '%</strong> (A' + (worstIdx + 1) + ') ' + (isJa ? 'から ' : 'to ') + '<strong>' + bestRate + '%</strong> (A' + (bestIdx + 1) + ')</p>';
  }
  if (m.tagData.length > 0) {
    const topTag = m.tagData[0];
    const tagName = isJa ? topTag.tag : (TAG_EN[topTag.tag] || topTag.tag);
    obsHtml += '<p style="margin-top:8px;"><strong>"' + escapeHtml(tagName) + '"</strong> ' + (isJa ? 'が最もいいねを集めている（' + topTag.total + '件）。' : 'has the most likes (' + topTag.total + ' total).') + '</p>';
  }
  obsHtml += '</div>';
  obs.innerHTML = obsHtml;
}
```

Add `renderArticlePerfChart(m)` and `renderTagsChart(m)` as separate functions.

- [ ] **Step 2: Verify in browser**

- Article performance chart shows grouped bars + like rate line
- X-axis shows A1, A2, ... labels
- Dual Y-axis labels correct
- Tags chart shows horizontal stacked bars, top 10
- "(showing top 10 of X)" appears if >10 tags
- Observation panel shows like rate range and top tag
- All text updates on language toggle

- [ ] **Step 3: Commit**

```bash
git add app/index.html
git commit -m "Add article performance and tag reach charts with observation panels"
```

---

### Task 7: Section 4 — Follower Journeys chart

**Files:**
- Modify: `app/index.html` (replace `renderSection4` stub)

- [ ] **Step 1: Implement renderSection4**

Scatter chart with connecting lines. Y = follower name, X = time. Blue dots for likes, green triangles for follows. Lines connect same person's events.

```javascript
let chartJourneys = null;

function renderSection4(m) {
  const main = document.getElementById('section4Main');
  const obs = document.getElementById('section4Obs');
  const isJa = currentLang === 'ja';

  if (m.journeys.length === 0) {
    main.innerHTML = '<div class="action-title">' + (isJa ? 'まだフォロワーデータがありません' : 'No follower data yet') + '</div>';
    obs.innerHTML = '';
    return;
  }

  let html = '<div class="action-title">' + m.actionTitles.journeys + '</div>';

  // Decide rendering mode based on count
  if (m.journeys.length <= THRESHOLDS.journeyAggregateAt) {
    // Individual journeys
    const maxH = m.journeys.length > THRESHOLDS.journeyScrollAt ? THRESHOLDS.timelineMaxHeight : undefined;
    const wrapStyle = maxH ? 'max-height:' + maxH + 'px;overflow-y:auto;' : '';
    html += '<div style="position:relative;' + wrapStyle + '"><canvas id="chartJourneys"></canvas></div>';
  } else {
    // Aggregated funnel view (placeholder until we have 50+ followers)
    html += '<div id="journeysFunnel"></div>';
  }

  html += '<div class="chart-legend">';
  html += '<div class="legend-item"><div class="legend-dot" style="background:' + COLOURS.accent + ';"></div> ' + (isJa ? 'いいね' : 'Like') + '</div>';
  html += '<div class="legend-item"><div class="legend-tri"></div> ' + (isJa ? 'フォロー' : 'Follow') + '</div>';
  html += '<div class="legend-item" style="color:var(--ink-muted);">' + (isJa ? '線は同じ人のアクションを時系列で接続' : "Lines connect the same person's actions over time") + '</div>';
  html += '</div>';

  main.innerHTML = html;

  if (m.journeys.length <= THRESHOLDS.journeyAggregateAt) {
    renderJourneysChart(m);
  } else {
    renderJourneysFunnel(m);
  }

  // Observation panel
  const likedMultiple = m.journeys.filter(j => j.likes.length > 1).length;
  const likedOne = m.journeys.filter(j => j.likes.length === 1).length;
  const likedNone = m.journeys.filter(j => j.likes.length === 0).length;

  let obsHtml = '<div class="obs-box">';
  obsHtml += '<p><strong>' + likedMultiple + '</strong> ' + (isJa ? 'フォロワーが複数記事にいいね後にフォロー。' : 'followers liked multiple articles before following.') + '</p>';
  obsHtml += '<p><strong>' + likedOne + '</strong> ' + (isJa ? 'フォロワーが1記事にいいね後にフォロー。' : 'liked one article then followed.') + '</p>';
  if (likedNone > 0) {
    obsHtml += '<p><strong>' + likedNone + '</strong> ' + (isJa ? 'フォロワーがいいねなしでフォロー。' : 'followed without liking first.') + '</p>';
  }
  obsHtml += '</div>';
  obs.innerHTML = obsHtml;
}
```

Add `renderJourneysChart(m)` — Chart.js scatter with `showLine: true` for each follower's events. Each follower is a dataset. Point styles vary by event kind.

Add `renderJourneysFunnel(m)` — simple HTML funnel showing "X liked 1 article → followed", "X liked 2+ articles → followed", "X followed without liking" as horizontal bars.

- [ ] **Step 2: Verify in browser**

- Journey chart shows connecting lines for each follower
- Blue dots for likes, green triangles for follows
- Y-axis shows follower names
- Observation panel shows multi-like/single-like/no-like breakdown
- At >15 followers: chart area has scroll
- Action title is computed

- [ ] **Step 3: Commit**

```bash
git add app/index.html
git commit -m "Add follower journeys chart with aggregation threshold"
```

---

### Task 8: Data tab + Dictionary tab + final wiring

**Files:**
- Modify: `app/index.html` (add Data tab and Dictionary tab content, final integration)

- [ ] **Step 1: Add Data tab HTML and functions**

Carry over from current file:
- Import zone HTML (upload, paste, export buttons, drag-drop area)
- Events table with sortable columns
- Import history section
- All handlers: `processImport`, `handleFileUpload`, `pasteFromClipboard`, `exportData`, `copyScript`
- Drag-and-drop event listeners
- `renderEventsTable`, `renderImportHistory`, `sortTable`, `filterTable`

Ensure the import zone text and button labels use `data-i18n` attributes for language switching.

- [ ] **Step 2: Add Dictionary tab HTML and functions**

Carry over from current file:
- Data definitions grid
- Three extraction scripts (notifications, profile, dashboard) with copy buttons
- Known limitations list

Add the BST/GMT limitation text: "GMT is shown year-round. During British Summer Time (late March to late October), UK local time is GMT+1."

- [ ] **Step 3: Add seed data**

Carry over the existing `SEED_DATA` constant and `seedIfEmpty()` function. The seed data provides initial demo data so the app isn't empty on first load.

- [ ] **Step 4: Wire up renderAll and verify complete app**

Ensure `renderAll` calls all render functions in order. Ensure `loadAndRender` loads from IndexedDB and calls `renderAll`. Ensure `init` opens DB, seeds if empty, and calls `loadAndRender`.

Open in browser and verify the complete flow:
- App loads with seed data
- All 4 chart sections render with correct visual language
- KPIs show correct values, no colour coding
- Article key bar visible at bottom
- Language toggle updates everything
- Data tab: import works, table populates
- Dictionary tab: definitions, scripts, limitations all render
- No console errors
- Tooltips render above peak zones
- Status bar shows "X articles, Y total readers"

- [ ] **Step 5: Commit**

```bash
git add app/index.html
git commit -m "Complete data tab, dictionary tab, seed data, and final wiring"
```

---

### Task 9: Polish and edge cases

**Files:**
- Modify: `app/index.html`

- [ ] **Step 1: Test and fix empty states**

Clear IndexedDB (`indexedDB.deleteDatabase('noteAnalytics')` in console, refresh). Verify:
- KPIs show "--" dashes
- Charts show "Import data to see this chart" message
- Article key bar is hidden
- Status bar shows empty state message
- No console errors

- [ ] **Step 2: Test responsive behaviour**

Resize browser to 768px width. Verify:
- Chart sections stack to single column (chart above, observation below)
- KPI grid wraps to 3 columns
- Article key bar still visible and functional
- No horizontal overflow

- [ ] **Step 3: Test language toggle end-to-end**

Switch to JP. Verify every visible text element is in Japanese:
- Header, status bar, tabs
- KPI labels and sub-labels
- Section headings
- Action titles
- Chart legends
- Observation panels
- Article key bar and panel
- Data tab labels
- Dictionary content

Switch back to EN and verify the reverse.

- [ ] **Step 4: Remove dead code and old comments**

Search for any leftover references to:
- Old colour variables (BLUE, GREEN, AMBER, etc.)
- Old function names (renderFunnel, renderActions)
- "floating" article key references
- "UK" timezone label (should be "GMT")
- Em dashes
- Any "may..." or advisory language

- [ ] **Step 5: Final commit**

```bash
git add app/index.html
git commit -m "Polish edge cases, empty states, responsive layout, and language toggle"
```

---

## Plan Self-Review

**Spec coverage check:**
- [x] App header: "Note.com Analytics", status bar — Task 1
- [x] KPI grid with trend arrows, no colours — Task 2
- [x] Article key sticky bottom bar — Task 3
- [x] Chart visual language (grey-out/colour-in, action titles, observation panels) — Tasks 4-7
- [x] Engagement timeline with toggle — Task 4
- [x] Activity by hour with peak zone, dual axis JST/GMT — Task 5
- [x] Day of week — Task 5
- [x] Article performance — Task 6
- [x] Tag reach — Task 6
- [x] Follower journeys with scaling — Task 7
- [x] Data tab — Task 8
- [x] Dictionary tab with BST note — Task 8
- [x] Empty states — Task 9
- [x] Scaling thresholds — Tasks 4-7 (each chart checks thresholds)
- [x] Section headings — Task 1 (CSS + renderHeadings)
- [x] Computed action titles — Task 1 (generators in computeMetrics)
- [x] Trend arrows stored in IndexedDB — Tasks 1-2

**Type consistency check:**
- `COLOURS` constant used consistently (not `COLORS` or inline hex)
- `ARTICLE_COLOURS` not `ARTICLE_COLORS`
- `getArticleColour` not `getArticleColor`
- `jstToGmt` not `jstToUk`
- `metrics_snapshot` store name consistent
- `THRESHOLDS` constant referenced by name in all render functions

**No placeholders found.** All steps contain code or exact verification criteria.
