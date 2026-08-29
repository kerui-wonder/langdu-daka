# 朗读打卡台 · 更新指南

> 项目目录：`C:\Users\kqm\.zcode\workspace\default`
> 所有改动只发生在 `recorder.html`（单文件应用），改完按下面流程发布即可。

## 〇、正式地址与发布（最重要）

- **固定网址：https://kerui-wonder.github.io/langdu-daka/recorder.html**（GitHub Pages，免费、HTTPS、不依赖本机开机）
- **发布/更新只需一条命令**：`python deploy.py`（自动把 8 个文件传到 GitHub 仓库 `kerui-wonder/langdu-daka` 并等待上线）
- GitHub 授权保存在本机 `ghcli` 目录里；若某天提示未授权，重新执行 `ghcli\bin\gh.exe auth login --web`
- 临时隧道方案（第三节）已降级为备用；正式入口请认准上面的固定网址
- **「数据管理」入口已隐藏**：「我的」页界面上不显示任何字样，点页面最底部的版本号（如「版本 v12」）即可展开备份/恢复/打包/清空面板，再点收起

## 一、每次改版的固定动作（缺一不可）

1. **只改 `recorder.html`**，保持单文件自包含
2. **版本号两处一起 +1**：`sw.js` 的 `CACHE`（如 `recorder-v7` → `recorder-v8`）和 `recorder.html`「我的」页底部的「版本 v7」文字
   —— 这是手机能拿到新版本的关键，忘了升号用户会一直看到旧页面；版本号显示出来是为了发布后一眼验收
3. 本地验证：浏览器打开 `http://127.0.0.1:8765/recorder.html`，确认功能正常
4. 新增数据字段必须三处同步：新建默认值 → `saveAll()` → `loadAll()` / `importBackup()`

## 二、本地服务（改版的验证环境）

在项目目录打开终端执行：

```
python -m http.server 8765 --bind 127.0.0.1
```

浏览器访问 `http://127.0.0.1:8765/recorder.html`。服务跑着的时候，所有文件改动**刷新即生效**，无需重启。

## 三、手机临时体验（二维码方案）

服务跑着的情况下，再开一个终端执行：

```
cloudflared.exe tunnel --url http://127.0.0.1:8765 --no-autoupdate
```

日志里会出现一个 `https://xxxx.trycloudflare.com` 地址，手机访问即可。
注意：这个地址**每次重启隧道都会变**，只适合临时用。改动会实时同步——改完文件手机刷新就是新版（配合第一步的版本号 +1，应用内会弹「已更新到最新版本」提示）。

## 四、固定发布（长期方案，二选一）

| 方案 | 操作 | 特点 |
|------|------|------|
| WorkBuddy 发布（推荐，已有） | 把本目录重新发布一次 | 永久地址不变；旧链接 `a6099c...` 目前还是老版本，发布后即覆盖 |
| GitHub Pages | 目录推到 GitHub 仓库，开启 Pages | 免费、永久、HTTPS；每次改版 push 即更新 |

发布渠道的原则：**本地隧道用来随时改随时看，WorkBuddy/GitHub 用来给手机一个不变的正式入口。**

## 五、版本更新如何到达用户手机

- `sw.js` 采用 cache-first + 后台静默更新，`CACHE` 升号后新版本会在用户打开应用时立即接管
- 应用内已加提示：新版本接管时会弹 toast「已更新到最新版本，请刷新页面」，用户刷新一次即可
- 如果用户长时间没打开，下次打开会自动拿到新版，无需任何手动清缓存操作

## 六、改版红线（来自交接包，勿违反）

- 不要改 IndexedDB 库名（`recorder-db`）和存储结构，旧数据会丢
- 不要动 `jszip.min.js` 和录音核心逻辑（`startRecording` / `pickMime`）
- 配色：蓝 `#4b7cf5`、紫 `#7c5bff`、红 `#ff4d5e`、绿 `#16b364`、背景 `#eef1f7`
## 七、任务执行追踪（task-tracker，独立小工具）

与录音打卡台相互独立的第二个单文件应用，同一目录发布即可：

- `task-tracker.html`：单文件应用（任务列表 + 执行计时 + 目标/连续天数 + 前置任务 + 统计报表 + CSV/JSON 备份）
- `task-tracker.webmanifest` / `task-tracker-sw.js` / `task-tracker-192.png` / `task-tracker-512.png`：PWA 配套（可离线、可加手机主屏）

### 改版规则（与 recorder 同思路）

1. 只改 `task-tracker.html`，保持单文件自包含
2. 版本号两处一起 +1：`task-tracker-sw.js` 的 `CACHE`（如 `task-tracker-v1` → `v2`）和页面底部「版本 v1」文字
3. 本地验证/日常运行：`python server.py 8765`（自带静态服务 + 每日提醒推送，替代 http.server）；或直接双击「打开任务追踪.bat」
4. 每日推送：页面「⚙ 提醒设置」里配置渠道与密钥（Server酱 / PushPlus / Bark）和提醒时间；页面每次改动会自动把复习排期同步给 server.py（tracker-sync.json），到点推送今日清单。推送需要电脑开机且服务在运行
4. 数据存于浏览器 localStorage，键名 `task-tracker-v1`（**不要改名**，旧数据会丢）；跨设备迁移用「备份 JSON / 导入备份」
5. 新增数据字段必须同步三处：`normalize()` 默认值 → `save()` → 导入合并（`doImport` 后统一走 `normalize()`）
6. 双击以 `file://` 打开也能用（Service Worker 会自动跳过），但 PWA 安装必须走 http(s)
