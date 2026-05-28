---
name: game-audio
description: 游戏音频设计技能，包含自适应音乐、3D空间音频、音效设计、音频中间件和音频优化
---

# 游戏音频设计技能

## 自适应音乐系统

### 横向/纵向编排
```typescript
// 自适应音乐系统
class AdaptiveMusic {
  // 横向切换(不同状态切换不同曲目)
  states: {
    explore:  { clip: 'explore_theme',  bpm: 80,  intensity: 0.3 },
    combat:   { clip: 'combat_theme',   bpm: 140, intensity: 0.8 },
    boss:     { clip: 'boss_theme',     bpm: 160, intensity: 1.0 },
    victory:  { clip: 'victory_fanfare', bpm: 120, intensity: 0.6 },
    defeat:   { clip: 'defeat_somber',  bpm: 60,  intensity: 0.2 },
  };

  // 纵向叠加(同一曲调上增减乐器层)
  layers: {
    percussion: { volume: 0.0, fadeTime: 2.0 },
    bass:       { volume: 0.3, fadeTime: 1.5 },
    melody:     { volume: 0.5, fadeTime: 2.0 },
    harmony:    { volume: 0.0, fadeTime: 2.5 },
    stings:     { volume: 0.0, fadeTime: 0.1 },  // 突发音效
  };

  // 根据游戏状态动态调整
  updateGameState(state: GameState): void {
    const threat = state.nearbyEnemies / state.maxEnemies;
    // threat > 0.7 → 加入全部层(combat模式)
    // threat > 0.3 → 加入percussion + bass
    // threat < 0.1 → 只有bass + melody(explore模式)
  }
}
```

### 音乐过渡技术
```typescript
// 平滑过渡
const TransitionTechniques = {
  // 同步点过渡(Bar-line transition)
  syncPoint: '等待当前小节结束再切换',

  // 淡入淡出(Crossfade)
  crossfade: 'A曲淡出的同时B曲淡入,重叠时间2-4秒',

  // 即时切换(Hard cut)
  hardCut: '适用于突发情况(被发现/战斗开始)',

  // 翻页器(Stinger)
  stinger: '在过渡点播放一个短音效,掩盖切换痕迹',
};
```

## 3D空间音频

### 空间音频模型
```typescript
// 3D音频参数
interface SpatialAudio {
  // 衰减模型
  attenuation: {
    model: 'linear' | 'logarithmic' | 'inverse',
    minDistance: 1.0,    // 最小距离(无衰减)
    maxDistance: 50.0,   // 最大距离(完全静音)
    rolloffFactor: 1.0,  // 衰减速度
  };

  // 多普勒效应
  doppler: {
    enabled: true,
    factor: 1.0,     // 0=关闭, 1=真实, >1=夸张
    listenerSpeed: Vector3,
    sourceSpeed: Vector3,
  };

  // 散播(Spread)
  spread: {
    // 0° = 点声源(精确定位)
    // 180° = 全向(环境声)
    angle: 0,         // 近距离时
    maxAngle: 90,     // 远距离时
  };

  // HRTF(头部相关传递函数)
  hrtf: {
    enabled: true,    // 3D耳机体验
    // 通过模拟耳廓/头部对不同方向声音的滤波实现3D定位
  };
}
```

### 环境音频
```typescript
// 环境音频区域
class AudioZone {
  // Reverb混响区域
  reverbZones: {
    outdoor:   { decay: 1.0s,  wetMix: 0.1 },  // 室外
    room:      { decay: 0.5s,  wetMix: 0.2 },   // 房间
    cave:      { decay: 2.0s,  wetMix: 0.5 },   // 洞穴
    cathedral: { decay: 4.0s,  wetMix: 0.6 },   // 大教堂
  };

  // 遮挡(Occlusion)
  occlusion: {
    // 声音被墙壁遮挡时低频通过,高频被阻
    raycast: true,
    layers: [
      { thickness: 0, filter: 'none' },        // 无遮挡
      { thickness: 1, filter: 'lowpass 4000Hz' },  // 墙壁
      { thickness: 2, filter: 'lowpass 1000Hz' },  // 厚墙
    ],
  };

  // 封闭感(Obstruction vs Occlusion)
  // Obstruction: 直达声被挡, 反射声正常
  // Occlusion: 直达声和反射声都被挡
}
```

## 音效设计

### 音效分类
| 类型 | 说明 | 数量预估 | 优先级 |
|------|------|----------|--------|
| 角色动作 | 走、跑、跳、攻击、受伤 | 200+ | P0 |
| UI音效 | 按钮点击、弹窗、获得奖励 | 100+ | P0 |
| 环境音 | 风声、水声、森林、城镇 | 50+ | P1 |
| 武器音效 | 不同武器挥动/命中/暴击 | 100+ | P0 |
| 怪物音效 | 叫声、攻击、死亡 | 150+ | P1 |
| 技能音效 | 施法、命中、特效 | 200+ | P0 |

### 音效变体系统
```typescript
// 避免重复感
class SoundVariation {
  // 随机变体
  playFootstep(surface: string): void {
    const variants = this.footstepBank[surface]; // 如 'concrete' 有5个变体
    const clip = variants[Math.floor(Math.random() * variants.length)];
    // 随机微调 pitch(0.9-1.1) 和 volume(0.8-1.0)
    clip.pitch = 0.9 + Math.random() * 0.2;
    clip.volume = 0.8 + Math.random() * 0.2;
    clip.play();
  }

  // 规则：
  // 同一音效连续播放间隔 > 100ms
  // 同一变体不连续播放两次
  // 距离上次播放越近，pitch偏移越大
}
```

## 音频中间件

### Wwise集成
```typescript
// Wwise事件调用
class WwiseIntegration {
  // 事件
  postEvent(eventName: string, gameObject: GameObject): void {
    // AK.SoundEngine.PostEvent(eventName, gameObjectID)
  }

  // RTPC(实时参数控制)
  setRTPC(parameter: string, value: number, gameObject: GameObject): void {
    // AK.SoundEngine.SetRTPCValue(parameter, value, gameObjectID)
    // 例：设置角色速度影响脚步声节奏
    // 例：设置血量影响心跳声频率
  }

  // State(全局状态)
  setState(group: string, state: string): void {
    // AK.SoundEngine.SetState(group, state)
    // 例：setState('Music', 'Combat')
    // 例：setState('Environment', 'Cave')
  }

  // Switch(对象级状态)
  setSwitch(group: string, value: string, gameObject: GameObject): void {
    // 例：setSwitch('Surface', 'Grass', player) → 草地脚步声
    // 例：setSwitch('Weapon', 'Sword', player) → 剑攻击音效
  }
}
```

### FMOD集成
```typescript
// FMOD事件调用
class FMODIntegration {
  // 创建事件实例
  createInstance(eventPath: string): FMODStudioEventInstance {
    // 如 "event:/Character/Footsteps/Grass"
  }

  // 参数控制
  setParameter(instance: FMODStudioEventInstance, name: string, value: number): void {
    // 例：setParameter(instance, 'Speed', 5.0) → 控制脚步节奏
    // 例：setParameter(instance, 'Intensity', 0.8) → 控制战斗音乐强度
  }

  // 快照(Snapshot) - 全局音频效果
  startSnapshot(snapshotPath: string): void {
    // 如 "snapshot:/LowHealth" → 降低音量,加滤波
  }
}
```

## 音频优化

### 内存预算
| 平台 | 音频内存上限 | 说明 |
|------|-------------|------|
| 移动端 | 30-50MB | 需严格控制 |
| PC | 100-200MB | 相对宽松 |
| 主机 | 50-100MB | 需预留系统音频 |

### 优化策略
```typescript
const AudioOptimization = {
  // 1. 压缩格式
  formats: {
    // 移动端: Vorbis/MP3(高压缩比)
    // PC: Vorbis/PCM(高质量)
    // 短音效: PCM(无解码开销)
    // 长音乐: Vorbis/MP3(省空间)
  },

  // 2. 加载策略
  loading: {
    // 预加载: UI音效、常驻环境音
    // 按需加载: 关卡音效、Boss音乐
    // 流式加载: 背景音乐(不占内存)
  },

  // 3. 语音池
  voicePool: {
    // 总Voice数: 移动端≤32, PC≤64
    // 虚拟Voice: 超出限制时按优先级虚拟化
    // 抢占策略: 优先级低的被优先级高的抢占
  },

  // 4. 采样率
  sampleRate: {
    // 音效: 44100Hz
    // 语音: 22050Hz(可接受)
    // 环境: 22050Hz
  },
};
```

### 音频性能检查清单
- [ ] 同时播放音效不超过Voice上限
- [ ] 音乐使用流式加载
- [ ] 远距离音效自动停止
- [ ] 3D音效使用距离衰减
- [ ] 大文件使用压缩格式
- [ ] 音效变体避免重复感
- [ ] 全局混音器层级合理(Master→Bus→SubBus)
