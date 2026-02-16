# SBT Plus 编码规范检查工具 - 用户手册

> **作者**: kk
> **版本**: 1.0.0
> **创建日期**: 2025-10-01
> **项目**: SBT Plus Framework

---

## 📖 简介

SBT Plus 编码规范检查工具是一个基于 Gradle 的自动化编码规范检查系统，集成到项目构建流程中，帮助团队保持代码质量和一致性。

### 规范来源

本工具的编码规范来自于以下开源项目：

**GitHub 仓库**: [kk-418/coding-standards](https://github.com/kk-418/coding-standards)

**规范基础**:
- **Java 编码规范**: 基于《阿里巴巴 Java 开发规范(嵩山版)》进行整理和优化
- **数据库规范**: 参考业界最佳实践和性能优化经验
- **前端规范**: 结合 React/Vue 官方推荐和 TypeScript 最佳实践

**规范文档地址**: [https://github.com/kk-418/coding-standards](https://github.com/kk-418/coding-standards)

---

## 📦 在其他 Gradle 项目中引入

### 方式一: 直接 Clone 项目（推荐）

1. **Clone 项目到你的项目目录**

```bash
cd /path/to/your-project
git clone https://github.com/kk-418/coding-standards-gradle.git
```

2. **在 build.gradle 中引入**

```groovy
// 在项目根目录的 build.gradle 文件顶部添加
plugins {
    id 'java'
    // ... 其他插件
}

// 应用编码规范检查配置
apply from: 'coding-standards-gradle/coding-standards.gradle'

// 其他配置...
subprojects {
    apply plugin: 'java'

    // validateVoTypes 任务需要在 subprojects 中定义
    // 如果没有，请参考本项目的 build.gradle 添加

    // ... 其他配置
}
```

3. **验证引入是否成功**

```bash
# 查看帮助
gradle codingStandardsHelp

# 执行检查
gradle checkCodingStandards
```

### 方式二: Git Submodule（适合多项目共享）

1. **添加为 Git Submodule**

```bash
cd /path/to/your-project
git submodule add https://github.com/kk-418/coding-standards-gradle.git coding-standards-gradle
```

2. **在 build.gradle 中引入**（同方式一步骤2）

3. **团队成员同步**

```bash
# 克隆项目时
git clone --recursive https://github.com/your/project.git

# 已有项目更新
git submodule update --init --recursive
```

### 关键配置说明

#### 1. validateVoTypes 任务（重要）

如果你的项目需要 VO 类型校验，需要在 `subprojects` 中添加以下配置：

```groovy
// 在 build.gradle 的 subprojects 块中添加
subprojects {
    apply plugin: 'java'

    // VO类型校验任务（需要 JavaParser 依赖）
    tasks.register('validateVoTypes') {
        group = 'verification'
        description = '校验VO类不使用BigDecimal、Long ID和时间类型'

        doLast {
            // 检查逻辑（参考本项目 build.gradle 第 137-222 行）
            // 这里需要完整的 validateVoTypes 实现
        }
    }

    // 编译时自动执行
    compileJava.dependsOn validateVoTypes
}
```

**完整实现请参考**: 本项目的 `build.gradle` 文件（第 136-222 行）

#### 2. 自定义检查规则

如需自定义检查范围，可修改以下文件：

- `coding-standards-gradle/common-coding.gradle` - 通用编码规范
- `coding-standards-gradle/database-standards.gradle` - 数据库规范
- `coding-standards-gradle/java-coding-standards.gradle` - Java编码规范
- `coding-standards-gradle/mybatis-flex-standards.gradle` - MyBatis-Flex规范

#### 3. 选择性启用检查

如果不需要某些检查，可以在引入时跳过：

```groovy
// 仅引入需要的模块
apply from: 'coding-standards-gradle/common-coding.gradle'
apply from: 'coding-standards-gradle/java-coding-standards.gradle'
// 不引入数据库和MyBatis-Flex规范
```

### 依赖要求

编码规范检查工具需要以下依赖：

1. **Gradle 版本**: 7.0+ (推荐 9.0+)
2. **Java 版本**: 11+ (如使用 validateVoTypes，推荐 21)
3. **JavaParser**: 用于 validateVoTypes 任务（可选）

如需使用 validateVoTypes，在 buildscript 中添加：

```groovy
buildscript {
    repositories {
        mavenCentral()
    }
    dependencies {
        classpath 'com.github.javaparser:javaparser-core:3.27.0'
    }
}
```

### 故障排查

#### 问题1: 找不到 validateVoTypes 任务

**原因**: validateVoTypes 任务在子项目中定义，根项目无法直接访问。

**解决方案**: 确保任务在 `subprojects` 块中定义，参考本项目 build.gradle。

#### 问题2: JavaParser 依赖缺失

**错误信息**: `Cannot resolve class StaticJavaParser`

**解决方案**: 在 buildscript 中添加 JavaParser 依赖（见上文）。

#### 问题3: 某些检查失败

**解决方案**:
1. 查看错误详情，了解违反的规范
2. 参考 [GitHub 规范文档](https://github.com/kk-418/coding-standards) 修改代码
3. 或临时跳过检查: `gradle compileJava -x validateVoTypes`

---

## 🚀 快速开始

### 1. 查看可用任务

```bash
gradle codingStandardsHelp
```

### 2. 执行编码规范检查

```bash
# 执行所有检查（强制性 + 建议性）
gradle checkCodingStandards

# 仅执行强制性检查
gradle checkMandatoryStandards

# 仅执行建议性检查
gradle checkAdvisoryStandards
```

### 3. 自动执行（编译时）

编码规范检查已集成到编译流程：

```bash
# 编译时自动执行 validateVoTypes 强制性检查
gradle compileJava

# 完整构建
gradle build
```

---

## 📋 检查规则概览

### 强制性检查（编译时执行，违反将中断构建）

| 检查项 | 任务名 | 说明 |
|--------|--------|------|
| VO类型校验 | `validateVoTypes` | 禁止VO类使用BigDecimal、Long ID、时间类型 |
| 全限定类名 | `checkFullyQualifiedNames` | 禁止使用全限定类名 |
| Long常量后缀 | `checkLongConstantSuffix` | Long常量必须使用大写L后缀 |
| SELECT * | `checkSelectStar` | 禁止在SQL中使用SELECT * |
| Entity必备字段 | `checkEntityRequiredFields` | Entity必须包含id/createdAt/updatedAt/isDeleted |
| 金额字段类型 | `checkAmountFieldType` | 金额字段必须使用BigDecimal |
| 枚举类命名 | `checkEnumNaming` | 枚举类必须以Enum结尾 |
| Result嵌套 | `checkResultPageResultNesting` | 禁止Result<PageResult>嵌套 |
| Entity注解 | `checkEntityAnnotations` | Entity必须有@Table/@Id注解 |
| Mapper接口 | `checkMapperInterface` | Mapper必须继承BaseMapper |
| 逻辑删除字段 | `checkLogicDeleteField` | 检查逻辑删除字段配置 |
| 乐观锁字段 | `checkVersionField` | 检查乐观锁版本号字段 |

### 建议性检查（仅警告，不中断构建）

| 检查项 | 任务名 | 说明 |
|--------|--------|------|
| 魔法值 | `checkMagicValues` | 避免使用魔法值 |
| 集合判空 | `checkCollectionEmpty` | 推荐使用CollectionUtils.isEmpty() |
| 索引命名 | `checkIndexNaming` | 索引命名规范检查 |
| 类命名规范 | `checkClassNaming` | 类命名规范检查 |
| Lombok注解 | `checkLombokAnnotations` | Lombok注解使用建议 |
| Record使用 | `checkRecordUsage` | 推荐DTO/VO使用Record |
| 枚举fromCode | `checkEnumFromCodeMethod` | 枚举应提供fromCode方法 |
| 枚举@EnumValue | `checkEnumValueAnnotation` | 枚举字段应使用@EnumValue |
| 时间字段自动填充 | `checkTimeFieldAutoFill` | 时间字段自动填充检查 |

---

## 🎯 使用场景

### 场景 1: 开发前检查

在开始编码前，了解当前项目的编码规范：

```bash
# 查看使用帮助
gradle codingStandardsHelp

# 生成规范报告
gradle generateCodingStandardsReport

# 查看报告
cat build/reports/coding-standards/coding-standards-report.md
```

### 场景 2: 提交代码前检查

在提交代码前，执行完整检查确保代码符合规范：

```bash
# 执行所有编码规范检查
gradle checkCodingStandards

# 如果检查通过，提交代码
git add .
git commit -m "feat: 实现新功能"
```

### 场景 3: CI/CD 集成

在 CI/CD 流程中自动执行检查：

```yaml
# GitHub Actions 示例
- name: 编码规范检查
  run: gradle checkMandatoryStandards

- name: 构建项目
  run: gradle build
```

### 场景 4: 修复规范问题

当检查失败时，查看详细错误信息并修复：

```bash
# 执行检查
gradle checkMandatoryStandards

# 查看错误信息（示例）
# ❌ VO类型校验失败！发现 3 个问题:
# /path/to/PaymentVO.java:15 - amount (BigDecimal) 应使用String
# /path/to/OrderVO.java:23 - userId (Long ID) 应使用String
# /path/to/OrderVO.java:30 - createdAt (LocalDateTime) 应使用String
```

**修复方法**:
```java
// ❌ 错误示例
public record PaymentVO(
    Long id,                    // Long ID
    BigDecimal amount,          // BigDecimal
    LocalDateTime createdAt     // LocalDateTime
) {}

// ✅ 正确示例
public record PaymentVO(
    String id,                  // String
    String amount,              // String
    String createdAt            // String (格式: yyyy-MM-dd HH:mm:ss)
) {}
```

---

## 🔧 分模块检查

根据需要执行特定模块的检查：

### 通用编码规范检查

```bash
gradle checkCommonCodingStandards
```

**包含检查项**:
- 禁止全限定类名
- 魔法值检查
- Long常量后缀
- 集合判空方法

### 数据库规范检查

```bash
gradle checkDatabaseStandards
```

**包含检查项**:
- 禁止SELECT *
- Entity必备字段
- 索引命名规范
- 金额字段类型

### Java编码规范检查

```bash
gradle checkJavaCodingStandards
```

**包含检查项**:
- 枚举类命名
- 类命名规范
- Lombok注解使用
- Result/PageResult嵌套
- Record类使用
- 枚举fromCode方法

### MyBatis-Flex规范检查

```bash
gradle checkMyBatisFlexStandards
```

**包含检查项**:
- Entity注解
- Mapper接口继承
- 枚举@EnumValue注解
- 逻辑删除字段
- 乐观锁版本号字段
- 时间字段自动填充

---

## 📊 生成检查报告

### 生成报告

```bash
gradle generateCodingStandardsReport
```

### 查看报告

报告位置: `build/reports/coding-standards/coding-standards-report.md`

```bash
# macOS
open build/reports/coding-standards/coding-standards-report.md

# Linux
cat build/reports/coding-standards/coding-standards-report.md
```

报告内容包括:
- 所有检查规则概览
- 使用说明
- 规范文档链接

---

## ⚙️ 配置说明

### 集成方式

本工具已集成到项目的 `build.gradle` 中：

```groovy
// 应用编码规范检查配置
apply from: 'coding-standards-gradle/coding-standards.gradle'
```

### 自定义配置

如需自定义检查规则，可以修改以下文件：

- `coding-standards-gradle/common-coding.gradle` - 通用编码规范
- `coding-standards-gradle/database-standards.gradle` - 数据库规范
- `coding-standards-gradle/java-coding-standards.gradle` - Java编码规范
- `coding-standards-gradle/mybatis-flex-standards.gradle` - MyBatis-Flex规范

### 跳过特定检查

如果需要临时跳过某个检查：

```bash
# 跳过VO类型校验编译
gradle compileJava -x validateVoTypes

# 跳过所有编码规范检查构建
gradle build -x validateVoTypes
```

⚠️ **注意**: 不推荐跳过强制性检查，这可能导致代码质量问题。

---

## 📚 规范文档

### 在线查看

访问 GitHub 仓库查看完整规范文档：

**项目地址**: [https://github.com/kk-418/coding-standards](https://github.com/kk-418/coding-standards)

### 在线查看

访问 GitHub 仓库查看所有规范文档：

**项目地址**: [https://github.com/kk-418/coding-standards](https://github.com/kk-418/coding-standards)

### 主要文档

| 文档 | 说明 | 链接 |
|------|------|------|
| 规范索引 | 所有规范的快速查找入口 | [index.md](https://github.com/kk-418/coding-standards/blob/main/index.md) |
| 工作规范 | AI辅助开发的基本工作规范 | [guidelines.md](https://github.com/kk-418/coding-standards/blob/main/guidelines.md) |
| 代码质量价值观 | 指导AI辅助开发的质量标准 | [values.md](https://github.com/kk-418/coding-standards/blob/main/values.md) |
| 调试规范 | 基于证据的调试方法论 | [debug.md](https://github.com/kk-418/coding-standards/blob/main/debug.md) |
| 通用编码规范 | 所有语言通用的编码标准 | [common-coding.md](https://github.com/kk-418/coding-standards/blob/main/common-coding.md) |
| Java编码规范 | Java 21 + Spring Boot规范 | [java-coding-standards.md](https://github.com/kk-418/coding-standards/blob/main/java-coding-standards.md) |
| 数据库规范 | MySQL数据库设计与使用 | [database-standards.md](https://github.com/kk-418/coding-standards/blob/main/database-standards.md) |
| 前端规范 | React/Vue + TypeScript规范 | [frontend-standards.md](https://github.com/kk-418/coding-standards/blob/main/frontend-standards.md) |

---

## ⚠️ 常见问题

### Q1: 为什么VO类型必须使用String？

**原因**:
1. **避免JSON序列化精度丢失**: BigDecimal序列化可能导致精度问题
2. **避免JavaScript精度问题**: JavaScript的Number类型最大安全整数为2^53-1，Long类型的ID可能超出范围
3. **统一时间格式**: 避免时区和格式不一致问题

**详细说明**: 查看 [java-coding-standards.md#3.1](https://github.com/kk-418/coding-standards/blob/main/java-coding-standards.md#31-vo类型校验任务-validatevotypes)

### Q2: 检查失败如何修复？

1. **查看错误详情**: 错误信息会显示文件路径和行号
2. **参考规范文档**: 根据检查项名称查找对应的规范文档
3. **修改代码**: 按照规范文档的要求修改代码
4. **重新检查**: 执行 `gradle checkMandatoryStandards` 验证修复

### Q3: 如何跳过某个检查？

**不推荐跳过强制性检查**，但如果确实需要：

```bash
# 临时跳过
gradle compileJava -x validateVoTypes

# 或在 build.gradle 中移除依赖
# compileJava.dependsOn validateVoTypes
```

### Q4: 检查太慢怎么办？

编码规范检查在编译时执行，性能影响很小。如果遇到性能问题：

1. **使用Gradle缓存**: 启用配置缓存加速构建
   ```bash
   gradle compileJava --configuration-cache
   ```

2. **并行构建**: 使用并行构建加速
   ```bash
   gradle build --parallel
   ```

3. **增量编译**: Gradle会自动使用增量编译

### Q5: 规范文档在哪里查看？

规范文档统一管理在 GitHub 仓库:
- **仓库地址**: [https://github.com/kk-418/coding-standards](https://github.com/kk-418/coding-standards)
- **在线查看**: 直接访问 GitHub 仓库查看最新文档
- **下载本地**: 可以 clone 到本地查看：
  ```bash
  git clone https://github.com/kk-418/coding-standards.git
  ```

### Q6: 为什么禁止SELECT *？

**原因**:
1. **性能问题**: 查询不需要的字段浪费网络带宽和内存
2. **索引失效**: 可能无法使用覆盖索引
3. **字段变更风险**: 表结构变更可能导致程序出错
4. **可读性差**: 不明确需要哪些字段

**详细说明**: 查看 [database-standards.md#SQL编写规范](https://github.com/kk-418/coding-standards/blob/main/database-standards.md#3-sql编写规范)

---

## 🔗 相关资源

### 官方规范

- **阿里巴巴Java开发手册**: [https://github.com/alibaba/p3c](https://github.com/alibaba/p3c)
- **Google Java Style Guide**: [https://google.github.io/styleguide/javaguide.html](https://google.github.io/styleguide/javaguide.html)

### 框架文档

- **Spring Boot**: [https://spring.io/projects/spring-boot](https://spring.io/projects/spring-boot)
- **MyBatis-Flex**: [https://mybatis-flex.com/](https://mybatis-flex.com/)
- **Gradle**: [https://docs.gradle.org/](https://docs.gradle.org/)

### 编码规范项目

- **本项目规范源**: [https://github.com/kk-418/coding-standards](https://github.com/kk-418/coding-standards)

---

## 📞 反馈和支持

### 问题反馈

如果发现检查规则有问题或需要改进：

1. **GitHub Issues**: [https://github.com/kk-418/coding-standards/issues](https://github.com/kk-418/coding-standards/issues)
2. **团队沟通**: 联系项目维护者

### 贡献规范

欢迎贡献新的编码规范或改进现有规范：

1. Fork 规范仓库: [https://github.com/kk-418/coding-standards](https://github.com/kk-418/coding-standards)
2. 创建分支并添加/修改规范
3. 提交 Pull Request
4. 等待审核和合并

---

## 📝 更新日志

### v1.0.0 (2025-10-01)

**首次发布**:
- ✅ 集成编码规范检查到项目
- ✅ 支持强制性和建议性检查
- ✅ 支持分模块检查
- ✅ 集成到编译流程
- ✅ 生成检查报告
- ✅ 完整的用户文档

**包含规范**:
- 通用编码规范
- Java编码规范（基于阿里巴巴规范）
- 数据库规范
- MyBatis-Flex规范

---

## 📄 许可证

本工具遵循 MIT License 许可证，详见 [LICENSE](./LICENSE) 文件。

编码规范文档来源于 [kk-418/coding-standards](https://github.com/kk-418/coding-standards) 项目。

---

**文档版本**: 1.0.0
**最后更新**: 2025-10-01
**维护者**: kk
**规范来源**: [https://github.com/kk-418/coding-standards](https://github.com/kk-418/coding-standards)
