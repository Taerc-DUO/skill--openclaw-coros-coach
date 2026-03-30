---
name: coros-coach
description: 查询 COROS 高驰跑步数据和训练状态，提供个性化训练指导，并在高驰 Training Hub 上创建结构化训练课程。仅在用户明确提到“高驰”“COROS”“coros”“Training Hub”，或已经提供高驰训练数据、希望基于高驰实际数据做训练评估、课表安排、课程创建、周训练复盘时使用。不要因为泛泛提到“训练计划”“间歇”“节奏跑”“LSD”“轻松跑”“今天能跑吗”就触发；这类普通跑步问题若没有 COROS/高驰上下文，不应使用本 skill。
---

# COROS 跑步教练

基于高驰 Training Hub 数据，为跑者提供个性化的跑步训练指导，并可直接在 Training Hub 上创建结构化训练课程推送到手表。

## 首次 Setup（必读）

### 1. 使用 openclaw 浏览器 profile 登录 COROS

本 skill 依赖 OpenClaw 自带的 `openclaw` 浏览器 profile 中的登录态读取数据和创建课程。首次使用时：

1. 在 OpenClaw 浏览器中使用 `openclaw` profile 打开：`https://coros.com/traininghub`
2. 如果提示未登录，由用户手动完成 COROS 登录
3. 登录成功后，账号密码、cookie 和会话会保存在该浏览器 profile 中，后续通常无需重复登录
4. 后续读取数据和创建课程时，直接复用这个 profile 的登录态，不要求用户在聊天中粘贴账号密码、cookie、access token 或 csrf token

### 2. 提供你的基本信息

告诉 AI 以下内容（一次性）：

- **跑步能力值**：COROS 仪表板的综合能力分（如 85）和乳酸阈配速
- **比赛目标**（可选）：如半马目标 1:35、全马 3:30 等
- **偏好**：距离+配速为主，或心率为主

---

## 数据获取

使用浏览器工具（profile: openclaw）访问高驰 Training Hub：

| 页面 | URL |
|---|---|
| 仪表板（数据总览） | `https://trainingcn.coros.com/admin/views/dash-board` |
| 日程（含课程库） | `https://trainingcn.coros.com/admin/views/schedule` |
| 活动列表 | `https://trainingcn.coros.com/admin/views/activity-list` |

### 从仪表板读取的关键数据

**跑步能力**
- 综合能力值 + 四项子分（有氧耐力/乳酸阈/速度耐力/冲刺）
- 各项对应的个人配速区间

**训练负荷**
- 短期负荷 / 长期负荷 / 负荷比
- 训练量评估（训练不足/正常/偏高/过高）

**体力恢复**
- 恢复百分比 + 状态描述

**心率与配速区间**
- 最大心率 / 静息心率 / 乳酸阈心率
- 乳酸阈配速
- 六个心率区间 BPM 范围
- 六个配速区间范围

**最近运动记录**
- 日期、距离、配速、训练负荷（TL）

**成绩预测 & 个人纪录**

### 登录态使用边界

- 如果页面能正常打开且显示你的个人训练数据，默认视为登录态可用
- 如果页面跳回登录页或接口返回未授权，提示用户在同一个 `openclaw` profile 中重新登录
- 不要要求用户把账号密码、cookie、token 或浏览器控制台输出贴到聊天里
- 如果确实需要调试请求参数，优先在已登录页面上下文中读取，避免把敏感信息暴露到对话中

---

## Jack Daniels 训练配速体系

基于 COROS 测得的乳酸阈配速（T pace）自动计算各区间配速：

| 类型 | 名称 | 作用 | 配速区间 |
|---|---|---|---|
| **E** | Easy（轻松跑） | 基础有氧恢复 | T配速 × 1.30-1.40（慢） |
| **M** | Marathon（马拉松配速） | 比赛专项 | T配速 × 1.05 |
| **T** | Threshold（乳酸阈） | 提升乳酸清除 | **你的乳酸阈配速** |
| **I** | Interval（间歇） | 提升 VO2max | T配速 × 0.90 |
| **R** | Repetition（重复跑） | 速度和神经肌肉 | T配速 × 0.85 |

> 配速换算示例（基于乳酸阈配速 4'10"/km = 250秒/km）：
> - E跑：~5'27"-5'47"（×1.30-1.40）
> - T跑：4'10"（乳酸阈配速）
> - I跑：3'46"（×0.90）

---

## 训练状态判断

根据负荷比和恢复数据给出训练建议：

| 负荷比（短期/长期） | 恢复状态 | 建议 |
|---|---|---|
| < 0.5 | 任意 | ⚠️ 训练不足，建议恢复跑 |
| 0.5 - 0.8 | > 80% | ✅ 可上强度（间歇/节奏跑） |
| 0.5 - 0.8 | < 60% | 🟡 中等强度（有氧跑） |
| 0.8 - 1.2 | > 70% | 🟡 正常训练，避免高强度 |
| 0.8 - 1.2 | < 50% | 🔴 休息或极轻活动 |
| > 1.2 | 任意 | 🔴 过度训练风险，必须休息 |

---

## 执行反馈原则

- 距离、时长、配速是课程的外部目标；心率用于判断当天执行时身体反馈是否正常
- 创建完课程后，不要只告诉用户“跑多远、跑多快”，还要补一句“执行时心率参考”
- 如果仪表板里已经有六个心率区间，优先直接引用用户自己的区间名称和 BPM 范围
- 如果没有完整心率区间，但有乳酸阈心率，则以乳酸阈心率为锚点给保守参考
- 对间歇快段，不要让用户实时追心率。快段以配速为主，心率用于判断困难程度是否失控，因为心率有滞后
- 对恢复跑、慢跑、LSD，心率反馈比瞬时配速更重要。如果配速达标但心率长期偏高，应允许用户主动降速

### 按训练类型给出的心率参考

- 轻松跑 / 恢复跑 / LSD 主段
  - 优先引用用户自己的低强度心率区间
  - 如果只能基于乳酸阈心率估算，保守参考为 `乳酸阈心率 - 15 到 25 bpm` 左右
  - 如果当天心率明显高于这个范围，即使配速达标，也应提示用户放慢
- 乳酸阈跑（T）
  - 优先引用用户自己的阈值或次阈值心率区间
  - 如果只能基于乳酸阈心率估算，保守参考为 `乳酸阈心率 ± 5 bpm`
  - 如果心率明显压不住且持续上飘，应建议缩短主段或改轻一点
- 间歇跑（I）快跑段
  - 快段仍以 I 配速为主，不要把心率写成唯一执行目标
  - 可以补充“后半组心率通常会上到阈值以上”，用于解释主观难度
  - 如果前几组就远超平时高强度区间，或恢复段心率迟迟降不下来，应建议减组、延长恢复或停止
- 间歇跑（I）慢跑段
  - 恢复段应重点关注心率是否开始回落
  - 如果恢复段结束前心率仍明显高于平时有氧/恢复区间，下一组不应机械开跑

---

## 训练类型参考

**日常训练组合（每周 3-4 次）：**

- **轻松跑/有氧跑**（每周 2-3 次）：5-10km，E 区间配速
- **乳酸阈跑**（每周 1 次）：6-8km，T 区间配速
- **长距离慢跑**（每周或隔周 1 次）：8-15km，E 区间偏慢侧，逐步增距
- **间歇跑**（状态好时）：800m × 4-6 组或 400m × 6-8 组，I 区间配速

> ⚠️ 业余跑者增距要循序渐进，每周增加不超过 10-15%。

---

## 课程生成规则（v1，证据优先）

以下规则只做两件事：

- 把本 skill 里已经验证过的 API 事实写死
- 在用户没说清楚时，提供保守默认，不冒充 COROS 官方唯一标准

### 规则来源与边界

- **已验证，可写死**
  - 已登录页面上下文内发起 `fetch`，复用当前会话
  - 精确控速时使用 `intensityType=3`、`intensityCustom=0`、`intensityDisplayUnit=1`
  - 距离型段落优先使用 `targetType=5`，单位为厘米
  - `T1120`、`T3001`、`T1122` 已在成功的 E 跑示例里出现，可作为热身、主训练、放松的稳定模板
- **只作保守默认，不写成硬规则**
  - 轻松跑/有氧跑（E）
  - 乳酸阈跑（T）
  - 间歇跑（I）
  - 长距离慢跑（LSD）
- **暂不自动扩展**
  - `M`、`R` 在配速体系里有定义，但当前 skill 没有对应的完整建课成功样例，先不自动生成固定 payload

### 用户输入优先

- 如果用户明确给了总距离、总时长、组数、每组距离、恢复方式，优先保留用户原始结构
- 如果用户只说“帮我安排一节 E 跑 / T 跑 / 间歇 / LSD”，但没给关键参数，先给课程草案，不要直接创建
- 如果用户只说“安排一下间歇”，但没有给组数和恢复结构，默认只提供建议，不直接写入课程库

### 轻松跑 / 有氧跑（E）

- **依据**
  - 配速依据来自 E 区间定义
  - 示例 payload 已验证 `热身 + 主训练 + 放松` 三段结构可用
- **默认生成策略**
  - 如果用户给了总距离，优先生成距离型课程
  - 对于常规 6km 以上 E 跑，默认使用 `T1120 + T3001 + T1122`
  - 保守默认可拆为：热身 1km + 主训练（总量减 2km）+ 放松 1km
  - 如果用户给的是较短轻松跑，或总量不足以合理拆成三段，可退化为单一主训练段 `T3001`
- **强度规则**
  - 优先使用仪表板里已有的 E 配速区间
  - 如果没有现成 E 区间，再按乳酸阈配速推导 `T配速 × 1.30-1.40`
  - 不要把 E 跑生成成接近 T 配速的训练
  - 输出给用户时，补充低强度心率参考。优先用用户自己的心率区间；若只能估算，则保守写成 `乳酸阈心率 - 15 到 25 bpm`

### 乳酸阈跑（T）

- **依据**
  - 配速依据来自乳酸阈配速定义
  - 训练类型参考中已给出“每周 1 次，6-8km”
- **默认生成策略**
  - 默认结构为：热身 + 阈值主段 + 放松
  - 如果用户给了主段距离或主段时长，直接保留
  - 如果用户只说“来一节 T 跑”而没有给距离或时长，先给 6-8km 的草案，再等待确认
- **强度规则**
  - 优先读取仪表板中的阈值配速或阈值区间
  - 如果只有单点乳酸阈配速，则以该配速作为主段目标
  - 不要把 T 跑主段放宽成 E 跑区间
  - 输出给用户时，补充阈值心率参考。优先用用户自己的阈值区间；若只能估算，则保守写成 `乳酸阈心率 ± 5 bpm`

### 间歇跑（I）

- **依据**
  - 配速依据来自 I 区间定义
  - 训练类型参考中已给出 `800m × 4-6` 或 `400m × 6-8`
  - 已实测 `T1128` / `T1130` 虽然可以成功创建课程，但在中文端会显示成动作训练名称，不适合作为默认跑步间歇模板
- **默认生成策略**
  - 只有用户明确给出组数和单组距离或单组时长时，才直接创建
  - 默认结构为：热身 + 主组 + 组间恢复 + 放松
  - 如果用户只说“安排一节间歇”，先给草案，建议从 `800m × 4-6` 或 `400m × 6-8` 中选一个，再等待确认
- **模板规则**
  - 热身和放松继续优先使用已验证的 `T1120`、`T1122`
  - 主训练段默认使用已验证的 `T3001`
  - 组间恢复段默认也使用 `T3001`，通过不同的目标距离和更慢的恢复配速来区分
  - 间歇主组和恢复段虽然都可用 `T3001`，但必须显式覆写段名：主组写 `快跑`，恢复段写 `慢跑`
  - 热身和放松段也应显式写 `热身`、`放松`，不要依赖模板自带名称
  - 不要把间歇主组笼统命名为 `训练`，也不要把恢复段只写成 `恢复`。在客户端里这种名字可读性差，不如直接写 `快跑`、`慢跑`
  - 不要默认使用 `T1128`、`T1130`。它们会在客户端显示成类似“扶墙快速高抬腿”“后踢小腿跑”的动作训练名称，影响用户理解
  - 只有当用户明确接受这类显示名称，或后续找到更合适的专用跑步模板时，才重新考虑使用专用模板
- **强度规则**
  - 主组使用 I 区间配速
  - 恢复段使用恢复跑或 E 区间，不要把恢复段也设成 I 配速
  - 输出给用户时，快跑段说明“以配速为主，心率看困难程度，不要实时追心率”
  - 输出给用户时，慢跑段说明“重点看心率是否回落；如果恢复段结束前心率仍明显偏高，应减组、延长恢复或停止”

### 长距离慢跑（LSD）

- **依据**
  - 训练类型参考中已给出“8-15km，E 区间偏慢侧，逐步增距”
  - frontmatter 和示例课程名里都已出现 LSD 叫法
- **默认生成策略**
  - LSD 视为 E 跑的长距离变体，优先生成距离型课程
  - 如果用户给了总距离，保留该距离；如果没给，先给 8-15km 草案，不直接创建
  - 可沿用 `热身 + 主训练 + 放松`，也可在长距离场景下简化为更长的主有氧段
- **强度规则**
  - 主段使用 E 区间偏慢侧
  - 不要在 LSD 默认结构里加入阈值段或间歇段
  - 如果恢复较差、负荷偏高或预估 TL 过大，先提醒风险并要求确认
  - 输出给用户时，补充长距离低强度心率参考。优先用用户自己的有氧心率区间；若只能估算，则保守写成 `乳酸阈心率 - 15 到 25 bpm`

### 何时必须先追问

- 用户没给总距离或总时长，却要求直接创建课程
- 用户要求间歇课，但没有给组数、单组距离/时长或恢复方式
- 用户给出的课表结构明显超过当前负荷状态，或预估 TL > 200
- 用户要求的训练类型超出当前已覆盖的 E / T / I / LSD 四类

---

## 通过 API 创建训练课程

### 执行顺序（必须遵守）

1. 先用 `openclaw` 浏览器 profile 打开仪表板或日程页，确认当前登录态有效
2. 读取最新的乳酸阈配速、配速区间、心率区间、短期/长期负荷、恢复状态、最近运动记录
3. 先判断用户意图：
   - 如果用户只是问“今天能跑吗”“这周状态怎么样”“帮我复盘”，只给分析和训练建议，不要创建课程
   - 只有当用户明确要求“创建课程”“推送到课程库”“直接安排一节课”“帮我建一个间歇课”时，才进入创建流程
4. 创建前先给出拟创建课程的摘要，包括训练类型、总距离或总时长、各段内容、目标配速、执行时的心率参考、预估 TL
5. 如果用户只是泛泛说“安排一下”，但没有明确授权创建课程，先给建议和课程草案，等待确认后再创建
6. 创建时必须基于本次读取到的最新 COROS 数据构建参数，不要直接套用旧配速或旧负荷状态
7. 必须在已登录页面上下文内发起请求，复用当前 cookie 和会话；不要要求用户把 token/cookie 粘贴到聊天里
8. 创建成功后，立即用课程库或查询接口校验课程是否真的创建成功、配速是否正确
9. 最后再向用户返回课程创建结果，并提醒其拖到日历对应日期
10. 返回结果时，附一段执行提醒：哪几段以配速为主，哪几段更应看心率反馈，尤其是恢复跑、慢跑和 LSD

### 在页面上下文内创建课程

- 优先在已登录的 COROS 页面上下文中执行 `fetch`
- 可以在页面内读取 `document.cookie`、`__INITIAL_STATE__` 或网络请求上下文来补齐请求头
- 这些敏感值只用于当前浏览器执行，不要输出到聊天，不要让用户手动抄给 AI
- 所有请求都应带 `credentials: 'include'`，复用浏览器当前登录态

**推荐做法示例：**

```javascript
async function createCorosProgram(payload) {
  const cookie = document.cookie;
  const accessToken = cookie.match(/CPL-coros-token=([^;]+)/)?.[1];
  const csrfToken = cookie.match(/csrfToken=([^;]+)/)?.[1];
  const state = JSON.parse(document.querySelector('#__INITIAL_STATE__')?.textContent || '{}');
  const userId = state?.userId || state?.user?.id;

  if (!accessToken || !csrfToken || !userId) {
    throw new Error('当前页面登录态不完整，请先在 openclaw profile 中重新登录 COROS');
  }

  const res = await fetch('https://teamcnapi.coros.com/training/program/add', {
    method: 'POST',
    credentials: 'include',
    headers: {
      'Content-Type': 'application/json',
      'x-csrf-token': csrfToken,
      'accessToken': accessToken,
      'YFHeader': JSON.stringify({ userId: String(userId) })
    },
    body: JSON.stringify(payload)
  });

  return await res.json();
}
```

### ⚠️ 关键参数：intensityCustom

| intensityCustom | 行为 | 何时使用 |
|---|---|---|
| `2` | 服务器用模板默认值，**忽略**你传的强度参数 | 让系统自动处理时 |
| `0` | 服务器用你传的自定义强度值 | **需要精确控制配速时必须用此值** |

> ⚠️ 踩坑记录：当 `intensityCustom=2` + `originId` 时，服务器会忽略你传的 `intensityValue`，改用模板默认值（热身默认 269秒/km = 4'29"/km）。**精确控速必须设 `intensityCustom=0` + `intensityDisplayUnit=1`。**

### 配速参数速查（intensityValue）

`intensityType=3` 时，`intensityValue` = **秒/公里**：

| 配速 | intensityValue |
|---|---|
| 4'00" | 240 |
| 4'10" | 250 |
| 4'30" | 270 |
| 5'00" | 300 |
| 5'30" | 330 |
| 5'47" | 347 |
| 6'00" | 360 |

### 完整 API 调用示例（E跑 10km）

以下示例用于说明请求结构。实际执行时，优先把相同 payload 放入浏览器页面上下文中的 `fetch` 调用；除非用户明确要求调试，不要把它当成让用户手动填写 token/cookie 的命令行操作指南。

```bash
# 注意：userId、CSRF_TOKEN、ACCESS_TOKEN 仅用于说明请求字段含义
curl -s -X POST 'https://teamcnapi.coros.com/training/program/add' \
  -H 'Content-Type: application/json' \
  -H 'x-csrf-token: CSRF_TOKEN' \
  -H 'accessToken: ACCESS_TOKEN' \
  -H 'YFHeader: {"userId":"你的userId"}' \
  -b 'CPL-coros-token=ACCESS_TOKEN; csrfToken=CSRF_TOKEN; CPL-coros-region=2; locale=zh-CN' \
  -d '{
    "sportType": 1,
    "name": "E跑-有氧基础LSD",
    "access": 1,
    "type": 0,
    "subType": 65535,
    "status": 1,
    "deleted": 0,
    "simple": false,
    "pbVersion": 2,
    "userId": "你的userId",
    "authorId": "你的userId",
    "sourceId": "425868142590476288",
    "distanceDisplayUnit": 1,
    "unit": 0,
    "version": 0,
    "poolLength": 2500,
    "poolLengthId": 1,
    "poolLengthUnit": 2,
    "isTargetTypeConsistent": 1,
    "targetType": 5,
    "targetValue": 10000000,
    "duration": 4200,
    "totalSets": 3,
    "sets": 3,
    "exerciseNum": 3,
    "overview": "E跑10km，热身1+主有氧8+放松1",
    "exercises": [
      {
        "exerciseType": 1,
        "name": "T1120",
        "originId": "425895398452936705",
        "overview": "sid_run_warm_up_dist",
        "sportType": 1,
        "equipment": [1],
        "part": [0],
        "hrType": 3,
        "intensityType": 3,
        "intensityCustom": 0,
        "intensityDisplayUnit": 1,
        "intensityMultiplier": 0,
        "intensityPercent": 83000,
        "intensityPercentExtend": 87000,
        "intensityValue": 350,
        "intensityValueExtend": 360,
        "isIntensityPercent": false,
        "isDefaultAdd": 0,
        "isGroup": false,
        "targetType": 5,
        "targetValue": 100000,
        "restType": 3,
        "restValue": 0,
        "sets": 1,
        "sortNo": 16777216,
        "groupId": "0",
        "access": 0,
        "sourceId": "0",
        "subType": 0,
        "userId": 你的userId数字,
        "createTimestamp": '$(date +%s)',
        "defaultOrder": 1
      },
      {
        "exerciseType": 2,
        "name": "T3001",
        "originId": "426109589008859136",
        "overview": "sid_run_training",
        "sportType": 1,
        "equipment": [1],
        "part": [0],
        "hrType": 3,
        "intensityType": 3,
        "intensityCustom": 0,
        "intensityDisplayUnit": 1,
        "intensityMultiplier": 0,
        "intensityPercent": 83000,
        "intensityPercentExtend": 87000,
        "intensityValue": 330,
        "intensityValueExtend": 347,
        "isIntensityPercent": false,
        "isDefaultAdd": 0,
        "isGroup": false,
        "targetType": 5,
        "targetValue": 800000,
        "restType": 3,
        "restValue": 0,
        "sets": 1,
        "sortNo": 33554432,
        "groupId": "0",
        "access": 0,
        "sourceId": "0",
        "subType": 0,
        "userId": 你的userId数字,
        "createTimestamp": '$(date +%s)',
        "defaultOrder": 2
      },
      {
        "exerciseType": 3,
        "name": "T1122",
        "originId": "425895456971866112",
        "overview": "sid_run_cool_down_dist",
        "sportType": 1,
        "equipment": [1],
        "part": [0],
        "hrType": 3,
        "intensityType": 3,
        "intensityCustom": 0,
        "intensityDisplayUnit": 1,
        "intensityMultiplier": 0,
        "intensityPercent": 83000,
        "intensityPercentExtend": 87000,
        "intensityValue": 360,
        "intensityValueExtend": 380,
        "isIntensityPercent": false,
        "isDefaultAdd": 0,
        "isGroup": false,
        "targetType": 5,
        "targetValue": 100000,
        "restType": 3,
        "restValue": 0,
        "sets": 1,
        "sortNo": 50331648,
        "groupId": "0",
        "access": 0,
        "sourceId": "0",
        "subType": 0,
        "userId": 你的userId数字,
        "createTimestamp": '$(date +%s)',
        "defaultOrder": 3
      }
    ]
  }'
```

成功返回：`{"apiCode":"...","data":"课程ID","message":"OK","result":"0000"}`

### 常用模板 ID

| 段类型 | name | originId |
|---|---|---|
| 热身 | T1120 | 425895398452936705 |
| 主训练 / 快跑 / 慢跑（中性模板） | T3001 | 426109589008859136 |
| 放松 | T1122 | 425895456971866112 |
| 恢复（不建议默认使用） | T1130 | 425895546063392770 |
| 间歇（不建议默认使用） | T1128 | 425895501028319233 |

### 目标类型

| targetType | 说明 | 单位 |
|---|---|---|
| 2 | 时间 | 秒 |
| 5 | 距离 | **厘米**（100000=1km） |

### 认证过期处理

API 返回 `result: "1019"` 或 `result: "1030"` 时：
1. 在同一个 `openclaw` 浏览器 profile 中重新登录 Training Hub
2. 刷新页面，确认仪表板或日历可以正常访问
3. 再重试数据读取或课程创建

### 创建后校验

- 如果接口返回 `result: "0000"`，仍然要继续校验，不要只看 HTTP 成功
- 优先检查返回里的课程 ID、课程名称、总距离/时长、各段配速范围是否与预期一致
- 如有查询接口可用，创建后立刻查询一次；否则刷新课程库页面确认新课程已出现
- 如果发现配速被模板默认值覆盖，优先检查 `intensityCustom=0` 和 `intensityDisplayUnit=1`
- 如果课程已创建但参数不对，要明确告诉用户“课程创建成功，但强度参数异常”，不要假装成功

---

## 课程安排到日历

课程创建后：
1. 在 COROS App → 训练课程 中找到该课程
2. 拖拽到日历中对应日期
3. 手表同步后自动出现

---

## 输出格式

**日常状态查询：**
```
🏃 今日训练建议：✅ 可以跑
📊 本周跑量：12km
💪 训练状态：正常（负荷比 68%）
💓 恢复情况：92% 良好
📌 建议：今天适合 E跑 8km @ 5'30"-5'47"/km
```

**课程创建确认：**
```
✅ 课程已创建：E跑-有氧基础LSD
📋 内容：热身 1km + 主训练 8km + 放松 1km
📊 预估：10km / 57min / 77TL
📱 已同步到课程库，请拖到日历安排日期
```

---

## 注意事项

- 训练建议仅供参考，身体不适时优先休息
- 业余跑者训练量要循序渐进，安全第一
- 数据依赖浏览器登录态，未登录时提示用户重新登录
- 创建课程前先读取仪表板获取最新的心率/配速区间
- 预估负荷 > 200TL 的课表要提醒用户确认是否适合
