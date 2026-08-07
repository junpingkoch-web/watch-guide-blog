---
description: Draft a new blog post under content/posts/
argument-hint: [topic and target category]
---

Draft a new post about: $ARGUMENTS

Pick the right existing menu category from `hugo.toml`'s `[[menu.main]]` entries (手表知识与鉴别 / 欧洲购表与免税攻略 / 二手表行情观察 / 瑞士本地生活) and set the post's category front matter accordingly — look at an existing post under `content/posts/` for the exact front matter fields and format to match.

After drafting, remind me to preview with `hugo server` before pushing — this repo builds via GitHub Actions on push, so a broken build only surfaces after commit.
