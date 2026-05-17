# social-media-cn

中文开发者社区推广技能 —— 为小红书 & 小黑盒自动生成推广内容。

## 安装

```bash
npx skills add therain2020/social-media-cn -g -y
```

## 使用

在 Claude Code 中提到以下需求时自动触发：
- "在小红书上推广我的项目"
- "生成一篇小红书笔记"
- "写个小黑盒帖子"
- "开源项目推广方案"

## 支持平台

| 平台 | 内容形式 |
|------|---------|
| 小红书 | 图文笔记 (标题 + 正文 + 话题标签 + 配图方案) |
| 知识星球/小黑盒 | 社区帖子 (技术分享/踩坑记录) |

## 目录结构

```
social-media-cn/
├── SKILL.md               # 技能定义
├── README.md              # 本文件
├── references/
│   └── templates.md       # 5套内容模板 + 话术库 + 标签库
└── promo-content/         # 生成的推广内容
    ├── xiaohongshu/       # 小红书笔记
    ├── xiaoheihe/         # 小黑盒帖子
    └── assets/            # 截图素材
```
