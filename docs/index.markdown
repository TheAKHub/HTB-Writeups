---
# Feel free to add content and custom Front Matter to this file.
# To modify the layout, see https://jekyllrb.com/docs/themes/#overriding-theme-defaults

layout: default
title: Home
---

# Writeups

# ✅ Jekyll + Hacker Theme Smoke Test

If you see this page with styled headings, links, and a post list below, the theme is working.

## Basics

- **Layout in use:** `{{ page.layout }}`
- **Build time (UTC):** `{{ site.time | date: "%Y-%m-%d %H:%M:%S" }}`
- **Base URL:** `{{ site.baseurl | default: "(none)" }}`
- **Site URL:** `{{ site.url | default: "(none)" }}`
- [HTB Labs]({{ "/HTB_Labs" | relative_url }})
> Tip: For project pages, set in `_config.yml`:
> ```
> url: "https://theakhub.github.io"
> baseurl: "/HTB-Writeups"
> ```

## Internal Links (checks `relative_url`)

- Home → [Index]({{ "/" | relative_url }})
- RSS → [feed.xml]({{ "/feed.xml" | relative_url }})
- Posts directory → [/_posts]({{ "/_posts/" | relative_url }})

## Code Block (syntax highlighting)

```bash
# ruby & jekyll versions (if building locally)
ruby -v
bundle exec jekyll -v
