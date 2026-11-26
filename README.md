# 租借女友空间 · Tsukuyomi Space

与千鹤、麻美、瑠夏、墨、弥妮相遇的沉浸式 AI 角色互动网站。
Vue 3 前端 + Express 后端，一个进程同时提供页面和 API。

> **想让别人在任何设备上随时打开（不需要你的电脑开机）？看 [DEPLOY.md](./DEPLOY.md)。**

---

## 本地运行

```bash
npm install          # 安装依赖
cp .env.example .env # 然后填 LLM_API_KEY（本地 .env 已经填好了）
npm run build        # 构建前端到 dist/
npm start            # 打开 http://127.0.0.1:3000
```

开发模式（热更新）：

```bash
npm run dev    # 同时启动 API(3000) + Vite(5173)
```

其它命令：

| 命令 | 作用 |
| --- | --- |
| `npm run smoke` | 13 项接口/路由自检，上线前跑一遍 |
| `npm run e2e -- https://你的域名` | 无头浏览器跑一轮真实对话 + 截图 |
| `npm run icons` | 重新生成 favicon / PWA 图标 |
| `python scripts/optimize-assets.py` | 把 `assets-src/` 里的原图重新压成 WebP |
| `npm run exe` | 打包成 Windows 绿色版 exe |

---

## 目录结构

```
backend/
  app.cjs              Express 应用（静态资源 / 安全头 / SPA fallback）
  server.cjs           启动、DNS 优化、LLM 预热、优雅退出
  config.cjs           集中配置 + 零依赖 .env 加载
  db/                  sqlite / json 双驱动（原生模块缺失时自动降级）
  middleware/          单 IP 限流
  routes/              health / chat / auth / tts
  services/            llm（对话）· voice（语音合成与降级）
src/frontend/          Vue 3 页面与组件
public/                站点实际发布的素材（WebP 图片、音乐、图标）
assets-src/            原始大图（不进仓库/镜像，用于重新压图）
dist/                  构建产物（已提交，云端无需再编译）
scripts/               dev / smoke / e2e / 图标生成 / 图片压缩
Dockerfile             node:22-slim，非 root 运行，自带 HEALTHCHECK
docker-compose.yml     国内服务器 / VPS 一键起站
render.yaml            Render Blueprint，一键部署
```

---

## 技术要点

- **对话**：DeepSeek（OpenAI 兼容接口），一次调用同时产出中文与日文，失败自动重试 3 次并带指数退避
- **语音**：本地有 GPT-SoVITS 时用角色声线；没有则无缝降级到浏览器 `speechSynthesis`（ja-JP），所以线上访客也能听见声音
- **数据库**：优先 `better-sqlite3`，原生模块不可用时自动用 JSON 文件，任何平台都能跑起来
- **限流**：聊天 20 次/分钟、语音 10 次/分钟、登录注册 20 次/分钟（按 IP）
- **缓存**：`/assets/*` 永久缓存（带 hash），图片 7 天，HTML 不缓存
