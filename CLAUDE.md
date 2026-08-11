# Project: watch-guide-blog（钟表与星辰：瑞士生活手记）

**这个仓库跟其他 sibling 工具不一样：是 Hugo 静态站点生成器项目，不是零构建纯 HTML。**
中文博客，主题 PaperMod（普通 vendored 目录，不是 git submodule）。内容围绕：手表知识与鉴别、
欧洲购表与免税攻略、二手表行情观察、瑞士本地生活。

## Commands
- 本地预览：`hugo server`（需要本机装 Hugo，版本参考 `.github/workflows/hugo.yml` 里的 `HUGO_VERSION`）
- 构建：`hugo --gc --minify`（CI 里用的确切命令，输出到 `public/`）
- 没有 npm/前端构建链——构建工具是 Hugo 本身，不是 Node 生态

## 部署流程
`.github/workflows/hugo.yml` 自动化：push 到 `main` 触发 GitHub Actions 用 Hugo 构建并发布到 GitHub Pages。
**不需要手动构建或手动部署**，跟其他零构建仓库"直接 push 生效"不同——这里是 push 触发 CI 构建。

## 文件结构
- `content/posts/` — 博客文章（Markdown），命名即 slug
- `content/about.md` / `contact.md` / `impressum.md` / `privacy-policy.md` — 信任页
- `content/tools/_index.md` — 小工具导航页
- `layouts/_partials/` — 主题覆盖（`header.html`、`extend_head.html`、`extend_footer.html`）——
  GA4 (`G-7LZZQ43QD6`) 挂在 `extend_head.html` 里
- `i18n/` — Hugo 内置多语言字符串（不是 sibling 工具那套手搓 `data-i18n` 方案）
- `static/tools/watch-valuator/index.html` — **watch-valuator 工具的博客内嵌副本**，
  跟独立仓库 `watch-valuator/index.html` 内容需要人工保持同步（两边都要改，不是自动同步）
- `hugo.toml` — 主配置，含分类菜单（`[[menu.main]]`）和外部工具链接

## 明确禁止的事
- 不要把这个项目当成零构建静态站处理——它有真实构建步骤（Hugo），改动后不能只靠"刷新看效果"，
  本地要跑 `hugo server` 预览
- 不要忘记 `static/tools/watch-valuator/index.html` 是独立仓库的手动同步副本，
  改了其中一份要记得另一份是否也要改（详见 [[secondhand-watch-valuator-project]] 记忆）
- 不要假设主题目录 (`themes/PaperMod`) 是 git submodule 去做 `git submodule update` 之类操作——它是普通目录

## Claude 工作方式
- 加新文章：在 `content/posts/` 下新建目录/文件，按现有文章的 front matter 格式
- 涉及 `hugo.toml` 里菜单结构改动时，注意 `[[menu.main]]` 的 `parent`/`weight` 字段决定嵌套和排序
- ads.txt 用的是 Hugo 的 `static/ads.txt` 约定（会被复制到站点根），不是仓库根目录的 `ads.txt`——
  跟其他零构建仓库放法不一样

## 持续维护
每次你需要重复纠正 Claude 同一件事三次以上，就把结论补进这个文件对应章节。
