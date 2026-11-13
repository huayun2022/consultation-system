# 云端后端结构

应用 ID：`wbapp_GbHQQpuLheDjEIvo6lBD4J`（WorkBuddy 云服务，一个应用对应一套独立环境）。

## 数据表

三张表都在 `public` schema，全部开启 RLS 且不给 `anon` / `authenticated` 任何直接读写权限，只能通过下面的函数访问。

| 表 | 主要字段 |
| --- | --- |
| `staff` | id, username, display_name, role(`admin` / `counselor`), password_hash, must_change, failed_attempts, locked_until, active, created_at |
| `appointments` | id, name, gender, age, phone, visit_date, reason, reply, status, replied_by, replied_at, created_at |
| `staff_sessions` | token_hash, staff_id, expires_at, created_at |

## 服务端函数

全部是 `SECURITY DEFINER` + `SET search_path = public, pg_temp`，由前端用 `cloud.database.rpc()` 调用。统一参数是 `p_action`（动作名）和 `p_payload`（JSON），受保护的动作在 payload 里带 `token`。函数名与前端 `RPC_OF` 一一对应。

| 函数 | 动作 | 权限 |
| --- | --- | --- |
| `staff_identity` | 会话 token 校验 | 内部调用 |
| `guest_api` | `submit` / `lookup` | 无需登录 |
| `auth_api` | `login` / `logout` / `change_password` | 登录后 |
| `staff_api` | `list` / `reply` / `overview` | 咨询师或管理员 |
| `admin_data_api` | `export` / `clear` | 仅管理员 |
| `admin_staff_api` | `staff_list` / `staff_add` / `staff_toggle` / `staff_reset_password` | 仅管理员 |

返回统一是 `{ok: true, ...}` 或 `{ok: false, message, code?}`；会话失效为 `UNAUTHENTICATED`，越权为 `FORBIDDEN`。前端收到 `UNAUTHENTICATED` 会清除本地会话并回到登录页。

## 安全设计

- 密码用 pgcrypto 的 `crypt(密码, gen_salt('bf', 8))` 生成 bcrypt 哈希，网页和接口都拿不到明文。
- 登录签发 32 字节随机 token，数据库只存 sha256 摘要，12 小时过期，退出即删除。
- 连续 5 次密码错误锁定 15 分钟；管理员重置密码、停用账号、修改密码都会让相关会话失效。
- 来访者查询必须手机号与姓名同时匹配，最多返回 20 条，且不返回其他人的手机号。
- 咨询师列表最多 200 条；姓名、手机号、日期、回复长度等都在服务端校验。
- 清空数据需要前端二次确认，并由服务端再校验 `confirm` 标记。

## 已验证

- 管理员、咨询师登录分别返回 `admin` / `counselor`；咨询师调用管理员接口返回 `FORBIDDEN`。
- 预约提交、按手机号加姓名查询、列表、回复、统计、导出、账号列表全部读回成功，测试数据已清除。
- 用真实 SDK 从公网域名依次调用 `auth_api` 登录、`staff_api` 统计、`admin_staff_api` 账号列表、`guest_api` 匿名查询、`auth_api` 退出，均成功，说明网关、密钥与来源校验通过。
