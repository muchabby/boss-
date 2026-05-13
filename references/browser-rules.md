# 浏览器操作规则

## 唯一路径

所有 BOSS 页面操作统一通过 `web-access` CDP Proxy 执行。

Proxy 地址：`http://localhost:3456`

不要使用 `browser-use` 或其他通用浏览器 skill 接管 BOSS 页面。

---

## 点击优先级

所有点击操作按以下顺序尝试，成功后立即停止：

1. `POST /clickAt?target=ID` — 模拟真实鼠标事件，首选
2. `POST /click?target=ID` — JS click，次选
3. eval 中 `element.click()` — 一次性兜底，不作默认路径

批量发消息时禁止用 `element.click()` 作为主路径。

---

## eval 使用规则

复杂脚本前必须先做最小探针：

```bash
# 探针示例
curl -s -X POST "http://localhost:3456/eval?target=ID" --data-raw 'document.title'
```

探针通过后再提交复杂逻辑。探针失败时优先排查：请求体编码、Content-Type、转义方式。

---

## 静默环境检查

进入真实业务前，优先静默检查：

1. `web-access` 依赖可用
2. `GET /targets` 可访问
3. 能在现有 target 上执行最小 eval 探针
4. 通过 `document.title` 做低成本登录态探针

只有检查失败时再提示用户修复。

---

## tab 复用规则

- 只复用用户已登录的 BOSS tab，记录其 `targetId`
- 不新开 tab，不新建浏览器上下文
- `targetId` 变化时，自动重新绑定（重新调 `/targets` 找 zhipin.com 的 tab）
- 只有找不到任何 BOSS tab 时，才提示用户重新打开

---

## 截图规则

截图默认禁止。

只有同时满足以下条件才允许：
1. DOM / CDP 主路径已尝试失败
2. 同一路径内最多又补试 2 次仍失败
3. 向用户说明了截图的额外风险（token 消耗、风控风险）
4. 用户明确同意

---

## 子代理限制

以下操作必须由主执行流完成，不起子代理：
- 找已登录 tab
- 读取候选人列表
- 点击候选人
- 发送消息

---

## 登录态探针

低成本登录态检查：

```bash
curl -s -X POST "http://localhost:3456/eval?target=ID" --data-raw 'document.title'
```

返回「BOSS直聘」或「有新消息啦！」→ 登录态正常。
返回登录页相关内容 → 提示用户重新登录。

`targetId` 失效或变化时，先重新调用 `/targets` 找新的 Boss target，再做上述探针。
