# 持久化记录

## 存储位置

```
C:/Users/{username}/.claude/projects/{project-hash}/hiring/
  candidates.json     — 候选人记录
  daily-stats.json    — 每日发消息计数
  job-policies.json   — 各岗位筛选标准缓存
```

skill 启动时自动创建目录，用户无需手动操作。

---

## candidates.json 结构

```json
[
  {
    "geekId": "692606697",
    "name": "林婧洋",
    "position": "游戏vip客服专员",
    "source": "inbound_chat",
    "screened_at": "2026-04-23T15:41:00+08:00",
    "status": "rejected",
    "reason": "无游戏经历，期望方向不符（人力资源）",
    "resume_summary": "旅游管理本科，门店店长3个月",
    "attachment_status": null,
    "message_sent_at": null,
    "interview_time": null,
    "notes": ""
  }
]
```

### status 枚举

| 值 | 含义 |
|----|------|
| `pending` | 已发现，未读取简历 |
| `resume_read` | 已读取在线简历摘要 |
| `maybe` | 信息不足，待人工复核 |
| `pass` | 通过筛选，待用户确认沟通 |
| `rejected` | 已淘汰 |
| `chatting` | 已进入站内沟通 |
| `attachment_requested` | 已发送求简历消息 |
| `attachment_received` | 已收到附件简历 |
| `interview_scheduled` | 已约面 |
| `thread_open_failed` | 线程打开失败，待重试 |
| `send_failed` | 消息发送失败，待重试 |

---

## daily-stats.json 结构

```json
{
  "2026-04-23": {
    "messages_sent": 5,
    "candidates_screened": 3,
    "last_updated": "2026-04-23T16:00:00+08:00"
  }
}
```

每日发消息上限 30 条，从此文件读取当日计数。
次日自动重置（按本地日期判断）。

---

## job-policies.json 结构

```json
{
  "游戏vip客服专员": {
    "created_at": "2026-04-23",
    "jd_summary": "负责游戏VIP用户的客服工作...",
    "must_have": ["客服相关经验"],
    "preferred": ["游戏行业经验", "VIP服务经验"],
    "hard_reject": ["完全无客服经验"],
    "confirmed_by_user": true
  }
}
```

同岗位筛选标准只需用户确认一次，后续直接复用。
用户可随时说「更新筛选标准」来修改。

---

## 读写规则

- 每次处理完一个候选人后立即写入，不批量延迟写
- 启动时读取，跳过已处理候选人（status 不为 `pending`）
- 候选人有新未读消息时，允许重新处理（不受 status 限制）
- 文件不存在时自动创建空结构，不报错
