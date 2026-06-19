# GitHub Actions 部署说明

这个项目是静态网页游戏，部署方式是：

1. GitHub Actions 手动触发部署工作流并构建 Docker 镜像。
2. Actions 通过 SSH 把镜像传到你的服务器。
3. 服务器用 Docker 启动一个 Nginx 容器。
4. 浏览器访问 `http://服务器IP:5000`。

默认不在每次 push 时自动部署，避免仓库还没配置 SSH secrets 时让 Actions 持续失败。配置完成后，在 GitHub 仓库里打开 `Actions` -> `Deploy` -> `Run workflow` 手动部署。

## 服务器准备

服务器需要能 SSH 登录，并且已经安装 Docker。登录服务器后可以用下面命令检查：

```bash
docker --version
```

如果当前登录用户运行 Docker 提示没权限，可以先用有 Docker 权限的用户作为 `DEPLOY_USER`，或者把该用户加入 Docker 用户组。

## 从 SSH 别名查出要填的值

GitHub Actions 运行在 GitHub 的机器上，不能直接使用你电脑里的 SSH 别名，所以要把别名里的真实信息填到 GitHub Secrets。

在你自己的电脑上运行，把 `your-alias` 换成你的 SSH 别名：

```powershell
ssh -G your-alias | Select-String '^(hostname|port|user|identityfile)\s'
```

对应关系：

| 输出项 | GitHub 里要填 |
| --- | --- |
| `hostname` | `DEPLOY_HOST` |
| `port` | `DEPLOY_PORT` |
| `user` | `DEPLOY_USER` |
| 私钥文件内容 | `DEPLOY_SSH_KEY` |

## 创建专门给 GitHub Actions 用的 SSH key

如果你现在是密码登录，GitHub Actions 没法帮你输入密码，建议创建一个专用 key：

```powershell
ssh-keygen -t ed25519 -C "github-actions-hecheng-cuizhiyu" -f $env:USERPROFILE\.ssh\hecheng_cuizhiyu_deploy
type $env:USERPROFILE\.ssh\hecheng_cuizhiyu_deploy.pub | ssh your-alias "mkdir -p ~/.ssh && cat >> ~/.ssh/authorized_keys && chmod 700 ~/.ssh && chmod 600 ~/.ssh/authorized_keys"
ssh -i $env:USERPROFILE\.ssh\hecheng_cuizhiyu_deploy your-user@your-host -p your-port
```

测试能登录后，把私钥内容填到 GitHub Secret `DEPLOY_SSH_KEY`：

```powershell
Get-Content $env:USERPROFILE\.ssh\hecheng_cuizhiyu_deploy -Raw
```

## GitHub 需要配置的 Secrets

进入仓库页面：

`Settings` -> `Secrets and variables` -> `Actions` -> `New repository secret`

添加：

| Secret | 示例 |
| --- | --- |
| `DEPLOY_HOST` | `203.0.113.10` |
| `DEPLOY_PORT` | `22` |
| `DEPLOY_USER` | `deploy` |
| `DEPLOY_SSH_KEY` | 上一步私钥完整内容 |

## 可选：修改访问端口

默认访问端口是 `5000`，也就是：

```text
http://服务器IP:5000
```

如果想换成别的端口，进入：

`Settings` -> `Secrets and variables` -> `Actions` -> `Variables` -> `New repository variable`

添加变量：

```text
APP_PORT=你的端口
```

记得在服务器防火墙或云服务器安全组里放行这个端口。

## 手动部署

配置好 Secrets 后，在 GitHub 仓库里打开：

`Actions` -> `Deploy` -> `Run workflow`

手动触发一次。
