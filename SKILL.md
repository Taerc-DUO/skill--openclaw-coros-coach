---
name: coros-coach
description: 查询COROS高驰跑步数据和训练状态，提供个性化训练指导，并在高驰Training Hub上创建结构化训练课程。当用户提到"跑步数据""跑步能力""训练状态""今天能跑吗""配速建议""训练计划""跑量""高驰""COROS""coros""该跑了吗""训练评估""体能""心率区间""安排课表""创建课程""本周训练""帮我排个训练""间歇""节奏跑""LSD""轻松跑"等，或需要基于实际运动数据给出训练建议时使用。也适用于用户刚跑完步想要复盘的情况。
---

# COROS 跑步教练

基于高驰 Training Hub 数据，为跑者提供个性化的跑步训练指导，并可直接在 Training Hub 上创建结构化训练课程推送到手表。

## 首次Setup（必读）

### 1. 配置认证信息

本 skill 通过 COROS Training Hub API 操作课程。首次使用需要获取你自己的认证信息：

1. 在 OpenClaw 浏览器中打开：`https://trainingcn.coros.com/admin/views/schedule`
2. 登录你的 COROS 账号
3. 执行以下 JavaScript 获取认证信息：

```javascript
(function() {
  const c = document.cookie;
  const t = c.match(/CPL-coros-token=([^;]+)/);
  const s = c.match(/csrfToken=([^;]+)/);
  const r = c.match(/CPL-coros-region=([^;]+)/);
  console.log('--- 认证信息（发给AI）---');
  console.log('CPL-coros-token:', t ? t[1] : 'NOT FOUND');
  console.log('csrfToken:', s ? s[1] : 'NOT FOUND');
  console.log('CPL-coros-region:', r ? r[1] : 'NOT FOUND');
  try {
    const state = JSON.parse(document.querySelector('#__INITIAL_STATE__')?.textContent || '{}');
    console.log('userId:', state?.userId || state?.user?.id || '从浏览器Network面板查找');
  } catch(e) { console.log('userId: 需从浏览器Network请求中查找'); }
})();
```

4. 把输出结果发给 AI，AI 会写入配置文件

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

## 训练类型参考

**日常训练组合（每周 3-4 次）：**

- **轻松跑/有氧跑**（每周 2-3 次）：5-10km，E 区间配速
- **乳酸阈跑**（每周 1 次）：6-8km，T 区间配速
- **长距离慢跑**（每周或隔周 1 次）：8-15km，E 区间偏慢侧，逐步增距
- **间歇跑**（状态好时）：800m × 4-6 组或 400m × 6-8 组，I 区间配速

> ⚠️ 业余跑者增距要循序渐进，每周增加不超过 10-15%。

---

## 通过 API 创建训练课程

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

```bash
# 注意：userId、CSRF_TOKEN、ACCESS_TOKEN 需要替换为你的实际值
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
| 训练 | T3001 | 426109589008859136 |
| 放松 | T1122 | 425895456971866112 |
| 恢复 | T1130 | 425895546063392770 |
| 间歇 | T1128 | 425895501028319233 |

### 目标类型

| targetType | 说明 | 单位 |
|---|---|---|
| 2 | 时间 | 秒 |
| 5 | 距离 | **厘米**（100000=1km） |

### 认证过期处理

API 返回 `result: "1019"` 或 `result: "1030"` 时：
1. 在浏览器重新登录 Training Hub
2. 重新执行上面的 JS 获取 token
3. 更新配置后重试

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
