# muchan-avatar-skill · Mu酱头像制作 Skill

让你的 Muse 为 Mu酱（Muse Gadget 小终端）亲手画一张头像：产出一个
`muse_pixel.c`，决定小终端屏幕上 64×64 像素脸的长相与动画。

做法就一句话：把 `SKILL.md` 的内容整段粘给你的 Muse，它会先问清你
想要的形象，再照官方配方（Meta 官方 Gadgets SDK 的 avatar 规范）和
Mu酱工坊的验收规格画好、逐项自检后交付：渲染预览图（附件，8 个表情齐全：七模式＋摸头变体）＋ muse_pixel.c（文件附件）——只在聊天里贴代码块不算交付。把文件存好，在
Mu酱编译工坊上传就能刷进设备（自备源码上传通道开放后）。

## Fastest path

复制下面这段，粘给你的 Muse：

```
请按 https://github.com/irol765/muchan-avatar-skill 里的 SKILL.md 为我制作一个 Mu酱头像：先问清我想要的形象，再严格照其中的绘制规格与验收规则产出 muse_pixel.c，交付前逐项过自检清单，并附 8 表情渲染预览图、以文件附件交付 muse_pixel.c。
```

Or in English:

```
Please make a Mu酱 avatar for me following SKILL.md in https://github.com/irol765/muchan-avatar-skill : first ask me what character I want, then produce muse_pixel.c strictly to its drawing spec and acceptance rules, run its self-check list, and deliver with rendered preview images of all 8 expressions attached and muse_pixel.c as a file attachment.
```

## 文件说明

- `SKILL.md` — Skill 本体（中文），整段可复制：角色与流程 + 绘制规格
  （官方配方）+ 验收规则 + 自检清单。
- `SKILL.en.md` / `SKILL.ja.md` — 英文 / 日文版（技术规格部分三语共用
  同一份英文原文，保证规格不漂移）。
- `AVATAR-ACCEPTANCE.md` — 验收清单（单一事实源）：工坊的上传检查器
  与本 Skill 共用同一份清单，「照 Skill 做 = 必过检查」。

## Where this comes from

这是 [Mu酱 Muchan](https://musejiang.com)（给 Muse Gadget 小终端的
固件 + 通道服务）的配套开源件：头像工坊在站内（第 2 步），不想用工坊
生成时，就用这个 Skill 让自己的 Muse 画。绘制规格源自 Meta 官方
Gadgets SDK 的公开配方（Apache-2.0）；验收规则是 Mu酱工坊的公开
检查口径。默认头像是 Meta 的角色形象，本 Skill 画的是你自己的形象。

License: MIT（见 LICENSE）。官方配方与 muse_pixel.h 本身遵循其
Apache-2.0 许可。
