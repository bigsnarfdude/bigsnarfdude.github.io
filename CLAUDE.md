# Blog Repo — bigsnarfdude.github.io

## Stack
- Jekyll with `minimal-mistakes` remote theme
- Posts in `_posts/` as markdown with YAML frontmatter
- Pages in `_pages/`
- Config: `_config.yml`

## Post Format

```markdown
---
title: "Post Title"
date: YYYY-MM-DD
categories:
  - research   # or: engineering, tools, etc.
tags:
  - tag1
  - tag2
---

*Month YYYY — bigsnarfdude*

---

Post body...
```

## Filename Convention
`_posts/YYYY-MM-DD-slug-with-hyphens.md`

## Content Focus
AI safety research blog. Topics include:
- Alignment faking detection (primary research area)
- Multi-agent systems (RRMA, TrustLoop)
- Mechanistic interpretability
- Software engineering for AI safety tooling

## When Writing Posts
- Match the author's voice: direct, technical, self-aware about complexity and failure modes
- Use `##` for section headers (not `#`)
- Code blocks use triple backticks with language hint
- Italicize the dateline: `*Month YYYY — bigsnarfdude*`
- No emojis unless the post tone calls for it
- Don't add a trailing "conclusion" section if the post ends naturally

## Deploying
Pushes to `master` auto-deploy via GitHub Pages.
