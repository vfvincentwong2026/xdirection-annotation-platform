# xdirection-annotation-platform
基于 Label Studio 的企业级 AI 辅助数据标注平台，具备智能预标注、印尼语本地化、RBAC 隔离与金标准质量风控机制。
# AI 辅助数据标注平台 (Enterprise Label Studio Edition)

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![Docker](https://img.shields.io/badge/Docker-Supported-blue.svg)](https://www.docker.com/)
[![Label Studio](https://img.shields.io/badge/Label%20Studio-v1.12+-orange.svg)](https://labelstud.io/)
[![Cloudflare](https://img.shields.io/badge/Cloudflare-Tunnel%20%26%20R2-F38020.svg)](https://www.cloudflare.com/)

基于开源 **Label Studio Community Edition** 进行深度定制与二次开发的企业级数据标注平台。结合 AI 预标注、智能任务路由、印尼语本地化以及多重防泄密与质量风控机制，专为高性价比的大规模众包/兼职团队打造。

---

## 目录
- [📖 项目概览](#-项目概览)
- [🎯 核心特性与优化](#-核心特性与优化)
- [🏗️ 系统架构](#️-系统架构)
- [🛠️ 技术栈](#️-技术栈)
- [🚀 快速开始与部署指南](#-快速开始与部署指南)
- [🔒 安全与质量控制 (QC)](#-安全与质量控制-qc)
- [💰 运营与算力成本估算](#-运营与算力成本估算)
- [📅 实施路线图](#-实施路线图)
- [🤝 贡献与反馈](#-贡献与反馈)
- [📄 开源协议](#-开源协议)

---

## 📖 项目概览

本项目解决传统数据标注在海外（如东南亚/印尼）众包场景下的三大痛点：**纯人工标注效率低**、**兼职团队质量难控** 以及 **数据安全泄露风险**。

通过在上层封装微服务代理，在不改动 Label Studio 核心代码的前提下，实现了企业级 RBAC 权限隔离、金标准盲测（HoneyPot）以及 AI 预标注（HITL）闭环。

---

## 🎯 核心特性与优化

| 优化模块 | 行业痛点 | 本平台解决方案 |
| :--- | :--- | :--- |
| **AI 预标注集成** | 纯人工从零标注效率极低 | 集成自定义 ML 后端（YOLO/SAM/LLM），自动生成预测框并标注置信度 |
| **UI/UX 本地化** | 原生英文界面，印尼学生理解成本高 | 全界面印尼语 (Bahasa Indonesia) + 动态 SOP 侧边栏 + 自定义快捷键 |
| **智能任务路由** | 任务无差别分发，造成算力浪费 | 基于置信度（High/Medium/Low）三级分发：高置信度直接入库，中低分配人工审校 |
| **主动学习闭环** | 模型无法持续进化 | 基于 Webhook 实时捕获人工修改数据，触发增量训练/微调 |
| **权限与防泄密** | 开源版无细粒度 RBAC，易被拖库 | 上层 Node.js 代理层拦截 Export 接口，结合动态前端水印与 API 数据脱敏 |
| **质量风控 (QC)** | 兼职学生“全盘接受”AI 错题 | 5% 金标准盲测（HoneyPot）+ 实时准确率监控 + 阶梯式限流/封号机制 |

---

## 🏗️ 系统架构

```text
┌──────────────────────────────────────────────────────────────┐
│                    前端层 (Cloudflare Pages)                  │
│  ┌──────────────┐  ┌──────────────┐  ┌────────────────────┐ │
│  │ 印尼语界面   │  │ SOP 侧边栏   │  │ 动态水印与快捷键  │ │
│  └──────────────┘  └──────────────┘  └────────────────────┘ │
└───────────────────────────┬──────────────────────────────────┘
                            │ HTTPS / WSS
                            ▼
┌──────────────────────────────────────────────────────────────┐
│                   接入与代理层 (Node.js API Gateway)          │
│  ┌──────────────┐  ┌──────────────┐  ┌────────────────────┐ │
│  │ RBAC 鉴权    │  │ 数据脱敏     │  │ 导出拦截器         │ │
│  └──────────────┘  └──────────────┘  └────────────────────┘ │
└───────────────────────────┬──────────────────────────────────┘
                            │ Cloudflare Tunnel
                            ▼
┌──────────────────────────────────────────────────────────────┐
│                 Label Studio 核心容器 (Docker)                │
│  ┌──────────────────────────────────────────────────────────┐ │
│  │  项目管理 / 标注界面 / 任务分配 / PostgreSQL 存储         │ │
│  └──────────────────────────────────────────────────────────┘ │
└──────────┬─────────────────────────────┬────────────────────┘
           │                             │
           ▼                             ▼
┌──────────────────┐          ┌─────────────────────────────┐
│ PostgreSQL 数据库 │          │  对象存储 (Cloudflare R2)    │
└──────────────────┘          └─────────────────────────────┘
           │                             │
           ▼                             ▼
┌─────────────────────────────────────────────────────────────┐
│              ML 后端微服务 (Python Label-Studio-ML)          │
│  ┌─────────────┐  ┌─────────────────┐  ┌─────────────────┐  │
│  │ 预标注生成   │  │ 置信度计算 (Score)│  │ 主动学习 Webhook │  │
│  └─────────────┘  └─────────────────┘  └─────────────────┘  │
└─────────────────────────────────────────────────────────────┘


🛠️ 技术栈
前端 UI: React.js / Vite, TailwindCSS (托管于 Cloudflare Pages)

网络与安全: Cloudflare Zero Trust (Tunnel), Cloudflare R2 Storage

标注引擎: Label Studio Community Edition (Dockerized)

网关与业务服务: Node.js (Express/Fastify) / Python (FastAPI)

数据存储: PostgreSQL 14+, Redis 7 (缓存与队列)

ML 后端: Python 3.10+, label-studio-ml SDK, PyTorch / ONNX Runtime

🚀 快速开始与部署指南

1. 环境准备
准备一台 Linux 服务器（推荐配置：4 核 CPU / 16GB 内存）

安装 Docker 及 Docker Compose

注册 Cloudflare 账号 并在域名管理中开启 Pages 与 R2

2. 启动核心服务 (Docker Compose)
克隆项目并配置环境参数：
git clone [https://github.com/YOUR_ORGANIZATION/label-studio-enterprise.git](https://github.com/YOUR_ORGANIZATION/label-studio-enterprise.git)
cd label-studio-enterprise
cp .env.example .env

编辑 .env 填入数据库密码与 Cloudflare 密钥后，运行：
docker-compose up -d

docker-compose.yml 示例配置：

version: '3.8'

services:
  labelstudio:
    image: heartexlabs/label-studio:latest
    restart: always
    environment:
      - LABEL_STUDIO_HOST=[https://your-tunnel-domain.trycloudflare.com](https://your-tunnel-domain.trycloudflare.com)
      - CSRF_TRUSTED_ORIGINS=[https://your-frontend.pages.dev](https://your-frontend.pages.dev)
      - USE_X_FORWARDED_HOST=true
      - SECURE_PROXY_SSL_HEADER=HTTP_X_FORWARDED_PROTO,https
      - DATABASE_URL=postgresql://ls_user:ls_pass@postgres:5432/labelstudio
      - REDIS_URL=redis://redis:6379/0
    volumes:
      - ./data:/label-studio/data
    ports:
      - "8080:8080"
    depends_on:
      - postgres
      - redis

  postgres:
    image: postgres:14-alpine
    environment:
      POSTGRES_DB: labelstudio
      POSTGRES_USER: ls_user
      POSTGRES_PASSWORD: ls_pass
    volumes:
      - pgdata:/var/lib/postgresql/data

  redis:
    image: redis:7-alpine
    volumes:
      - redisdata:/data

  ml-backend:
    build: ./ml-backend
    environment:
      - MODEL_PATH=/models/latest.onnx
    volumes:
      - ./models:/models
    ports:
      - "9090:9090"

volumes:
  pgdata:
    driver: local
  redisdata:
    driver: local

3. 配置 Cloudflare Tunnel (安全内网穿透)
在服务器上运行以下命令建立加密隧道：

# 安装 cloudflared
curl -L --output cloudflared.deb [https://github.com/cloudflare/cloudflared/releases/latest/download/cloudflared-linux-amd64.deb](https://github.com/cloudflare/cloudflared/releases/latest/download/cloudflared-linux-amd64.deb)
sudo dpkg -i cloudflared.deb

# 绑定并运行
cloudflared tunnel --url http://localhost:8080

🔒 安全与质量控制 (QC)
1. 动态防泄密 (DLP)
代理层拦截: 屏蔽所有标注人员对 /api/projects/{id}/export 接口的访问权，导出权限仅限 Team Lead 角色。
数据脱敏: 前端仅读取 Cloudflare R2 的临时预签名 URL（Presigned URL），后端 API 响应自动过滤原始客户敏感元数据。
水印防截图: 标注界面强制叠加带有兼职人员名、邮箱与 IP 地址的半透明动态水印。
2. 金标准盲测引擎 (HoneyPot)
导入数据集时，系统会自动植入 5% 的标准参考答案（Gold Standard Tasks）。
标注人员在无感知的情况下进行审核与修正。
当触发 Webhook 提交时，计算准确率（Accuracy Score）：
Accuracy ≥ 90%: 正常派单，全额结算。75% ≤ Accuracy < 90%: 触发警报，暂停派单，进入 SOP 复核。
Accuracy < 75%: 自动冻结账户，对应批次任务重新释放回公共任务池。

💰 运营与算力成本估算

服务组件	预估用量	月度费用 (USD)
Cloudflare Pages / Tunnel	无限构建 / 无限流量	$0 (免费层)
Cloudflare R2	1TB 存储 + 2,000 万次 API 读取	~$18.60
云服务器 (应用+ML)	1 台 4C/16GB (如 DigitalOcean / Hetzner)	$40.00 - $80.00
PostgreSQL 托管数据库	100 GB 规格	$20.00 - $40.00
合计预估	小规模试点 (50 标注员团队)	~$80.00 - $140.00 / 月

📅 实施路线图

[x] Phase 1: 基础架构搭建 (已完成) — 部署 Docker Compose、Cloudflare Tunnel 及 Cloudflare R2 接入。

[ ] Phase 2: 本地化与 UI 增强 (推进中) — 完成印尼语 i18n、SOP 侧边栏和快捷键插件开发。

[ ] Phase 3: 业务代理与 QC 闭环 — 完成 Node.js 代理层 (RBAC)、HoneyPot 盲测逻辑与阶梯限流。

[ ] Phase 4: 种子团队试点与优化 — 招募 5-10 名印尼大学生进行全流程 PoC 跑通并上线运营。

🤝 贡献与反馈
欢迎提 PR 或 Issue。如有任何业务合作或技术架构疑问，请联系产品与工程团队：

公司官网: www.xdirectionai.com

商务/技术支持: support@xdirectionai.com

📄 开源协议
本项目基于 MIT License 协议开源。
