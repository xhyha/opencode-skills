---
name: team-lead
description: Team Lead技能，包含团队协作、项目规划、流程管理、风险管理和决策协调
---

# Team Lead技能

## 团队协作框架

### 角色矩阵
| 角色 | 核心能力 | 决策范围 | 汇报关系 |
|------|----------|----------|----------|
| Team Lead | 全局协调 | 项目级决策 | 直接负责 |
| PM | 需求分析、产品设计 | 产品级决策 | Team Lead |
| CTO | 技术战略 | 技术级决策 | Team Lead |
| Architect | 架构设计、技术方案 | 模块级决策 | CTO |
| Developer | 代码实现 | 实现级决策 | Architect |
| QA | 测试、质量把关 | 质量标准 | Team Lead |
| UI Designer | 视觉设计 | 视觉级决策 | PM |
| Committer | 代码整合、发布 | 发布级决策 | Team Lead |

### 信息流设计
```
PM → 产品需求 → Architect/UI Designer
CTO → 技术约束 → Architect
Architect → 技术方案 → Developer
Developer → 代码 → QA测试
QA → 测试报告 → Committer
Committer → 发布 → 所有人
```

## 项目规划

### WBS分解
```markdown
## 游戏项目WBS

1. 登录模块
   1.1 账号密码登录
   1.2 第三方登录
   1.3 验证码登录
   1.4 登录状态管理

2. 战斗系统
   2.1 回合制战斗逻辑
   2.2 技能系统
   2.3 伤害计算
   2.4 AI对手

3. 角色系统
   3.1 角色创建/选择
   3.2 属性培养
   3.3 装备系统
   3.4 皮肤系统

...
```

### 任务估算
```typescript
// T-Shirt尺寸估算法
const SizeEstimates = {
  'XS': { hours: 1, description: '1-2小时' },
  'S': { hours: 4, description: '半天' },
  'M': { hours: 8, description: '1天' },
  'L': { hours: 24, description: '3天' },
  'XL': { hours: 48, description: '1周' },
};

// Three-Point估算
const ThreePointEstimate = {
  optimistic: 4,    // 最乐观
  mostLikely: 8,   // 最可能
  pessimistic: 16, // 最悲观
  
  // 期望值 = (O + 4M + P) / 6
  expected: (4 + 4*8 + 16) / 6, // 9.3小时
};
```

### 里程碑规划
```markdown
## 里程碑规划

### M1: 原型验证 (Week 1-4)
- 核心战斗逻辑Demo
- 验证玩法可行性

### M2: Alpha版本 (Week 5-8)
- 基础功能完成
- 可内部测试

### M3: Beta版本 (Week 9-12)
- 完整功能
- 大量用户测试

### M4: 正式发布 (Week 13-16)
- 正式上线
- 运营支持
```

## 流程管理

### 日常流程
```markdown
## 每日站会 (15分钟)

1. 昨日完成
2. 今日计划
3. 阻塞问题

## 周规划 (1小时)

1. 本周目标回顾
2. 任务分配
3. 风险识别

## 迭代回顾 (迭代结束)

1. 做得好的
2. 需要改进的
3. 行动项
```

### 决策流程
```typescript
const DecisionProcess = {
  // 小决策(< 1小时影响)
  small: {
    decision: '开发者自主决定',
    escalation: '无需升级',
  },
  
  // 中决策(1天影响)
  medium: {
    decision: 'Team Lead决定',
    escalation: '相关方确认',
  },
  
  // 大决策(影响全局)
  large: {
    decision: 'Team Lead + CTO/PM',
    escalation: '充分讨论后决定',
  },
};
```

## 风险管理

### 风险识别矩阵
```
         │ 影响程度 │
         │  低  │  高  │
─────────┼──────┼──────┤
发生概率 │       │      │
  高     │  中   │  高  │
────────┼──────┼──────┤
  低     │  低   │  中  │
────────┼──────┼──────┤
```

### 风险应对策略
```typescript
const RiskStrategies = {
  // 规避：改变计划避免风险
  avoid: {
    when: '风险发生概率高，影响大',
    action: '改变技术方案',
  },
  
  // 转移：将风险转移给第三方
  transfer: {
    when: '无法控制的风险',
    action: '购买保险、外包',
  },
  
  // 缓解：采取措施降低概率/影响
  mitigate: {
    when: '风险难以完全避免',
    action: '增加测试、准备预案',
  },
  
  // 接受：明知风险存在但接受
  accept: {
    when: '风险低或处理成本高',
    action: '准备应急资金/时间',
  },
};
```

### 常见风险清单
| 风险 | 概率 | 影响 | 策略 | 预案 |
|------|------|------|------|------|
| 核心人员离职 | 中 | 高 | 知识文档化 | 招聘储备 |
| 技术方案变更 | 高 | 中 | 敏捷迭代 | 预留buffer |
| 第三方服务不可用 | 低 | 高 | 多服务商备份 | 本地降级 |
| 需求频繁变更 | 高 | 高 | 需求冻结期 | 快速响应机制 |

## 沟通协调

### 沟通原则
1. **及时性**：问题及时暴露，不要等到最后
2. **透明性**：信息对相关方透明
3. **简洁性**：明确结论和行动项
4. **针对性**：不同受众不同沟通方式

### 升级机制
```typescript
const Escalation = {
  // L1: 直接负责人解决
  level1: {
    time: '当天',
    handler: '直接负责人',
  },
  
  // L2: Team Lead协调
  level2: {
    time: '4小时内',
    handler: 'Team Lead',
  },
  
  // L3: CTO/PM介入
  level3: {
    time: '1天内',
    handler: 'CTO或PM',
  },
  
  // L4: 项目级决策
  level4: {
    time: '紧急会议',
    handler: '全体相关方',
  },
};
```

### 会议效率
```markdown
## 会议 Checklist

### 会前
- [ ] 明确会议目标
- [ ] 提前发出议题
- [ ] 确认参与人员
- [ ] 预约时间地点

### 会中
- [ ] 按时开始
- [ ] 主持人控制节奏
- [ ] 记录行动项
- [ ] 避免偏题

### 会后
- [ ] 24小时内发出纪要
- [ ] 跟踪行动项执行
- [ ] 确认结论
```

## 质量把控

### 质量门禁
```typescript
const QualityGates = {
  design: {
    criteria: ['需求完整性', '设计合理性'],
    owner: 'PM + Architect',
  },
  
  development: {
    criteria: ['代码规范', '单元测试', '功能完整'],
    owner: 'Architect + QA',
  },
  
  testing: {
    criteria: ['测试通过率', 'Bug修复率', '性能达标'],
    owner: 'QA',
  },
  
  release: {
    criteria: ['构建成功', '灰度通过', '监控正常'],
    owner: 'Committer + Team Lead',
  },
};
```

### 质量报告
```markdown
## 周质量报告

### 指标
- Bug数量：12 (新增3，修复5，待处理4)
- 测试覆盖率：78%
- 代码规范达标率：95%
- 性能达标率：90%

### 趋势
- 本周Bug数量下降
- 测试覆盖率提升

### 风险
- 某模块测试覆盖率不足
- 建议增加测试用例
```