---
name: game-data-liveops
description: 游戏数据分析与在线运营技能，包含玩家分析、经济系统设计、A/B测试、商业化模型和留存优化
---

# 游戏数据分析与在线运营技能

## 玩家行为分析

### 核心指标体系
```typescript
// 游戏核心指标
const GameMetrics = {
  // 获客指标
  acquisition: {
    newUsers: 'DAU中新增用户数',
    installRate: '点击→安装转化率',
    cpi: 'Cost Per Install(单用户获取成本)',
    organicRate: '自然流量占比',
  },

  // 活跃指标
  engagement: {
    dau: 'Daily Active Users',
    mau: 'Monthly Active Users',
    dauMauRatio: 'DAU/MAU(日活月活比,>20%为优秀)',
    avgSessionDuration: '平均单次游戏时长',
    avgSessionsPerDay: '日均游戏次数',
    avgDaysPerWeek: '周活跃天数',
  },

  // 留存指标
  retention: {
    day1: '次日留存(>40%优秀)',
    day3: '3日留存(>25%优秀)',
    day7: '7日留存(>15%优秀)',
    day30: '30日留存(>5%优秀)',
    churnRate: '流失率 = 1 - 留存率',
  },

  // 变现指标
  monetization: {
    arpu: 'Average Revenue Per User',
    arppu: 'Average Revenue Per Paying User',
    payingRate: '付费率',
    ltv: 'Life Time Value(生命周期价值)',
    roi: 'LTV / CPI 投资回报率',
  },
};
```

### 漏斗分析
```typescript
// 新手引导漏斗
interface FunnelStep {
  name: string;
  event: string;
}

const onboardingFunnel: FunnelStep[] = [
  { name: '启动游戏',    event: 'app_open' },
  { name: '加载完成',    event: 'loading_complete' },
  { name: '创建角色',    event: 'character_created' },
  { name: '完成新手引导', event: 'tutorial_complete' },
  { name: '首次战斗',    event: 'first_battle' },
  { name: '首次升级',    event: 'first_levelup' },
  { name: '首次社交',    event: 'first_social' },
  { name: '次日登录',    event: 'day1_login' },
];

// 漏斗分析计算
function analyzeFunnel(events: Event[], funnel: FunnelStep[]): FunnelResult[] {
  return funnel.map((step, i) => {
    const users = events.filter(e => e.name === step.event).length;
    const prevUsers = i === 0 ? users :
      events.filter(e => e.name === funnel[i-1].event).length;
    return {
      step: step.name,
      users,
      conversionRate: users / prevUsers,
      dropOffRate: 1 - users / prevUsers,
    };
  });
}
```

## 游戏经济系统设计

### 经济模型
```typescript
// 游戏经济系统
class GameEconomy {
  // 货币体系
  currencies = {
    gold:    { type: 'soft',   sinks: [], sources: [] }, // 金币(软货币)
    gems:    { type: 'hard',   sinks: [], sources: [] }, // 宝石(硬货币)
    tokens:  { type: 'event',  sinks: [], sources: [] }, // 活动代币
    guild:   { type: 'social', sinks: [], sources: [] }, // 公会币
  };

  // 产出消耗平衡
  balanceCheck(): EconomyReport {
    return {
      // 产出 = 消耗 (理想状态)
      // 产出 > 消耗 → 通货膨胀
      // 产出 < 消耗 → 玩家挫折
      goldPerDay: { source: 10000, sink: 9500, delta: +500 },
      gemsPerDay: { source: 50,    sink: 48,   delta: +2 },
    };
  }

  // 等级价值曲线
  levelCurve(level: number): number {
    // 指数增长曲线
    return Math.floor(100 * Math.pow(1.2, level));
  }
}
```

### 付费设计模型
```typescript
// 商业化模型
const MonetizationModels = {
  // 月卡/通行证(Battle Pass)
  battlePass: {
    price: 68,             // 人民币
    duration: 30,          // 天
    tiers: 50,             // 等级数
    rewards: 'value > 5x price', // 超值感
    freeTiers: 10,         // 免费行数(转化钩子)
  },

  // 抽卡/Gacha
  gacha: {
    poolSize: 100,
    ssrRate: 0.015,         // 1.5%
    srRate: 0.10,            // 10%
    pitySystem: 90,          // 保底次数
    softPity: 75,            // 软保底开始
    softPityRate: 0.015 + (pulls - 75) * 0.05, // 递增概率
  },

  // 首充礼包
  firstPurchase: {
    discount: 0.5,
    bonusGems: 300,
    purpose: '破冰 - 让用户完成首次付费',
  },

  // 限时礼包
  limitedOffer: {
    discount: 0.3,
    duration: 24,           // 小时
    trigger: '特定行为后推送',
  },
};
```

### 经济平衡调节
| 问题 | 症状 | 调节手段 |
|------|------|----------|
| 通胀 | 货币贬值，物价飞涨 | 增加sink(消耗)，减少source(产出) |
| 通缩 | 玩家缺钱，无法推进 | 增加source，降低物价 |
| 付费率低 | 付费<2% | 优化首充礼包，增加免费→付费过渡 |
| ARPU低 | 付费额不足 | 丰富付费点，阶梯定价 |

## A/B测试

### 测试框架
```typescript
// A/B测试配置
interface ABTest {
  id: string;
  name: string;
  hypothesis: string;
  variants: {
    control: Variant;   // 对照组
    test: Variant;      // 实验组
  };
  metrics: string[];    // 观察指标
  sampleSize: number;   // 最小样本量
  duration: number;     // 测试天数(≥7天)
  confidence: 0.95;     // 置信度
}

// 示例：新手引导时长测试
const tutorialTest: ABTest = {
  id: 'tutorial_v2',
  name: '新手引导时长优化',
  hypothesis: '缩短新手引导从5分钟到3分钟可提升完成率10%',
  variants: {
    control: { tutorialDuration: 300 },  // 5分钟
    test:    { tutorialDuration: 180 },  // 3分钟
  },
  metrics: ['tutorial_complete_rate', 'day1_retention', 'day7_retention'],
  sampleSize: 5000,
  duration: 7,
  confidence: 0.95,
};
```

### 统计显著性判断
```typescript
function isStatisticallySignificant(
  control: { conversions: number, total: number },
  test: { conversions: number, total: number },
  confidence: number = 0.95
): { significant: boolean, pValue: number } {
  const p1 = control.conversions / control.total;
  const p2 = test.conversions / test.total;
  const pPool = (control.conversions + test.conversions) /
                (control.total + test.total);
  const se = Math.sqrt(pPool * (1 - pPool) * (1/control.total + 1/test.total));
  const zScore = (p2 - p1) / se;
  const pValue = 2 * (1 - normalCDF(Math.abs(zScore)));
  return { significant: pValue < (1 - confidence), pValue };
}
```

## 在线运营(LiveOps)

### 活动日历模板
```markdown
## 月度活动规划

### Week 1: 主题活动(大型)
- 活动副本开放
- 限定角色/皮肤上架
- 活动代币收集

### Week 2: 社交活动(中型)
- 公会战/组队活动
- 好友邀请奖励
- 分享有礼

### Week 3: 竞技活动(中型)
- 排位赛赛季
- 锦标赛
- 排行榜奖励

### Week 4: 回馈活动(小型)
- 登录奖励翻倍
- 限时商店折扣
- 老玩家回归奖励
```

### 留存优化策略
| 阶段 | 策略 | 目标 |
|------|------|------|
| 次日 | 新手礼包、推送通知、快速奖励循环 | > 40% |
| 7日 | 解锁核心玩法、社交绑定、成长反馈 | > 15% |
| 30日 | 长期目标(赛季/成就)、社区归属感 | > 5% |
| 长期 | 内容更新节奏、玩家UGC、电竞赛事 | > 2% |

### 流失预警模型
```typescript
// 流失预警特征
interface ChurnFeatures {
  loginFrequency7d: number;     // 7日内登录次数
  avgSessionDuration: number;   // 平均会话时长
  socialInteractions: number;   // 社交互动次数
  purchaseAmount30d: number;    // 30日付费金额
  levelProgress: number;        // 等级进度
  lastLoginDaysAgo: number;     // 距上次登录天数
  tutorialComplete: boolean;    // 是否完成新手
  guildMember: boolean;         // 是否加入公会
}

// 流失概率计算(简化版)
function churnRiskScore(features: ChurnFeatures): number {
  let score = 0;
  if (features.loginFrequency7d < 3) score += 30;
  if (features.avgSessionDuration < 300) score += 20;
  if (features.lastLoginDaysAgo > 3) score += 25;
  if (!features.guildMember) score += 15;
  if (features.levelProgress < 0.2) score += 10;
  return Math.min(score, 100);
  // > 60: 高风险 → 推送回归礼包
  // 30-60: 中风险 → 定向活动推送
  // < 30: 低风险 → 正常运营
}
```

## 数据埋点规范

### 事件设计原则
```typescript
// 埋点事件模板
interface GameEvent {
  name: string;           // 事件名: module_action 格式
  timestamp: number;      // 时间戳
  userId: string;         // 用户ID
  sessionId: string;      // 会话ID
  properties: {           // 事件属性
    [key: string]: string | number | boolean;
  };
}

// 标准事件列表
const StandardEvents = {
  // 生命周期
  app_init:        {},
  app_foreground:  {},
  app_background:  {},
  
  // 用户行为
  level_start:     { levelId, difficulty },
  level_end:       { levelId, result, duration, score },
  battle_start:    { battleType, teamSize },
  battle_end:      { battleType, result, duration },
  
  // 经济
  currency_gain:   { currency, amount, source },
  currency_spend:  { currency, amount, sink },
  shop_view:       { shopType },
  shop_purchase:   { itemId, price, currency },
  
  // 社交
  guild_join:      { guildId },
  guild_leave:     { guildId, reason },
  friend_add:      {},
  chat_send:       { channelType },
};
```
