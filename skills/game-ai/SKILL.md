---
name: game-ai
description: 游戏AI编程技能，包含行为树、状态机、寻路算法、群体AI和动态难度调节
---

# 游戏AI编程技能

## AI架构模式

### 行为树(Behavior Tree)
```typescript
// 行为树节点类型
abstract class BTNode {
  abstract execute(context: AIContext): NodeStatus;
}

enum NodeStatus { Success, Failure, Running }

// 组合节点
class Selector extends BTNode {
  constructor(private children: BTNode[]) { super(); }
  execute(ctx: AIContext): NodeStatus {
    for (const child of this.children) {
      const status = child.execute(ctx);
      if (status !== NodeStatus.Failure) return status;
    }
    return NodeStatus.Failure;
  }
}

class Sequence extends BTNode {
  constructor(private children: BTNode[]) { super(); }
  execute(ctx: AIContext): NodeStatus {
    for (const child of this.children) {
      const status = child.execute(ctx);
      if (status !== NodeStatus.Success) return status;
    }
    return NodeStatus.Success;
  }
}

// 装饰器节点
class Inverter extends BTNode {
  constructor(private child: BTNode) { super(); }
  execute(ctx: AIContext): NodeStatus {
    const status = this.child.execute(ctx);
    if (status === NodeStatus.Success) return NodeStatus.Failure;
    if (status === NodeStatus.Failure) return NodeStatus.Success;
    return NodeStatus.Running;
  }
}
```

### 有限状态机(FSM)
```typescript
// 通用状态机
class StateMachine<T> {
  private currentState: State<T>;
  private transitions: Map<string, Transition<T>[]> = new Map();

  update(entity: T, dt: number): void {
    this.currentState.onUpdate(entity, dt);
    this.checkTransitions(entity);
  }

  private checkTransitions(entity: T): void {
    const transitions = this.transitions.get(this.currentState.name) || [];
    for (const t of transitions) {
      if (t.condition(entity)) {
        this.currentState.onExit(entity);
        this.currentState = t.targetState;
        this.currentState.onEnter(entity);
        return;
      }
    }
  }
}

// 游戏角色状态示例
const PlayerFSM = {
  idle: { transitions: ['move', 'attack', 'stunned'] },
  move:  { transitions: ['idle', 'attack', 'dodge'] },
  attack: { transitions: ['idle', 'combo', 'stunned'] },
  stunned: { transitions: ['idle'] },
};
```

### GOAP(Goal-Oriented Action Planning)
```typescript
// 目标导向行动规划
interface GOAPAction {
  name: string;
  cost: number;
  preconditions: Map<string, boolean>;
  effects: Map<string, boolean>;
  execute(agent: AIAgent): void;
}

class GOAPPlanner {
  plan(agent: AIAgent, goal: Map<string, boolean>): GOAPAction[] {
    // A*搜索从当前状态到目标状态的最优行动序列
    // 每个节点 = 世界状态
    // 每条边 = 一个行动
    // 边权重 = 行动cost
    return this.aStarSearch(agent.getWorldState(), goal);
  }
}
```

## 寻路算法

### A*寻路
```typescript
// A*寻路实现
class AStar {
  findPath(start: Point, goal: Point, grid: Grid): Point[] {
    const openSet = new PriorityQueue<Node>();
    const closedSet = new Set<string>();

    openSet.enqueue({ pos: start, g: 0, f: this.heuristic(start, goal) });

    while (!openSet.isEmpty()) {
      const current = openSet.dequeue();
      if (current.pos.equals(goal)) return this.reconstructPath(current);

      closedSet.add(current.pos.key());

      for (const neighbor of grid.getNeighbors(current.pos)) {
        if (closedSet.has(neighbor.key())) continue;
        const g = current.g + this.moveCost(current.pos, neighbor);
        const f = g + this.heuristic(neighbor, goal);
        openSet.enqueue({ pos: neighbor, g, f, parent: current });
      }
    }
    return []; // 无路径
  }

  // 启发函数
  private heuristic(a: Point, b: Point): number {
    // 曼哈顿距离(4方向) 或 对角线距离(8方向)
    return Math.abs(a.x - b.x) + Math.abs(a.y - b.y);
  }
}
```

### NavMesh导航
```typescript
// NavMesh导航网格
class NavMesh {
  private mesh: Triangle[];

  findPath(start: Vector3, goal: Vector3): Vector3[] {
    // 1. 找到起点和终点所在三角形
    const startTri = this.findTriangle(start);
    const goalTri = this.findTriangle(goal);

    // 2. A*在三角形之间搜索
    const triPath = this.aStarTriangles(startTri, goalTri);

    // 3. 漏斗算法平滑路径(Funnel Algorithm)
    return this.funnelSmooth(start, goal, triPath);
  }
}
```

### 群体寻路(Flow Field)
```typescript
// 流场寻路 - 适用于大量单位同时移动
class FlowField {
  private field: Direction[][];

  calculate(goal: Point, grid: Grid): void {
    // BFS从目标向外的cost传播
    const queue = [goal];
    this.costField[goal.y][goal.x] = 0;

    while (queue.length > 0) {
      const current = queue.shift();
      for (const neighbor of grid.getNeighbors(current)) {
        const newCost = this.costField[current.y][current.x] + 1;
        if (newCost < this.costField[neighbor.y][neighbor.x]) {
          this.costField[neighbor.y][neighbor.x] = newCost;
          queue.push(neighbor);
        }
      }
    }

    // 生成方向场 - 每个格子指向cost最低的邻居
    for (let y = 0; y < grid.height; y++) {
      for (let x = 0; x < grid.width; x++) {
        this.field[y][x] = this.getLowestCostNeighbor(x, y);
      }
    }
  }
}
```

## 战斗AI

### 敌人AI决策
```typescript
// 敌人AI控制器
class EnemyAI {
  private behaviorTree: BTNode;

  constructor() {
    this.behaviorTree = new Selector([
      // 高优先级：逃跑
      new Sequence([
        new Condition(ctx => ctx.hp < ctx.maxHp * 0.2),
        new Action(ctx => ctx.flee()),
      ]),
      // 中优先级：攻击
      new Sequence([
        new Condition(ctx => ctx.canSeePlayer()),
        new Selector([
          new Sequence([
            new Condition(ctx => ctx.inMeleeRange()),
            new Action(ctx => ctx.meleeAttack()),
          ]),
          new Sequence([
            new Condition(ctx => ctx.hasRangedAbility()),
            new Action(ctx => ctx.rangedAttack()),
          ]),
        ]),
      ]),
      // 低优先级：巡逻
      new Action(ctx => ctx.patrol()),
    ]);
  }
}
```

### 动态难度调节(DDA)
```typescript
// 动态难度系统
class DynamicDifficulty {
  private playerPerformance: number = 0.5; // 0=很差, 1=极好

  update(events: GameEvent[]): void {
    // 根据玩家表现调整
    const recentDeaths = events.filter(e => e.type === 'death').length;
    const recentKills = events.filter(e => e.type === 'kill').length;
    const accuracy = events.filter(e => e.type === 'hit').length /
                     Math.max(1, events.filter(e => e.type === 'shot').length);

    // 滑动平均
    this.playerPerformance = this.playerPerformance * 0.8 +
      (recentKills / (recentDeaths + 1) * 0.5 + accuracy * 0.5) * 0.2;

    this.adjustDifficulty();
  }

  private adjustDifficulty(): void {
    // performance > 0.7 → 提升难度
    // performance < 0.3 → 降低难度
    const enemyHpMultiplier = 0.8 + this.playerPerformance * 0.4;
    const enemyDamageMultiplier = 0.7 + this.playerPerformance * 0.6;
    const spawnRateMultiplier = 0.8 + this.playerPerformance * 0.4;
  }
}
```

## 群体AI

### Boids算法(鸟群/鱼群)
```typescript
class Boid {
  // 三大规则
  separation(neighbors: Boid[]): Vector3 {
    // 分离：远离过近的邻居
    let steer = Vector3.zero;
    for (const other of neighbors) {
      const d = this.position.distance(other.position);
      if (d < separationRadius) {
        steer.add(Vector3.sub(this.position, other.position).normalize().divide(d));
      }
    }
    return steer;
  }

  alignment(neighbors: Boid[]): Vector3 {
    // 对齐：匹配邻居的平均方向
    const avgVelocity = Vector3.zero;
    for (const other of neighbors) avgVelocity.add(other.velocity);
    return avgVelocity.divide(neighbors.length).sub(this.velocity);
  }

  cohesion(neighbors: Boid[]): Vector3 {
    // 聚合：朝邻居中心移动
    const center = Vector3.zero;
    for (const other of neighbors) center.add(other.position);
    return center.divide(neighbors.length).sub(this.position);
  }
}
```

## AI优化策略

### LOD AI(Level of Detail)
| 玩家距离 | AI等级 | 行为 |
|----------|--------|------|
| 近(<20m) | 完整 | 完整行为树，精确寻路 |
| 中(20-50m) | 简化 | 简化FSM，直线移动 |
| 远(>50m) | 模拟 | 仅播放动画，定时模拟位置 |

### AI性能优化
- 帧分片：每帧只更新1/N的AI
- 空间分区：只处理视野范围内的AI
- 预计算：静态路径预烘焙
- 协程/异步：复杂决策分散到多帧
