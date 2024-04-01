---
title: "follow Nicole's video"
description:
aliases:
  - 
tags:
  -
draft: false
date: "2024-04-01"

create: "2024-04-01 22:11"
---

Guide video: [How to publish your notes for free with Quartz - YouTube](https://www.youtube.com/watch?v=6s6DT1yN4dw)

Quartz documentation: [Welcome to Quartz 4 (jzhao.xyz)](https://quartz.jzhao.xyz/)


## 照著 Nicole 影片，中途遇到的一些個人問題

### NVM 切換 node.js 版本

[NVM 教學 - 如何使用 NVM 切換 node.js 版本 - nvm 基本指令 - Sean 工作版 (seanacnet.com)](https://seanacnet.com/node-js/nvm/)

不知道何時裝了 nvm

在 git bash 下
```bash
nvm list

nvm install v20.11.1  # 這是我安裝時候的 stable
nvm use v20.11.1
```

### git clone
這步是要把 [jackyzha0/quartz: 🌱 a fast, batteries-included static-site generator that transforms Markdown content into fully functional websites (github.com)](https://github.com/jackyzha0/quartz) 複製(?) 一份到自己這裡

有3種方法:
1. Fork
2. Use this template
3. `git clone https://github.com/jackyzha0/quartz.git`

前兩種都是點開 github repo 可以看到的

我思考要用哪個，最後採用 Fork，理由是 remote 更新可以很方便地 sync (我不知道其他兩種是否也可以做到 sync)

> [!NOTE]- 過程遇到的小坑
>  Nicole 是用 git clone
>  https://youtu.be/6s6DT1yN4dw?t=444
>  
>  git remote add origin <選git那個URL，不要https>
> 
> 我一開始照 Quartz documentation 用 https 的，到下面這步
> 
> `npx quartz sync --no-pull`
> 發生
> ```
> ! [remote rejected] v4 -> v4 (refusing to allow an OAuth App to create or update workflow `.github/workflows/ci.yaml` without `workflow` scope
> ```
> 
> 用 fork 就不會有這個問題~算是繞過去了
> 
> 還沒用 fork 之前，我為了這個問題花費一番功夫都沒成功，後來是注意到 Nicole 用 git那個URL
> 
> 不過 走這條路要加 SSH key 到 github account

然後依照 [Setting up your GitHub repository (jzhao.xyz)](https://quartz.jzhao.xyz/setting-up-your-GitHub-repository)
```bash
git remote set-url origin REMOTE-URL(git那個URL)
git remote add upstream https://github.com/jackyzha0/quartz.git
```

### windows10, git bash

> [!NOTE] teminal
>  nvm 在 git bash 可以，powershell 說沒有指令
>  
>  不過 切換好 node.js 版本後，powershell 就可以用新版本的 node.js, npm 了~

照著 Guide video 或 Quartz documentation，做到這一步
```bash
npx quartz create
```
出現 跟這個發問者 一樣的問題
[node.js - Cannot create Sveltekit app - ERR_TTY_INIT_FAILED uv_tty_init returned EBADF (bad file descriptor) - Stack Overflow](https://stackoverflow.com/questions/75750730/cannot-create-sveltekit-app-err-tty-init-failed-uv-tty-init-returned-ebadf-ba)
```
SystemError [ERR_TTY_INIT_FAILED]: TTY initialization failed: uv_tty_init returned EBADF (bad file descriptor)

```

底下回答說 這是 git bash 的問題，所以我直接在 vscode 用 powershell

一換就成功了~

### 瀏覽器縮放倍率
我的瀏覽器 110% 縮放，這時 Explorer (Component) 被吃掉，剛好我又特別在意這塊

正愁是哪裡出問題，發現 縮放倍率100% 時 版面跟網路上大家示範的一樣~

> [!NOTE] 今日結語
> 雖然今天是愚人節，但我寫這篇文章是認真的~
> 
> 前幾周有在研究怎麼做 blog，看到 Hugo 等 static site generator 琳瑯滿目 (所以迷失了一陣子><)
> 
> 然後找到 quartz 說是 "專門為 Obsidian 配套設計" (https://discord.com/channels/686053708261228577/735629542906920993/1219863381989916762)
> 
> 又看到 Nicole 有拍影片示範
> 
> 於是就決定從這裡下手



