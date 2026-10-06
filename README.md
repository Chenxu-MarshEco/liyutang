# 黎语堂 · 评论服务（Waline 服务端）

这个仓库不属于花涧堂那个站点仓库，它是**黎语堂（/liyutang）的评论 / 账号服务**：
[Waline](https://waline.js.org/) 的服务端，跑在 Vercel 上，评论数据存在自己的 Postgres 里。

代码来自 Waline 官方模板（`walinejs/waline` 仓库的 `example/` 目录），一共就几行：

| 文件 | 是什么 |
| --- | --- |
| `index.cjs` | 服务的入口：`module.exports = Application({ plugins: [] })`（Waline 的全部逻辑在 `@waline/vercel` 包里） |
| `package.json` | 只有一个依赖：`@waline/vercel` |
| `vercel.json` | Vercel 的构建说明（把 `index.cjs` 当 Node 函数，其余路径都改写给它） |
| `robots.txt` | 不让搜索引擎抓 |
| `.env.example` | 数据库要填的环境变量长什么样 |

> 为什么不直接用 GitHub Discussions（Giscus）：那要求**每个留言的人都先有 GitHub 账号**。
> 黎语堂要的是"邮箱注册的普通用户 + 站长指定的共创者"两级，所以走 Waline 自带账号体系。

## 部署（在 Vercel 里做）

1. Vercel → **Add New… → Project** → 选中这个 `liyutang` 仓库 → **Deploy**
   （**不用**改任何设置；根目录就是它，因为这里只有服务端这几个文件）。
   第一次部署**大概率报 `500 This Serverless Function has crashed`** —— 正常，
   因为还没有数据库，连上之后重新部署就会恢复。
2. 建数据库：项目页左侧 **Storage → Create Database → Neon → Continue →
   Accept and Create** → 一路 Continue → **Create** → **Connect → Connect Project**。
   连上之后 Vercel 会把 `POSTGRES_*` 那几项环境变量自动注入这个项目。
3. 建表：在数据库页点 **Open in Neon → 左侧 SQL Editor**，把官方这份语句整个粘进去 Run：
   https://github.com/walinejs/waline/blob/main/assets/waline.pgsql
4. 回 Vercel → **Deployments → 最新那条 → Redeploy** → 等 STATUS 变成 **Ready** → **Visit**。
   那个地址就是要填进花涧堂 `src/data/liyutang.json` 的 `forum.waline.serverURL`。

## 三个入口（把 `<地址>` 换成上一步拿到的地址）

| 地址 | 干什么 |
| --- | --- |
| `<地址>` | 服务本身（花涧堂的帖子页会用它挂评论区） |
| `<地址>/ui/register` | **注册 —— 第一个注册的人自动成为管理员**，所以服务一通就去占位 |
| `<地址>/ui` | 评论管理后台：改 / 标记 / 删除评论，给用户打**专属标签**（比如给共创者打「共创者」） |

## 也可以用命令行部署

```shell
npx vercel        # 在这个目录里跑，会引导你登录并建项目
npx vercel --prod # 直接发到生产环境
```

数据库那一步仍然要在 Vercel 网页里点（它的商品页命令行代替不了）。
