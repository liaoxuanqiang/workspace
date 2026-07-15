# RDP via GitHub Actions + Tailscale

通过 GitHub Actions 在 Windows Runner 上自动配置远程桌面（RDP），并借助 [Tailscale](https://tailscale.com/) 组网实现安全的远程访问。

## 工作原理

1. **配置 RDP** — 修改注册表启用远程桌面，禁用网络级身份认证（NLA），开放 3389 端口防火墙规则
2. **创建用户** — 自动创建管理员用户 `liaox`，密码为 8 位纯数字
3. **安装 Tailscale** — 下载并静默安装 Tailscale 客户端
4. **建立连接** — 使用 Auth Key 将 Runner 接入 Tailscale 网络，同时开启 **Subnet Router** 和 **Exit Node** 功能，获取虚拟 IP
5. **验证连通性** — 测试 3389 端口 TCP 连接是否正常
6. **保持运行** — 持续输出连接信息并保持 Runner 活跃，直到手动取消

## 前置准备

### 1. Tailscale Auth Key

前往 [Tailscale Admin Console](https://login.tailscale.com/admin/settings/keys) 生成一个 Auth Key，然后在 GitHub 仓库中添加 Secret：

- **Settings → Secrets and variables → Actions → New repository secret**
- Name: `TAILSCALE_AUTH_KEY`
- Value: 你的 Auth Key

### 2. 本机安装 Tailscale

用于连接 Runner 的客户端设备也需要安装 Tailscale 并登录同一账号。

## 使用方法

1. 进入仓库 **Actions** 页面
2. 选择 **RDP** workflow
3. 点击 **Run workflow** 手动触发
4. 等待运行完成，在日志中找到连接信息：

```
=== RDP ACCESS ===
Address:   <Tailscale IP>
Username:  liaox
Password:  <随机生成的密码>
==================
```

5. 在本地打开 Windows 远程桌面连接（`mstsc`），输入 Tailscale IP 地址
6. 使用上述用户名和密码登录

## 连接信息

| 项目 | 值 |
|------|-----|
| 用户名 | `liaox` |
| 密码 | 每次运行随机生成（8 位纯数字） |
| 地址 | Tailscale 分配的虚拟 IP（运行日志中查看） |
| 端口 | 3389 |
| 最大时长 | 3600 分钟（60 小时），超时自动断开 |

## Tailscale 功能说明

### Subnet Router（子网路由）
- 自动检测 Runner 的所有 IPv4 网卡地址并广播对应子网
- 允许 Tailscale 网络中的其他设备访问 Runner 所在子网的资源

### Exit Node（出口节点）
- 将 Runner 作为 Tailscale 网络的出口节点
- 其他设备可将流量通过此 Runner 转发到互联网

> ⚠️ 需要在 [Tailscale Admin Console](https://login.tailscale.com/admin/machines) 中手动审批 Subnet Router 和 Exit Node 的广播请求

## 文件结构

```
rdp/
├── .github/
│   └── workflows/
│       └── main.yml      # GitHub Actions 工作流定义
└── README.md
```

## 注意事项

- ⚠️ 此方案仅用于学习和测试，不建议用于生产环境
- ⚠️ 禁用 NLA 会降低安全性，请确保仅通过 Tailscale 网络访问
- ⚠️ GitHub Actions Runner 最长运行 6 小时（免费账号），`timeout-minutes` 设为 3600 以最大化利用
- ⚠️ 每次运行会重新生成密码，请从日志中获取最新凭据
- 🔑 确保客户端设备与 Runner 处于同一 Tailscale 网络中

## 终止连接

在 GitHub Actions 页面点击 **Cancel workflow** 即可终止 Runner，连接自动断开。
