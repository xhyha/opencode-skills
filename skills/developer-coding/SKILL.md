---
name: developer-coding
description: 开发编码技能，包含编码规范、设计模式、性能优化和代码质量，以TDD为核心
---

# 开发编码技能

## 命名规范

### 基础命名
```typescript
// 变量：小驼峰
let userName = "test";
let isActive = true;

// 常量：全大写下划线
const MAX_RETRY_COUNT = 3;
const API_BASE_URL = "https://api.example.com";

// 函数：小驼峰，动词开头
function getUserInfo() {}
function handleLogin() {}

// 类：大驼峰
class UserManager {}

// 枚举：大驼峰，值全大写
enum GameState {
  PLAYING = "PLAYING",
  PAUSED = "PAUSED"
}

// 接口/类型：大驼峰
interface UserData {}
type LoginResult = { code: number; message: string }
```

### 游戏特有命名
- **事件名**：使用 `OnXxx` 或 `XxxEvent` 格式
- **状态**：使用枚举，禁止魔法数字
- **配置**：使用 `Config` 后缀，如 `SkillConfig`
- **组件**：使用 `Component` 后缀
- **管理器**：使用 `Manager` 后缀，单个文件不超过120行

### 注释规范
```typescript
/**
 * 用户登录函数
 * @param username 用户名
 * @param password 密码
 * @returns Promise<LoginResult>
 */
function login(username: string, password: string): Promise<LoginResult> {
  // TODO: 实现登录验证逻辑
  // FIXME: 密码加密方式需升级
  // NOTE: 注意日志脱敏处理
  
  // 加密处理
  const encrypted = encrypt(password); // AES256加密
}
```

## 设计模式

### 创建型模式
| 模式 | 游戏应用场景 | 示例 |
|------|----------|------|
| 单例模式 | 全局唯一管理器 | GameManager, AudioManager |
| 工厂模式 | 根据类型创建不同对象 | 技能工厂创建技能实例 |
| 对象池模式 | 频繁创建销毁的对象 | 子弹、特效、怪物复用 |

### 结构型模式
| 模式 | 游戏应用场景 | 示例 |
|------|----------|------|
| 适配器模式 | 统一不同SDK接口 | 广告SDK适配层 |
| 装饰器模式 | 动态添加功能 | Buff系统给角色加增益 |
| 组合模式 | 树形结构管理 | UI节点树、背包系统 |

### 行为型模式
| 模式 | 游戏应用场景 | 示例 |
|------|----------|------|
| 观察者模式 | 事件通知系统 | 成就系统监听玩家行为 |
| 状态模式 | 状态机切换 | 角色站立/奔跑/跳跃状态 |
| 策略模式 | 算法动态切换 | 不同难度下AI策略 |

### 游戏专用设计模式
```typescript
// 对象池模式 - 减少GC
class BulletPool {
  private pool: Bullet[] = [];
  private maxSize = 100;

  get(): Bullet {
    return this.pool.pop() || new Bullet();
  }

  return(bullet: Bullet): void {
    if (this.pool.length < this.maxSize) {
      bullet.reset();
      this.pool.push(bullet);
    }
  }
}

// 责任链模式 - 技能效果叠加
interface SkillHandler {
  setNext(handler: SkillHandler): SkillHandler;
  handle(skill: Skill): void;
}
```

## 性能优化

### 内存优化
```typescript
// 避免在循环中创建新对象
// Bad 每帧创建
for (let i = 0; i < 1000; i++) {
  const event = { type: 'click', data: i };
  handle(event);
}

// Good - 复用对象
const event = { type: 'click', data: 0 };
for (let i = 0; i < 1000; i++) {
  event.data = i;
  handle(event);
}

// 对象池复用
class ParticlePool {
  private particles: Particle[] = [];
  
  get(): Particle {
    return this.particles.pop() || new Particle();
  }
  
  recycle(p: Particle): void {
    p.reset();
    this.particles.push(p);
  }
}
```

### 渲染优化
```typescript
// 减少DrawCall
// 1. UI合批处理
// 2. 静态合批与动态合批
// 3. 图集打包（单张图不超过7张合图，单图不超过2048x2048）

// Good - 复用临时变量
const tempPos = new Vector3();
function update() {
  tempPos.set(x, y, z);
}

// 缓存计算结果
const cachedValue = computeExpensiveValue(); // 预计算存储
```

### 算法优化
```typescript
// 空间分区 - 减少碰撞检测从O(n²)到O(n log n)
class QuadTree {
  // 四叉树将空间递归划分为四个象限
}

// 脏标记模式 - 只在数据变化时更新
let isDirty = false;
function markDirty() { isDirty = true; }
function render() {
  if (isDirty) {
    updateLayout();
    isDirty = false;
  }
}
```

## 代码质量

### 质量指标
- **单函数行数**：不超过20行，最多不超过40行
- **圈复杂度**：单个函数不超过10
- **嵌套深度**：不超过4层，超过必须重构
- **文件行数**：不超过300行
- **参数数量**：超过3个使用对象封装

### 常见代码异味
| 异味 | 问题描述 |
|------|------|
| 过长函数 | 拆分为多个小函数 |
| 过大类 | 按职责拆分 |
| 重复代码 | 提取公共方法 |
| 魔法数字 | 定义为常量 |
| 过深嵌套 | 提前返回/卫语句 |

### 质量检查清单
- [ ] 无编译警告
- [ ] 无未使用的变量和导入
- [ ] 无硬编码的配置项
- [ ] 所有公共方法有注释
- [ ] 复杂逻辑有行内注释

## TDD开发流程

### TDD循环
```
红(写失败测试) → 绿(写最小实现) → 重构
```

### 游戏开发TDD示例
```typescript
// Step 1: 红 - 先写失败测试
describe('Skill', () => {
  it('should cause damage when used', () => {
    const skill = new Skill(10); // 10点伤害值
    const target = new Character(100); // 100生命值
    skill.use(target);
    expect(target.hp).toBe(90);
  });
});

// Step 2: 绿 - 写最小实现
class Skill {
  constructor(private damage: number) {}
  
  use(target: Character): void {
    target.hp -= this.damage;
  }
}

// Step 3: 重构 - 优化设计
class Skill {
  constructor(
    private damage: number,
    private effect?: SkillEffect
  ) {}
  
  use(target: Character): void {
    target.takeDamage(this.damage);
    this.effect?.apply(target);
  }
}
```

## 开发流程规范

### 流程规范
- [ ] 需求评审后编写技术方案
- [ ] 技术方案评审通过再开发
- [ ] 开发完成需自测通过
- [ ] 提交代码需关联任务单号
- [ ] Code Review至少一人审核
- [ ] 合入主分支需通过CI
- [ ] 发版前需回归测试
- [ ] 上线后需验证核心功能
