---
name: content-pipeline
description: Use when the user wants to run a content production pipeline across one or more markets — keyword research followed by blog post writing. Also use when the user says "create content for all markets", "I need blog posts", "run content sprint", "keyword research and blog posts", "write posts for [market]", "content batch", or wants to go from keyword research to published blog posts in one workflow.
---

# Content Pipeline

Orchestrate a full content production sprint: parallel keyword research across markets, topic selection with user approval, then parallel blog post writing — all optimized for SEO, GEO, and AEO.

---

## Step 1: Intake

Collect everything in one message. If the user already provided some inputs, extract them and only ask for what's missing.

1. **Which markets?** — pull active markets from `client-config.md` Markets table; ask the user to pick a subset or default to all
2. **Keyword focus / period?** — topic area or seasonal focus (e.g., "spring product launch", "enrollment campaign September")
3. **How many blog posts per market?** — default 2-3 if not specified

Also confirm:
- **Brand context** — pull from `client-config.md` (brand name, tone, markets, channels). If missing, route to `client-onboarding` first.
- **Competitors** — pull from `notes/competitors.md` if it exists, or ask the user.

---

## Step 2: Parallel Keyword Research

Spawn one sub-agent per market. All run simultaneously.

Each agent gets the keyword-researcher skill instructions plus market-specific context:

```
Agent({
  name: "keywords-[market]",
  description: "Keyword research for [market]",
  prompt: `
    [Insert keyword-researcher skill instructions from skills/keyword-researcher.md]

    CONTEXT:
    - Brand: [brand name]
    - Website: [market-specific URL]
    - Market: [country] / Language: [language]
    - Focus: [keyword focus / period from intake]
    - Competitors: [competitor list]
    - Audience: [audience description]

    Skip the discovery interview — all inputs are provided above.
    Start from Phase 2 (Website Scan) and run through to Phase 6 (Deliver Report).

    Save the keyword report to: outputs/keyword-research/keyword-report-[market].md

    IMPORTANT: For each keyword cluster, tag which channels it serves:
    - SEO (organic ranking)
    - AEO (featured snippets, answer boxes)
    - GEO (AI-generated responses, LLM citations)
    - Ads (paid search)

    After delivering the full report, also output a separate section:
    TOP BLOG POST IDEAS — list 5-10 blog post ideas derived from P1 keywords.
    For each idea include:
    - Title (in target market language)
    - Primary keyword + secondary keywords
    - Channel fit: which of SEO/AEO/GEO this post targets
    - Hook: one sentence on the angle
    - Urgency: flag if seasonal or time-sensitive
  `,
  run_in_background: true
})
```

While agents run, wait for all to complete before proceeding.

---

## Step 3: Review Gate

Once all keyword research agents return, present a consolidated topic selection table to the user.

### Presentation Format

For each market, show the top blog post ideas:

```
## [Market Name] — Top Blog Post Ideas

| # | Title | Primary Keyword | Channel Fit | Urgency |
|---|-------|----------------|-------------|---------|
| 1 | [title in market language] | [keyword] | SEO + AEO | Peak season |
| 2 | ... | ... | GEO + SEO | — |

Full keyword report: [link to keyword-report-[market].md]
```

Then ask:

> "Here are the top content opportunities across your markets. Pick which posts to write — you can select by number, tweak angles, or add your own topics. How many total posts do you want produced?"

**Wait for user selection before proceeding.** Do not auto-pick topics.

---

## Step 4: Parallel Blog Writing

Spawn one sub-agent per selected blog post. All run simultaneously.

Each agent gets the blog-post-writer skill instructions plus keyword data and SEO/GEO/AEO optimization layer:

```
Agent({
  name: "blog-[market]-[slug]",
  description: "Write blog post: [short title]",
  prompt: `
    [Insert blog-post-writer skill instructions from skills/blog-post-writer.md]

    CONTEXT:
    - Brand: [brand name]
    - Market: [country] / Language: [language]
    - Website: [market-specific URL]
    - Audience: [audience description]
    - Tone: [tone]
    - Primary keyword: [keyword from selection]
    - Secondary keywords: [from keyword research]
    - Channel fit: [SEO/AEO/GEO tags from keyword research]
    - Goal: [user's stated goal or default to "sign up"]

    Skip Phase 1 (Collect Brand Inputs) and Phase 3 (Propose Ideas) — the topic
    is already selected. Start with Phase 2 (Research) and write the full post.

    === SEO / GEO / AEO OPTIMIZATION LAYER ===

    This post must be optimized for all three search surfaces, not just
    traditional SEO. Apply these principles throughout:

    **AEO (Answer Engine Optimization):**
    - Open each major section with a direct, concise answer paragraph
      (40-60 words) that could be pulled as a featured snippet
    - Use question-format H2s where natural: "How do I...", "What is...",
      "Why does..."
    - Include a FAQ section (3-5 questions) at the end with short,
      direct answers — this is featured snippet and People Also Ask fuel
    - Use numbered steps and tables where the content supports it —
      these formats win answer boxes

    **GEO (Generative Engine Optimization):**
    - Write entity-rich content: use specific names, statistics, dates,
      and citations rather than vague claims
    - Include "AI-citable" paragraphs — standalone statements that an LLM
      could quote verbatim as an authoritative source (factual, specific,
      well-attributed)
    - Add author/brand credibility signals: "According to [Brand]...",
      "Based on data from [X students]..."
    - Reference specific studies, data points, or expert perspectives
      where available — LLMs favor content they can attribute
    - Make the content comprehensive enough to be THE definitive resource
      on this topic in this language

    **SEO (Traditional Organic):**
    - Primary keyword in: title, H1, first 100 words, 2-3 H2s, meta
      description, conclusion
    - Secondary keywords distributed naturally (1-2 uses each)
    - Internal link suggestions after each major section
    - Short paragraphs, varied sentence length, scannable structure
    - Target word count: 1,200-1,800 words (comprehensive enough for
      GEO authority, structured enough for AEO snippets)

    === END OPTIMIZATION LAYER ===

    Save the finished post to: outputs/blog/[brand]-[market]-blog-[slug].[ext]
  `,
  run_in_background: true
})
```

---

## Step 5: Delivery

Once all blog post agents return, present a summary table:

```
## Content Sprint Complete

| Market | Title | Primary Keyword | Word Count | Channels | File |
|--------|-------|----------------|------------|----------|------|
| SI | [title] | [keyword] | ~1,400 | SEO+AEO | [link] |
| HU | [title] | [keyword] | ~1,300 | GEO+SEO | [link] |
| ... | ... | ... | ... | ... | ... |

Keyword reports: [links to each market's keyword report]
```

Then offer next steps:

> "All posts are ready. Want me to:
> 1. Publish any of these via WordPress? (wp-blog-publisher skill)
> 2. Create social media snippets from the posts?
> 3. Export the keyword reports and content calendar to Google Sheets?
> 4. Run another batch for different topics?"

---

## Error Handling

- If a keyword research agent fails (e.g., website fetch fails), report which market failed and offer to retry or skip
- If a blog writing agent produces a post that's too short (<800 words) or misses the optimization layer, flag it in the summary table
- If the user selects 0 posts for a market at the review gate, skip that market for writing — don't force content

---

## Tone

This is a production pipeline, not a tutorial. Move fast, present data cleanly, wait at the review gate, then execute. The user is a CMO running content operations — match that pace.
