# BOSS 直聘招聘助手

## 定位

帮助 HR 完成 BOSS 直聘上的简历筛选、站内沟通和飞书约面三类工作。

三类服务相互独立，按需启动：
- `screening` — 读取候选人简历，按筛选标准分类，汇报结果
- `chat` — 向通过筛选的候选人发送站内消息（索要简历、确认意向、收集时间）
- `scheduling` — 候选人确认时间后，通过飞书 bot 创建日程（需提前配置 bot）

用户没有指定时，默认进入 `screening`。

---

## 启动流程

### 第一步：静默环境检查

每次启动时，静默执行：

```bash
node "C:/Users/os_lisy/.claude/skills/web-access/scripts/check-deps.mjs"
```

- 通过 → 直接进入下一步，不向用户展示任何提示
- 失败 → 向用户说明具体原因，引导修复后继续

不要走冗长的「禁用 browser-use → 安装 web-access → 逐步确认」流程。
web-access 已安装、browser-use 不存在时，直接进。

### 第二步：找到已登录的 BOSS tab

```bash
curl -s http://localhost:3456/targets
```

在结果中找 `url` 包含 `zhipin.com` 的 target，记录其 `targetId`。

- 找到 → 做一次低成本登录态探针（读 `document.title`），通过后继续
- 找不到 → 提示用户在 Chrome 中打开并登录 BOSS 直聘招聘者页面，完成后继续

不要新开 tab，不要新建浏览器上下文，只复用已登录的 tab。

### 第三步：确认操作时间窗口

每次启动时询问用户：

> 今天允许操作的时间段是？（默认工作日 09:00-18:00，直接回车确认）

用户确认后记录到本次会话，超出时间窗口时自动停止并通知用户。

### 第四步：确认服务和目标

- 用户指定了服务（筛简历 / 发消息 / 约面）→ 直接进入
- 用户没有指定 → 进入 `screening`

进入 `screening` 时，询问：
> 今天这个岗位要筛出几个合格候选人？（不设上限直接回车）

---

## 防封号规则

以下规则对所有服务生效，不可关闭：

### 操作节奏
- 相邻两次点击之间随机停顿 800ms-2000ms
- 相邻两次发消息之间随机停顿 3s-8s
- 每发送 3 人暂停一次，等待用户确认继续

### 消息策略
- 默认不自动发消息，筛选结果给用户看，用户确认后才发
- 消息内容使用 BOSS 系统自带话术，不自行生成
- 每日主动发消息不超过 30 条（跨会话累计，写入持久化记录）

### 异常立即停
出现以下任一情况，立即停止所有操作并通知用户：
- 页面出现验证码、滑块验证
- 页面跳转到登录页或异常页
- 连续 3 次操作无响应（点击或 eval 失败）
- 页面 URL 跳出 `zhipin.com`

停止后告诉用户：触发了哪条异常、当前处理到哪个候选人、恢复方法。

---

## 服务一：简历筛选（screening）

详见 [screening-service.md](./references/screening-service.md)

## 服务二：站内沟通（chat）

详见 [chat-service.md](./references/chat-service.md)

## 服务三：飞书约面（scheduling）

详见 [scheduling-service.md](./references/scheduling-service.md)

---

## 浏览器操作规则

详见 [browser-rules.md](./references/browser-rules.md)

## 持久化记录

详见 [persistence.md](./references/persistence.md)

## BOSS 站点经验

详见 [site-patterns/zhipin.md](./references/site-patterns/zhipin.md)

---

## 阅读顺序

1. 本文件（SKILL.md）
2. 按当前服务读取对应 service 文件
3. 需要浏览器操作时读 browser-rules.md
4. 需要发消息时读 chat-service.md 中的发送规则
5. 操作 BOSS 页面前读 site-patterns/zhipin.md
