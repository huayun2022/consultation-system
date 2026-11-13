# 心理咨询预约系统

来访者填表预约、凭「手机号 + 姓名」查回复；咨询师登录回复；管理员管理账号与数据。

| 项目 | 内容 |
| --- | --- |
| 正式入口（公网可用） | https://wbapp-gbhqqpulhedjeivo6lbd4j.app.workbuddy.host/?sharecode=US7YDLSDFD |
| 仓库 | https://github.com/huayun2022/consultation-system |
| 技术支持电话 | 15684510919 |
| 当前使用版本 | `cloud-version/`（云端版，2026-11-14 起） |

## 目录

| 路径 | 说明 |
| --- | --- |
| `cloud-version/index.html` | 云端版前端，HTML / CSS / JavaScript 单文件 |
| `cloud-version/vendor/index.global.js` | 云端 SDK 本地副本，CDN 加载失败时兜底 |
| `cloud-version/README.md` | 云端版使用与部署说明 |
| `cloud-version/server.md` | 数据表结构与服务端接口 |
| `index.html` | 旧版单机演示页，仅存档，已停用 |
| `.github/workflows/deploy.yml` | GitHub Pages 部署工作流 |

## 两个版本

| | 旧版（根目录 `index.html`） | 云端版（`cloud-version/`） |
| --- | --- | --- |
| 预约数据 | 浏览器 localStorage，换设备看不到 | 云端数据库，多设备共享同一份 |
| 登录 | 密码明文写在网页源码，全员共用一个密码 | 服务端 bcrypt 校验，一人一账号 |
| 查回复 | 只凭手机号 | 手机号与姓名同时匹配 |
| 状态 | 已停用 | 正在使用 |

旧版是纯静态页面，没有服务器和数据库，密码明文写在 `index.html` 第 390–392 行。这些问题在云端版都已处理，细节见 `cloud-version/server.md`。

## 关于账号密码

本仓库公开，**不存放任何初始密码**。账号由管理员在后台创建或重置，密码通过私下渠道分发；首次登录后请立即在「修改密码」中更换。旧版演示页里的弱密码属于历史遗留，与云端版无关。

## 部署

云端版运行在 WorkBuddy 托管环境，入口地址见顶部；改动前端须在托管环境重新发布，推送本仓库不会自动更新它。GitHub Pages 仍提供根目录的旧版页面（https://huayun2022.github.io/consultation-system/），仅作存档。

`deploy.yml` 原先上传的是不存在的 `consultation/` 目录，已改为仓库根目录 `.`。
