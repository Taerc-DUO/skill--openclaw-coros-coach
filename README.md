# COROS 跑步教练 Skill for OpenClaw

让 OpenClaw 成为你的 COROS 手表智能跑步教练。基于 **Jack Daniels 训练体系**，读取你的 COROS 数据，生成个性化训练计划，并直接 API 推送到手表。

## ✨ 功能

- 📊 **读取训练数据**：跑步能力、训练负荷、体力恢复、心率/配速区间
- 🏃 **训练状态判断**：根据负荷比和恢复情况给出当天训练建议
- 📝 **Jack Daniels 配速体系**：基于乳酸阈配速自动计算 E/M/T/I/R 各区间
- 📱 **API 创建课程**：精确控制配速，直接推送到 COROS Training Hub
- 📅 **日历安排**：创建后拖拽到日历，手表同步即可

## 🔧 安装

### 前置要求

- [OpenClaw](https://openclaw.ai) 已安装
- COROS 高驰手表 + COROS 账号
- COROS Training Hub 使用权限

### 安装步骤

1. 将 `SKILL.md` 复制到 OpenClaw skill 目录：

```bash
# 默认 skill 目录
cp SKILL.md ~/.openclaw/skills/coros-coach/SKILL.md
```

2. 在 OpenClaw 中激活 skill（说"我要使用 COROS 跑步教练"或类似触发词）

3. **首次 Setup**（必须）：按照下方步骤完成浏览器登录

## 🔑 首次 Setup：使用浏览器登录态

### 使用 `openclaw` profile 登录

1. 在 OpenClaw 浏览器中使用 `openclaw` profile 打开：`https://coros.com/traininghub`
2. 如果提示未登录，手动登录你的 COROS 账号
3. 登录后，浏览器会保存账号密码、cookie 和会话，后续通常不用重复登录
4. AI 直接复用这个浏览器 profile 的登录态读取数据和创建课程，不需要你在对话里发送账号密码、cookie、token 或 csrf 信息

### 提供基本信息

告诉 AI 你的：

- **跑步能力**：COROS 仪表板的综合能力分和乳酸阈配速
- **比赛目标**（可选）：如半马目标、全马目标等
- **偏好**：距离+配速为主，或心率为主

## 🚀 快速开始

配置完成后，直接对 AI 说：

- "我今天能跑吗？"
- "帮我出个明天的跑步课表"
- "这周训练状态怎么样？"
- "我要跑间歇，帮我安排"
- "复盘一下我上周的跑步数据"

## 🧭 工作方式

- 普通状态查询或训练复盘：AI 只读取 COROS 数据并给建议，不会自动创建课程
- 明确说出“创建课程”“推到课程库”“直接给我建一节课”时：AI 才会调用 Training Hub 接口创建课程
- 创建课程时，AI 会优先复用 `openclaw` 浏览器 profile 的当前登录态，不要求你在聊天里提供 token 或 cookie
- 创建完成后，AI 应继续校验课程是否真的写入成功、配速是否符合预期
- 当前只为轻松跑/有氧跑（E）、乳酸阈跑（T）、间歇跑（I）、长距离慢跑（LSD）定义了保守默认生成规则
- 如果用户没有说清总距离、总时长或间歇结构，AI 应先给课程草案，不直接创建
- 间歇课默认使用中性跑步模板，并把主组段名写成 `快跑`、恢复段名写成 `慢跑`，避免客户端出现动作训练名称或笼统的“训练”
- 创建或建议课程时，AI 不只给配速和距离，还应补充执行时的心率参考。快跑段以配速为主，恢复跑和 LSD 更要看心率是否落在合理范围

## 📚 Jack Daniels 配速参考

| 类型 | 名称 | 作用 | 典型配速（VDOT≈50）|
|---|---|---|---|
| **E** | Easy（轻松跑） | 基础有氧恢复 | 5'27"-5'47"/km |
| **M** | Marathon（马拉松配速） | 比赛专项 | 4'23"/km |
| **T** | Threshold（乳酸阈） | 提升乳酸清除 | 4'10"/km |
| **I** | Interval（间歇） | 提升 VO2max | 3'46"/km |
| **R** | Repetition（重复跑） | 速度和神经肌肉 | 3'32"/km |

> 实际配速会根据你的乳酸阈配速自动计算，上表为 VDOT≈50（半马 1:35 水平）的参考值。

## ⚠️ 常见问题

**Q: API 报 "Access token is invalid"**
A: 登录态过期了。请在同一个 `openclaw` 浏览器 profile 中重新登录 COROS Training Hub，再重试。

**Q: 课表配速和预期不符**
A: 创建课程后用 `program/query` API 验证参数。精确控速必须用 `intensityCustom=0`，否则服务器会使用模板默认值。

**Q: 手表收不到课程**
A: 在 COROS App → 训练课程 中找到课程，点击同步图标发送到手表。

## 🤝 贡献

欢迎提交 Issue 和 PR！

- 问题反馈：https://github.com/Taerc-DUO/skill--openclaw-coros-coach/issues
- 功能建议：https://github.com/Taerc-DUO/skill--openclaw-coros-coach/discussions

## 📄 许可

MIT License

## 🙏 致谢

- 训练体系：**Jack Daniels' Running Formula**
- 技术平台：**OpenClaw**
- 数据支持：**COROS 高驰**
