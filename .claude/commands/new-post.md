---
description: Draft a new blog post under content/posts/
argument-hint: [topic and target category]
---

Draft a new post about: $ARGUMENTS

Pick the right existing menu category from `hugo.toml`'s `[[menu.main]]` entries (手表知识与鉴别 / 欧洲购表与免税攻略 / 二手表行情观察 / 瑞士本地生活) and set the post's category front matter accordingly — look at an existing post under `content/posts/` for the exact front matter fields and format to match.

Structure: a title and opening that state plainly what the reader will get, H2/H3 sections that actually answer the question (not filler), a short FAQ addressing 2-4 real follow-up questions where it fits the topic, and a closing that's specific rather than a generic summary. Write from the author's real, stated expertise (瑞士腕表零售从业者) where the topic touches on watches — don't claim expertise the author doesn't have; for tax/legal/customs topics, hedge and point to official sources, matching the "not tax advice" disclaimer pattern already used in the duty-free-calculator tool. Target genuinely useful length for the topic, don't pad to hit a word count.

Run `/qa-content` on the draft before considering it done. After drafting, remind me to preview with `hugo server` before pushing — this repo builds via GitHub Actions on push, so a broken build only surfaces after commit.
