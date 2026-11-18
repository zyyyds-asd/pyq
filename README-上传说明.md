# Moment 云端朋友圈 · 上传说明

这个文件夹里的文件可以直接拖进 GitHub 仓库根目录。**index.html 必须在根目录**，不能放进子文件夹。

| 文件 | GitHub Pages | Cloudflare Workers | 作用 |
|---|---|---|---|
| `index.html` | ✅ 必传 | ✅ 必传 | 首页（根路径 `/` 靠它） |
| `404.html` | 建议 | 建议 | 访问不存在地址时显示 |
| `.nojekyll` | 建议（隐藏文件） | 不需要 | 阻止 Jekyll 处理，避免文件被忽略 |
| `wrangler.jsonc` | 不需要 | ✅ 必传 | `assets.directory: "./"`，修复静态目录报错 |
| `.assetsignore` | 不需要 | 建议（隐藏文件） | 发布时排除配置文件 |
| `README-上传说明.md` | ❌ | ❌ | 本文件 |

> Windows 资源管理器默认隐藏 `.assetsignore`、`.nojekyll` 这类点开头文件：
> 在「查看」里勾上「文件扩展名」和「隐藏的项目」才看得到。

---

## 一、GitHub Pages（你说的仓库，最快）

1. 打开仓库，根目录若还有旧的「我的网页.html」「web_moments.html」先删掉，避免两个入口打架。
2. 上传 `index.html`（+ `404.html`、`.nojekyll`）。
3. Settings → Pages → Source 选 `Deploy from a branch`；分支 `main`、目录 `/ (root)`，保存。
4. 等 1–2 分钟，访问 `https://<用户名>.github.io/<仓库名>/`。
5. 关键一步：到 Firebase 控制台把 `xxx.github.io` 加进 **Authentication → Settings → Authorized domains**，
   否则匿名登录会报 `auth/unauthorized-domain`，页面顶栏显示「云端连接异常」。

## 二、Cloudflare Workers（对应你之前的报错）

1. 根目录上传 `index.html`、`wrangler.jsonc`、`.assetsignore`、`404.html`。
2. Cloudflare → Workers & Pages → 你的项目 → 重试部署。
   构建命令留空、部署命令 `npx wrangler deploy`、根目录 `/`。
3. `wrangler.jsonc` 里的 `"name": "zyyyds-asd"` 必须和你已有 Worker 名字一致，否则会新建一个。
4. 部署完同样要把 `xxx.workers.dev` 加进 Firebase 的 Authorized domains。

---

## 三、这个版本自带了 Firebase 配置，但你要保证 3 件事

`index.html` 里写死了一份配置（项目 `zxcv-1f16e`），所以页面打开就会去连那个项目。要能正常用：

1. **Firebase → Authentication → Sign-in method**：启用 **匿名（Anonymous）**。
2. **Firestore Database / Storage** 已创建，且安全规则允许读写（见下）。
3. **Authorized domains** 加上你的部署域名。

### 安全提醒（重要）

页面里的 `apiKey` 是 Firebase **Web 配置**，公开在前端属于正常设计，不算泄密；
真正决定安全的是**规则**。如果规则写成 `allow read, write: if true`，任何人拿到你的网址就能读、改、删数据。
测试可以这么写，等要公开给人用，至少要限制到登录用户：

Firestore 规则（示例，先跑通再收紧）：

```
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {
    match /{document=**} {
      allow read, write: if request.auth != null;
    }
  }
}
```

Storage 规则同理。

---

## 四、国内网络提醒

这个版本从 `www.gstatic.com` 加载 Firebase SDK，运行时要访问 `*.googleapis.com`（Auth / Firestore / Storage）。
**中国大陆直连通常不通**，表现为：顶栏红字「云端连接异常」、一直转圈、控制台 `net::ERR_...`。
这不是配置问题，是网络到不了 Google。要么用能访问 Google 的网络环境，要么换掉数据后端。

另外 FontAwesome 图标走 `cdnjs.cloudflare.com`，国内一般能用，慢的话不影响主功能。

## 五、本地预览

双击 `index.html` 能看界面。但 `file://` 下匿名登录可能被浏览器限制，
**请以部署后的 https 站点为准**验证功能（发帖、传图、登录）。
