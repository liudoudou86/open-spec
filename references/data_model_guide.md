# 数据模型编写指南

## 基本结构

每个数据模型规格应包含以下部分：

```
### 4.1 [Model名称]
| 字段 | 类型 | 必填 | 说明 | 备注 |
|------|------|------|------|------|
| id | UUID | 是 | 主键 | 系统生成 |
| name | String(255) | 是 | 名称 | |
| status | Enum | 是 | 状态 | active/inactive/deleted |
| created_at | DateTime | 是 | 创建时间 | |
| updated_at | DateTime | 是 | 更新时间 | |
```

## 字段类型

### 常见类型

| 类型 | 说明 | 示例 |
|------|------|------|
| UUID | 全局唯一标识 | 550e8400-e29b-41d4-a716-446655440000 |
| String(n) | 字符串，n为最大长度 | String(255) |
| Text | 长文本 | |
| Integer | 整数 | |
| Float | 浮点数 | |
| Boolean | 布尔值 | true/false |
| Date | 日期 | 2024-01-01 |
| DateTime | 日期时间 | 2024-01-01T00:00:00Z |
| Enum | 枚举值 | active/inactive |
| JSON | JSON对象 | {"key": "value"} |
| Array | 数组 | [1, 2, 3] |

### 特殊类型

| 类型 | 说明 |
|------|------|
| Email | 邮箱地址 |
| Phone | 手机号 |
| URL | 网址 |
| IPAddress | IP地址 |
| Money | 金额（建议使用整数单位：分） |
| Percentage | 百分比 |

## 字段属性

### 必填性

| 标记 | 说明 |
|------|------|
| 是 | 必须提供，不能为空 |
| 可选 | 可以不提供，有默认值 |
| 条件必填 | 满足某条件时必填 |

### 约束

| 约束 | 说明 |
|------|------|
| 唯一 | 值不能重复 |
| 主键 | 主键字段 |
| 外键 | 关联其他表 |
| 默认值 | 不提供时的默认值 |
| 范围 | 数值范围限制 |

### 示例

| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| email | String(255) | 是 | 邮箱，唯一 |
| age | Integer | 可选，默认0 | 年龄，范围0-150 |
| status | Enum | 是 | 状态，默认active |
| name | String(100) | 是 | 名称，最长100字符 |

## 通用字段

大多数表应该包含以下字段：

```markdown
| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| id | UUID | 是 | 主键 |
| created_at | DateTime | 是 | 创建时间 |
| updated_at | DateTime | 是 | 更新时间 |
| deleted_at | DateTime | 可选 | 软删除时间 |
```

## 关系设计

### 一对多关系

```
用户 (1) ---- (N) 订单
```

在"多"的一方存储外键：

| 字段 | 类型 | 说明 |
|------|------|------|
| user_id | UUID | 外键，关联users表 |

### 多对多关系

```
用户 (N) ---- (N) 角色
```

使用中间表：

| 字段 | 类型 | 说明 |
|------|------|------|
| id | UUID | 主键 |
| user_id | UUID | 外键，关联users表 |
| role_id | UUID | 外键，关联roles表 |

### 一对一关系

```
用户 (1) ---- (1) 用户详情
```

可以在任一方存储外键，或合并到一张表。

## 软删除

建议使用软删除（is_deleted 或 deleted_at 字段）而非物理删除：

```markdown
| 字段 | 类型 | 必填 | 说明 |
|------|------|------|------|
| deleted_at | DateTime | 可选 | 删除时间，为空表示未删除 |
| is_deleted | Boolean | 是 | 是否删除，默认false |
```

查询时自动过滤已删除记录。

## 索引设计

### 何时创建索引

| 场景 | 示例 |
|------|------|
| WHERE条件字段 | user_id, status |
| 排序字段 | created_at |
| 唯一性字段 | email (唯一索引) |
| 联合查询 | (user_id, status) |

### 索引命名

| 类型 | 命名规则 | 示例 |
|------|----------|------|
| 普通索引 | idx_表名_字段 | idx_users_status |
| 唯一索引 | uk_表名_字段 | uk_users_email |
| 主键 | pk_表名 | pk_users |

## 枚举值

### 状态类

```markdown
| status | 说明 |
|--------|------|
| pending | 待处理 |
| active | 激活 |
| inactive | 禁用 |
| deleted | 已删除 |
```

### 性别

```markdown
| gender | 说明 |
|--------|------|
| male | 男 |
| female | 女 |
| other | 其他 |
| unknown | 未知 |
```

### 订单状态

```markdown
| order_status | 说明 |
|--------------|------|
| pending | 待支付 |
| paid | 已支付 |
| processing | 处理中 |
| shipped | 已发货 |
| delivered | 已送达 |
| cancelled | 已取消 |
| refunded | 已退款 |
```

## 示例：完整用户模型

```markdown
### User 用户表
| 字段 | 类型 | 必填 | 说明 | 备注 |
|------|------|------|------|------|
| id | UUID | 是 | 主键 | 系统生成 |
| email | String(255) | 是 | 邮箱 | 唯一索引 |
| username | String(50) | 是 | 用户名 | 唯一索引 |
| password_hash | String(255) | 是 | 密码hash | |
| nickname | String(100) | 可选 | 昵称 | 默认等于username |
| avatar_url | String(500) | 可选 | 头像URL | |
| phone | String(20) | 可选 | 手机号 | |
| status | Enum | 是 | 状态 | active/inactive，默认active |
| last_login_at | DateTime | 可选 | 最后登录时间 | |
| created_at | DateTime | 是 | 创建时间 | |
| updated_at | DateTime | 是 | 更新时间 | |
| deleted_at | DateTime | 可选 | 删除时间 | 软删除 |
```
