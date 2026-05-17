# 小红书推广笔记 · ssh-mobile-workflow 首发

---

## 笔记一：痛点解决型

📌 不想坐电脑前写代码？试试这个

谁懂啊，Claude Code 任务还在跑，想出个门拿快递都在纠结要不要带电脑 😭

以前用 VS Code Remote 和 JetBrains Gateway 搞过远程开发，但这些方案都绑在桌面端上。我想实现的是——人在外面只带手机，随时 SSH 回家里电脑继续写代码。搜了一圈，没找到现成的方案。

于是自己动手搞了一个一键脚本：

✨ Tailscale 虚拟组网 — 手机和电脑自动组网，不用公网 IP，不用配端口转发
✨ OpenSSH + 受限用户 — 创建专用的 gitops 用户，不暴露主用户，NTFS 权限硬化
✨ Claude Code 配置共享 — 用 NTFS 符号链接把主用户的 skills、settings 全透传给 SSH 用户
✨ 断点续跑 — 脚本中途断了重跑就行，完成的步骤自动跳过

📍 第一步：git clone 项目，管理员身份运行 setup.ps1
📍 第二步：电脑和手机都装 Tailscale，同一个账号登录
📍 第三步：手机 Termius 输入 `ssh gitops@<tailscale-ip>`，搞定 🎉

全程不到 15 分钟。实测通勤路上用手机跑了遍 /mobiledev，除了手机打字慢点，体验跟坐在电脑前没差。

ssh-mobile-workflow 已经在 GitHub 开源了，链接放评论区～
有什么问题直接问我，看到就回 🙌

🏷️
#远程开发 #Claude #SSH #程序员日常 #效率工具 #开源项目 #GitHub #Windows技巧 #AI编程 #移动办公

---

## 笔记二：教程型

📌 手机远程Claude Code保姆级教程

想不想出门只带手机，随时连回电脑写 Claude Code？保姆级教程来了 🤝

需要准备：
- Windows 11 电脑一台（Win10 应该也行但没测过）
- 管理员权限
- 一台手机（iPhone 或安卓）
- 一个 GitHub 账号（Tailscale 登录用）

跟着一步步来：

🔧 Step 1：跑一键配置脚本
克隆项目后管理员运行 setup.ps1，脚本会自动装 OpenSSH、创建 gitops 用户、配 NTFS 权限、设环境变量、建 Claude 符号链接。中间会暂停等你登录 Tailscale。

📱 Step 2：手机装两个 App
App Store / 应用商店搜「Tailscale」和「Termius」，下载安装。Tailscale 用同一个 GitHub 账号登录。

🔗 Step 3：Termius 连上电脑
在 Tailscale 管理后台找到电脑的 100.x.x.x 地址，Termius 里新建连接：
- Host: 100.x.x.x（你电脑的 Tailscale IP）
- 用户: gitops
- 密码: 脚本执行时你设的密码
连接成功就能在手机上用 Claude Code 了 🎉

Q: 需要公网 IP 吗？
A: 不需要。Tailscale 走的是 WireGuard 虚拟组网，电脑和手机点对点直连

Q: 会不会不安全？
A: 脚本创建的是受限 gitops 用户，对 Desktop/Documents 等敏感目录设了拒绝访问。加上 Tailscale 的加密隧道，安全性比直接暴露 SSH 端口高得多

Q: 手机端能用所有 Claude Code 功能吗？
A: git、maven、pnpm、Claude Code 全套都能用。浏览网页类的技能（/browse）不行，gh CLI 需要额外配 GitHub Token

整个过程 15 分钟搞定。配置好之后出门掏手机就能连回去，不用背电脑了。

有问题评论区见，看到就回！

🏷️
#保姆级教程 #远程开发 #Claude #程序员教程 #效率工具 #Windows #SSH #技术分享 #AI工具

---

## 笔记三：开源推荐型

📌 开源：手机远程操控Claude Code写代码

最近在 GitHub 上搞了一个小项目：ssh-mobile-workflow

一句话概括：一个 PowerShell 脚本，把你的 Windows 电脑变成手机随时可连接(SSH)的 Claude Code 开发工作站。

🔧 Tailscale + OpenSSH 一键配置
脚本自动搞定组网和 SSH 服务，不用手搓配置文件

📱 手机端 Termius + Tailscale 直连
出门只带手机，通勤/排队/躺床上都能 remote 回电脑写代码

🛡️ NTFS 权限精准控制
创建专用 gitops 用户，对敏感目录做 DENY，不暴露主用户数据

🔗 符号链接共享 Claude 配置
主用户的 skills、settings、mcp 配置透明共享给 SSH 用户，改一处两边生效

⚡ 断点续跑，全程 < 15 分钟
已完成的步骤自动跳过，中断了重跑就行，不用担心重复执行

我自己的使用场景：周末出门不想背电脑，但又想随时改点代码——手机掏出来 Termius 一连，Claude Code 照常用。通勤路上 reviewer 的 PR 也能顺手处理。

Star 已经在路上了 ⭐ GitHub 链接放评论区～

🏷️
#开源项目 #Claude #远程开发 #程序员日常 #效率工具 #GitHub #SSH #Windows工具 #AI编程
