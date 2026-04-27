# API 规格编写指南

## 基本结构

每个API规格应包含以下部分：

```
### 3.1 [API名称]
| 字段 | 值 |
|------|-----|
| Endpoint | /api/v1/xxx |
| Method | GET/POST/PUT/DELETE |
| Auth | Bearer Token / None |

#### Request
```json
{
  "field": "类型 - 说明 - 必填/可选"
}
```

#### Response - 成功 (200)
```json
{
  "code": 200,
  "message": "success",
  "data": {}
}
```

#### Response - 错误
```json
{
  "code": 400,
  "message": "错误描述"
}
```
```

## HTTP Method 选择

| Method | 用途 | 幂等性 |
|--------|------|--------|
| GET | 获取资源 | 幂等 |
| POST | 创建资源 | 非幂等 |
| PUT | 完整更新资源 | 幂等 |
| PATCH | 部分更新资源 | 非幂等 |
| DELETE | 删除资源 | 幂等 |

## URL 设计规范

### 命名规则
- 使用小写字母
- 使用复数名词表示资源集合：`/users`, `/orders`
- 使用 kebab-case：`/user-profiles`
- 层级不超过3层：`/users/{id}/orders`

### 常见模式

| 场景 | URL示例 |
|------|---------|
| 获取资源列表 | GET /users |
| 获取单个资源 | GET /users/{id} |
| 创建资源 | POST /users |
| 更新资源 | PUT /users/{id} |
| 删除资源 | DELETE /users/{id} |
| 获取子资源 | GET /users/{id}/orders |
| 特殊操作 | POST /users/{id}/activate |

## Request Body 设计

### 字段类型

| 类型 | 说明 | 示例 |
|------|------|------|
| String | 字符串 | "hello" |
| Integer | 整数 | 42 |
| Number | 数字 | 3.14 |
| Boolean | 布尔值 | true/false |
| Array | 数组 | [1, 2, 3] |
| Object | 对象 | {"key": "value"} |
| DateTime | 日期时间 | "2024-01-01T00:00:00Z" |
| UUID | 唯一标识 | "550e8400-e29b-41d4-a716-446655440000" |

### 字段标注

| 标注 | 说明 |
|------|------|
| 必填 | 必须提供，否则400 |
| 可选 | 可以不提供，有默认值 |
| 条件必填 | 满足某条件时必填 |

### 示例

```json
{
  "username": "string - 用户名 - 必填",
  "email": "string - 邮箱 - 必填",
  "age": "integer - 年龄 - 可选，默认0",
  "tags": "array[string] - 标签 - 可选",
  "profile": "object - 用户资料 - 可选"
}
```

## Response 设计

### 成功响应结构

```json
{
  "code": 200,
  "message": "success",
  "data": {
    // 实际数据
  },
  "pagination": {
    // 分页信息（如果适用）
    "page": 1,
    "page_size": 20,
    "total": 100
  }
}
```

### 错误响应结构

```json
{
  "code": 400,
  "message": "错误描述",
  "errors": [
    {
      "field": "email",
      "message": "邮箱格式不正确"
    }
  ]
}
```

### 常见HTTP状态码

| 状态码 | 含义 | 使用场景 |
|--------|------|----------|
| 200 | OK | 成功获取/更新资源 |
| 201 | Created | 成功创建资源 |
| 204 | No Content | 成功删除，无返回内容 |
| 400 | Bad Request | 请求参数错误 |
| 401 | Unauthorized | 未认证 |
| 403 | Forbidden | 无权限 |
| 404 | Not Found | 资源不存在 |
| 409 | Conflict | 资源冲突（如重复） |
| 500 | Internal Error | 服务器内部错误 |

## 认证设计

### 认证方式

| 方式 | 适用场景 |
|------|----------|
| None | 公开接口 |
| Bearer Token | 用户端API |
| API Key | 服务端API调用 |
| OAuth 2.0 | 第三方集成 |

### 示例

| 字段 | 值 |
|------|-----|
| Auth | Bearer Token |

## 分页设计

### 请求参数

| 参数 | 类型 | 说明 |
|------|------|------|
| page | integer | 页码，默认1 |
| page_size | integer | 每页数量，默认20 |

### 响应结构

```json
{
  "data": [],
  "pagination": {
    "page": 1,
    "page_size": 20,
    "total": 100,
    "total_pages": 5
  }
}
```

## 排序与过滤

### 排序
```
GET /users?sort=created_at:desc,name:asc
```

### 过滤
```
GET /users?status=active&age_gte=18
```

## 版本控制

### URL版本
```
/api/v1/users
/api/v2/users
```

### Header版本
```
Accept: application/vnd.myapp.v1+json
```
