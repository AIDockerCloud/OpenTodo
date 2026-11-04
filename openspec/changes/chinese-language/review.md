# 审校清单（提交 PR 之前，由母语维护者执行）

这个变更里的 7 个面、约 7,400 条中文全部是 AI 初译（Web 词条 7109 条，其余六面 326 条）。`localization` 规格写明
「已发布词条须由人撰写与审阅，不得机器翻译」，所以**下面每一步都要真人走完**，
缺一步就不能提 PR。整套流程的产出物是：一份你签字认可的 catalog + 一条通过的 CI。

---

## 0. 先把工具链装上（一次就够）

> 本分支已经变基到 `upstream/main`（`ed97c6cdd`）。上游那 14 个提交往词条表加了 6 条
> （评论 webhook 三个事件、单键快捷键开关、Android 麦克风提示），已补译。
> 后来上游又新增了 22 条 OAuth 相关词条并删除 1 条，条数从 7082 变成 7109。

```bash
cd ~/ai/github.com/AIDockerCloud/OpenTodo
npm ci --prefix web          # 约 700 MB–1 GB，走热点注意流量
go mod download              # Go 侧编译与测试
```

装完后确认机器上能跑：

```bash
make i18n-check              # 应输出 ok (7109 messages, every shipped catalog complete)
```

> 我这边只装了 `web/node_modules/typescript`（4.4 MB），够跑 i18n 检查，但 `make check`
> 里的 vitest / storybook / tsc 全都没跑过。第 5 节是补这一课。

## 1. 生成审校稿（5 分钟内完成）

```bash
cd ~/ai/github.com/AIDockerCloud/OpenTodo
mkdir -p ~/tmp/zh/review
python3 - <<'PY'
import json, os
en = list(json.load(open('web/src/locales/messages.json', encoding='utf-8')))
zh = json.load(open('web/src/locales/zh.json', encoding='utf-8'))
out = os.path.expanduser('~/tmp/zh/review')
CHUNK = 400
for i in range(0, len(en), CHUNK):
    with open(f'{out}/web-{i//CHUNK+1:02d}.tsv', 'w', encoding='utf-8') as f:
        for k in en[i:i+CHUNK]:
            f.write(f'{k}\t{zh[k]}\n')
print('已生成', (len(en)+CHUNK-1)//CHUNK, '个文件，每个 400 条')
PY
```

`~/tmp/zh/review/` 下每个文件 400 条，左边英文、右边中文，用表格软件或编辑器并排看。

## 2. 逐条审（这是唯一不能用脚本替代的部分）

判据只有四条，逐条过，不合就改：

1. **读得懂**：当作陌生界面上的一句话，能不能一眼明白它在说什么动作或状态。
2. **像中文**：没有英式语序（"你有 1 个提醒等待"→"有 1 个提醒"）、没有直译腔（"可用的"堆叠）、
   动词开头、不堆"进行/的/了"。
3. **术语对**：与 `docs/translating.md` 的术语表一致。抽查这 12 个高频词——
   任务 / 项目 / 分区 / 标签 / 收集箱 / 今天 / 回收站 / 截止日期 / 最后期限 / 提醒 / 设置 / 例行任务。
4. **不越界**：`{name}`、`{n}`、`<a>` 这些没被翻掉（校验器会挡，但你顺手看一眼更快）。

**优先审这几类**，性价比最高（用户天天看，翻错最刺眼）：

| 优先级 | 范围 | 条数 | 怎么找 |
|---|---|---|---|
| P0 | 按钮与菜单（短句） | 约 1,200 | 按长度排序，≤ 3 个英文单词的 |
| P0 | 导航与视图名 | 29 | 搜 `Today` `Upcoming` `Anytime` `Someday` `Inbox` `Trash` |
| P1 | 设置项标题与说明 | 871 | `lib/settingsCopy.ts` 那一段（批 01） |
| P1 | 错误提示 | 154 | `lib/i18n/errors.ts` 那一段（批 04 后半） |
| P2 | 快捷键速查 | 308 | 批 02，长句多，允许保留英文按键名 |
| P2 | 其余 | 约 5,000 | 按批顺序过 |

## 3. 改法

**直接改 `web/src/locales/zh.json`**（不要回头改 `~/tmp/zh/batches/*.tsv`，那是我的中间产物）。
改完必跑：

```bash
make i18n-check              # 占位符、标签、复数、Settings 路径都会被校验
```

改一条就验一条太慢，建议**攒 20–30 条改一次再跑**，报错会直接指名是哪条。

## 4. 其余六个面

| 面 | 文件 | 条数 | 注意 |
|---|---|---|---|
| 浏览器扩展 | `extension/_locales/zh/messages.json` | 57 | `extName` ≤75、`extDescription` ≤132 字符 |
| Android | `mobile/android/app/src/main/res/values-zh/strings.xml` | 49 | `%s`/`%d` 不能动 |
| iOS | `mobile/ios/Shared/Localizable.xcstrings` | 30 | `%@`/`%lld` 不能动 |
| 邮件 | `internal/mail/templates/*.zh.txt` | 12 封 | 纯文本靠换行排版，注意段落别挤 |
| 通知渠道 | `internal/channels/messages.go` 的 `"zh"` 组 | 22 | `{name}` 保留 |
| Obsidian 插件 | `obsidian/locales/zh.json` | 150 | 有 29 条复用了主应用译文，改的时候两边一起改 |

```bash
node --test extension/locales.test.js          # 17 项，应全绿
node scripts/i18n-check.mjs                    # 含 shell + obsidian 检查
```

## 5. 机器检查结果（本机已实测，2026-11-04）

工具链：`web/node_modules` 444 MB（`npm install`，**不要用 `npm ci`**，它会先清空 node_modules）、
Go 1.27.3 解包在 `~/go-sdk/go`（无需 sudo）、`GOPROXY=https://goproxy.cn,direct`。

| 检查 | 结果 |
|---|---|
| `node scripts/i18n-check.mjs` | ok（7109 条，含 shell、obsidian） |
| `tsc --noEmit` + `tsc --noEmit -p tsconfig.sw.json` | 通过 |
| `eslint --max-warnings 0 src` | 通过 |
| `node scripts/ui-check.mjs` | ok |
| `node --test extension/locales.test.js` | 17 项全绿 |
| `go test ./internal/mail/... ./internal/channels/... ./internal/quickadd/...` | 全过 |
| `make fmt-check` / `templates-check` / `docs-check` | 通过 |
| `openspec validate --all --strict` | 145 项全过 |
| `make bundle-budget` | ok：入口 141.2 KB / 预算 142；首屏 678.5 KB / 预算 679 |
| `vitest` | 7611 通过、1 个既有 flaky、7 skipped（见下） |

**中文包体积**：`zh-*.js` 515 KB 原始 / **178 KB gzip**，独立懒加载 chunk
（德语 574 KB / 178 KB，法语 588 KB / 178 KB —— 中文下载量与德语持平）。

`internal/mail/testdata/digest.zh.golden` **不用重新生成**：`TestDigestGolden` 已通过，
说明它与 Go 的真实渲染逐字节一致。

### 两个会让你误判的环境问题

1. **直接在中文 locale 下跑 `npm test` 会看到约 70 个失败**，全部是 `waitFor ... 10000 ms` 超时和
   日期断言失败。根因是 `web/public/push-sw.js` 里的 `const locale = lang === 'en' ? undefined : lang`：
   英文界面用**系统默认 locale** 格式化日期，你这台机器是 zh_CN，于是得到
   `Due 11月 3, 3:00 下午`。这不是汉化引入的，main 上同样如此，值得单独给上游提 issue。
   ```bash
   cd /home/inverstar/ai/github.com/AIDockerCloud/OpenTodo/web
   LANG=en_US.UTF-8 LC_ALL=en_US.UTF-8 npm test
   ```
2. `src/app/todayAgenda.test.tsx > says how old the cached events are while offline` 在 671 个
   worker 全并发时会偶发超时，单跑 10/10 通过。既有 flaky。

## 6. 视觉验收

```bash
cd web && npm run dev
```

浏览器切到简体中文，按这四个视口各看一遍：**375 px / 1280 px × 浅色 / 深色**。只看三件事：

- 换行后的中文行高是否挤（中文比西文需要更多行距）；
- 侧边栏、任务行、按钮里的文字是否被截断；
- 数字与中文混排（`截止 2026-11-06`）是否别扭。

## 7. 提 PR

```bash
git checkout -b feat/simplified-chinese
git push -u origin feat/simplified-chinese
```

PR 描述里必须写清楚三件事，否则维护者会打回：

1. **这是 AI 初稿 + 母语者逐条审校**：说明审校覆盖了多少条、改了多少处；
2. **维护者署名**：把你的名字加进 `docs/translating.md` 的维护者表（现在还没有这张表，
   如果 `more-languages` 变更先合了，按它的格式加；否则在 "Adding a language" 第 5 步那句话旁注明）；
3. **明确不在本次范围**：中文 quick add（需要分词器）、繁体、文档与内置帮助保持英文。

---

## 已知遗留（写进 PR 的 "Not included" 一节）

- **中文 quick add 不支持**：`tokenize()` 按空格切词，中文整句是一个 token。需要分词器，
  属独立变更。`docs/language.md` 已如实写明。
- **繁体中文**：`resolveLanguage` 只取主语言标签，`zh-TW` 会落到 `zh`（简体）。要支持繁体需先改 locale 解析。
- **中文字体**：不打包 CJK 子集（几 MB，会破首屏预算），中文用系统字体渲染。
- **上游的 locale 依赖**：`push-sw.js` 在英文界面用系统默认 locale 格式化日期，测试因此依赖宿主
  locale（见第 5 节）。本变更没有动它，但值得单独提一个 issue。
