---
layout: default
title: Today I Learned
---

# Today I Learned

Short notes on things I learned today — mostly devops, networking, and engineering.

## Posts (Chronological)

| # | Date | Title | Category |
|---|------|-------|----------|
| 001 | 2023-11-22 | [Nginx Proxy Manager](devops/2023-11-22-001-nginx-proxy-manager.md) | devops |
| 002 | 2023-11-22 | [CI/CD with GitLab Runner](devops/2023-11-22-002-add-ci-cd-with-gitlab-runner.md) | devops |
| 003 | 2026-06-12 | [Tailscale + VPN + Docker Network Conflict](networking/2026-06-12-003-tailscale-network-conflict.md) | networking |

---

## Structure

```
today-i-learned/
├── _posts/
│   ├── devops/
│   │   ├── 2023-11-22-001-nginx-proxy-manager.md
│   │   └── 2023-11-22-002-add-ci-cd-with-gitlab-runner.md
│   └── networking/
│       └── 2026-06-12-003-tailscale-network-conflict.md
├── _templates/
│   └── post.md
├── _config.yml
├── Gemfile
└── .github/workflows/deploy.yml
```

Each post follows the format:
```
YYYY-MM-DD-NNN-slug-with-title.md
```

Posts include frontmatter with title, date, category, tags, and summary.
