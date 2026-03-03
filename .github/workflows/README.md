# CI/CD 工作流说明

本目录包含 ShareSDU 项目的 GitHub Actions 工作流配置。

## 📋 工作流列表

### 1. Web CI (`web-ci.yml`)

**触发条件：**
- Push 到 `main` 或 `develop` 分支（web 目录有变更）
- Pull Request 到 `main` 或 `develop` 分支（web 目录有变更）

**工作流程：**
1. **Lint** - ESLint 代码检查
2. **Test** - 运行单元测试，上传覆盖率报告
3. **Build** - 构建生产版本，上传构建产物
4. **E2E** - 运行端到端测试（可选）

**产物：**
- `web-dist` - 构建后的前端文件
- `e2e-screenshots` - E2E 测试失败时的截图

### 2. Android CI (`android-ci.yml`)

**触发条件：**
- Push 到 `main` 或 `develop` 分支（android 目录有变更）
- Pull Request 到 `main` 或 `develop` 分支（android 目录有变更）

**工作流程：**
1. **Build** - Gradle 构建
2. **Test** - 运行单元测试
3. **Build APK** - 构建 Release APK

**产物：**
- `android-apk` - 构建的 APK 文件
- `android-test-results` - 测试结果

### 3. Documentation Check (`docs-check.yml`)

**触发条件：**
- Push 到 `main` 或 `develop` 分支（Markdown 文件有变更）
- Pull Request 到 `main` 或 `develop` 分支（Markdown 文件有变更）

**检查项：**
1. **Markdown Lint** - Markdown 格式检查
2. **Link Check** - 检查文档中的链接是否有效
3. **Spell Check** - 拼写检查

### 4. PR Check (`pr-check.yml`)

**触发条件：**
- Pull Request 打开、同步或重新打开

**检查项：**
1. **Commit Check** - 检查提交信息格式和签名
2. **PR Title Check** - 检查 PR 标题格式
3. **Size Check** - 检查 PR 大小，超过 1000 行会警告
4. **Auto Label** - 根据文件变更自动添加标签

## 🔧 配置文件

所有配置文件都位于 `.github/config/` 目录下，保持项目根目录整洁。

### `.github/config/.markdownlint.json`
Markdown 格式检查配置，定义了允许的 Markdown 规则。

### `.github/config/.markdown-link-check.json`
链接检查配置，定义了忽略的链接模式和重试策略。

### `.github/config/.commitlintrc.json`
提交信息格式检查配置，遵循 Conventional Commits 规范。

### `.github/config/.cspell.json`
拼写检查配置，定义了项目特定的词汇表。

### `.github/labeler.yml`
自动标签配置，根据文件变更自动添加 PR 标签。

## 📊 状态徽章

可以在 README.md 中添加以下徽章：

```markdown
![Web CI](https://github.com/W1412X/sharesdu/workflows/Web%20CI/badge.svg)
![Android CI](https://github.com/W1412X/sharesdu/workflows/Android%20CI/badge.svg)
![Documentation Check](https://github.com/W1412X/sharesdu/workflows/Documentation%20Check/badge.svg)
```

## 🚀 本地运行

### Web 项目

```bash
cd web

# Lint
npm run lint

# Test
npm run test:unit

# Build
npm run build

# E2E
npm run test:e2e
```

### Android 项目

```bash
cd android

# Build
./gradlew build

# Test
./gradlew test

# Build APK
./gradlew assembleRelease
```

## 🔐 Secrets 配置

如果需要部署或其他敏感操作，需要在 GitHub 仓库设置中配置以下 Secrets：

- `CODECOV_TOKEN` - Codecov 上传 token（可选）
- 其他部署相关的 secrets

## 📝 添加新的工作流

1. 在 `.github/workflows/` 目录下创建新的 `.yml` 文件
2. 定义触发条件和工作流程
3. 更新本文档说明新工作流的用途

## 🐛 故障排查

### 工作流失败

1. 查看 Actions 标签页的详细日志
2. 检查是否是依赖安装问题
3. 确认配置文件格式正确
4. 本地运行相同的命令验证

### 提交签名检查失败

参考 [CONTRIBUTING.md](../../CONTRIBUTING.md) 配置提交签名。

### 文档检查失败

- Markdown Lint: 修复格式问题或更新 `.github/config/.markdownlint.json`
- Link Check: 修复失效链接或添加到忽略列表 `.github/config/.markdown-link-check.json`
- Spell Check: 修正拼写或添加到 `.github/config/.cspell.json` 词汇表

## 📚 相关资源

- [GitHub Actions 文档](https://docs.github.com/en/actions)
- [Conventional Commits](https://www.conventionalcommits.org/)
- [贡献指南](../../CONTRIBUTING.md)
