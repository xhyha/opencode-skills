---
name: game-art-tech
description: 游戏美术与技术美术技能，包含3D建模、动画、Shader编程、VFX特效和美术管线优化
---

# 游戏美术与技术美术技能

## 3D建模规范

### 模型规范
| 类型 | 面数上限 | 贴图尺寸 | 骨骼数 |
|------|----------|----------|--------|
| 主角 | 15k-30k | 2048x2048 | 80-120 |
| NPC | 5k-10k | 1024x1024 | 40-60 |
| 怪物 | 3k-8k | 1024x1024 | 30-50 |
| 场景道具 | 500-3k | 512x512 | 0-10 |
| 建筑 | 2k-8k | 1024x1024 | 0 |

### LOD规范
```
LOD0: 原始模型 (100%面数) - 近距离<15m
LOD1: 简化模型 (50%面数)  - 中距离15-30m
LOD2: 低模     (25%面数)  - 远距离30-60m
LOD3: Billboard           - 极远距离>60m
```

## PBR材质系统

### PBR工作流
```typescript
// PBR材质参数
interface PBRMaterial {
  albedo: Texture;      // 基础色，无光照信息
  normal: Texture;      // 法线贴图
  metallic: Texture;    // 金属度 (0=非金属, 1=金属)
  roughness: Texture;   // 粗糙度 (0=光滑镜面, 1=粗糙漫反射)
  ao: Texture;          // 环境光遮蔽
  emission: Texture;    // 自发光
  
  // 参数校验规则
  // albedo RGB各通道值范围: 50-240 (sRGB)
  // metallic: 大多数材质为0或1，金属才为1
  // roughness: 避免纯0(除镜子外)
  // 非金属的specular = 0.04 (4%菲涅尔反射)
}
```

### 材质分类
| 材质类型 | metallic | roughness | 备注 |
|----------|----------|-----------|------|
| 塑料 | 0 | 0.3-0.7 | 低金属度 |
| 木材 | 0 | 0.7-0.9 | 高粗糙度 |
| 金属(抛光) | 1 | 0.1-0.3 | 高金属度低粗糙度 |
| 金属(生锈) | 0.5-0.8 | 0.6-0.9 | 混合态 |
| 皮肤 | 0 | 0.5-0.7 | 需SSS次表面散射 |
| 布料 | 0 | 0.8-1.0 | 需各向异性 |

## Shader编程

### 基础Shader结构(GLSL)
```glsl
// Vertex Shader
#version 330 core
layout (location = 0) in vec3 aPos;
layout (location = 1) in vec3 aNormal;
layout (location = 2) in vec2 aTexCoord;

out vec3 WorldPos;
out vec3 Normal;
out vec2 TexCoord;

uniform mat4 model;
uniform mat4 view;
uniform mat4 projection;

void main() {
    WorldPos = vec3(model * vec4(aPos, 1.0));
    Normal = mat3(transpose(inverse(model))) * aNormal;
    TexCoord = aTexCoord;
    gl_Position = projection * view * vec4(WorldPos, 1.0);
}

// Fragment Shader - PBR光照
uniform vec3 cameraPos;
uniform vec3 lightPos;
uniform vec3 lightColor;
uniform sampler2D albedoMap;
uniform sampler2D normalMap;
uniform sampler2D metallicMap;
uniform sampler2D roughnessMap;

const float PI = 3.14159265359;

// Cook-Torrance BRDF
float DistributionGGX(vec3 N, vec3 H, float roughness) {
    float a = roughness * roughness;
    float a2 = a * a;
    float NdotH = max(dot(N, H), 0.0);
    float denom = (NdotH * NdotH * (a2 - 1.0) + 1.0);
    return a2 / (PI * denom * denom);
}

float GeometrySmith(vec3 N, vec3 V, vec3 L, float roughness) {
    float r = roughness + 1.0;
    float k = (r * r) / 8.0;
    float NdotV = max(dot(N, V), 0.0);
    float NdotL = max(dot(N, L), 0.0);
    float ggx1 = NdotV / (NdotV * (1.0 - k) + k);
    float ggx2 = NdotL / (NdotL * (1.0 - k) + k);
    return ggx1 * ggx2;
}

vec3 fresnelSchlick(float cosTheta, vec3 F0) {
    return F0 + (1.0 - F0) * pow(clamp(1.0 - cosTheta, 0.0, 1.0), 5.0);
}
```

### 常用游戏Shader效果
```glsl
// 描边Shader
void main() {
    vec3 outlineOffset = aNormal * outlineWidth;
    gl_Position = projection * view * model * vec4(aPos + outlineOffset, 1.0);
    fragColor = outlineColor;
}

// 溶解效果
uniform float dissolveThreshold;
void main() {
    float noise = texture(dissolveMap, TexCoord).r;
    if (noise < dissolveThreshold) discard;
    float edge = smoothstep(dissolveThreshold, dissolveThreshold + edgeWidth, noise);
    fragColor = mix(edgeColor, baseColor, edge);
}

// 水面Shader核心
void main() {
    vec2 uv1 = TexCoord * scale + time * speed1;
    vec2 uv2 = TexCoord * scale * 1.5 + time * speed2;
    vec3 normal1 = texture(normalMap1, uv1).rgb;
    vec3 normal2 = texture(normalMap2, uv2).rgb;
    vec3 waterNormal = normalize(normal1 + normal2);
    // 菲涅尔反射 + 折射
    float fresnel = pow(1.0 - max(dot(V, waterNormal), 0.0), 5.0);
    fragColor = mix(refractionColor, reflectionColor, fresnel);
}
```

## VFX特效系统

### 粒子系统
```typescript
// 粒子发射器配置
interface ParticleEmitter {
  // 发射
  rate: number;                    // 每秒发射数
  burst: { count: number, probability: number }[];

  // 生命周期
  lifetime: { min: number, max: number };

  // 初始属性
  startSpeed: { min: number, max: number };
  startSize: { min: number, max: number };
  startColor: Gradient;
  startRotation: { min: number, max: number };

  // 随时间变化
  sizeOverLifetime: AnimationCurve;
  colorOverLifetime: Gradient;
  speedOverLifetime: AnimationCurve;

  // 力场
  gravity: number;
  wind: Vector3;
  turbulence: number;
  attractor: { position: Vector3, strength: number }[];

  // 渲染
  renderMode: 'billboard' | 'stretched' | 'mesh';
  material: Material;
  maxParticles: number;
}
```

### 特效性能预算
| 特效类型 | 粒子上限 | DrawCall | 贴图尺寸 |
|----------|----------|----------|----------|
| 角色技能 | 200 | 3 | 512x512 |
| 环境特效 | 100 | 2 | 256x256 |
| UI特效 | 50 | 1 | 256x256 |
| 全屏特效 | 300 | 5 | 1024x1024 |

## 动画系统

### 动画状态机
```typescript
// 动画状态机
class AnimatorController {
  // 状态
  states: {
    idle:    { clip: 'idle',    speed: 1.0 },
    walk:    { clip: 'walk',    speed: 1.0 },
    run:     { clip: 'run',     speed: 1.2 },
    attack1: { clip: 'atk1',    speed: 1.5 },
    attack2: { clip: 'atk2',    speed: 1.5 },
  };

  // 过渡规则
  transitions: [
    { from: 'idle',    to: 'walk',    condition: 'speed > 0.1', duration: 0.2 },
    { from: 'walk',    to: 'run',     condition: 'speed > 5.0', duration: 0.15 },
    { from: 'attack1', to: 'attack2', condition: 'combo_input', duration: 0.1, hasExitTime: true },
  ];

  // 混合树(Blend Tree)
  blendTree: {
    type: '2D',
    parameters: ['speedX', 'speedY'],
    motions: [
      { clip: 'idle',  position: [0, 0] },
      { clip: 'walkF', position: [0, 1] },
      { clip: 'walkB', position: [0, -1] },
      { clip: 'walkL', position: [-1, 0] },
      { clip: 'walkR', position: [1, 0] },
    ],
  };
}
```

### IK(反向动力学)
```typescript
// 常用IK应用
const IKSetup = {
  // 脚部IK - 踩在斜面上脚对齐地面
  footIK: {
    raycastDown: true,
    adjustPelvis: true,
    maxStepHeight: 0.5,
  },
  // 手部IK - 精确抓取物品
  handIK: {
    target: 'weaponHandle',
    transitionSpeed: 10,
  },
  // 头部IK - 看向目标
  headIK: {
    lookAtTarget: 'cameraOrEnemy',
    weight: 0.8,
    bodyWeight: 0.3,
    headWeight: 0.7,
  },
};
```

## 美术管线优化

### 贴图压缩格式
| 平台 | 格式 | 压缩比 | 备注 |
|------|------|--------|------|
| iOS | ASTC 4x4 | 8:1 | PVR已弃用 |
| Android | ETC2 | 4:1 | 广泛兼容 |
| PC | BC7 | 4:1 | 高质量 |
| 法线贴图 | BC5 | 2:1 | 双通道 |

### 资源优化清单
- [ ] 贴图使用2的幂次方尺寸
- [ ] 静态物体标记为Static(启用 batching)
  
- [ ] 合理使用LOD(至少3级)
- [ ] 材质合并(同材质物体合批)
- [ ] 图集打包(减少DrawCall)
- [ ] 去除不可见面
- [ ] 模型减面优化拓扑
