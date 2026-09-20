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
- **上面这条不止对 `hugo.toml` 成立，`content/*.md` 正文里手写的根相对链接/图片同样会 404**
  （2026-08-13 在 `content/about.md` 里插入图片时用 `read_network_requests` 实测确认）——主题模板
  （导航栏、页脚）用 `relURL`/`absURL` 会正确带上 `/watch-guide-blog/` 前缀，但 Goldmark 渲染
  `content/*.md` 正文时是原样透传 `[text](/images/foo.png)`，同样缺前缀。正文里引用站内资源要么用
  不带开头斜杠的相对路径，要么直接写完整线上 URL（`https://junpingkoch-web.github.io/watch-guide-blog/...`）
  ——写完整 URL 更保险，跟其他 sibling 仓库互相引用时的习惯也一致
- **`**粗体**` 如果结尾紧贴中文全角标点（`）」』】”`等），且标点后面直接接字母/汉字（没有空格
  或另一个标点跟着），Goldmark 会判定这个 `**` 不构成合法的闭合定界符，整段原样显示成字面的
  `**文字**`（不报错，肉眼刷新页面容易漏看，2026-08-26 用一个临时 draft 测试页逐条排除后才定位到——
  同样的粗体写法只要结尾是字母/数字，或标点后面跟空格，就完全正常，只有"标点直接接闭合`**`+
  后面无空格直接接字母/汉字"这一种组合会触发）——这是 CommonMark 的 flanking-delimiter 规则本身
  的行为（右闭合定界符前面是标点、后面又不是空白或标点时不算 right-flanking），不是这个主题或
  这份配置的 bug，换其他 Goldmark/CommonMark 渲染器大概率一样。**写法上怎么避免**：全角括号一类
  标点收尾的短语要加粗时，要么在闭合 `**` 后面手动留一个空格，要么把结尾标点挪到 `**` 外面
  （例如 `**异步沟通（Asynchronous Updates）**替代` 会炸，`**异步沟通（Asynchronous Updates）** 替代`
  或 `**异步沟通（Asynchronous Updates）替代**` 都不会）。以后写正文里的粗体，只要闭合 `**`
  前一个字符是全角标点，就要留意这条，构建后最好搜一下 `<strong>` 附近有没有漏网的字面 `**`

- **`absLangURL`（或 `absURL`）套在一个字面字符串上，只是拼 URL，不代表那个页面真的存在**
  （2026-09-02 用独立工具 `npx broken-link-checker` 实际爬线上站点才发现，`check_dead_links.py`
  自己会跳过这个仓库，测不到）：`extend_footer.html` 原来写的是
  `{{ "impressum/" | absLangURL }}`，在英文页面上无条件拼出 `/en/impressum/`、`/en/contact/`，
  但 `content/impressum.md`、`content/contact.md` 从来没建过 `.en.md` 翻译，这两个英文页面
  压根不存在，页脚链接在所有英文页上全部 404。**正确写法是先用 `.Site.GetPage "impressum"`
  判断当前语言下这个页面是否真的存在，不存在就 fallback 到默认语言版本**：
  `{{ with .Site.GetPage "impressum" }}{{ .Permalink }}{{ else }}{{ "impressum/" | absURL }}{{ end }}`
  ——因为 `defaultContentLanguageInSubdir = false`，中文版没有 `/en/` 前缀，`absURL`（不加语言
  前缀）天然就会落在中文页面上，不需要额外判断语言代码。以后任何"页脚/导航链接到一个可能没有
  翻译的页面"场景，都用这个 `with...else` 模式，不要直接对字面字符串套 `absLangURL`。
  **修的时候也要小心不要犯同一类错误的变种**：把正文里裸的 `/contact/` 改成完整 URL 时，第一次
  改成了 `.../en/contact/`——这个页面同样不存在，等于把 404 换了个新地址，本质没修。改任何
  "链接指向哪"的问题时，替换后的目标 URL 本身也要实际验证过（`curl` 一下状态码），不能只看
  "这个 URL 格式上更对"就直接信。

- **广告位和 `adsbygoogle.js` 现在只在"文章页"渲染，这是有意为之，不是漏了**（2026-09-20，AdSense 因
  "Minderwertige Inhalte" 被拒两次后的整改）：`header.html`（广告位）和 `extend_head.html`（loader 脚本）
  都用 `and (eq .Kind "page") (eq .Section "posts") (not .Params.noAds)` 把关。首页、列表页、标签/分类页、
  about/contact/impressum/privacy/tools 页都没有广告。单篇文章在 front matter 加 `noAds: true`（YAML）或
  `noAds = true`（TOML，这里两种格式都有）即可单独去广告——目前 14 篇正文不足约 1000 汉字的薄文章用了它。
  **审核通过后要恢复广告：删掉这些 `noAds` 行即可；不要把 header/extend_head 里的条件判断拆掉，
  非文章页本来就不该挂广告。**
- **标签/分类页（`/tags/`、`/categories/`）是 noindex 且不进 sitemap**：靠 `content/tags/_index{,.en}.md`、
  `content/categories/_index{,.en}.md` 里的 `robotsNoIndex: true` + `sitemap.disable: true`（顶层和
  `cascade` 各写一份，缺 `.en` 版本英文页不生效）。新增分类菜单项时，这几个文件不要删。sitemap 里只应该出现
  文章页和几个真实页面（验证：`hugo -d <临时目录>` 后数 `sitemap.xml` 里 `/tags/`、`/categories/` 应为 0）。
  注意根仓库 `robots.txt` 里对这两个路径的 `Disallow` **挡不住 AdSense 爬虫**（它无视 `User-agent: *`），
  真正起作用的是这里的 noindex + 不挂广告。

- **不要编造"第一手内容"**：文章里的成交价、退税到手比例、客户/面交案例、照片、从业年限，只能来自作者
  本人（用户）亲口提供的素材，不能凭"听起来合理"补写——这个站正因"缺可核查的第一手内容"被 AdSense 判为
  "Minderwertige Inhalte"，编出来的细节既不诚实，也恰好是审核员最容易识破的一类。用户说"帮我补进我的真实
  经历"但没给素材时，先生成一份**素材清单**（每篇 3–6 个具体问题，最低标准：≥3 条真实事实 + 1 张自己拍的
  照片，客户匿名，不涉雇主机密；样例见 2026-09-20 的 `Desktop\Blog-First-Hand-Material-Request.md`），
  等用户回答后再写，措辞保持用户口吻、数字原样写"约/大概"。同时顺手核对文章里已有的硬性事实
  （比如序列号位置、退税门槛、税率），发现和自己已知不符的不要悄悄改，列出来让用户确认；发现文章之间
  数字互相矛盾（如附件差价 5–15% vs 10–20%）也同样列出，别自行挑一个。

- **改文章里的事实时，中英文两份必须成对改**：有英文版的文章是 `index.md` + `index.en.md` 两个文件
  （2026-09 起至少 `rolex-serial-number-authentication-guide`、`swiss-watch-customs-and-tax-guide`、
  `europe-five-country-…`、`secondhand-watch-valuation-…` 有），只改一份会让两个语种互相矛盾。批量改用脚本时
  每个替换都断言"恰好匹配一次"（Python 里写个 `sub1()`），匹配 0 次或多次就报错，别用会悄悄漏改的
  `replace`；文件可能是 CRLF，读写时保留原换行符，标题/引号用 `[“"]` 兼容中英文引号。
- **核实/补充硬事实的来源优先级**：官方机构原文（瑞士联邦税务局 estv.admin.ch、联邦海关 bazg.admin.ch、
  机场官网）> 行业/商家页面 > 旅游宣传页/博客。`WebSearch` 可用 `allowed_domains: ["admin.ch", "bazg.admin.ch"]`
  限定官方域名。**来源之间冲突时不要自己挑一个**：以法定/主管机关为准，并把差异写进文章（例如退税出境期限：
  联邦税务局 90 天 vs 瑞士旅游局 30 天）。官方页面没提到的说法（如退税公司手续费比例）要在文中明说
  "官方页面未说明，属作者经验"，不要包装成规则。引用的来源在文中用链接标出并写明查询日期。
  `curl` 访问部分官方/旅游站会返回 406（拦机器人），不代表链接失效，用 `WebFetch` 或浏览器再确认。
- **改了已发布文章的事实内容，要在文末加可见的"校订说明"并设 `lastmod`**（front matter 里
  `lastmod: 2026-09-20T…`），写清改了什么、依据哪些来源——既诚实，也是审核员能看到的可信度信号。

## Claude 工作方式
- 加新文章：在 `content/posts/` 下新建目录/文件，按现有文章的 front matter 格式
- **加新文章前先检查是否和已有文章主题重叠**（2026-08-26 真实发生：用户粘贴的新草稿
  `swiss-alps-day-trip-route-guide` 和已发布的 `alpine-day-trip-planning-brienz-giessbach`
  几乎是同一条布里恩茨湖/吉斯巴赫瀑布路线，只是多了图恩这一段——如果直接照发会变成两篇互相
  抢同一批关键词的近似重复内容）。发现重叠时不要自己决定合并/跳过/硬发，抛给用户选（这次
  用户选了"差异化重写"）。差异化重写的具体做法：新文章开头直接点名并链接旧文章、明确说"这篇
  不重复讲 XX，只讲新增的 YY"，把内容重心完全放在真正增量的部分；同时回头在旧文章末尾加一句
  指向新文章的链接，让两篇形成双向互链而不是各自孤立
- 对文章做 SEO/GEO 审查时，先读 `.claude/rules/seo-geo-review.md`——里面有 CJK 标题/描述的
  正确量长方法（不能直接用字符数）、`schema_json.html` 的字段回退逻辑、以及两个还没修的
  长期待办（博客文章没有 FAQ 结构化内容、标签因为共享回链工具而topic稀释），不用每次重新摸索
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
