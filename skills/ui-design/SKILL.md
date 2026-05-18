---
name: ui-design
description: UI设计师技能，包含视觉设计、布局规范、动效设计、设计系统和多平台适配
---

# UI设计师技能

## 视觉设计原则

### 视觉层级
1. **首要元素**：最大、最亮、对比最强
2. **次要元素**：辅助信息
3. **背景元素**：营造氛围

### 色彩理论
```typescript
// 色彩系统示例
const ColorSystem = {
  // 主色：品牌代表
  primary: '#FF6B35',
  
  // 辅助色：功能区分
  secondary: '#4ECDC4',
  
  // 中性色：文字、背景
  neutral: {
    900: '#1A1A2E',
    700: '#16213E',
    500: '#0F3460',
    300: '#E94560',
  },
  
  // 功能色：状态指示
  success: '#2ECC71',
  warning: '#F39C12',
  error: '#E74C3C',
  info: '#3498DB',
}

// 渐变
gradient: 'linear-gradient(135deg, #FF6B35, #F39C12)'
```

### 字体系统
```typescript
const Typography = {
  // 游戏标题
  display: {
    fontFamily: 'GameFont',
    size: 48,
    weight: 700,
    lineHeight: 1.2,
  },
  
  // 界面标题
  heading: {
    fontFamily: 'SystemFont',
    size: 24,
    weight: 600,
    lineHeight: 1.3,
  },
  
  // 正文
  body: {
    fontFamily: 'SystemFont',
    size: 16,
    weight: 400,
    lineHeight: 1.5,
  },
  
  // 辅助文字
  caption: {
    fontFamily: 'SystemFont',
    size: 12,
    weight: 400,
    lineHeight: 1.4,
  }
}
```

## 布局规范

### 栅格系统
```
12列网格
  │ gutters │ margins │
  │         │         │
  │← gutter →│← gutter →│
  
边距：16px (移动端)
列间距：16px
安全区：左右各留12px
```

### 安全区指南
```typescript
// iOS Safe Area
const SafeArea = {
  top: 44,     // 刘海屏44，普通20
  bottom: 34,  // home indicator
  horizontal: 16,
}

// 适配策略
// - 顶部使用StatusBar高度
// - 底部使用HomeIndicator高度
// - 横向内容不超SafeArea
```

### 间距系统
```typescript
const Spacing = {
  xs: 4,
  sm: 8,
  md: 16,
  lg: 24,
  xl: 32,
  xxl: 48,
}

// 使用示例
// 内边距：组件内部间距
// 外边距：组件之间间距
// 统一使用4px倍数
```

## 动效设计

### 动效原则
1. **目的性**：动效传递信息，而非装饰
2. **自然性**：符合物理规律
3. **高效性**：快速响应，不拖沓
4. **一致性**：同类型元素动效一致

### 常用动效曲线
```typescript
// 常用缓动曲线
const Easing = {
  // 标准缓入缓出
  standard: 'cubic-bezier(0.4, 0, 0.2, 1)',
  
  // 减速(出现)
  decelerate: 'cubic-bezier(0, 0, 0.2, 1)',
  
  // 加速(消失)
  accelerate: 'cubic-bezier(0.4, 0, 1, 1)',
  
  // 弹性
  spring: 'cubic-bezier(0.175, 0.885, 0.32, 1.275)',
  
  // 回弹
  bounce: 'cubic-bezier(0.68, -0.55, 0.265, 1.55)',
}

// 时长
const Duration = {
  instant: 100,   // 立即
  fast: 200,      // 快，如hover
  normal: 300,    // 标准
  slow: 500,      // 慢，如页面切换
}
```

### 游戏特定动效
```typescript
// 伤害数字
const DamageNumber = {
  duration: 800,
  offsetY: -50,
  scale: [1.2, 1.0],
  opacity: [1, 0],
  easing: 'easeOut',
}

// 技能特效
const SkillEffect = {
  duration: 500,
  scale: [0.5, 1.0],
  rotation: 360,
  particles: true,
}

// 界面切换
const UITransition = {
  push: { direction: 'left', duration: 300 },
  pop: { direction: 'right', duration: 300 },
  modal: { direction: 'bottom', duration: 400 },
}
```

## 设计系统

### 组件规范
```typescript
// Button组件状态
interface ButtonState {
  default: { bg: '#FF6B35', text: '#FFFFFF' };
  hover: { bg: '#FF8555', text: '#FFFFFF' };
  active: { bg: '#E55A25', text: '#FFFFFF' };
  disabled: { bg: '#CCCCCC', text: '#888888' };
}

// Card组件
interface CardSpec {
  borderRadius: 12,
  padding: 16,
  shadow: '0 4px 12px rgba(0,0,0,0.1)',
  bgColor: '#FFFFFF',
}
```

### 图标规范
```typescript
// 图标尺寸
const IconSize = {
  xs: 16,
  sm: 24,
  md: 32,
  lg: 48,
};

// 图标样式
// - 线性图标：细线风格
// - 填充图标：实心风格
// - 统一描边宽度：2px
// - 统一颜色：currentColor
```

## 多平台适配

### iOS适配
```typescript
// 设备尺寸
const iOSDevices = {
  iPhoneSE: { width: 320, height: 568, scale: 2 },
  iPhone8: { width: 375, height: 667, scale: 2 },
  iPhone8Plus: { width: 414, height: 736, scale: 3 },
  iPhoneX: { width: 375, height: 812, scale: 3 },
  iPhone12: { width: 390, height: 844, scale: 3 },
};

// 安全区处理
@View {
// 使用SafeAreaView包裹
// 确保内容不被刘海遮挡
}

// 字体适配
const iOSFontScale = (baseFont: number) => {
  // 使用系统字体，系统会自动适配
  return baseFont;
};
```

### Android适配
```typescript
// 设备尺寸
const AndroidDevices = {
  mdpi: { scale: 1 },
  hdpi: { scale: 1.5 },
  xhdpi: { scale: 2 },
  xxhdpi: { scale: 3 },
  xxxhdpi: { scale: 4 },
};

// 适配策略
// 1. 使用sp作为字体单位
// 2. 使用dp作为尺寸单位
// 3. 使用相对布局
// 4. 提供多套切图资源
```

### 横屏适配
```typescript
// 横屏布局
const LandscapeLayout = {
  // UI按16:9比例设计
  // 竖屏元素需要横屏版本
  // 横屏特有布局：左右分栏
};

// 适配要点
// - 主要内容放中间安全区
// - 两边预留功能区
// - 避免横屏遮挡
```

## 资源输出

### 切图规范
```bash
# 命名规范
[类型]_[名称]_[状态]@[倍率].png

# 示例
btn_login_normal@2x.png
btn_login_pressed@2x.png
btn_login_disabled@2x.png
icon_coin@3x.png

# 输出格式
# iOS: PNG/PDF
# Android: PNG/WebP
```

### 标注规范
```markdown
## 登录按钮标注

### 位置
- X: 150
- Y: 400
- Width: 200
- Height: 48

### 样式
- 背景色: #FF6B35
- 圆角: 24
- 文字: #FFFFFF
- 字号: 18sp
- 字重: 600

### 状态
- Normal: #FF6B35
- Pressed: #E55A25
- Disabled: #CCCCCC

### 间距
- 内边距: 水平16dp，垂直12dp
- 元素间距: 8dp
```