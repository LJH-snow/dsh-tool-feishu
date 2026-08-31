# dsh-tool-feishu 开发文档

## 1. 项目概览

| 项 | 内容 |
|---|---|
| 项目名 | `dsh-tool-feishu` |
| 定位 | DeepSeek Harness 的飞书/Lark 集成插件 |
| 版本 | v0.1.0 |
| 架构 | Cordis 插件 + `ctx.tools.register(defineTool(...))` |
| API | 飞书 Open API v1/v3/v4 |
| 认证 | tenant_access_token（appId + appSecret）或静态 token |

### 1.1 目录

```text
src/client.ts      FeishuClient：token 管理、fetch 注入、超时、错误映射
src/index.ts       18 个 defineTool 定义与插件 apply
tests/client.spec.ts  客户端契约测试
tests/tools.spec.ts   工具注册、凭证保护、业务值测试
examples/cordis.yml   dsh 组合配置示例
```

## 2. 技术决策

### 2.1 认证

飞书 Open API 使用 tenant_access_token 认证。客户端支持两种模式：
- appId + appSecret：自动获取并缓存 token，过期前 60 秒刷新。
- 静态 token：直接使用，适用于外部 token 管理场景。

### 2.2 工具范围

v0.1 覆盖飞书核心只读 + 消息发送：
- 群组：列表、详情、成员。
- 消息：列表、详情、发送（写操作）。
- 用户：详情、部门列表。
- 部门：详情、列表、子部门。
- 日历：列表、事件列表。
- 审批：实例列表、实例详情、同意/拒绝。

### 2.3 错误映射

| 场景 | 返回/行为 |
|---|---|
| 未配置凭证 | `{ found: false, reason }` 或 `{ ok: false, reason }` |
| 飞书 API code !== 0 | 抛 `FeishuError` |
| HTTP 4xx/5xx | 抛 `FeishuError` |

## 3. 测试

```sh
npm install
npm run typecheck
npm test
npm run build
```

当前测试覆盖：

- tenant_access_token 获取与缓存。
- 静态 token 直接使用。
- 群组列表/详情/成员映射与分页。
- 消息列表/详情/发送映射。
- 用户详情/部门列表映射。
- 日历/事件/审批实例与审批详情/同意/拒绝映射。
- 部门详情/部门列表/子部门映射。
- 凭证缺失保护与错误处理。
- 18 个工具注册、render 函数与 present 卡片。

## 4. 后续方向

- 飞书文档（docx）与多维表格（bitable）只读巡检。
- 飞书机器人事件订阅（webhook）。
- Lark 国际版 baseUrl 自动检测。
