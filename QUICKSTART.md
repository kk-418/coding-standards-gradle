# 编码规范检查 - 5分钟快速入门

> **快速上手指南** - 新手必读 ⭐⭐⭐

---

## 🎯 1分钟了解

**什么是编码规范检查?**
- 自动检查代码是否符合团队规范
- 在编译时自动执行，防止不规范代码进入代码库
- 提供详细的错误提示和修改建议

**规范来源**:
- GitHub: [kk-418/coding-standards](https://github.com/kk-418/coding-standards)
- 基于《阿里巴巴 Java 开发规范(嵩山版)》

---

## ⚡ 常用命令

```bash
# 查看帮助
gradle codingStandardsHelp

# 执行所有检查
gradle checkCodingStandards

# 编译时自动检查（包含validateVoTypes）
gradle compileJava
gradle build
```

---

## ❗ 最重要的规则

### 1. VO类型约束 ⭐⭐⭐

**规则**: VO类所有字段必须使用 `String` 类型

```java
// ❌ 错误 - 编译失败
public record PaymentVO(
    Long id,                    // 错误: Long ID
    BigDecimal amount,          // 错误: BigDecimal
    LocalDateTime createTime    // 错误: LocalDateTime
) {}

// ✅ 正确
public record PaymentVO(
    String id,                  // 正确: String
    String amount,              // 正确: String
    String createTime           // 正确: String (格式: yyyy-MM-dd HH:mm:ss)
) {}
```

**为什么?**
- 避免JSON序列化精度丢失
- 避免JavaScript精度问题（Long超出安全整数范围）
- 统一时间格式，避免时区问题

### 2. 禁止SELECT * ⭐⭐

```sql
-- ❌ 错误
SELECT * FROM payment WHERE id = 1;

-- ✅ 正确
SELECT id, amount, status, create_time FROM payment WHERE id = 1;
```

### 3. 枚举类命名 ⭐⭐

```java
// ❌ 错误
public enum PaymentStatus {}

// ✅ 正确
public enum PaymentStatusEnum {}
```

### 4. 禁止Result嵌套PageResult ⭐

```java
// ❌ 错误
public Result<PageResult<OrderVO>> queryOrders() {}

// ✅ 正确
public PageResult<OrderVO> queryOrders() {}
```

---

## 🔧 出现错误怎么办?

### 第1步: 查看错误信息

```bash
gradle compileJava
```

输出示例:
```
❌ VO类型校验失败！发现 2 个问题:
/path/to/PaymentVO.java:15 - amount (BigDecimal) 应使用String
/path/to/OrderVO.java:23 - userId (Long ID) 应使用String
```

### 第2步: 修改代码

根据提示修改代码类型为 `String`

### 第3步: 重新编译

```bash
gradle compileJava
```

看到 `✅ VO类型校验通过！` 表示修复成功。

---

## 📚 深入学习

### 查看完整文档

```bash
# 用户手册（本地）
open coding-standards-gradle/README.md

# 规范文档（在线 - GitHub）
open https://github.com/kk-418/coding-standards
```

### 主要文档索引

| 文档 | 说明 | 何时查看 |
|------|------|---------|
| [README.md](./README.md) | 完整用户手册 | 详细了解所有功能 |
| [index.md](https://github.com/kk-418/coding-standards/blob/main/index.md) | 规范索引 | 快速查找具体规范 |
| [java-coding-standards.md](https://github.com/kk-418/coding-standards/blob/main/java-coding-standards.md) | Java规范 | Java开发必读 |
| [debug.md](https://github.com/kk-418/coding-standards/blob/main/debug.md) | 调试规范 | 调试问题时参考 |

---

## ❓ 常见问题

**Q: 能否跳过检查?**

A: 不推荐。但临时可以用：
```bash
gradle compileJava -x validateVoTypes
```

**Q: 检查太严格了?**

A: 规范基于《阿里巴巴 Java 开发规范》，目的是保证代码质量。如有问题请在GitHub提Issue。

**Q: 如何查看最新规范文档?**

A: 规范文档托管在 GitHub，直接访问查看最新版本：
```bash
open https://github.com/kk-418/coding-standards
```

---

## 🔗 相关链接

- **规范源项目**: [https://github.com/kk-418/coding-standards](https://github.com/kk-418/coding-standards)
- **阿里巴巴规范**: [https://github.com/alibaba/p3c](https://github.com/alibaba/p3c)
- **完整手册**: [README.md](./README.md)

---

**编写时间**: 2025-10-01
**作者**: kk
**版本**: 1.0.0
