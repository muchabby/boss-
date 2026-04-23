# leiting-hiring

BOSS 直聘招聘助手，基于 web-access CDP 接管浏览器，支持简历筛选、站内沟通、飞书约面三类服务。

## 依赖

- [web-access](https://github.com/eze-is/web-access) — 必须提前安装
- Node.js 22+
- Chrome / Chromium（开启远程调试）

## 快速开始

1. 安装 web-access skill
2. 在 Chrome 地址栏打开 `chrome://inspect/#remote-debugging`，勾选 Allow remote debugging
3. 打开并登录 BOSS 直聘招聘者页面
4. 对 Claude 说：「帮我筛简历」

## 文件结构

```
leiting-hiring/
  SKILL.md                          # 主入口，启动流程和防封号规则
  references/
    screening-service.md            # 简历筛选服务
    chat-service.md                 # 站内沟通服务
    scheduling-service.md           # 飞书约面服务（待接入）
    browser-rules.md                # 浏览器操作规则
    persistence.md                  # 持久化记录结构
    site-patterns/
      zhipin.md                     # BOSS 直聘 DOM 路径经验（2026-04-23 验证）
```

## 核心设计原则

- **启动轻**：静默探针，环境 OK 直接进
- **防封号**：操作间隔随机化、异常立即停、每日发消息上限 30 条
- **人工确认**：筛选结果给用户看，用户确认后才发消息
- **持久化**：候选人记录跨会话保留，不重复处理
- **站点经验**：BOSS DOM 路径已验证，不重复探索

## 触发词

- 「帮我筛简历」「筛一下投递」→ 进入 screening
- 「帮我联系这些候选人」「发消息」→ 进入 chat
- 「帮我约面」→ 进入 scheduling
