# 示例：Spec

这是一个完整的Spec示例，对应 `demo_proposal.md` 中的Proposal。

---

# Spec: 用户登录注册系统

## 1. 概述

实现邮箱密码注册和登录功能，支持JWT Token认证，用户可修改个人资料。
目标用户为已有产品用户，业务价值在于完善用户体系，支持个性化体验。

## 2. 功能列表

### 2.1 用户注册

- 描述：用户通过邮箱密码注册账户
- 优先级：P0
- 详细说明：
  - 输入邮箱、密码、确认密码
  - 校验邮箱格式（正则验证）
  - 校验密码强度（至少8位，包含数字和字母）
  - 校验两次密码一致
  - 校验邮箱唯一性
  - 创建用户记录，密码bcrypt加密存储

### 2.2 用户登录

- 描述：用户通过邮箱密码登录
- 优先级：P0
- 详细说明：
  - 输入邮箱、密码
  - 校验凭证，返回JWT Token
  - Token有效期7天
  - 记录最后登录时间

### 2.3 修改昵称

- 描述：用户修改个人昵称
- 优先级：P1
- 详细说明：
  - 最长20字符
  - 仅登录用户可修改

### 2.4 修改头像

- 描述：用户上传头像
- 优先级：P1
- 详细说明：
  - 支持jpg、png
  - 最大2MB
  - 上传到云存储，返回URL

---

## 3. API规格

### 3.1 用户注册

| 字段 | 值 |
|------|-----|
| Endpoint | /api/v1/users/register |
| Method | POST |
| Auth | None |

#### Request
```json
{
  "email": "string - 邮箱 - 必填",
  "password": "string - 密码 - 必填",
  "confirm_password": "string - 确认密码 - 必填"
}
```

#### Response - 成功 (201)
```json
{
  "code": 201,
  "message": "注册成功",
  "data": {
    "user_id": "uuid",
    "email": "user@example.com"
  }
}
```

#### Response - 错误
```json
{
  "code": 400,
  "message": "注册失败",
  "errors": [
    {"field": "email", "message": "邮箱格式不正确"}
  ]
}
```

### 3.2 用户登录

| 字段 | 值 |
|------|-----|
| Endpoint | /api/v1/users/login |
| Method | POST |
| Auth | None |

#### Request
```json
{
  "email": "string - 邮箱 - 必填",
  "password": "string - 密码 - 必填"
}
```

#### Response - 成功 (200)
```json
{
  "code": 200,
  "message": "登录成功",
  "data": {
    "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
    "expires_in": 604800
  }
}
```

#### Response - 错误
```json
{
  "code": 401,
  "message": "邮箱或密码错误"
}
```

### 3.3 修改昵称

| 字段 | 值 |
|------|-----|
| Endpoint | /api/v1/users/me/nickname |
| Method | PUT |
| Auth | Bearer Token |

#### Request
```json
{
  "nickname": "string - 昵称 - 必填，最多20字符"
}
```

#### Response - 成功 (200)
```json
{
  "code": 200,
  "message": "修改成功",
  "data": {
    "nickname": "新昵称"
  }
}
```

### 3.4 上传头像

| 字段 | 值 |
|------|-----|
| Endpoint | /api/v1/users/me/avatar |
| Method | POST |
| Auth | Bearer Token |
| Content-Type | multipart/form-data |

#### Request
```json
{
  "avatar": "file - 头像文件 - 必填，支持jpg/png，最大2MB"
}
```

#### Response - 成功 (200)
```json
{
  "code": 200,
  "message": "上传成功",
  "data": {
    "avatar_url": "https://cdn.example.com/avatars/xxx.jpg"
  }
}
```

---

## 4. 数据模型

### 4.1 User 用户表

| 字段 | 类型 | 必填 | 说明 | 备注 |
|------|------|------|------|------|
| id | UUID | 是 | 主键 | 系统生成 |
| email | String(255) | 是 | 邮箱 | 唯一索引 |
| password_hash | String(255) | 是 | 密码hash | bcrypt加密 |
| nickname | String(20) | 可选 | 昵称 | 默认等于email前缀 |
| avatar_url | String(500) | 可选 | 头像URL | |
| status | Enum | 是 | 状态 | active/inactive，默认active |
| last_login_at | DateTime | 可选 | 最后登录时间 | |
| created_at | DateTime | 是 | 创建时间 | |
| updated_at | DateTime | 是 | 更新时间 | |
| deleted_at | DateTime | 可选 | 删除时间 | 软删除 |

### 4.2 AccessToken 访问令牌表

| 字段 | 类型 | 必填 | 说明 | 备注 |
|------|------|------|------|------|
| id | UUID | 是 | 主键 | 系统生成 |
| user_id | UUID | 是 | 用户ID | 外键，关联users表 |
| token | String(512) | 是 | Token | |
| expires_at | DateTime | 是 | 过期时间 | |
| created_at | DateTime | 是 | 创建时间 | |
| deleted_at | DateTime | 可选 | 删除时间 | 软删除 |

---

## 5. 业务流程

### 5.1 注册流程
```
[用户] -> 输入信息 -> [接口校验] -> [业务校验] -> [创建用户] -> [返回成功]
                                        ↓
                                   [业务校验失败] -> [返回错误]
```

### 5.2 登录流程
```
[用户] -> 输入凭证 -> [查找用户] -> [验证密码] -> [生成Token] -> [返回成功]
                                              ↓
                                         [验证失败] -> [返回错误]
```

---

## 6. 验收标准

| ID | 功能点 | 验收条件 | 优先级 |
|----|--------|----------|--------|
| AC1 | 注册 | 输入有效邮箱和密码，返回201 | P0 |
| AC2 | 注册 | 输入已存在邮箱，返回409 | P0 |
| AC3 | 注册 | 密码少于8位，返回400 | P0 |
| AC4 | 登录 | 输入正确凭证，返回200和Token | P0 |
| AC5 | 登录 | 输入错误密码，返回401 | P0 |
| AC6 | 登录 | 输入不存在邮箱，返回401 | P0 |
| AC7 | 修改昵称 | 未登录，返回401 | P0 |
| AC8 | 修改昵称 | 昵称超长，返回400 | P1 |
| AC9 | 上传头像 | 文件超过2MB，返回400 | P1 |
| AC10 | Token | 过期后请求，返回401 | P0 |

---

## 7. 边界条件

- 邮箱格式：需符合RFC 5322标准
- 密码强度：至少8位，包含数字和字母
- 昵称空白：默认使用邮箱前缀
- 头像格式：仅支持jpg、png，其他格式返回400
- 并发注册：同一邮箱同时请求，只有一个成功
- Token刷新：需在过期前7天内刷新

---

## 8. 潜在风险

| 风险 | 影响 | 缓解措施 |
|------|------|----------|
| 密码泄露 | 高 | 使用bcrypt加密，不记录明文 |
| Token被劫持 | 高 | HTTPS传输，设置短期过期 |
| 撞库攻击 | 中 | 登录失败5次后需要验证码 |
| 头像存储 | 中 | 使用云存储，配额限制 |

---

## 9. 监控与日志

- 注册API：记录注册成功、失败
- 登录API：记录登录成功、失败
- 错误日志：记录所有400/401错误
- 性能指标：API响应时间<200ms

---

**注意**：这个Spec已经过用户确认，可以进入Tasks阶段。

对应Proposal：`examples/demo_proposal.md`