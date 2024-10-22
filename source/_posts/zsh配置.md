---
uuid: d8ebab90-907d-11ef-933b-47a111c02c08
title: oh-my-zsh配置
index_img: /imgs/zsh/image.png
abbrlink: 1285
date: 2024-10-22 21:59:25
updated:
tags: zsh
categories: linux
---

1. 下载插件
```shell
git clone https://github.com/zsh-users/zsh-autosuggestions ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-autosuggestions
git clone https://github.com/zsh-users/zsh-syntax-highlighting.git ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-syntax-highlighting
```

2. 修改.zshrc
```bashrc
plugins=(
        git
        extract
        zsh-syntax-highlighting
        zsh-autosuggestions
)
```