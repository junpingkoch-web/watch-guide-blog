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
  GA4 (`G-7LZZQ43QD6`) 挂在 `extend_head.html` 里；`extend_footer.html` 里的"所有工具 · All tools"
  链接指回 `junpingkoch-web.github.io` 首页——跟其他 sibling 仓库 `footer-links` 里的同款链接是
  同一套约定，2026-08 审查时发现这个仓库当初漏加了，之后新增/改这个 partial 时留意别再漏
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
- 不要直接信任本地 `hugo --gc --minify` 编译出的 `public/` 内容——如果同时有 `hugo server` 在跑，或者
  `resources/_gen` 缓存没清，编译产物可能是**改动前的旧版本**（曾经真实发生：CSS 源文件已经改了，
  编译出的 `public/assets/css/*.css` 却还是旧规则）。每次验证改动前先
  `rm -rf resources/_gen public .hugo_build.lock` 再重新构建，确认 `grep` 编译产物能看到新内容
- 不要在 `ins.adsbygoogle` 标签本身用 CSS 覆盖 `display`（尤其 `display:flex !important`）——这会跟
  AdSense 自己的自适应尺寸测量逻辑冲突，导致 `push()` 时报错 `No slot size for availableWidthX`、
  广告位彻底不出。占位框的虚线边框/背景色样式要放在外层 `.ad-slot` 包裹层上，`ins` 标签本身除了
  `width:100%` 不要碰其他布局属性（参照 `assets/css/extended/custom.css` 里 `.ad-slot` 的写法，
  以及 `static/tools/watch-valuator/index.html` 里未被覆盖过的写法）
- 不要在 `.ad-slot` 用 flex 布局时，只给外层容器设 `min-height` 而不给 `ins` 标签本身设
  ——flex 默认的 `align-items:center` 不会把子元素撑满高度，`ins` 会塌陷成 0 高度，
  占位框视觉上直接消失（外层容器尺寸对了，但看起来还是空的）
- 不要给 `hugo.toml` 里 `homeInfoParams.Content` 这类会被当 markdown 渲染的字符串写绝对路径链接
  （如 `[text](/posts/slug/)`）——TOML 字符串走的是纯 goldmark 渲染，不会自动加上 baseURL 的
  `/watch-guide-blog/` 子路径前缀，绝对路径线上会 404。要用不带开头斜杠的相对路径（`posts/slug/`）

## Claude 工作方式
- 加新文章：在 `content/posts/` 下新建目录/文件，按现有文章的 front matter 格式
- 涉及 `hugo.toml` 里菜单结构改动时，注意 `[[menu.main]]` 的 `parent`/`weight` 字段决定嵌套和排序
- ads.txt 用的是 Hugo 的 `static/ads.txt` 约定（会被复制到站点根），不是仓库根目录的 `ads.txt`——
  跟其他零构建仓库放法不一样
- 新增分类（`[[menu.main]]` 里加一条指向 `/categories/<name>/` 的菜单项）时，如果这个分类下暂时
  没有任何文章打了对应 tag，Hugo **不会**自动生成该分类的页面——菜单链接会直接 404。必须手动建
  `content/categories/<name>/_index.md`（带 `title`/`description` front matter）撑住这个页面，
  等真正有文章打上这个分类标签后内容会自动汇入
- 覆盖主题 partial 时（如 `layouts/_partials/home_info.html`），文件名要跟
  `themes/PaperMod/layouts/_partials/` 下的原始文件名完全一致，Hugo 才会用项目自己的版本覆盖主题版本
- 验证任何改动前，先跑一遍「清缓存重建 → 本地 `hugo server` 打开页面确认视觉效果 → git push →
  轮询 GitHub Actions run 状态直到 `completed`/`success` → `curl` 线上 URL 确认内容真的更新了」
  这一整套流程，不要只跑到某一步就下结论说"做完了"

## 持续维护
每次你需要重复纠正 Claude 同一件事三次以上，就把结论补进这个文件对应章节。
