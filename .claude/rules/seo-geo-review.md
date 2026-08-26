# SEO / GEO review checklist for this blog

Reusable checklist from the 2026-08-26 review of 5 new posts. Re-apply this instead of re-deriving a method each time new posts (or edits to title/description) get reviewed.

## Title / description length — use CJK display-width, not raw character count

Google's SERP truncation is pixel-based, and a CJK glyph renders roughly **2x** the width of a Latin character/digit. Raw `len()` on a Chinese string is misleading — a 46-character Chinese title is *not* the same risk as a 46-character English one. Compute a width score instead:

```python
def width(s):
    w = 0
    for ch in s:
        w += 2 if ord(ch) > 0x2E80 else 1  # CJK/fullwidth vs Latin/digit/punctuation
    return w
```

Practical thresholds found to hold up against actual rendered `<title>` output on this site: keep **title ≲ 55–60 width-units**, **description ≲ 100–110 width-units**. Above that, expect mid-sentence truncation in search results. (Bash `${#var}` on this Windows/Git-Bash setup reports *byte* length for UTF-8 multi-byte strings, not character count — don't use it for this check; use the Python snippet above instead.)

## Before claiming a schema/meta gap, read the actual partial

`themes/PaperMod/layouts/_partials/templates/schema_json.html` is the source of truth for what `BlogPosting` JSON-LD actually emits per post:
- `description`: uses front matter `description` if set, else falls back to `.Summary` — so always set `description` explicitly rather than relying on the fallback.
- `keywords`: uses front matter `keywords` if set, else falls back to `.Params.tags` — this site has never set `keywords` on any post (checked all 36), so relying on the tags fallback is the established convention, not a gap.
- `image`: uses `cover.image` if the post sets one, else falls back to `site.Params.images` (the site-wide default image in `hugo.toml`) — so a post with no cover still gets *some* image in its schema, just not a post-specific one. Don't report "no schema image" as a finding without checking this fallback chain first.

## Standing GEO gap: no FAQ/Q&A block on blog posts

The site's **tool pages** (e.g. `alpine-route-planner`, `timezone-planner`) all have a `<details class="faq-item">` FAQ block feeding structured, highly-extractable Q&A content to AI answer engines and Google featured snippets. **Blog posts under `content/posts/` have never had this** — checked across the 5 posts added 2026-08-26 and it's absent site-wide. This is the single highest-leverage GEO improvement available on this blog and hasn't been done yet. Flag it again on every future SEO/GEO pass rather than assuming a prior pass added it, until it's actually implemented.

## Tag-taxonomy dilution from shared backlink tags

When multiple posts link to the same tool (e.g. `random-grouping-website`) and all get tagged with that tool's name as a topic tag, the resulting tag archive page clusters unrelated topics together (e.g. VIP 腕表沙龙 + DACH sales roleplay + 二手表鉴定, whose only real commonality is "happens to link this tool"). Not wrong, but worth a second thought before reflexively adding a "tool name" tag to every post that merely mentions the tool — a tag should reflect a real topical cluster a reader would want to browse, not just an internal cross-link marker.
