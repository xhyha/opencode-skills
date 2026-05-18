---
name: committer-release
description: Committer技能，包含Git工作流、CI/CD、版本管理、发布流程和回滚策略
---

# Committer技能

## Git工作流

### 分支模型
```
main (发布分支)
  ├── develop (开发分支)
  │     ├── feature/login (功能分支)
  │     ├── feature/battle (功能分支)
  │     └── bugfix/fix-login (修复分支)
  └── release/v1.0 (发布候选分支)
```

### Git命令规范
```bash
# 功能分支命名
git checkout -b feature/[功能名]
git checkout -b feature/user-login

# 修复分支命名
git checkout -b bugfix/_问题描述]
git checkout -b bugfix/fix-login-crash

# 发布分支命名
git checkout -b release/v1.0.0

# 合并前变基(保持线性历史)
git rebase main

# 合并(保留历史)
git merge --no-ff feature/xxx
```

### Commit规范
```
<type>(<scope>): <subject>

body

footer
```

Type类型：
- **feat**: 新功能
- **fix**: Bug修复
- **docs**: 文档更新
- **style**: 代码格式(不影响运行)
- **refactor**: 重构
- **perf**: 性能优化
- **test**: 测试相关
- **chore**: 构建/工具

示例：
```
feat(login): 添加短信验证码登录

- 添加SMS服务集成
- 添加验证码发送接口
- 添加验证码输入界面

Closes #123
```

## CI/CD流程

### CI阶段
```yaml
# .gitlab-ci.yml 示例
stages:
  - lint
  - test
  - build

lint:
  stage: lint
  script:
    - npm run lint
    - npm run type-check

unit-test:
  stage: test
  script:
    - npm run test:unit
  coverage: '/Coverage: \d+\.\d+/'

integration-test:
  stage: test
  script:
    - npm run test:integration
  services:
    - postgres:13
    - redis:6

build:
  stage: build
  script:
    - npm run build
  artifacts:
    paths:
      - dist/
    expire_in: 1 week
```

### CD阶段
```yaml
deploy:
  stage: deploy
  script:
    - ./deploy.sh ${DEPLOY_ENV}
  only:
    - main
  when: manual

# 部署脚本示例
deploy.sh:
  # 1. 拉取最新代码
  # 2. 安装依赖
  # 3. 构建
  # 4. 执行数据库迁移
  # 5. 重启服务
  # 6. 健康检查
```

## 版本管理

### 语义化版本( SemVer )
```
主版本.次版本.修订号
 MAJOR  MINOR  PATCH

1.2.3
 │     │     │
 │     │     └── 补丁版本：Bug修复
 │     └──────── 次版本：新功能(向下兼容)
 └────────────── 主版本：破坏性变更
```

### 版本标签
```bash
# 创建版本标签
git tag -a v1.0.0 -m "版本1.0.0发布"
git push origin v1.0.0

# 删除远程标签
git push origin --delete v1.0.0
git tag -d v1.0.0
```

## 发布流程

### 发布前检查清单
- [ ] 所有功能已测试通过
- [ ] 变更日志已更新
- [ ] 版本号已更新
- [ ] CI/CD pipeline全部通过
- [ ] 安全扫描无高危漏洞
- [ ] 性能测试达标
- [ ] 准备回滚方案
- [ ] 通知相关团队

### 发布步骤
```bash
# 1. 创建发布分支
git checkout -b release/v1.0.0

# 2. 更新版本号和变更日志
# 编辑 CHANGELOG.md
# 修改 package.json version

# 3. 提交并打标签
git add .
git commit -m "release: v1.0.0"
git tag -a v1.0.0 -m "Release v1.0.0"
git push origin release/v1.0.0 --tags

# 4. 合并到main
git checkout main
git merge release/v1.0.0
git push origin main

# 5. 执行CD部署
# 在CI/CD平台触发部署

# 6. 监控发布
# 观察监控仪表盘
# 检查日志和报警
```

### 灰度发布策略
```
阶段1: 10% 用户
  └── 监控错误率、性能指标
  
阶段3: 30% 用户
  └── 扩大监控范围
  
阶段3: 50% 用户
  └── 准备回滚
  
阶段4: 100% 用户
  └── 全量发布
```

## 回滚策略

### 回滚触发条件
- 错误率上升超过阈值(>1%)
- 关键指标明显下降
- P0级Bug影响大量用户
- 安全漏洞

### 回滚执行
```bash
# 快速回滚到上一版本
./rollback.sh

# rollback.sh 示例
#!/bin/bash
VERSION=${1:-"previous"}

# 停止当前服务
kubectl scale deployment game-api --replicas=0

# 回滚数据库(如需要)
# ./rollback-db.sh

# 部署上一个稳定版本
kubectl set image deployment/game-api game-api=game-api:${VERSION}

# 验证
curl -f http://game-api/health

# 重启服务
kubectl scale deployment game-api --replicas=3
```

### 数据库回滚
- 每次发布前备份数据库
- 使用事务确保数据一致性
- 大规模数据变更准备回滚SQL

## Release Notes编写

### 模板
```markdown
# Release Notes - v1.0.0

## 发布日期
2026-01-15

## 新功能
### 🎮 核心功能
- 添加角色培养系统
- 添加好友对战功能

### ⚡ 性能优化
- 登录速度提升50%
- 减少30%内存占用

## Bug修复
- 修复登录偶发崩溃问题 #123
- 修复商城购买异常 #124

## 破坏性变更
- 旧版API已废弃，请使用 /api/v2/

## 升级注意
- 需要执行数据库迁移脚本
- 请在设置中重新登录
```

### 自动生成
```bash
# 使用git-changelog自动生成
npm install -g git-changelog
git-changelog --tag v1.0.0
```