# Note.com Content Analytics

A bilingual analytics dashboard built for a Japanese content creator, turning raw engagement data into editorial decisions.

## The Brief

A strategy consultant publishing on [note.com](https://note.com) (Japan's leading blogging platform) needed to understand what was working and why. With no built-in analytics beyond basic view counts, she had no way to connect publishing behaviour to reader engagement. The goal: a single dashboard that surfaces patterns across content, timing, and audience growth.

## What It Does

- **Conversion funnel** tracking views through to likes, follows, and repeat readers, with period-over-period trend indicators
- **Content performance** comparison across articles and tags, with automatic detection of ubiquitous tags that skew results
- **Publish timing analysis** by hour and day of week across dual timezones (JST/GMT), identifying peak engagement windows
- **Audience growth** visualisation showing cumulative follower journeys and how individual articles drive new followers
- **Computed observations** that surface findings as plain-language sentences rather than raw numbers
- **Full bilingual support** in Japanese and English, matching the author's cross-cultural audience

## Decisions Enabled

The dashboard is organised around three dimensions that map to the editorial questions that matter:

**Timing** - When should I publish?
Hourly and day-of-week engagement patterns revealed that weekday evening posts (JST) consistently outperformed weekend publishing, directly informing the author's scheduling.

**Content** - What should I write about?
Article-level like rates and tag-based reader segmentation (repeat vs one-time readers) showed which topics retained readers versus which attracted one-off traffic. Ubiquitous tags are automatically filtered so niche topics get a fair comparison.

**Audience** - Are readers staying?
Follower journey tracking and growth curves tied to specific publish dates showed which articles converted casual readers into followers, and at what rate.

## Screenshot

![Dashboard](screenshots/dashboard.png)

## Technical Note

Single-page browser app with no backend or database. Data is stored locally via browser storage and imported from note.com's notification feed. Fully bilingual JP/EN interface.

[Live demo](https://sujay-suresh.github.io/note-analytics/app/)

## About

Built by [Sujay Suresh](https://github.com/sujay-suresh). If you need help making sense of your data, get in touch.
