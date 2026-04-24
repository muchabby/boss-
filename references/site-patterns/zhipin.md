---
domain: zhipin.com
aliases: [BOSS直聘, boss直聘]
updated: 2026-04-24
verified: true
---

## 平台特征

- SPA，Vue.js 构建，所有操作在同一 URL `/web/chat/index` 完成
- 候选人聊天列表左侧，简历摘要显示右侧（conversation-box）
- 候选人列表使用虚拟滚动，DOM 节点用 `role="listitem"` 标记
- 每个 listitem 内有一个 `[data-id]` 元素，格式为 `{geekId}-0`，对应的 DOM id 为 `_${geekId}-0`

## 有效模式

### 找到指定候选人并点击
```js
// 先取所有 listitem
var items = document.querySelectorAll('[role="listitem"]');
// 找到目标候选人的 data-id
var item = items[N]; // N 为目标索引
var dataId = item.querySelector("[data-id]").getAttribute("data-id"); // 格式: "692606697-0"
```
然后用 clickAt：
```
curl -s -X POST "http://localhost:3456/clickAt?target=ID" --data-raw '#_692606697-0'
```
**注意：直接 `.click()` 无效，必须用 clickAt 配合 `#_${dataId}` CSS 选择器。**

### 读取候选人简历摘要（右侧面板）
点击候选人后等待 1-2 秒，再读：
```js
document.querySelector(".conversation-box").innerText.replace(/\s+/g, " ").trim()
```
返回字段包含：姓名、年龄、工龄、学历、工作经历摘要、教育经历、沟通职位、期望薪资。

**学历注意**：「专升本」字样出现在教育经历文字中，不在学历标签里，必须读完整文本才能识别。

### 求简历按钮（已验证，2026-04-24）
右侧面板底部操作区有「求简历」按钮：
```js
// 找到按钮
var btn = Array.from(document.querySelectorAll('span.operate-btn')).find(el => el.textContent.trim() === '求简历');
btn.click(); // 触发求简历弹窗
```
弹窗出现后，点击确认按钮：
```js
// 确认弹窗的主按钮
var confirmBtn = document.querySelector('.boss-btn-primary');
confirmBtn.click(); // 用 eval 方式，不用 clickAt（clickAt 在此场景首次可能不触发）
```
**发送前必须检查**：聊天记录中是否已有「求简历」消息或候选人已发来附件，避免重复发送。

### 读取所有候选人列表（含岗位过滤）
顶部岗位过滤器位于 `.chat-top-filter`，点击对应岗位名可过滤列表：
```js
document.querySelector(".chat-top-filter").textContent // 包含所有岗位名
```

### 读取未读候选人
```js
var items = document.querySelectorAll('[role="listitem"]');
var unread = Array.from(items).filter(function(el) {
  return el.querySelector("[class*='badge'], [class*='num'], [class*='unread']");
});
```

### 标记不合适
候选人右侧面板底部有"不合适"按钮，点击后会弹出原因选择框。
（路径待验证，2026-04-23）

## 已知陷阱

- `a.resume-btn-online` 点击后会弹出 dialog，但 dialog 内容（`.boss-dialog__body`）在初次渲染时几乎是空的，`innerText` 只返回一行通知文字。简历详情需要通过 conversation-box 的摘要获取，而不是依赖弹窗。
- `[role="listitem"]` 内的 `.click()` 不触发 Vue 事件，必须用 `clickAt` 触发真实鼠标事件。
- 在线简历弹窗（`display: none` 的 `.attachment-resume-top-info-content`）内容不通过 innerText 可达，建议跳过弹窗，直接用 conversation-box 摘要判断。
- 页面标题会从"有新消息啦！"变为"BOSS直聘"，不代表登录态失效。
- 求简历确认弹窗：`clickAt` 在首次点击时可能不触发，必须改用 eval 中的 `btn.click()`。
- 通过 CDP eval 向输入框写入中文会产生乱码，绝对不能用 `innerText =` 或 `value =` 方式发消息。

## 平台特征

- SPA，Vue.js 构建，所有操作在同一 URL `/web/chat/index` 完成
- 候选人聊天列表左侧，简历摘要显示右侧（conversation-box）
- 候选人列表使用虚拟滚动，DOM 节点用 `role="listitem"` 标记
- 每个 listitem 内有一个 `[data-id]` 元素，格式为 `{geekId}-0`，对应的 DOM id 为 `_${geekId}-0`

## 有效模式

### 找到指定候选人并点击
```js
// 先取所有 listitem
var items = document.querySelectorAll('[role="listitem"]');
// 找到目标候选人的 data-id
var item = items[N]; // N 为目标索引
var dataId = item.querySelector("[data-id]").getAttribute("data-id"); // 格式: "692606697-0"
```
然后用 clickAt：
```
curl -s -X POST "http://localhost:3456/clickAt?target=ID" --data-raw '#_692606697-0'
```
**注意：直接 `.click()` 无效，必须用 clickAt 配合 `#_${dataId}` CSS 选择器。**

### 读取候选人简历摘要（右侧面板）
点击候选人后等待 1-2 秒，再读：
```js
document.querySelector(".conversation-box").innerText.replace(/\s+/g, " ").trim()
```
返回字段包含：姓名、年龄、工龄、学历、工作经历摘要、教育经历、沟通职位、期望薪资。

### 读取所有候选人列表（含岗位过滤）
顶部岗位过滤器位于 `.chat-top-filter`，点击对应岗位名可过滤列表：
```js
document.querySelector(".chat-top-filter").textContent // 包含所有岗位名
```

### 读取未读候选人
```js
var items = document.querySelectorAll('[role="listitem"]');
var unread = Array.from(items).filter(function(el) {
  return el.querySelector("[class*='badge'], [class*='num'], [class*='unread']");
});
```

### 标记不合适
候选人右侧面板底部有"不合适"按钮，点击后会弹出原因选择框。
（路径待验证，2026-04-23）

## 已知陷阱

- `a.resume-btn-online` 点击后会弹出 dialog，但 dialog 内容（`.boss-dialog__body`）在初次渲染时几乎是空的，`innerText` 只返回一行通知文字。简历详情需要通过 conversation-box 的摘要获取，而不是依赖弹窗。
- `[role="listitem"]` 内的 `.click()` 不触发 Vue 事件，必须用 `clickAt` 触发真实鼠标事件。
- 在线简历弹窗（`display: none` 的 `.attachment-resume-top-info-content`）内容不通过 innerText 可达，建议跳过弹窗，直接用 conversation-box 摘要判断。
- 页面标题会从"有新消息啦！"变为"BOSS直聘"，不代表登录态失效。
