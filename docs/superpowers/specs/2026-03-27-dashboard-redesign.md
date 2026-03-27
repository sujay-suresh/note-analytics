# Note.com Analytics Dashboard Redesign

## Context

Single-page analytics dashboard for a note.com content creator (strategy consultant based in UK, publishing to a Japanese audience). Built as a standalone HTML/JS app with Chart.js, IndexedDB for persistence, JSON import via browser console extraction scripts. EN/JP bilingual.

The current implementation has 41 identified UX issues across colour consistency, wording clarity, chart storytelling, scalability, and assumed knowledge. This spec defines the complete redesign.

## Audience

The primary user is the author herself. She is data-literate (strategy consultant), can interpret charts and draw her own conclusions. The dashboard should surface patterns clearly and let her decide what to do. No prescriptive advice. No unexplained judgements.

## Design Principles

1. **Observations, not advice.** State what the data shows. Never "you should..." or "this may...". The author is a consultant; she decides.
2. **Computed, not authored.** Every annotation, title, and observation is derived from the data and updates when data changes. No static editorial text.
3. **Grey-out, colour-in.** Default everything to grey. Use one accent colour to highlight only the finding the title refers to. The reader's eye goes to colour first.
4. **Action titles.** Every chart title is a sentence stating the computed finding, not a label. "Most likes arrive between 17:00 and 21:00 JST" not "Activity by Hour".
5. **No traffic-light judgements.** No red/amber/green on metrics. No benchmarks exist yet. Show trends over time instead.
6. **Plain language in both languages.** No jargon, no abbreviations without context. No em dashes. UK English spelling.

## App Header

- Title: "Note.com Analytics" (capital N)
- Right side: status summary (e.g., "6 articles, 28 total readers"), language toggle (EN/JP)

## Tabs

Three tabs, unchanged: Briefing, Data, Dictionary.

## Briefing Tab Structure

Top-to-bottom reading order:

1. KPI grid (6 metrics)
2. Article key (sticky bottom bar, expandable)
3. Section 1: Engagement timeline (chart + right panel)
4. Section 2: Activity by hour + Day of week (charts + right panel)
5. Section 3: Article performance + Tags (charts + right panel)
6. Section 4: Follower journeys (chart + right panel)

Each section uses the consulting slide layout: chart on the left, observation panel on the right.

## KPI Grid

Six metrics in a single row. No colour coding on the numbers. All use the same neutral text colour.

| Metric | Value | Sub-label (EN) | Sub-label (JP) |
|--------|-------|-----------------|-----------------|
| Articles Published | count | "over X days" | "X日間に公開" |
| Avg. Views per Article | number | "X total views" | "合計X閲覧" |
| Avg. Likes per Article | number | "X total likes" | "合計Xいいね" |
| Like Rate | percentage | "% of viewers who liked" | "閲覧者のうちいいねした割合" |
| Follow Rate | percentage | "X followed out of Y who liked" | "いいねしたY人中X人がフォロー" |
| Return Rate | percentage | "X came back to like another article (of Y)" | "X人が2件以上の記事にいいね（Y人中）" |

### Trend Arrows

From the second data import onwards, each KPI shows a small trend indicator comparing current value to previous import:

- Up arrow (neutral colour, e.g., dark grey) if metric increased
- Down arrow if decreased
- No arrow on first import (no comparison available)
- Arrow is purely directional, no colour judgement (not green for up, red for down)

The trend comparison stores the previous import's computed values in IndexedDB alongside the current values.

## Article Key

Sticky bottom bar pinned to the viewport bottom. Visible on the Briefing tab only.

**Collapsed state (default):** Single line showing colour dot + "A1", "A2", etc. for each article. Full article count visible.

**Expanded state (click to expand):** Expands upward into a panel showing:
- Colour dot + A-number + full title + view count for each article
- At 10+ articles: adds a search/filter input at the top
- Scrollable list

**Scaling behaviour:**
- 1 to ~15 articles: all visible inline in collapsed bar
- 15+: collapsed bar shows first ~12 with "+X more", expanded view has search

**Purpose label:** The bar itself communicates that colours in charts correspond to these articles.

## Chart Visual Language

Applied consistently to every chart on the Briefing tab.

### Layout

Each chart section is a card with two columns:
- **Left (wider):** Action title, chart, axis labels, legend
- **Right (220px):** Observation panel with computed facts in a grey rounded box (#f3f4f6 background, 6px border-radius, padded)

### Action Titles

Every chart title is a complete sentence computed from the data:
- "Most likes arrive between 17:00 and 21:00 JST"
- "A1 has the highest like rate at 29%"
- "The tag 'consulting' attracts the most likes"
- "3 followers liked before they followed"

Titles update automatically when data changes.

### Colour System

- **Accent (bars in focus):** #1e40af (darker blue for bars), #1e3a8a (border/emphasis)
- **Peak zone background:** #dbeafe (lighter blue, clearly distinct from the darker bars)
- **Grey (context bars):** #d1d5db
- **Publish time markers:** #dc2626 (red, small axis ticks)
- **Follow events:** #059669 (green, triangles)
- **Article colours:** Cycling palette for article identification: ['#2563eb', '#7c3aed', '#059669', '#d97706', '#dc2626', '#0891b2']. Used only in charts where article identity matters (timeline, article performance). Not used in the activity-by-hour chart (which uses grey/accent only).

### Annotations

- **Peak zone:** Computed from data (top N hours by engagement, or hours above 75th percentile). Rendered as a shaded background region (#dbeafe) behind the relevant bars. "Peak" label sits inside the shaded region at the top, not hitting the box ceiling (adequate top padding).
- **Publish time markers:** Small red ticks on the x-axis. Progressive scaling:
  - 1 to 10 articles: individual ticks, hoverable for article name
  - 11+: density heatmap strip (darker = more articles published at that hour)
- **Tooltips:** z-index set above all other elements including peak zone labels. Show both timezones and the count.

### Axis Labels

For the activity-by-hour chart:
- **JST row** directly beneath the chart bars (closest to x-axis). Every hour labelled (0-23). Font: 10px, weight 500, colour #1a1a2e. Row label: "JST" in 9px grey.
- **GMT row** directly beneath JST. Every hour labelled. Font: 9px, colour #9ca3af. Row label: "GMT" in 9px grey.
- No extra text like "audience timezone" or "your timezone". Just "JST" and "GMT".

Note on BST: GMT is used year-round for simplicity. A footnote in the Dictionary tab notes that during British Summer Time (late March to late October), UK local time is GMT+1.

### Legend

Compact, below the axis. Uses the same visual elements as the chart (dot for bar colours, tick for publish markers, triangle for follows). No redundant items.

### Observation Panel (Right Side)

- Grey rounded box (#f3f4f6 background, #e5e7eb border, 6px radius, 12px 14px padding)
- Contains computed facts with bold numbers: "**3 of 5 articles** were published before the peak zone. **2 of 5** were published during it."
- Below the box, lighter text (11px, #6b7280) for supplementary computed stats: "50 likes (70%) arrived during peak hours across all articles."
- No label ("Observation:" etc.). Just the facts.
- Text formatted to avoid widows/orphans. Sentences should not leave single words on a new line.

## Chart Specifications

### Chart 1: Engagement Timeline

**Purpose:** Show when each like and follow happened relative to article publish dates.

**Chart type:** Scatter (x = time, y = article row). Blue dots for likes, green triangles for follows. All likes use one blue colour (not per-article colours). The Y-axis row position identifies the article.

**Action title example:** "Most engagement arrived within 48 hours of publish" (computed from data).

**Publish date markers:** Dashed grey vertical lines at each article's publish date.

**Toggle:** "Show People" / "Show Timeline"
- Under ~100 unique readers: "Show People" displays follower journey drill-down view (individual paths: liked A1, then A3, then followed)
- Over ~100: "Show People" displays aggregated view (e.g., "X% of followers liked 2+ articles before following", distribution charts)

**Scalability:** Chart height computed as max(280px, (articleCount + 2) * 50px). At 60 articles this is 3100px which is too tall. At 15+ articles, switch to a condensed row height (30px per row) and add vertical scroll within the chart container (max height 600px).

### Chart 2: Activity by Hour

**Purpose:** Show which hours of the day get the most engagement.

**Chart type:** Bar (24 bars, one per hour). Grey for non-peak, darker blue (#1e40af) for peak zone. Peak zone background shading in #dbeafe.

**Action title example:** "Most likes arrive between 17:00 and 21:00 JST" (computed).

**Observation panel:** How many articles were published before/during/after the peak zone.

### Chart 3: Day of Week

**Purpose:** Show which days of the week get the most engagement.

**Chart type:** Stacked bar (7 bars). Likes in blue, follows in green. Published-day markers as a separate subtle bar layer.

**Action title example:** "Wednesday and Thursday see the most engagement" (computed).

### Chart 4: Article Performance

**Purpose:** Compare views, likes, and like rate across articles.

**Chart type:** Grouped bar (views as light bars, likes as solid bars) with a line overlay for like rate on a secondary Y-axis.

**Action title example:** "A1 has the highest like rate at 29%" (computed from data).

**Axis labels:** Left Y = "Views / Likes", Right Y = "Like Rate (%)". X = article short labels (A1, A2...).

### Chart 5: Tag Reach

**Purpose:** Show which tags attract the most likes, split by repeat vs one-time readers.

**Chart type:** Horizontal stacked bar. Dark purple for repeat readers, light blue for one-time readers.

**Action title example:** "Articles tagged 'consulting' attract the most likes" (computed).

**Scaling:** Show top 10 tags. If there are more than 10, note "(showing top 10 of X)" in the legend area.

### Chart 6: Follower Journeys

**Purpose:** Show the path each follower took (liked which articles, in what order, before following).

**Chart type:** Scatter with connecting lines (x = time, y = follower). Blue dots for likes, green triangles for follows. Lines connect same person's actions.

**Action title example:** "3 followers liked before they followed" (computed).

**Scaling:** At 15+ followers, chart height capped at 600px with internal scroll. At 50+, switch from individual rows to an aggregated funnel view (how many liked 1 article before following, how many liked 2+, etc.).

## Data Tab

Unchanged in structure. Three areas:

1. **Import zone:** Upload JSON, paste from clipboard, export all data. Drag-and-drop support.
2. **Events table:** Sortable, searchable table of all events (likes, follows). Columns: Time (JST), Kind, Who, Article, Source.
3. **Import history:** List of past imports with event/article counts.

## Dictionary Tab

Unchanged in structure. Three areas:

1. **Data definitions:** Terms and their meanings (Like, Follow, Views, Like Rate, etc.)
2. **Extraction scripts:** Three browser console scripts (notifications, profile, dashboard) with copy buttons.
3. **Known limitations:** Including the BST/GMT note.

Add to limitations: "GMT is shown year-round. During British Summer Time (late March to late October), UK local time is GMT+1. Adjust mentally or check a converter."

## Internals

### IndexedDB Stores

- `events`: Individual like/follow events (keyPath: id)
- `articles`: Article metadata (keyPath: article_url)
- `imports`: Import history records (keyPath: timestamp)
- `metrics_snapshot`: Previous import's computed KPI values for trend comparison (keyPath: 'latest')

### Computed Metrics

All metrics computed client-side from IndexedDB data. Key computations:

- **Peak zone:** Hours where engagement count exceeds the 75th percentile of hourly counts.
- **Action titles:** Template strings with computed values inserted. Each chart has a title-generation function that reads the metrics and returns a sentence.
- **Trend arrows:** Current KPI values compared to `metrics_snapshot` store. On each import, current values are saved to the snapshot before recomputing.

### Scaling Thresholds

| Element | Threshold | Behaviour Change |
|---------|-----------|-----------------|
| Publish ticks | >10 articles | Switch from individual ticks to density strip |
| Timeline chart height | >15 articles | Condensed row height + internal scroll (max 600px) |
| Article key (collapsed) | >15 articles | Show first ~12 inline + "+X more" |
| Article key (expanded) | >10 articles | Add search/filter input |
| Follower journeys | >15 followers | Cap height at 600px with scroll |
| Follower journeys | >50 followers | Switch to aggregated funnel view |
| Timeline "Show People" | >100 readers | Switch from individual cards to aggregated patterns |
| Tag chart | >10 tags | Show top 10 with "(showing top 10 of X)" note |

## Section Headings

Each chart section has a heading above it. Use plain numbered text, not circled emoji numbers:
- "1. How engagement built over time"
- "2. When readers respond"
- "3. Which articles attract and retain readers"
- "4. How followers discovered and converted"

Font: 13px, weight 700, uppercase, letter-spacing 0.08em, colour #9ca3af.

## Empty States

When no data has been imported yet:

- **KPIs:** Show dashes ("--") for all values. No trend arrows.
- **Charts:** Show the card skeleton (title area, empty chart area with a centred message: "Import data to see this chart" / "データをインポートすると表示されます"). Grey text, no chart elements.
- **Article key:** Hidden (no articles to show).
- **Status bar:** "No data yet. Go to the Data tab to import." / "データなし。データタブからインポートしてください。"

## Clarifications

- **Article colours vs uniform blue:** The timeline chart (Chart 1) and activity-by-hour chart (Chart 2) use uniform blue for likes. Article identity is conveyed by row position (Y-axis), not colour. Charts where article identity is the point of comparison (Chart 4: Article Performance) use the per-article colour palette. The article key's colour mapping applies to Charts 4 and 5 only.
- **Trend arrows:** All current KPIs are "up is positive" metrics. If a metric were added where down is better (e.g., churn rate), the arrow would still be neutral colour. The direction is informational, not judgemental.

## Out of Scope

- AI-generated insights or API calls
- Comparative benchmarks against other creators
- Real-time data (all data is manual import)
- Mobile-optimised layout (responsive basics only)
- Dark mode
