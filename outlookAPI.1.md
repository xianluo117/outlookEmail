# Outlook 邮件管理工具 API 文档

## 1. 基本信息

- 服务地址：`http://127.0.0.1:5001`
- Web 框架：`Flask`
- 代码入口：[`web_outlook_app.py`](web_outlook_app.py)
- API 鉴权：所有 [`/api/*`](web_outlook_app.py:736) 接口支持 Session 或 API Key 鉴权，逻辑见 [`api_key_required()`](web_outlook_app.py:709) 与 [`login_required()`](web_outlook_app.py:732)
- 默认登录密码：`admin123`，可通过 [`/api/settings`](web_outlook_app.py:1535) 修改
- 系统 API Key：可通过 [`/api/settings`](web_outlook_app.py:1535) 配置
- 默认返回格式：`application/json`
- 导出接口返回格式：`text/plain; charset=utf-8`

## 2. 认证说明

### 2.1 认证方式

当前 [`/api/*`](web_outlook_app.py:736) 接口支持两种认证方式：

1. 浏览器登录 Session
2. API Key

API Key 提取顺序见 [`get_request_api_key()`](web_outlook_app.py:697)：

- `Authorization: Bearer <key>`
- `X-API-Key: <key>`
- 查询参数 `api_key=<key>`

接口鉴权逻辑见 [`api_key_required()`](web_outlook_app.py:709)：

- 已登录 Session 有效时直接放行
- 未登录时校验系统 API Key
- API Key 未配置或错误时返回 `401`

### 2.2 登录

接口：`POST /login`

源码位置：[`login()`](web_outlook_app.py:748)

请求体示例：

```json
{
  "password": "admin123"
}
```

成功响应示例：

```json
{
  "success": true,
  "message": "登录成功"
}
```

失败响应示例：

```json
{
  "success": false,
  "error": "密码错误"
}
```

### 2.3 API 鉴权失败响应

当访问受保护接口且 Session 与 API Key 都未通过时，由 [`api_key_required()`](web_outlook_app.py:709) 返回：

```json
{
  "success": false,
  "error": "API 鉴权失败",
  "need_login": true,
  "need_api_key": true,
  "auth_methods": ["session", "api_key"]
}
```

HTTP 状态码：`401`

### 2.4 认证调用示例

Bearer Token：

```bash
curl http://127.0.0.1:5001/api/groups \
  -H "Authorization: Bearer your-api-key"
```

X-API-Key：

```bash
curl http://127.0.0.1:5001/api/groups \
  -H "X-API-Key: your-api-key"
```

---

## 3. 页面与会话接口

### 3.1 获取登录页

- 方法：`GET`
- 路径：`/login`
- 说明：返回登录页面 HTML
- 源码：[`login()`](web_outlook_app.py:686)

### 3.2 登录

- 方法：`POST`
- 路径：`/login`
- 说明：提交密码，写入会话
- 源码：[`login()`](web_outlook_app.py:686)

### 3.3 退出登录

- 方法：`GET`
- 路径：`/logout`
- 说明：清除登录状态并跳转登录页
- 源码：[`logout()`](web_outlook_app.py:707)

### 3.4 获取主页

- 方法：`GET`
- 路径：`/`
- 说明：返回主页 HTML
- 源码：[`index()`](web_outlook_app.py:715)

---

## 4. 分组接口

### 4.1 获取所有分组

- 方法：`GET`
- 路径：`/api/groups`
- 源码：[`api_get_groups()`](web_outlook_app.py:724)

成功响应示例：

```json
{
  "success": true,
  "groups": [
    {
      "id": 1,
      "name": "默认分组",
      "description": "未分组的邮箱",
      "color": "#666666",
      "is_system": 0,
      "created_at": "2026-01-01 00:00:00",
      "account_count": 3
    }
  ]
}
```

### 4.2 获取单个分组

- 方法：`GET`
- 路径：`/api/groups/<group_id>`
- 源码：[`api_get_group()`](web_outlook_app.py:739)

成功响应示例：

```json
{
  "success": true,
  "group": {
    "id": 1,
    "name": "默认分组",
    "description": "未分组的邮箱",
    "color": "#666666",
    "is_system": 0,
    "created_at": "2026-01-01 00:00:00",
    "account_count": 3
  }
}
```

失败响应示例：

```json
{
  "success": false,
  "error": "分组不存在"
}
```

### 4.3 创建分组

- 方法：`POST`
- 路径：`/api/groups`
- 源码：[`api_add_group()`](web_outlook_app.py:750)

请求体：

```json
{
  "name": "业务组",
  "description": "业务邮箱",
  "color": "#1a1a1a"
}
```

成功响应：

```json
{
  "success": true,
  "message": "分组创建成功",
  "group_id": 3
}
```

失败响应：

```json
{
  "success": false,
  "error": "分组名称不能为空"
}
```

或：

```json
{
  "success": false,
  "error": "分组名称已存在"
}
```

### 4.4 更新分组

- 方法：`PUT`
- 路径：`/api/groups/<group_id>`
- 源码：[`api_update_group()`](web_outlook_app.py:769)

请求体：

```json
{
  "name": "业务组",
  "description": "业务邮箱分组",
  "color": "#00bcf2"
}
```

成功响应：

```json
{
  "success": true,
  "message": "分组更新成功"
}
```

失败响应：

```json
{
  "success": false,
  "error": "更新失败"
}
```

### 4.5 删除分组

- 方法：`DELETE`
- 路径：`/api/groups/<group_id>`
- 源码：[`api_delete_group()`](web_outlook_app.py:787)

说明：

- `1` 号默认分组不可删除
- 删除后，分组下账号会迁移到默认分组

成功响应：

```json
{
  "success": true,
  "message": "分组已删除，邮箱已移至默认分组"
}
```

失败响应：

```json
{
  "success": false,
  "error": "默认分组不能删除"
}
```

### 4.6 导出分组账号

- 方法：`GET`
- 路径：`/api/groups/<group_id>/export`
- 源码：[`api_export_group()`](web_outlook_app.py:800)
- 返回：TXT 文件下载

导出内容格式：

```text
email----password----client_id----refresh_token
```

失败响应：

```json
{
  "success": false,
  "error": "该分组下没有邮箱账号"
}
```

---

## 5. 账号接口

### 5.1 获取账号列表

- 方法：`GET`
- 路径：`/api/accounts`
- 源码：[`api_get_accounts()`](web_outlook_app.py:929)

查询参数：

- `group_id`：可选，按分组筛选

请求示例：

```http
GET /api/accounts?group_id=1
```

成功响应示例：

```json
{
  "success": true,
  "accounts": [
    {
      "id": 1,
      "email": "demo@example.com",
      "client_id": "12345678...",
      "group_id": 1,
      "group_name": "默认分组",
      "group_color": "#666666",
      "remark": "测试账号",
      "status": "active",
      "created_at": "2026-01-01 00:00:00",
      "updated_at": "2026-01-01 00:00:00"
    }
  ]
}
```

说明：列表接口会隐藏敏感字段，不返回完整 `password` 和 `refresh_token`。

### 5.2 获取单个账号详情

- 方法：`GET`
- 路径：`/api/accounts/<account_id>`
- 源码：[`api_get_account()`](web_outlook_app.py:954)

成功响应示例：

```json
{
  "success": true,
  "account": {
    "id": 1,
    "email": "demo@example.com",
    "password": "password123",
    "client_id": "client_id_value",
    "refresh_token": "refresh_token_value",
    "group_id": 1,
    "group_name": "默认分组",
    "remark": "测试账号",
    "status": "active",
    "created_at": "2026-01-01 00:00:00",
    "updated_at": "2026-01-01 00:00:00"
  }
}
```

失败响应：

```json
{
  "success": false,
  "error": "账号不存在"
}
```

### 5.3 添加账号

- 方法：`POST`
- 路径：`/api/accounts`
- 源码：[`api_add_account()`](web_outlook_app.py:980)

请求体字段：

- `account_string`：必填，支持单行或多行
- `group_id`：可选，默认 `1`

单条导入格式：

```text
邮箱----密码----client_id----refresh_token
```

请求体示例：

```json
{
  "account_string": "demo@example.com----pass123----clientid123----refreshtoken123",
  "group_id": 1
}
```

批量导入示例：

```json
{
  "account_string": "a@example.com----pass1----client1----token1\nb@example.com----pass2----client2----token2",
  "group_id": 2
}
```

成功响应：

```json
{
  "success": true,
  "message": "成功添加 2 个账号"
}
```

失败响应：

```json
{
  "success": false,
  "error": "没有新账号被添加（可能格式错误或已存在）"
}
```

### 5.4 更新账号

- 方法：`PUT`
- 路径：`/api/accounts/<account_id>`
- 源码：[`api_update_account()`](web_outlook_app.py:1012)

#### 方式 A：完整更新

请求体示例：

```json
{
  "email": "demo@example.com",
  "password": "password123",
  "client_id": "client_id_value",
  "refresh_token": "refresh_token_value",
  "group_id": 1,
  "remark": "新备注",
  "status": "active"
}
```

成功响应：

```json
{
  "success": true,
  "message": "账号更新成功"
}
```

失败响应：

```json
{
  "success": false,
  "error": "邮箱、Client ID 和 Refresh Token 不能为空"
}
```

#### 方式 B：仅更新状态

请求体示例：

```json
{
  "status": "inactive"
}
```

成功响应：

```json
{
  "success": true,
  "message": "状态更新成功"
}
```

### 5.5 按 ID 删除账号

- 方法：`DELETE`
- 路径：`/api/accounts/<account_id>`
- 源码：[`api_delete_account()`](web_outlook_app.py:1055)

成功响应：

```json
{
  "success": true
}
```

失败响应：

```json
{
  "success": false,
  "error": "删除失败"
}
```

### 5.6 按邮箱地址删除账号

- 方法：`DELETE`
- 路径：`/api/accounts/email/<email_addr>`
- 源码：[`api_delete_account_by_email()`](web_outlook_app.py:1065)

成功响应：

```json
{
  "success": true
}
```

### 5.7 导出全部账号

- 方法：`GET`
- 路径：`/api/accounts/export`
- 源码：[`api_export_all_accounts()`](web_outlook_app.py:843)
- 返回：TXT 文件下载

### 5.8 导出选中分组账号

- 方法：`POST`
- 路径：`/api/accounts/export-selected`
- 源码：[`api_export_selected_accounts()`](web_outlook_app.py:881)
- 返回：TXT 文件下载

请求体示例：

```json
{
  "group_ids": [1, 2]
}
```

失败响应：

```json
{
  "success": false,
  "error": "请选择要导出的分组"
}
```

---

## 6. 邮件接口

### 6.1 获取邮件列表

- 方法：`GET`
- 路径：`/api/emails/<email_addr>`
- 源码：[`api_get_emails()`](web_outlook_app.py:1077)

查询参数：

- `method`：可选，默认 `graph`，支持 `graph`
- `top`：可选，默认 `20`

说明：

- 当 `method=graph` 时，优先使用 Graph API
- Graph API 获取失败时，自动回退到 IMAP

请求示例：

```http
GET /api/emails/demo@example.com?method=graph&top=10
```

成功响应示例：

```json
{
  "success": true,
  "emails": [
    {
      "id": "message-id",
      "subject": "测试邮件",
      "from": "sender@example.com",
      "date": "2026-01-01T00:00:00Z",
      "is_read": false,
      "has_attachments": false,
      "body_preview": "邮件预览"
    }
  ],
  "method": "Graph API"
}
```

IMAP 返回结构示例：

```json
{
  "success": true,
  "emails": [
    {
      "id": "1",
      "subject": "测试邮件",
      "from": "sender@example.com",
      "date": "Mon, 01 Jan 2026 08:00:00 +0800",
      "body_preview": "邮件正文预览..."
    }
  ],
  "method": "IMAP"
}
```

失败响应：

```json
{
  "success": false,
  "error": "获取邮件失败，请检查账号配置"
}
```

### 6.2 获取邮件详情

- 方法：`GET`
- 路径：`/api/email/<email_addr>/<message_id>`
- 源码：[`api_get_email_detail()`](web_outlook_app.py:1114)

查询参数：

- `method`：可选，默认 `graph`

成功响应示例：

```json
{
  "success": true,
  "email": {
    "id": "message-id",
    "subject": "测试邮件",
    "from": "sender@example.com",
    "to": "receiver@example.com",
    "cc": "copy@example.com",
    "date": "2026-01-01T00:00:00Z",
    "body": "<p>正文</p>",
    "body_type": "html"
  }
}
```

失败响应：

```json
{
  "success": false,
  "error": "获取邮件详情失败"
}
```

---

## 7. 临时邮箱接口

### 7.1 获取临时邮箱列表

- 方法：`GET`
- 路径：`/api/temp-emails`
- 源码：[`api_get_temp_emails()`](web_outlook_app.py:1345)

成功响应示例：

```json
{
  "success": true,
  "emails": [
    {
      "id": 1,
      "email": "temp@example.com",
      "status": "active",
      "created_at": "2026-01-01 00:00:00",
      "updated_at": "2026-01-01 00:00:00"
    }
  ]
}
```

### 7.2 生成临时邮箱

- 方法：`POST`
- 路径：`/api/temp-emails/generate`
- 源码：[`api_generate_temp_email()`](web_outlook_app.py:1353)

请求体示例：

```json
{
  "prefix": "demo",
  "domain": "mail.example.com"
}
```

说明：

- `prefix`、`domain` 均可省略
- 实际生成逻辑依赖 GPTMail 服务

成功响应：

```json
{
  "success": true,
  "email": "demo@mail.example.com",
  "message": "临时邮箱创建成功"
}
```

失败响应：

```json
{
  "success": false,
  "error": "生成临时邮箱失败，请稍后重试"
}
```

### 7.3 删除临时邮箱

- 方法：`DELETE`
- 路径：`/api/temp-emails/<email_addr>`
- 源码：[`api_delete_temp_email()`](web_outlook_app.py:1372)

成功响应：

```json
{
  "success": true,
  "message": "临时邮箱已删除"
}
```

### 7.4 获取临时邮箱邮件列表

- 方法：`GET`
- 路径：`/api/temp-emails/<email_addr>/messages`
- 源码：[`api_get_temp_email_messages()`](web_outlook_app.py:1382)

成功响应示例：

```json
{
  "success": true,
  "emails": [
    {
      "id": "message-id",
      "from": "sender@example.com",
      "subject": "验证码",
      "body_preview": "您的验证码是 123456",
      "date": "2026-01-01 00:00:00",
      "timestamp": 1735689600,
      "has_html": 0
    }
  ],
  "count": 1,
  "method": "GPTMail"
}
```

### 7.5 获取临时邮件详情

- 方法：`GET`
- 路径：`/api/temp-emails/<email_addr>/messages/<message_id>`
- 源码：[`api_get_temp_email_message_detail()`](web_outlook_app.py:1413)

成功响应示例：

```json
{
  "success": true,
  "email": {
    "id": "message-id",
    "from": "sender@example.com",
    "to": "temp@example.com",
    "subject": "验证码",
    "body": "<p>您的验证码是 123456</p>",
    "body_type": "html",
    "date": "2026-01-01 00:00:00",
    "timestamp": 1735689600
  }
}
```

失败响应：

```json
{
  "success": false,
  "error": "邮件不存在"
}
```

### 7.6 删除单封临时邮件

- 方法：`DELETE`
- 路径：`/api/temp-emails/<email_addr>/messages/<message_id>`
- 源码：[`api_delete_temp_email_message()`](web_outlook_app.py:1443)

成功响应：

```json
{
  "success": true,
  "message": "邮件已删除"
}
```

### 7.7 清空临时邮箱邮件

- 方法：`DELETE`
- 路径：`/api/temp-emails/<email_addr>/clear`
- 源码：[`api_clear_temp_email_messages()`](web_outlook_app.py:1454)

成功响应：

```json
{
  "success": true,
  "message": "邮件已清空"
}
```

失败响应：

```json
{
  "success": false,
  "error": "清空失败"
}
```

### 7.8 刷新临时邮箱邮件

- 方法：`POST`
- 路径：`/api/temp-emails/<email_addr>/refresh`
- 源码：[`api_refresh_temp_email_messages()`](web_outlook_app.py:1468)

成功响应示例：

```json
{
  "success": true,
  "emails": [
    {
      "id": "message-id",
      "from": "sender@example.com",
      "subject": "验证码",
      "body_preview": "您的验证码是 123456",
      "date": "2026-01-01 00:00:00",
      "timestamp": 1735689600,
      "has_html": 0
    }
  ],
  "count": 1,
  "new_count": 1,
  "method": "GPTMail"
}
```

失败响应：

```json
{
  "success": false,
  "error": "获取邮件失败"
}
```

---

## 8. 设置接口

### 8.1 获取系统设置

- 方法：`GET`
- 路径：`/api/settings`
- 源码：[`api_get_settings()`](web_outlook_app.py:1503)

成功响应示例：

```json
{
  "success": true,
  "settings": {
    "login_password_masked": "a******3",
    "gptmail_api_key": "gpt-test",
    "api_auth_key_masked": "abcd********wxyz",
    "api_auth_key_configured": true
  }
}
```

说明：

- 返回 `login_password_masked`，不会返回明文登录密码
- 返回 `api_auth_key_masked` 与 `api_auth_key_configured`，不会返回明文系统 API Key
- 当前仍会返回 `gptmail_api_key` 明文值

### 8.2 更新系统设置

- 方法：`PUT`
- 路径：`/api/settings`
- 源码：[`api_update_settings()`](web_outlook_app.py:1518)

请求体示例：

```json
{
  "login_password": "newpass123",
  "gptmail_api_key": "gpt-prod-key",
  "api_auth_key": "your-secure-api-key"
}
```

成功响应：

```json
{
  "success": true,
  "message": "已更新：登录密码, GPTMail API Key"
}
```

失败响应示例：

```json
{
  "success": false,
  "error": "密码长度至少为 4 位"
}
```

或：

```json
{
  "success": false,
  "error": "没有需要更新的设置"
}
```

---

## 9. 通用调用示例

### 9.1 使用 API Key 调用接口

```bash
curl http://127.0.0.1:5001/api/groups \
  -H "Authorization: Bearer your-api-key"
```

### 9.2 登录后调用接口

```bash
curl -X POST http://127.0.0.1:5001/login \
  -H "Content-Type: application/json" \
  -c cookies.txt \
  -d "{\"password\":\"admin123\"}"
```

```bash
curl http://127.0.0.1:5001/api/groups \
  -b cookies.txt
```

### 9.3 添加账号

```bash
curl -X POST http://127.0.0.1:5001/api/accounts \
  -H "Content-Type: application/json" \
  -b cookies.txt \
  -d "{\"account_string\":\"demo@example.com----pass123----clientid----token123\",\"group_id\":1}"
```

### 9.4 获取邮件列表

```bash
curl "http://127.0.0.1:5001/api/emails/demo@example.com?method=graph&top=10" \
  -b cookies.txt
```

---

## 10. 实现备注

- 服务监听端口定义为 `5001`，见 [`port = 5001`](web_outlook_app.py:1645)
- 服务启动调用见 [`app.run()`](web_outlook_app.py:1654)
- 所有 [`/api/*`](web_outlook_app.py:736) 路由支持 Session 或 API Key 鉴权
- 分组导出、账号导出、选中分组导出均返回 TXT 文件，不是 JSON
- 邮件接口优先 Graph API，失败后回退 IMAP，见 [`api_get_emails()`](web_outlook_app.py:1160) 与 [`api_get_email_detail()`](web_outlook_app.py:1197)
- 临时邮箱能力依赖 GPTMail 外部服务封装，入口见 [`gptmail_request()`](web_outlook_app.py:1233)
