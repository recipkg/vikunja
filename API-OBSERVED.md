# 账户、项目、任务与权限：局部 API 合同

观察目标是 [固定提交](https://github.com/go-vikunja/vikunja/tree/e37ebc0d10db366ff111bc4bb74a696d7cfc912e) 的 Linux 后端，使用 SQLite、默认本地认证、禁用邮件的隔离配置。Go 编译时前端 embed 使用占位页面，未观察浏览器界面。除明确注明的 `/api/v2` 外，下文路径以 `/api/v1` 为前缀。下述状态和字段只覆盖具体样例，不推断其他认证、数据库、版本或配置。

## 账户与项目

- `POST /register` 的 JSON 提供 `username`、`password`、`email`；启用注册、邮箱未要求确认时创建账户，样例返回 200 和含 id、username 的用户对象。`POST /login` 提供用户名和密码，返回 200 和 `token`。后续请求以 `Authorization: Bearer <token>` 认证；未认证 `GET /projects` 返回 401。密码、真实 token 和数据库不得进入配方。
- 认证用户 `PUT /projects`，例如 `{"title":"Work"}`，样例返回 201 和带 `id` 的项目。该样例响应中的四个默认视图 `view_kind` 为 `list`、`gantt`、`table`、`kanban`；视图元数据不证明这些界面或拖动操作已实现。
- 使用返回的项目 ID，`PUT /projects/{project}/tasks` 与 `{"title":"First task","due_date":"2026-12-01T12:00:00Z"}` 创建任务；样例返回 201，包含 id、project_id、title、done=false 和原截止时间。`GET /tasks/{task}` 返回该任务。字段、格式与权限尚需覆盖更多情况。

## 项目用户权限

对于仅由 Alice 创建、尚未分享的项目和任务，Bob 的 `GET /projects/{id}` 与 `GET /tasks/{id}` 在样例中均返回 403。Alice `PUT /projects/{id}/users` 提交 `{"username":"bob","permission":0}`，返回 201；Bob 可读取任务但不能以 v1 POST 更新，后者返回 403。

Alice `POST /projects/{id}/users/bob` 提交 `{"permission":1}` 返回 200 后，Bob 对任务的 v2 PATCH 可以成功。再次提交 `{"permission":0}` 后，Bob 的同类 PATCH 返回 403。正常停止并重启同一后端、重新登录后任务和只读共享仍存在；Alice `DELETE /projects/{id}/users/bob` 返回 200，之后 Bob 读取任务返回 403。此处只验证直接用户共享的 0 和 1，不覆盖管理员、团队、层级继承或竞态。

## v1 POST 与 v2 PATCH 的重要区别

样例中对有截止时间的普通任务执行 v1 `POST /tasks/{id}`，请求体只有 `{"done":true}`，返回 200；之后 GET 显示 `done=true`，但 `due_date` 被清为 `0001-01-01T00:00:00Z`。不能把此 v1 请求当作保留所有未发送字段的局部更新。不要据此推断其他每个字段的具体处理。

另建相同截止时间的任务，通过 `PATCH /api/v2/tasks/{id}` 仅提交 `{"done":true}`，本次观察返回 200；v1 GET 显示任务已完成且截止时间保留。给 Bob 读写权限后，他 PATCH 标题也保留了截止时间。v2 源码将 PATCH 作为局部更新路径；并发冲突、null、其他字段及重复任务未实测。

重建须实现并测试以上可观察行为，不能仅依据路由名称或状态码宣称完整兼容。固定源码的服务端路由及 v2 更新定义分别见 [`pkg/routes/routes.go`](https://github.com/go-vikunja/vikunja/blob/e37ebc0d10db366ff111bc4bb74a696d7cfc912e/pkg/routes/routes.go) 和 [`pkg/routes/api/v2/tasks.go`](https://github.com/go-vikunja/vikunja/blob/e37ebc0d10db366ff111bc4bb74a696d7cfc912e/pkg/routes/api/v2/tasks.go)。
