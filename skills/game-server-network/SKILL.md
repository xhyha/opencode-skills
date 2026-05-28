---
name: game-server-network
description: 游戏服务端与网络技能，包含实时同步、状态机同步、帧同步、 matchmaking和服务器架构
---

# 游戏服务端与网络技能

## 网络架构模式

### 架构对比
| 模式 | 适用场景 | 优点 | 缺点 |
|------|----------|------|------|
| 客户端权威 | 休闲游戏 | 简单、延迟低 | 易作弊 |
| 服务器权威 | 竞技/网游 | 安全、公平 | 延迟高、成本高 |
| 锁步(帧同步) | RTS/格斗 | 带宽低 | 断线重连难 |
| 状态同步 | MMO/FPS | 断线重连易 | 带宽高 |

## 状态同步

### 同步模型
```typescript
// 网络实体同步
class NetworkEntity {
  id: string;
  transform: { position: Vector3, rotation: Quaternion };
  lastSyncTime: number;

  // 客户端预测
  private predictedStates: EntityState[] = [];

  clientPredict(input: PlayerInput): void {
    // 本地立即执行，不等服务端确认
    const newState = this.simulate(input);
    this.predictedStates.push({
      sequence: input.sequence,
      state: newState,
    });
    this.applyState(newState);
  }

  // 服务端校正
  serverCorrect(ackSequence: number, serverState: EntityState): void {
    // 移除已确认的预测
    this.predictedStates = this.predictedStates.filter(
      s => s.sequence > ackSequence
    );
    // 重新模拟未确认的输入
    let state = serverState;
    for (const predicted of this.predictedStates) {
      state = this.simulateFrom(state, predicted.input);
      predicted.state = state;
    }
    this.applyState(state);
  }
}
```

### 插值与外推
```typescript
// 实体插值器
class EntityInterpolator {
  private stateBuffer: EntityState[] = []; // 保存最近N个状态
  private renderDelay = 100; // ms，比服务端延迟100ms渲染

  update(currentTime: number): EntityState {
    const renderTime = currentTime - this.renderDelay;

    // 从状态缓冲区找到插值点
    let older: EntityState, newer: EntityState;
    for (let i = 0; i < this.stateBuffer.length - 1; i++) {
      if (this.stateBuffer[i].time <= renderTime &&
          this.stateBuffer[i + 1].time >= renderTime) {
        older = this.stateBuffer[i];
        newer = this.stateBuffer[i + 1];
        break;
      }
    }

    if (!older || !newer) return this.stateBuffer[this.stateBuffer.length - 1];

    // 线性插值
    const t = (renderTime - older.time) / (newer.time - older.time);
    return EntityState.lerp(older, newer, t);
  }
}
```

## 帧同步(Lockstep)

### 核心流程
```
帧1: 收集所有玩家输入 → 广播
帧2: 所有客户端收到全部输入 → 确定性模拟
帧3: 重复...

关键：所有客户端必须产生完全相同的模拟结果
```

### 确定性要求
```typescript
// 帧同步确定性检查清单
const DeterministicChecklist = {
  // 浮点数：必须使用定点数或浮点一致性模式
  math: '使用FixedPoint(定点数)代替float',

  // 随机数：使用确定性随机数生成器
  random: '所有客户端使用相同seed的PRNG',

  // 排序：不稳定排序会导致不同结果
  sort: '使用稳定排序，相同元素保持原始顺序',

  // 遍历顺序：容器遍历顺序必须一致
  iteration: '使用数组代替HashMap，遍历顺序固定',

  // 初始化：所有客户端初始状态必须相同
  init: '场景配置由服务器下发，不依赖本地资源',

  // 物理引擎：必须使用确定性物理引擎
  physics: '使用确定性物理(如Unity DOTS/自研物理)',
};
```

## Matchmaking系统

### 匹配算法
```typescript
class MatchMaker {
  // ELO匹配
  findMatch(player: Player): Match {
    const searchRange = this.getSearchRange(player.waitTime);

    const candidates = this.pool.filter(p =>
      Math.abs(p.elo - player.elo) <= searchRange &&
      p.region === player.region &&
      !p.inMatch
    );

    // 按ELO差距排序，取最近的
    candidates.sort((a, b) =>
      Math.abs(a.elo - player.elo) - Math.abs(b.elo - player.elo)
    );

    return this.createMatch(player, candidates[0]);
  }

  // 搜索范围随等待时间扩大
  private getSearchRange(waitSeconds: number): number {
    return Math.min(100 + waitSeconds * 10, 500);
  }
}
```

### 房间系统
```typescript
class Room {
  id: string;
  maxPlayers: number;
  gameMode: string;
  host: Player;
  players: Player[] = [];

  // 房间状态机
  state: 'waiting' | 'loading' | 'playing' | 'finished';

  // 广播房间状态
  broadcastState(): void {
    const snapshot = {
      players: this.players.map(p => p.serialize()),
      state: this.state,
      timestamp: Date.now(),
    };
    this.players.forEach(p => p.send('room:update', snapshot));
  }
}
```

## 服务器架构

### 游戏服务器架构
```
                    ┌─────────────┐
                    │  Load Balancer │
                    └──────┬──────┘
                           │
              ┌────────────┼────────────┐
              │            │            │
        ┌─────┴────┐ ┌────┴─────┐ ┌────┴─────┐
        │ Game Svr 1│ │Game Svr 2│ │Game Svr 3│
        │ (战斗房间) │ │(战斗房间)│ │(战斗房间)│
        └─────┬────┘ └────┬─────┘ └────┬─────┘
              │            │            │
              └────────────┼────────────┘
                           │
                    ┌──────┴──────┐
                    │   Redis     │ (房间状态/排行榜)
                    └──────┬──────┘
                           │
                    ┌──────┴──────┐
                    │  MongoDB    │ (玩家数据)
                    └──────┬──────┘
                           │
                    ┌──────┴──────┐
                    │ Kafka/MQ    │ (事件/日志)
                    └─────────────┘
```

### 热更新机制
```typescript
// 服务器热更新
class HotReload {
  // Lua脚本热更新(不停服)
  reloadScript(moduleName: string): void {
    const newCode = this.fetchFromConfig(moduleName);
    const oldModule = require.cache[moduleName];
    // 保留旧状态，替换函数实现
    Object.assign(oldModule.exports, this.parseModule(newCode));
    this.broadcast('hotfix:applied', { module: moduleName });
  }

  // 配置热更新
  reloadConfig(configKey: string): void {
    const newConfig = this.fetchFromDB(configKey);
    // 原子替换，不影响进行中的游戏
    process.nextTick(() => {
      configs[configKey] = newConfig;
    });
  }
}
```

## 反作弊系统

### 检测机制
```typescript
class AntiCheat {
  // 速度检测
  checkSpeedHacking(player: Player): boolean {
    const maxSpeed = player.moveSpeed * 1.2; // 20%容差
    const actualSpeed = player.velocity.magnitude();
    return actualSpeed > maxSpeed;
  }

  // 位置校验(服务器权威)
  validatePosition(player: Player, reported: Vector3): boolean {
    const expected = player.lastKnownPos;
    const distance = reported.distance(expected);
    const maxDistance = player.moveSpeed * (Date.now() - player.lastSyncTime) / 1000;
    return distance <= maxDistance * 1.3;
  }

  // 操作频率检测
  checkInputFrequency(player: Player): boolean {
    const inputsPerSecond = player.recentInputs.length;
    return inputsPerSecond > MAX_INPUT_RATE; // 如 > 20次/秒
  }
}
```

### 数据安全
- 所有战斗计算在服务器执行
- 关键数值服务器校验
- 签名验证防篡改
- 通信加密(TLS)
- 敏感操作二次验证
