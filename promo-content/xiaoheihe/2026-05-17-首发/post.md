# 小黑盒推广帖子 · ssh-mobile-workflow 首发

---

## 帖子一：技术分享型

**[开源] 一个脚本把Windows电脑变成手机SSH开发工作站**

# 背景

我是 Claude Code 的重度用户，日常用 `/mobiledev` 和 `/module-audit` 这些 skill 写代码。但有个问题一直很烦——你必须坐在电脑前。Claude 跑任务的时候出门拿个快递都不踏实。

市面上的远程开发方案（VS Code Remote、JetBrains Gateway）都依赖桌面端。我想做的是：手机掏出来就能 SSH 回去继续写，不需要开电脑。搜了一圈没找到现成的，那就自己造。

# 技术方案

- 组网：Tailscale（WireGuard 虚拟组网，100.x.x.x 网段，无需公网 IP）
- 服务：Windows OpenSSH Server
- 客户端：手机端 Termius + Tailscale App
- 脚本：PowerShell（单文件 setup.ps1，断点续跑）
- 权限：NTFS ACL 精细控制（DENY 敏感目录，不碰根目录）

架构很简单：手机 → Tailscale 加密隧道 → 家里 Windows 电脑 OpenSSH → gitops 受限用户 → Claude Code

# 关键配置

gitops 用户对敏感目录的 NTFS 权限控制：

```
# Desktop/Documents/AppData 等敏感目录 → DENY 写入
$acl.AddAccessRule((New-Object System.Security.AccessControl.FileSystemAccessRule(
    "gitops", "Write", "Deny"
)))

# D:\GitHub → 读写
# 开发工具目录（Java/Maven/Node.js）→ 只读
# Claude 配置目录 → 读写（symlink 共享主用户配置）
```

Claude Code 配置共享用的 NTFS 符号链接：

```powershell
New-Item -ItemType SymbolicLink `
  -Path "C:\Users\gitops\.claude" `
  -Target "C:\Users\主用户\.claude"
```

主用户的 skills、settings.json、mcp.json 全部透传，改一处两边生效。

# 效果

- 脚本执行 < 10 分钟（7 步全自动 + 2 步人工装 App）
- 出门只带手机，Termius 连上就能用 Claude Code 全套 skill
- git / maven / pnpm / Claude Code 全部可用
- 不需要公网 IP，Tailscale 走 WireGuard 端到端加密

# 安装

```bash
git clone https://github.com/therain2020/ssh-mobile-workflow.git
cd ssh-mobile-workflow
.\setup.ps1   # 管理员身份运行
```

GitHub：https://github.com/therain2020/ssh-mobile-workflow

README 里每一步都写得很详细，包括 4 个真实踩坑记录（根目录 DENY 陷阱、Oracle javapath 冲突、Tailscale 被代理拦截等）。

# 讨论

- 大家有类似移动办公的需求吗？你们现在是怎么解决的？
- NTFS 权限这块有没有更好的实践？特别是 DENY 继承那部分，Windows 的行为跟预期不太一样

---

## 帖子二：踩坑记录型

**[踩坑] Windows SSH 远程开发 NTFS 权限配置的血泪史**

# 需求

给 ssh-mobile-workflow 项目做安全加固：SSH 登录的用户只能访问开发相关目录，不能碰主用户的 Desktop、Documents、AppData 等敏感区域。听起来很简单的需求，在 Windows NTFS 权限系统上踩了 4 个坑。

# 环境

- Windows 11 Home China
- OpenSSH Server（Windows 自带）
- PowerShell 5.1
- 目标：创建受限 gitops 用户，NTFS 权限精准控制

# 踩坑记录

## 坑 1：DENY 了 D:\ 根目录，Maven 和 Java 直接罢工

- 现象：Maven 编译报错「拒绝访问」，Java 侧没有任何报错信息，`java -v` 直接无输出，进程静默退出
- 原因：Windows 对根目录的 DENY 权限会通过子进程继承传播。Maven 启动子进程时，子进程尝试访问当前工作目录的父级路径，一路回溯到根目录时触发了 DENY。加上 Oracle Java 安装器在 System32 下放了一个 `java.exe` 快捷方式，PATH 里它的优先级比系统 JDK 还高，导致问题更难排查
- 解决：**不要 DENY 根目录**。只 DENY 到具体的用户目录层级（Desktop、Documents、AppData 等），根目录不做任何 DENY 规则。Oracle javapath 从系统 PATH 里删掉或者在用户 PATH 里把 JDK 路径排到前面去

## 坑 2：DENY 不加 (OI)(CI) 的诡异行为

- 现象：在某个目录上加了不带继承标志的 DENY，看起来只影响该目录本身。但实际上该目录下的子进程仍会受影响
- 原因：Windows 对 DENY ACE 的处理跟 ALLOW 不同。即便是只对目录本身设置的 DENY，子进程在尝试访问路径中的某个上级目录时仍可能触发。这和 Linux 的权限模型完全不同
- 解决：所有的 DENY 规则明确加上 `(OI)(CI)`（Object Inherit + Container Inherit），行为就可预测了——只会影响你指定的目录及其子内容

## 坑 3：SSH 登录看不到 Machine 级别的环境变量

- 现象：配了 `JAVA_HOME` 和 `MAVEN_HOME`，SSH 登录后 `echo %JAVA_HOME%` 返回空
- 原因：环境变量有 User 和 Machine 两个 scope。SSH 登录的 gitops 用户看不到主用户的 User scope 变量。而之前图方便把变量都设在了 User scope
- 解决：环境变量全部设到 Machine scope。而且每次修改后必须 `Restart-Service sshd`，sshd 不会自动感知新的环境变量

## 坑 4：Tailscale 登录弹窗被 Clash 代理拦截

- 现象：脚本执行到 Tailscale 登录步骤时，浏览器弹不出登录页面
- 原因：Clash TUN 模式或系统代理设置拦截了 Tailscale 的本地认证请求。Tailscale 登录走的是 `http://127.0.0.1:xxxx` 本地回调，被代理规则误伤了
- 解决：临时关掉系统代理或者把 `127.0.0.1` 加入代理绕过列表，登录完再开回来

# 最终方案

这些坑都沉淀在了 ssh-mobile-workflow 的 setup.ps1 里。脚本已处理好所有权限配置，不会踩上面的坑。如果手动配 Windows SSH 环境，上面四条可以作为避坑指南。

GitHub：https://github.com/therain2020/ssh-mobile-workflow
README 的「踩坑记录」那节有更详细的排查过程。
