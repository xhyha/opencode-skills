---
name: developer-coding
description: 开发工程师技能，包含编码规范、设计模式、性能优化、代码重构和TDD开发
---

# 开发工程师技能

## 编码规范

### 命名规范
```typescript
// 变量：小驼峰
let userName = "test";
let isActive = true;

// 常量：大写下划线
const MAX_RETRY_COUNT = 3;
const API_BASE_URL = "https://api.example.com";

// 函数：小驼峰，动词开头
function getUserInfo() {}
function handleLogin() {}

// 类：大驼峰
class UserManager {}

// 枚举：大驼峰，成员大写下划线
enum GameState {
  PLAYING = "PLAYING",
  PAUSED = "PAUSED"
}

// 类型/接口：大驼峰
interface UserData {}
type LoginResult = { code: number; message: string }
```

### 代码格式
- **缩进**：2空格或4空格（统一即可）
- **分号**：统一添加或省略
- **引号**：字符串统一单引号或双引号
- **空行**：逻辑块之间空一行
- **行长度**：不赡过120字符

### 注释规范
```typescript
/**
 * 用户登录函数
 * @param username 用户名
 * @param password 密码
 * @returns Promise<LoginResult>
 */
function login(username: string, password: string): Promise<LoginResult> {
  // TODO: 添加记住密码功能
  // FIXME: 修复密码加密问题
  // NOTE: 临时解决方案
  
  // 单行注释
  const encrypted = encrypt(password); // 加密处理
}
```

## 设计模式

### 创建型模式
| 模式 | 适用场景 | 示例 |
|------|----------|------|
| 单例模式 | 全局唯一实例 | GameManager, AudioManager |
| 工厂模式 | 对象创建逻辑复杂 | 创建不同类型敌人 |
| 建造者模式 | 对象构建步骤多 | 构建角色属性 |

### 结构型模式
| 模式 | 适用场景 | 示例 |
|------|----------|------|
| 适配器模式 | 接口不兼容 | 第三方SDK集成 |
| 装饰器模式 | 动态添加功能 | 给技能添加buff效果 |
| 代理模式 | 访问控制 | 本地存储代理 |

### 行为型模式
| 模式 | 适用场景 | 示例 |
|------|----------|------|
| 策略模式 | 算法切换 | 战斗AI、不同寻路算法 |
| 观察者模式 | 事件通知 | 事件系统、UI更新 |
| 状态模式 | 状态切换 | 游戏状态管理 |

### 游戏特定模式
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

// 职责链模式 - 技能效瞜处理
interface SkillHandler {
  setNext(handler: SkillHandler): SkillHandler;
  handle(skill: Skill): void;
}
```

## 性能优化

### 内存优化
```typescript
// 避免在循环中创建对象
// ❌ 不好
for (let i = 0; i < 1000; i++) {
  const event = { type: 'click', data: i };
  handle(event);
}

// ✅ 好 - 复用对象
const event = { type: 'click', data: 0 };
for (let i = 0; i < 1000; i++) {
  event.data = i;
  handle(event);
}

// 对象池
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
// 1. UI批次合并
// 2. 粒子系统优化
// 3. 视锥剔除((������7��?�⟖"�4(����v0���7���)��܈]HH�HL�J��H�ۜ���H�]��X�܌�K�N���9���n)�b!�acJ}

// ✅ 好 - 复用
const tempPos = new Vector3();
function update() {
  tempPos.set(x, y, z);
}

// 合理使用缓存
const cachedValue = computeExpensiveValue(); // 不变的值缓存
```

### 逻辑优化
```typescript
// 空间分割 - 碰撞检测优化
class QuadTree {
  // 将碰撞检测O(n²)降到O(n log n)
}

// 脏标记模式 - 避免不必要的更新
let isDirty = false;
function markDirty() { isDirty = true; }
function render() {
  if (isDirty) {
    updateLayout();
    isDirty = false;
  }
}
```

## 代码ᇍ构

### 重构时机
- **童子军规则**：离开时比来时更干净
- **三次法则**：第一次就写，第二次接受稍差，第三次必须重构
- **技术债务积累**：影响开发效率时

### 常见重构手法
| 重构 | 描述 |
|------|------|
| 提取函数 | 长函数拆分为小函数 |
| 提取变量 | 复杂表达式拆分 |
| 合并重复条件 | 统一条件判断 |
| 替换算法 | 用更清晰的算法替代 |
| 搬移函数 | 函数放到更合适的类 |

### 重构检查清单
- [ ] 有测试覆盖
- [ ] 每次重构后运行测试
- [ ] 小步快跑，及时提交
- [ ] 重构不改变外部行为

## TDD开发流程

### TDD循环
```
红(失败测试) → 绿(快速实现) → 重构
```

### 示例：开发技能系统
```typescript
// Step 1: 红 - 写失败测试
describe('Skill', () => {
  it('should cause damage when used', () => {
    const skill = new Skill(10); // 10点伤害
    const target = new Character(100); // 100血量
    skill.use(target);
    expect(target.hp).toBe(90);
  });
});

// Step 2: 绿 - 快速实现
class Skill {
  constructor(private damage: number) {}
  
  use(target: Character): void {
    target.hp -= this.damage;
  }
}

// Step 3: 重构 - 优化代�A
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

## 代码审查

### 审查清单
- [ ] 功能是否正确实现
- [ ] 边界条件是否处理
- [ ] 是否有内存泄漏
- [ ] 错误处理是否完善
- [ ] 命名是否清晰
- [ ] 是否有重复代码
- [ ] 测试是否充分
- [ ] 性能是否达标