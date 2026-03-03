# Android CI 修复说明

## 🐛 发现的问题

### 1. Kotlin 插件缺失
**问题：** Android 项目使用了 Kotlin DSL (`.gradle.kts` 文件)，但没有应用 Kotlin 插件。

**影响：** 可能导致构建失败或 Kotlin 相关功能无法使用。

**修复：**
- 在 `android/build.gradle.kts` 中添加 Kotlin 插件
- 在 `android/app/build.gradle.kts` 中应用 Kotlin 插件
- 添加 `kotlinOptions` 配置

### 2. Java 版本不匹配
**问题：**
- CI 配置使用 JDK 17
- Gradle 配置使用 Java 1.8
- targetSdk 设置为 33，但 compileSdk 为 34

**影响：** 版本不一致可能导致编译错误或运行时问题。

**修复：**
- 统一使用 Java 17
- 更新 `compileOptions` 和 `kotlinOptions` 的 JVM 目标
- 统一 targetSdk 为 34

### 3. Gradle 缓存配置问题
**问题：** `cache: 'gradle'` 在子目录项目中可能无法正确工作。

**修复：** 使用 `gradle/gradle-build-action@v2` 替代，明确指定构建根目录。

### 4. 错误处理不足
**问题：** 构建失败时没有详细的错误信息和报告。

**修复：**
- 添加 `--stacktrace` 参数获取详细错误信息
- 添加 Gradle 版本验证步骤
- 分离 Debug 和 Release APK 构建
- 失败时上传构建报告

## ✅ 修复内容

### CI 工作流改进 (`.github/workflows/android-ci.yml`)

1. **使用专用的 Gradle Action**
   ```yaml
   - name: Setup Gradle
     uses: gradle/gradle-build-action@v2
     with:
       gradle-version: wrapper
       build-root-directory: android
   ```

2. **添加 Gradle 验证**
   ```yaml
   - name: Validate Gradle wrapper
     working-directory: ./android
     run: ./gradlew --version
   ```

3. **添加详细的错误追踪**
   ```yaml
   run: ./gradlew build --stacktrace
   ```

4. **分离 Debug 和 Release 构建**
   - Debug APK 总是构建（用于测试）
   - Release APK 可能失败（需要签名配置）

5. **改进错误报告**
   - 失败时上传构建报告
   - 测试结果总是上传

### Gradle 配置改进

#### `android/build.gradle.kts`
```kotlin
plugins {
    id("com.android.application") version "8.1.3" apply false
    id("org.jetbrains.kotlin.android") version "1.9.0" apply false  // 新增
}
```

#### `android/app/build.gradle.kts`
```kotlin
plugins {
    id("com.android.application")
    id("org.jetbrains.kotlin.android")  // 新增
}

android {
    // ...
    defaultConfig {
        targetSdk = 34  // 从 33 更新到 34
        // ...
    }
    
    compileOptions {
        sourceCompatibility = JavaVersion.VERSION_17  // 从 1.8 更新
        targetCompatibility = JavaVersion.VERSION_17  // 从 1.8 更新
    }
    
    kotlinOptions {  // 新增
        jvmTarget = "17"
    }
}

dependencies {
    implementation("androidx.core:core-ktx:1.13.1")  // 新增 Kotlin 扩展
    // ...
}
```

## 🔍 验证步骤

### 本地验证

```bash
cd android

# 1. 验证 Gradle wrapper
./gradlew --version

# 2. 清理构建
./gradlew clean

# 3. 构建项目
./gradlew build --stacktrace

# 4. 运行测试
./gradlew test --stacktrace

# 5. 构建 Debug APK
./gradlew assembleDebug

# 6. 构建 Release APK（可能需要签名配置）
./gradlew assembleRelease
```

### CI 验证

推送代码后，检查 GitHub Actions：
1. 查看 "Android CI" 工作流
2. 检查每个步骤的日志
3. 下载构建产物验证 APK

## 📝 注意事项

### Release APK 签名

如果需要构建签名的 Release APK，需要：

1. **配置签名密钥**
   ```kotlin
   android {
       signingConfigs {
           create("release") {
               storeFile = file("path/to/keystore.jks")
               storePassword = System.getenv("KEYSTORE_PASSWORD")
               keyAlias = System.getenv("KEY_ALIAS")
               keyPassword = System.getenv("KEY_PASSWORD")
           }
       }
       buildTypes {
           release {
               signingConfig = signingConfigs.getByName("release")
               // ...
           }
       }
   }
   ```

2. **在 GitHub Secrets 中配置**
   - `KEYSTORE_PASSWORD`
   - `KEY_ALIAS`
   - `KEY_PASSWORD`
   - 上传 keystore 文件（base64 编码）

3. **更新 CI 工作流**
   ```yaml
   - name: Decode keystore
     run: |
       echo "${{ secrets.KEYSTORE_BASE64 }}" | base64 -d > android/app/keystore.jks
   
   - name: Build Release APK
     env:
       KEYSTORE_PASSWORD: ${{ secrets.KEYSTORE_PASSWORD }}
       KEY_ALIAS: ${{ secrets.KEY_ALIAS }}
       KEY_PASSWORD: ${{ secrets.KEY_PASSWORD }}
     run: ./gradlew assembleRelease
   ```

### Kotlin 版本兼容性

- Kotlin 1.9.0 与 Android Gradle Plugin 8.1.3 兼容
- 如果遇到兼容性问题，可以调整版本

### Java 17 要求

- Android Gradle Plugin 8.x 需要 Java 17
- 确保本地开发环境也使用 Java 17

## 🚀 后续优化建议

1. **添加代码检查**
   - Lint 检查
   - Detekt（Kotlin 静态分析）

2. **添加测试覆盖率**
   - JaCoCo 覆盖率报告
   - 上传到 Codecov

3. **优化构建速度**
   - 使用 Gradle 构建缓存
   - 并行构建

4. **添加自动化测试**
   - UI 测试（Espresso）
   - 集成测试

## 📚 参考资源

- [Android Gradle Plugin 8.x 发布说明](https://developer.android.com/studio/releases/gradle-plugin)
- [Kotlin 版本兼容性](https://kotlinlang.org/docs/gradle-configure-project.html)
- [GitHub Actions for Android](https://github.com/actions/setup-java)
