# AGENTS.md

## Project Overview

This is a **Today I Learned (TIL)** blog — a collection of short notes on things learned, published as a static site via GitHub Pages + Jekyll.

## Adding a New Post

### Steps

1. Find the next chronological number by checking existing posts:
   ```bash
   ls _posts/*/
   ```
   Use `NNN` as the next number (e.g., if max is 003, use 004).

2. Create the post file in the correct category directory:
   ```
   _posts/{category}/YYYY-MM-DD-NNN-slug.md
   ```

3. Use the template from `_templates/post.md` as a starting point.

4. Update `README.md` — add a row to the Posts table with the new entry.

### File Naming

```
YYYY-MM-DD-NNN-slug-with-title.md
```

- `YYYY-MM-DD` — creation date
- `NNN` — chronological sequence number (001, 002, 003...)
- `slug` — lowercase, hyphenated title

### Frontmatter

Every post MUST include this frontmatter:

```yaml
---
title: "Title Here"
date: YYYY-MM-DD
category: CATEGORY
tags: [tag1, tag2]
summary: "One-line summary"
---
```

### Categories

Use existing category directories under `_posts/`. Create a new directory if the topic doesn't fit existing categories:
- `devops/` — CI/CD, Docker, deployment, infrastructure
- `networking/` — VPN, Tailscale, routing, DNS
- `security/` — auth, encryption, hardening
- `backend/` — APIs, databases, server-side
- `frontend/` — UI, React, CSS
- `git/` — git workflows, branching

### Content Guidelines

- Start with the problem or what was learned
- Include code snippets with language tags
- Add key takeaways or lessons learned
- Keep it concise — TIL format, not a tutorial

### After Creating

Run `ls _posts/*/` to verify the file is in the right place with correct naming.
