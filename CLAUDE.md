# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## 项目概述

**Coding Standards Gradle** GRADLE 项目的编码规范检查工具集，通过 Gradle 任务自动化执行编码规范检查，确保代码质量和一致性。

## 技术架构

- **Gradle 脚本** - 纯 Gradle 脚本实现的规范检查工具
- **模块化设计** - 四大规范模块独立加载
- **静态代码分析** - 基于正则表达式和文件扫描的静态检查
- **编译时集成** - 强制性检查集成到编译流程

## 核心模块

项目由 4 个独立的规范检查模块组成：

```
coding-standards-gradle/
├── coding-standards.gradle           # 主入口，整合所有检查模块
├── common-coding.gradle              # 通用编码规范检查
├── database-standards.gradle         # 数据库规范检查
├── java-coding-standards.gradle      # Java 编码规范检查
└── mybatis-flex-standards.gradle     # MyBatis-Flex 规范检查
```

### 模块职责

#### 1. common-coding.gradle（通用编码规范）
- `checkFullyQualifiedNames` - 禁止在方法中使用全限定类名
- `checkMagicValues` - 检测魔法值（未定义的常量）
- `checkLongConstantSuffix` - Long 常量必须使用大写 L 后缀
- `checkCollectionEmpty` - 集合判空应使用 CollectionUtils

#### 2. database-standards.gradle（数据库规范）
- `checkSelectStar` - 禁止使用 SELECT *
- `checkEntityRequiredFields` - Entity 必须包含 id/createTime/updateTime/isDeleted
- `checkIndexNaming` - 索引命名规范（idx_/uk_ 前缀）
- `checkAmountFieldType` - 金额字段必须使用 BigDecimal

#### 3. java-coding-standards.gradle（Java 规范）
- `checkEnumNaming` - 枚举类必须以 Enum 结尾
- `checkClassNaming` - 类命名规范（Service/Controller/Mapper）
- `checkLombokAnnotations` - Lombok 注解使用规范
- `checkResultPageResultNesting` - 禁止 Result<PageResult> 嵌套
- `checkRecordUsage` - 推荐 DTO/VO 使用 Record
- `checkEnumFromCodeMethod` - 枚举类应提供 fromCode 方法

#### 4. mybatis-flex-standards.gradle（MyBatis-Flex 规范）
- `checkEntityAnnotations` - Entity 必须有 @Table 和 @Id 注解
- `checkMapperInterface` - Mapper 必须继承 BaseMapper
- `checkEnumValueAnnotation` - 枚举 code 字段应标注 @EnumValue
- `checkLogicDeleteField` - isDeleted 字段配置检查
- `checkVersionField` - version 字段配置检查（乐观锁）
- `checkTimeFieldAutoFill` - 时间字段自动填充配置建议

## 常用命令

### 执行检查任务

```bash
# 查看所有可用任务和帮助
gradle codingStandardsHelp

# 执行所有规范检查
gradle checkCodingStandards

# 执行强制性检查（编译前必须通过）
gradle checkMandatoryStandards

# 执行建议性检查（仅警告）
gradle checkAdvisoryStandards
```

### 分模块检查

```bash
# 通用编码规范
gradle checkCommonCodingStandards

# 数据库规范
gradle checkDatabaseStandards

# Java 编码规范
gradle checkJavaCodingStandards

# MyBatis-Flex 规范
gradle checkMyBatisFlexStandards
```

### 单项检查示例

```bash
# 检查枚举类命名
gradle checkEnumNaming

# 检查 SELECT * 使用
gradle checkSelectStar

# 检查 Entity 必备字段
gradle checkEntityRequiredFields

# 检查 Result/PageResult 嵌套
gradle checkResultPageResultNesting
```

### 生成检查报告

```bash
# 生成 Markdown 格式的检查报告
gradle generateCodingStandardsReport

# 报告路径: build/reports/coding-standards/coding-standards-report.md
```

## 检查级别说明

### 强制性检查（❌ 编译失败）
以下检查失败会抛出 `GradleException`，中断编译流程：

- `checkFullyQualifiedNames` - 全限定类名检查
- `checkLongConstantSuffix` - Long 常量后缀检查
- `checkSelectStar` - SELECT * 检查
- `checkEntityRequiredFields` - Entity 必备字段检查
- `checkAmountFieldType` - 金额字段类型检查
- `checkEnumNaming` - 枚举类命名检查
- `checkResultPageResultNesting` - Result/PageResult 嵌套检查
- `checkEntityAnnotations` - Entity 注解检查
- `checkMapperInterface` - Mapper 接口检查
- `checkLogicDeleteField` - 逻辑删除字段检查
- `checkVersionField` - 乐观锁版本号检查

### 建议性检查（⚠️ 仅警告）
以下检查失败仅输出警告，不中断编译：

- `checkMagicValues` - 魔法值检查
- `checkCollectionEmpty` - 集合判空检查
- `checkIndexNaming` - 索引命名检查
- `checkClassNaming` - 类命名规范检查
- `checkLombokAnnotations` - Lombok 注解检查
- `checkRecordUsage` - Record 使用建议
- `checkEnumFromCodeMethod` - 枚举 fromCode 方法检查
- `checkEnumValueAnnotation` - @EnumValue 注解建议
- `checkTimeFieldAutoFill` - 时间字段自动填充建议

## 集成到项目

### 在其他 Gradle 项目中使用

在目标项目的 `build.gradle` 中添加：

```gradle
// 应用编码规范检查
apply from: 'path/to/coding-standards-gradle/coding-standards.gradle'

// 将强制性检查集成到编译任务
tasks.named('compileJava') {
    dependsOn 'checkMandatoryStandards'
}
```

### 配置检查范围

默认检查路径为 `src/main/java` 和 `src/main/resources`。如需自定义，可在各检查任务中修改 `fileTree` 配置。

## 开发指南

### 添加新的检查规则

1. 在对应的规范模块文件中添加新任务：

```gradle
tasks.register('checkNewRule') {
    group = 'verification'
    description = '检查规则描述'

    doLast {
        def violations = []
        def javaFiles = fileTree(dir: 'src/main/java', include: '**/*.java')

        javaFiles.each { file ->
            // 实现检查逻辑
        }

        if (!violations.isEmpty()) {
            println "\n" + "="*60
            println "❌ 检查失败！发现 ${violations.size()} 个问题:"
            violations.each { println it }
            println "="*60 + "\n"
            throw new GradleException("检查失败")
        } else {
            println "✅ 检查通过"
        }
    }
}
```

2. 将新任务添加到综合检查任务的依赖中：

```gradle
tasks.register('checkModuleStandards') {
    dependsOn checkExistingRule
    dependsOn checkNewRule  // 添加新规则
}
```

3. 如果是强制性检查，还需在 `coding-standards.gradle` 的 `checkMandatoryStandards` 中添加依赖。

### 检查逻辑实现技巧

- 使用正则表达式匹配代码模式：`content =~ /pattern/`
- 跳过注释行：`if (line.trim().startsWith('//') || line.trim().startsWith('*'))`
- 区分接口和实现：检查 `content.contains('interface ')`
- 检查注解存在：`content.contains('@Table')`
- 使用 `fileTree` 指定检查文件范围

## 规范来源

本工具的编码规范来源于开源项目：

**GitHub 仓库**: [kk-418/coding-standards](https://github.com/kk-418/coding-standards)

**规范基础**:
- **Java 编码规范**: 基于《阿里巴巴 Java 开发规范(嵩山版)》进行整理和优化
- **数据库规范**: 参考业界最佳实践和性能优化经验
- **通用规范**: 综合各大公司编码规范的最佳实践
- **MyBatis-Flex 规范**: 结合 MyBatis-Flex 官方文档和项目实践

**规范文档地址**: [https://github.com/kk-418/coding-standards](https://github.com/kk-418/coding-standards)

所有检查规则基于以下编码规范文档：

- **通用规范**: [common-coding.md](https://github.com/kk-418/coding-standards/blob/main/common-coding.md)
- **数据库规范**: [database-standards.md](https://github.com/kk-418/coding-standards/blob/main/database-standards.md)
- **Java 规范**: [java-coding-standards.md](https://github.com/kk-418/coding-standards/blob/main/java-coding-standards.md)
- **MyBatis-Flex 规范**: [mybatis-flex-standards.md](https://github.com/kk-418/coding-standards/blob/main/mybatis-flex-standards.md)

详细规范查看: [规范文档索引](https://github.com/kk-418/coding-standards/blob/main/index.md)

## 重要开发原则

- **静态检查优先** - 所有检查基于静态代码分析，无需运行代码
- **快速失败** - 强制性检查失败立即抛出异常，提前发现问题
- **渐进式提示** - 建议性检查仅警告，支持渐进式改进
- **模块化扩展** - 每个规范模块独立，易于扩展新规则
- **零依赖** - 纯 Gradle 脚本实现，无需额外依赖

## 文件约定

- 所有 `.gradle` 文件使用 UTF-8 编码
- 文件头部包含作者、创建日期、规范来源注释
- 每个检查任务独立定义，职责单一
- 违规信息格式：`文件路径:行号 - 问题描述`
- 使用 `println` 输出彩色标记（✅ ❌ ⚠️ 💡）提升可读性
