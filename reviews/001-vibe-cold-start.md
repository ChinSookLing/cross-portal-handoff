# 001 · Vibe cold-start review

Reviewer: Vibe  
Purpose: cold-start comprehension  
Reviewed commit: not named in the review. The text matches the tree at 3f249d9, before clarification commit 192a37a.  
Outcome: comprehension PASS at the release gate

---

我用中文继续。我已经通读了仓库的 README、SPEC.md、SECURITY.md 和 `examples/minimal/` 的三个文件。以下只基于这些文档，不带任何未写明的假设。

## 按文档执行一个回合的流程

如果我是协议中的 Courier,为两个不同门户里的 AI(Contestant)之间传递一手棋,文档规定的流程是:

1. **获取一次完整的 handoff**(§4):一条纯文本消息,包含表头行(`PLAY HANDOFF — one complete message...`)、核心字段(`GAME`、`MOVE`、`EXPECTED_MOVE_NUMBER`、`CONTESTANT_SESSION_ID`、`BOARD` 等)、领域字段,并以 `END HANDOFF · <GAME> · move <N>` 结尾。不得拆分;投递失败必须整条重发。
2. **把 handoff 原样发给当前该走棋的 Contestant**。信任规则:只有在对方被明确邀请之后、在其自己受信任的会话里送达当前 handoff,它才行动(§3、SECURITY.md)。
3. **读取 Contestant 的回复**:只看第一行——先去空白、去掉 `**` 和反引号、大小写不敏感(§5)。第一行是坐标(如 `E5`)、`pass`、`resign` 或 `NO MOVE`;其余全是 commentary,不得解析。若缺 END HANDOFF 行、局面不完整或字段自相矛盾,Contestant 必须回 `NO MOVE`。
4. **Courier 把原始回复一字不改地提交给 Table**(§6):连同 `session_id`(来自 handoff 的 `CONTESTANT_SESSION_ID`)、`expected_move_number` 和 Courier 秘钥。Table 按顺序校验:秘钥 → 座位 → 回合号 → 重复检测 → 首行解析 → 合法性。重复提交只接受一次;无回复或连续三次 `NO MOVE` 则 Table 暂停,绝不自动落子。
5. **Table 记录带溯源的结果**(§8):`raw_response` 原样保存,`source: courier`,人工中转必须记为 `human_bridge`。之后 Table 生成下一位玩家的 handoff,循环往复。

## 阻碍我实施或安全使用的问题

- **没有可获得的服务端或实现代码。** Table 是协议的核心(生成 handoff、校验、记录),但仓库里只有规范文本和记录示例,没有代码、API 规范之外的部署说明、甚至没有说明 Table 软件如何获取或搭建。作为外部读者,我无法根据仓库本身运行一个 Table。
- **Courier 秘钥的分发完全未定义。** 提交授权唯一来自 Courier secret(§3、§12),但它如何生成、如何交给 Courier、如何轮换,文档只说"不得出现在任何公开处"。这保护了安全边界,却也意味着仓库本身不足以让外人开始一局——而 README 明确说"尚未被项目外任何人测试过",这与我读到的内容一致。
- **"明确邀请"机制未规定。** Contestant 只在被明确邀请后才能行动,但邀请的格式、渠道、由谁发出、如何撤销,均无定义。这一步是信任链的起点,却是规范里最大的未指定环节。
- **若干校验细节留白**:`STATE_VERSION` 被要求单调,但验证表里没有对应的检查项;"returned to the same seat with the reason" 没有定义错误消息的格式或返回通道;重复检测只比对"上一次已接受的提交",回放/截断重发的交互没有完全展开;也没有超时时长(只有"可能要几分钟"的定性说明)。
- **第一行解析仍有边角风险**:规范排除了 `Move: G3` 这种带标签的形式,但没有说明如何处理第一行含前后缀、多坐标、或引号包裹的情况;不过这些属于 v0.1 草稿可接受的模糊。

**总体评价**:文档本身很诚实——信任边界清楚("阅读不是行动许可")、失败模式都有对应机制、已知失败被保留而非美化。它是可读、可遵循的草稿,但目前是一个"描述了已被内部验证的协议"的仓库,而不是一个外人拿起就能跑的协议:缺实现、缺秘钥分发、缺邀请机制。这正是 README 自己承认的下一步。
