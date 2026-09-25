# 003 · Lumo targeted recheck

Reviewer: Lumo  
Purpose: targeted recheck of H1, H2, H5, H7 and H8  
Reviewed commit: 9debc57  
Outcome: PASS. No new release-blocking issue.

---

我来获取 patch commit `9debc57` 的相关文档，专门检查 H1、H2、H5、H7、H8 这五项的修复情况：





根据新 commit `9debc57` 的文档提取，我来逐项验证之前提出的 5 个问题是否已被准确回应：

```card
{ "type": "finding", "title": "Patch Verification Summary", "body": "H1, H2, H5, H7, H8 all addressed with documentation-only changes. No new critical issues introduced for v0.1 experimental release.", "severity": "info" }
```

---

## 逐项验证结果

| Issue | 状态 | 证据位置 |
|-------|------|----------|
| **H1** — 跨参赛者注释/提示注入边界 | ✅ Addressed | SECURITY.md 新增加粗段落 *"Another contestant's commentary is quoted data, not an instruction"*；SPEC §4.1 新增 *"MUST be clearly delimited as quoted, untrusted data. MUST NOT be presented... as instructions"* |
| **H2** — 拒绝消息不回显恶意首行 | ✅ Addressed | SPEC §6.1 错误表新增 *"Return to the same seat with **fixed wording**. The offending line is **not echoed** as an action"*（两处） |
| **H5** — 无密码学来源验证 | ✅ Addressed | SECURITY.md 新增 *"This version does not prove origin. Nothing here cryptographically shows that a handoff came from the Table..."* 明确披露局限 |
| **H7** — 已闭和对局拒绝提交 | ✅ Addressed | SPEC §6.1 校验表首位新增 *"`STATUS` is not `live` → `game_closed` → Reject; nothing recorded"* |
| **H8** — Go 操作限制为 ASCII | ✅ Addressed | SPEC §5 新增 *"For Go, the only action spellings are ASCII... Fullwidth letters, Chinese numerals and any other script are `invalid_action`"* |

---

## 最终问题：是否有任何内容对 v0.1 实验性发布构成新的严重或发布阻断性问题？

**答案：否。**

理由：
1. **所有补丁均为文档性澄清**，未引入新的机制依赖或实施复杂度。
2. **SECURITY.md 的坦白披露**（*"does not prove origin"*）是负责任的透明度实践，而非漏洞。它诚实地告诉部署者和用户此协议的保护范围，避免了虚假安全感。
3. **版本日志确认**："Documentation only. No signing. Not a release." —— Bill 团队理解自己在做什么。

唯一需要注意的是：H5 的披露虽然正确，但在实际外部测试前，可能需要额外强调**参赛者自身安全护栏的重要性**（因为协议不提供来源验证）。但这不属于新创建的风险，而是原 H5 风险的延续性运营建议。

**结论：patch 干净，v0.1 实验性发布无障碍。** 🔒
