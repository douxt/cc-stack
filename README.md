# cc-stack

**Claude Code 后端协议栈 — 最简透传路由代理，零依赖 Node.js 单文件。**

> Claude Code backend protocol stack — a minimal passthrough proxy in a single zero-dependency Node.js file.

---

## 是什么 / What

[mini-router.js](scripts/mini-router.js) — **196 行**，解析 model 字段做路由选择，其余 body 原样透传。不做格式翻译（Anthropic ↔ OpenAI），只做最纯粹的 HTTP 转发。

对比 [claude-code-router](https://github.com/musistudio/claude-code-router)（35k+ stars）等全功能网关，cc-stack 的哲学是：**如果上游已原生支持 Anthropic Messages API，就不要在中间加翻译层。**

> **196 lines**. Routes by model field, forwards everything else as-is. No format translation (Anthropic ↔ OpenAI), just pure HTTP passthrough. Built for providers that already speak Anthropic natively (DeepSeek, vLLM, etc.).

---

## 为什么不用 cc-router / Why not cc-router?

| | cc-router | **cc-stack (mini-router)** |
|---|---|---|
| 代码量 | 数千行 TypeScript | **196 行 JavaScript** |
| 格式翻译 | 默认翻译，passthrough 是二等公民 | **只透传，零翻译** |
| 依赖 | npm 生态 | **零依赖（Node.js 内置模块）** |
| 兼容性风险 | CC 升级频繁导致 break | **只依赖 HTTP 协议，无版本绑定** |
| 适用 | 多 Provider 不同协议 | **Anthropic-native Provider** |

---

## 快速开始 / Quick Start

### 1. 配置

```bash
cp config/secret.template.json config/secret.json
# 编辑 secret.json 填入 API Key
```

### 2. 启动

```bash
# 直连 DeepSeek
./scripts/switch-profile.sh direct deepseek

# mini-router 透传（推荐）
./scripts/switch-profile.sh deepqwen
```

### 3. Claude Code 使用

启动后 `ANTHROPIC_BASE_URL` 已自动写入 `~/.claude/settings.local.json`，Reload VSCode 窗口即可。

> After switching, `ANTHROPIC_BASE_URL` is auto-configured in `~/.claude/settings.local.json`. Reload your VSCode window to apply.

---

## 架构 / Architecture

```
Claude Code (VSCode / CLI)
    │
    │  ANTHROPIC_BASE_URL=http://localhost:3457
    ▼
┌─────────────────────────────┐
│  mini-router.js  :3457      │
│                             │
│  ① 解析 model 字段           │
│  ② 匹配路由规则              │
│  ③ 原样透传 body 到上游       │
│     不改一个字               │
└─────────────────────────────┘
    │
    ├─ Haiku → DeepSeek V4 Flash ──► api.deepseek.com/anthropic
    ├─ Sonnet → DeepSeek V4 Pro ───► api.deepseek.com/anthropic
    └─ Opus → Qwen 3.7 Max ──────► dashscope.aliyuncs.com/apps/anthropic
```

---

## 路由模式 / Profiles

| Profile | 说明 |
|---------|------|
| `deepqwen` | DeepSeek + Qwen 组合，mini-router 真透传 |
| `direct deepseek` | 直连 DeepSeek，不走代理 |
| `direct qwen` | 直连阿里百炼 Qwen |
| `deepseek` | CCR 全量 DeepSeek 路由 |

更多 profile 见 `./scripts/switch-profile.sh help`。

---

## 目录 / Structure

```
cc-stack/
├── scripts/
│   ├── mini-router.js          ← 核心：196 行透传代理
│   ├── switch-profile.sh       ← 一键切换路由模式
│   └── merge-config.py         ← 配置合并工具
├── profiles/                   ← 各 Provider 路由配置
│   ├── deepqwen/                 ← DeepSeek+Qwen 透传
│   ├── deepseek/               ← DeepSeek CCR 路由
│   ├── direct/                 ← 直连模式
│   └── ...
├── config/
│   └── secret.template.json    ← 密钥模板（不提交真实密钥）
└── .github/workflows/          ← GitHub → Gitee 自动镜像
```

---

## 许可 / License

MIT
