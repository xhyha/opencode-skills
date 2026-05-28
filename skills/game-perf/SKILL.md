---
name: game-perf
description: 游戏性能优化技能，包含CPU/GPU分析、帧时间预算、内存管理、渲染优化和多平台调优
---

# 游戏性能优化技能

## 帧时间预算

### 60FPS预算分配
```
一帧总预算: 16.67ms (1000ms / 60fps)

推荐分配:
┌─────────────────────────────────────────┐
│ 游戏逻辑(CPU)    │  4-5ms  │  ~30%     │
│ 物理模拟(CPU)     │  2-3ms  │  ~15%     │
│ AI更新(CPU)       │  1-2ms  │  ~10%     │
│ 动画更新(CPU)     │  1-2ms  │  ~10%     │
│ 渲染提交(CPU→GPU) │  2-3ms  │  ~15%     │
│ GPU渲染           │  4-5ms  │  ~30%     │
│ 预留buffer        │  1-2ms  │  ~10%     │
└─────────────────────────────────────────┘
```

### 帧率标准
| 平台 | 目标帧率 | 最低帧率 | 说明 |
|------|----------|----------|------|
| 移动端 | 30/60fps | 25fps | 考虑发热降频 |
| PC | 60fps | 30fps | 支持高刷新率 |
| 主机 | 60fps | 30fps | 画质/帧率模式 |
| VR | 90/120fps | 90fps | 低于90fps会眩晕 |

## CPU性能优化

### Profiling工具
```typescript
// 自定义性能探针
class PerformanceProfiler {
  private markers: Map<string, { start: number, total: number, calls: number }> = new Map();

  beginSample(name: string): void {
    this.markers.set(name, { start: performance.now(), total: 0, calls: 0 });
  }

  endSample(name: string): void {
    const marker = this.markers.get(name);
    if (marker) {
      marker.total += performance.now() - marker.start;
      marker.calls++;
    }
  }

  getReport(): PerformanceReport {
    return Array.from(this.markers.entries()).map(([name, data]) => ({
      name,
      totalTime: data.total,
      avgTime: data.total / data.calls,
      calls: data.calls,
    })).sort((a, b) => b.totalTime - a.totalTime);
  }
}
```

### 帧分片(Time-slicing)
```typescript
// 将耗时操作分摊到多帧执行
class TimeSlicedTask {
  private budgetMs = 2; // 每帧最多2ms

  execute(tasks: Task[]): void {
    const startTime = performance.now();

    while (tasks.length > 0) {
      const task = tasks.shift();
      task.execute();

      // 超出预算，剩余推到下一帧
      if (performance.now() - startTime > this.budgetMs) {
        break;
      }
    }
  }
}

// 适用场景：
// - AI更新(每帧只更新1/N的AI)
// - 寻路计算
// - 资源加载
// - 大规模物理模拟
```

### 多线程/Job System
```typescript
// Job System任务调度
class JobSystem {
  // 主线程: 游戏逻辑、渲染提交
  // Worker线程: 物理、AI、动画、资源加载

  scheduleJobs(frameJobs: Job[]): void {
    // 按依赖关系排序
    const sorted = this.topologicalSort(frameJobs);

    // 分配到Worker
    for (const job of sorted) {
      if (job.canRunOnWorker) {
        this.workerPool.enqueue(job);
      } else {
        this.mainThreadJobs.push(job);
      }
    }
  }
}

// 典型多线程分配
const ThreadAssignment = {
  main:     ['游戏逻辑', '输入处理', '渲染提交'],
  worker1:  ['物理模拟'],
  worker2:  ['AI决策', '寻路'],
  worker3:  ['动画更新', '粒子更新'],
  worker4:  ['资源加载', '网络IO'],
  gpu:      ['顶点处理', '光栅化', '像素着色'],
};
```

## GPU渲染优化

### DrawCall优化
```typescript
// 减少DrawCall策略
const DrawCallOptimization = {
  // 1. 静态合批(Static Batching)
  staticBatch: {
    rule: '标记为Static的物体自动合批',
    limit: '顶点数<300的物体',
    gain: '100个物体 → 1个DrawCall',
  },

  // 2. 动态合批(Dynamic Batching)
  dynamicBatch: {
    rule: '相同材质的小物体自动合批',
    limit: '顶点数<300, 使用相同材质',
    note: 'CPU开销可能抵消GPU收益',
  },

  // 3. GPU Instancing
  instancing: {
    rule: '相同Mesh+材质的物体一次绘制',
    useCase: '树木、草、石头、弹幕',
    limit: '参数通过Instanced Buffer传递',
    gain: '10000棵树 → 1个DrawCall',
  },

  // 4. SRP Batcher (Unity URP/HDRP)
  srpBatch: {
    rule: '相同Shader Variant自动合批',
    gain: '不要求相同材质，只要求相同Shader',
  },
};
```

### 裁剪优化
```typescript
// 多级裁剪
const CullingPipeline = {
  // 1. 距离裁剪(Distance Culling)
  distance: {
    rule: '超出可视距离的物体不渲染',
    threshold: { near: 50, far: 200, veryFar: 500 },
  },

  // 2. 视锥裁剪(Frustum Culling)
  frustum: {
    rule: '摄像机视野外的物体不渲染',
    note: '引擎自动处理，一般不需要手动',
  },

  // 3. 遮挡裁剪(Occlusion Culling)
  occlusion: {
    rule: '被其他物体完全遮挡的不渲染',
    baking: '烘焙遮挡数据(静态场景)',
    runtime: '运行时计算(动态物体)',
  },

  // 4. 层级裁剪
  layered: {
    rule: '不同质量设置不同裁剪距离',
    low:    { shadow: 20, particle: 30, detail: 50 },
    medium: { shadow: 40, particle: 60, detail: 100 },
    high:   { shadow: 80, particle: 120, detail: 200 },
  },
};
```

### Shader优化
```typescript
const ShaderOptimization = {
  // 减少指令数
  instructions: {
    '避免分支': '使用step/mix/lerp代替if/else',
    '减少纹理采样': '合并贴图(金属度+粗糙度=一张图)',
    '简化数学': '用近似函数代替精确计算',
    '减少varying': 'Vertex→Fragment传递的数据最小化',
  },

  // 移动端Shader指南
  mobile: {
    '避免discard': '导致Early-Z失效',
    '半精度浮点': '使用mediump代替highp',
    '减少ALU': '移动端GPU ALU有限',
    '简化光照': '移动端最多2盏实时光',
  },
};
```

## 内存优化

### 内存预算
```
移动端总内存预算: ~300-400MB

分配方案:
┌──────────────────────────────────────┐
│ 渲染资源  │  150-200MB │ 贴图、Mesh   │
│ 音频资源  │   30-50MB  │ 音效、音乐    │
│ 代码&逻辑 │   30-50MB  │ 脚本、配置    │
│ 系统预留  │   50-80MB  │ OS、驱动     │
│ Buffer   │   20-30MB  │ 渲染缓冲     │
└──────────────────────────────────────┘
```

### 内存优化策略
```typescript
const MemoryOptimization = {
  // 1. 资源压缩
  texture: {
    format: 'ASTC(移动) / BC(PC)',
    size: '贴图尺寸不超过必要大小',
    mipmap: '远距离物体用低级mipmap',
  },

  // 2. 资源生命周期
  lifecycle: {
    preload: '场景加载时预加载必要资源',
    stream: '大文件流式加载(不占常驻内存)',
    unload: '离开场景后卸载不用的资源',
    pool: '复用对象，避免频繁new/GC',
  },

  // 3. GC优化
  gc: {
    avoid: '避免在热路径中分配新对象',
    pool: '使用对象池复用',
    struct: '值类型代替引用类型(减少堆分配)',
    span: '使用Span<T>避免数组拷贝',
  },
};
```

### 内存泄漏检测
```typescript
// 资源引用追踪
class ResourceTracker {
  private resources: Map<string, { ref: any, stackTrace: string, time: number }> = new Map();

  track(name: string, resource: any): void {
    this.resources.set(name, {
      ref: resource,
      stackTrace: new Error().stack,
      time: Date.now(),
    });
  }

  // 检测泄漏：资源被追踪但无引用
  detectLeaks(): LeakReport[] {
    const leaks: LeakReport[] = [];
    for (const [name, info] of this.resources) {
      if (info.ref === null || info.ref === undefined) {
        leaks.push({
          name,
          stackTrace: info.stackTrace,
          heldTime: Date.now() - info.time,
        });
      }
    }
    return leaks.sort((a, b) => b.heldTime - a.heldTime);
  }
}
```

## 多平台调优

### 移动端专项
| 优化项 | 策略 |
|--------|------|
| 发热 | 帧率动态调整(60→30)，复杂场景降低特效 |
| 电量 | 减少GPS/网络请求频率，后台时暂停 |
| 存储包体 | 资源按需下载，首包<200MB |
| 兼容性 | OpenGL ES 3.0基准，降级方案 |

### 性能测试流程
```markdown
1. 建立基线(Baseline)
   - 目标设备上记录各场景的平均帧率、帧时间
   - 记录内存峰值、DrawCall数

2. Profiling
   - CPU: 函数耗时排序，找热点
   - GPU: 渲染统计，DrawCall/三角形数/显存
   - 内存: 快照对比，找增长

3. 优化迭代
   - 每次只改一个变量
   - 对比优化前后数据
   - 确认无副作用

4. 回归测试
   - 全场景性能测试
   - 多设备验证
   - 长时间运行稳定性(4小时monkey test)
```

### 性能监控看板
```typescript
// 运行时性能监控
class PerfMonitor {
  // 实时FPS显示
  fps: number;
  frameTime: number;      // ms
  cpuTime: number;         // ms
  gpuTime: number;         // ms

  // 内存
  totalMemory: number;     // MB
  gcCount: number;
  gcTotalTime: number;     // ms

  // 渲染
  drawCalls: number;
  triangles: number;
  setPassCalls: number;    // 材质切换次数

  // 告警阈值
  warnings: {
    fpsBelow30: true,
    frameTimeAbove20ms: true,
    memoryAbove300MB: true,
    drawCallsAbove500: true,
    gcPerFrame: true,
  }
}
```
