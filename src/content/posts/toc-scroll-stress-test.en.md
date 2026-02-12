---
locale: "en-US"
translationKey: "toc-scroll-stress-test"
title: "Long-Form Stress Test: Static Blog Operations from 0 to 1"
description: "A long-form test article for TOC scroll highlighting, ECharts rendering, tables, formulas, and code blocks."
date: "2026-02-10"
tags: ["architecture", "performance", "seo", "testing", "astro"]
draft: false
---

This article is intentionally long. Its purpose is to validate the full rendering pipeline, not to optimize literary style.

## 01. Goals and Constraints

The blog should not only "work", but remain operable, portable, and maintainable over time.

- Markdown is the single source of truth.
- Build output stays pure static HTML.
- Runtime scripts stay minimal.
- The architecture must support future VPS migration via `git pull + build`.

### 01.1 What we avoid

Most failures come from front-loading complexity. Introducing CMS/database/auth too early raises long-term maintenance cost.

### 01.2 Why static-first

A static-first architecture keeps operations simple and hosting portable.

### 01.3 Core assumption

Content lifecycle is longer than platform lifecycle. You may change CDN, server, or build stack, but content should remain stable.

## 02. Information Architecture

Three layers:

- Interface pages (home, tags, posts, about)
- Content (Markdown + metadata)
- Distribution (sitemap, RSS, structured data)

### 02.1 URL policy

Recommended paths:

- `/posts/{slug}/`
- `/tags/{tag}/`

### 02.2 Slug naming recommendation

- Lowercase English + hyphen.
- Avoid Chinese slugs for cross-system stability.
- Do not change slugs after publishing.

### 02.3 Tag taxonomy recommendation

Keep tag count bounded (for example 20-40 stable topics) and avoid creating one-off tags for every post.

## 03. Performance Model

$$
T_{user} \approx T_{network} + T_{html} + T_{paint} + T_{js}
$$

The static-site advantage is minimizing `T_js` so first-view latency depends mostly on network + HTML.

### 03.1 Above-the-fold budget

Practical baseline:

- First-screen CSS < 70KB (gzip)
- First-screen JS < 80KB (gzip)
- LCP < 2.0s (4G simulation)

### 03.2 Image strategy

- Use `webp/avif` for cover images.
- Export body images near content width; avoid raw 4K uploads.
- Keep naming deterministic under `/public/images/posts/{slug}/...`.

### 03.2.1 Media rendering stress test (images + GIF)

![Stress Image 1](https://images.unsplash.com/photo-1770885653391-5cd1f045aba8?w=700&auto=format&fit=crop&q=60&ixlib=rb-4.1.0&ixid=M3wxMjA3fDB8MHxmZWF0dXJlZC1waG90b3MtZmVlZHwzMXx8fGVufDB8fHx8fA%3D%3D)

![Stress Image 2](https://images.unsplash.com/photo-1641141109253-6c1fc4f53e66?w=700&auto=format&fit=crop&q=60&ixlib=rb-4.1.0&ixid=M3wxMjA3fDB8MHxzZWFyY2h8M3x8Z2lmc3xlbnwwfHwwfHx8MA%3D%3D)

![Stress GIF](https://media.tenor.com/NDTi9GwBHUAAAAAM/gif-tenor.gif)

### 03.3 When to add client-side scripts

Add runtime JS only when interaction value is explicit (charts, search suggestion, focused motion).

## 04. Markdown Rendering Baseline

### 04.1 Table rendering

| Dimension | Target | Note |
| --- | --- | --- |
| Readability | High | Comfortable line-height and spacing |
| Mobile | Horizontal scroll | Avoid forced squeezing |
| Semantics | Keep native `table` | Better SEO and accessibility |

### 04.2 Code block rendering

```ts
interface PostMeta {
  title: string;
  slug: string;
  date: string;
  tags: string[];
  draft?: boolean;
}

export function toRoute(meta: PostMeta): string {
  return `/posts/${meta.slug}/`;
}
```

### 04.3 Math rendering

Inline example: $E = mc^2$.

Block example:

$$
\text{Score}_{seo} = 0.4 \cdot C + 0.3 \cdot I + 0.3 \cdot U
$$

Where:

- $C$: content quality
- $I$: index coverage
- $U$: URL stability

### 04.4 ECharts rendering

```echarts
{
  "tooltip": { "trigger": "axis" },
  "legend": { "data": ["Organic", "Direct"] },
  "xAxis": {
    "type": "category",
    "data": ["W1", "W2", "W3", "W4", "W5", "W6"]
  },
  "yAxis": { "type": "value", "name": "Visits" },
  "series": [
    {
      "name": "Organic",
      "type": "line",
      "smooth": true,
      "data": [1200, 1480, 1710, 1940, 2130, 2410],
      "areaStyle": {}
    },
    {
      "name": "Direct",
      "type": "line",
      "smooth": true,
      "data": [600, 720, 810, 900, 940, 1030]
    }
  ]
}
```

## 05. SEO Infrastructure

SEO is not keyword stuffing. The core is discoverability + machine readability + sustainable updates.

### 05.1 Required technical items

- `sitemap.xml`
- `rss.xml`
- canonical
- Open Graph / Twitter cards
- JSON-LD article metadata

### 05.2 Content strategy

- Each post should solve one concrete problem.
- Avoid vague headline wording.
- Provide the answer early, then expand.

### 05.3 Topic clusters

Series around stable themes accumulate authority better than random disconnected topics.

## 06. Multilingual Strategy

You already decided on language switching. Recommended approach is independent content per locale.

### 06.1 Recommended approach

- One Markdown file per language.
- Link variants through `translationKey`.
- Language switch should jump to the corresponding localized page.

### 06.2 Why not front-end text-only replacement

Replacing only UI labels does not cover article body and is weak for language-specific indexing.

### 06.3 Fallback when translation is missing

If a target locale is missing:

- fallback to default locale
- show a clear "translation not available yet" message

## 07. TOC UX Requirements

TOC is navigation infrastructure for long-form content, not decoration.

### 07.1 Highlighting rules

- Active state follows scroll position.
- Click on TOC item should immediately highlight the item.
- Highlighting should remain correct after resize/zoom changes.

### 07.2 Desktop and mobile

- Desktop: right-side sticky TOC.
- Mobile: collapsible TOC to protect content width.

### 07.3 Visual rhythm

TOC should stay subtle while active sections remain clearly identifiable.

## 08. Content Production Workflow

Standardize writing flow so the system supports you, not the other way around.

### 08.1 Writing process

1. Draft 5-8 section headings first.
2. Keep each paragraph focused on one point.
3. Add images/links after core writing is complete.

### 08.2 Working with AI

- You define structure and opinions.
- AI helps expand and polish.
- Final fact/tone review stays human.

### 08.3 Pre-publish checklist

- Title/description length is reasonable.
- TOC hierarchy is clean.
- Code/charts are readable.
- Mobile reading experience is acceptable.

## 09. Maintainability and Migration

### 09.1 Why static sites migrate easily

Content and hosting platform are naturally decoupled. Build output is regular files deployable on any web server.

### 09.2 Migration from Cloudflare to VPS

Minimal steps:

1. `git pull`
2. `npm ci`
3. `npm run build`
4. serve `dist/` via Nginx

### 09.3 Minimal operations surface

- reduce runtime services
- reduce online dependencies
- move complexity to build time

## 10. Final Note

If you optimize for "looks powerful", the project gets heavier. If you optimize for long-term operation, it gets more stable.

This long-form post is designed to test TOC behavior and rendering reliability. Scroll through and verify:

- TOC highlighting remains stable
- ECharts renders correctly
- tables and formulas stay readable on mobile

After these fundamentals are stable, then optimize homepage visual effects.
