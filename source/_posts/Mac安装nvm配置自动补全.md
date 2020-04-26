---
title: Mac 安装 nvm 配置自动补全
date: 2020-04-26 12:14:01
categories: 
- Web前端
tags:
- Mac
- node.js
- nvm
---


使用 Homebrew 安装 nvm 之后发现输入 `nvm lis` 按 <kbd>Tab</kbd> 键不能自动补全出 `nvm list`， 谷歌了一下，发现  [Homebrew Community Discussion](https://discourse.brew.sh/t/solved-nvm-bash-completion-not-working-w-o-manual-sourcing/2670) 上已经有人提了这个问题，但是他的解决方法是写了一串脚本，但是试了下不能解决问题，而且我对写一串脚本来解决有点洁癖，而且 Homebrew 上的 nvm 不是最新版，故而换用另外的方法。

<!--more-->

直接使用 nvm 官方推荐的安装方式：
```bash
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.35.3/install.sh | bash
# 或者
wget -qO- https://raw.githubusercontent.com/nvm-sh/nvm/v0.35.3/install.sh | bash
```

安装结束后，根据提示将以下代码加入 ~/.bash_profile 中：
```shell
export NVM_DIR="$HOME/.nvm"
[ -s "$NVM_DIR/nvm.sh" ] && . "$NVM_DIR/nvm.sh"  # This loads nvm
[ -s "$NVM_DIR/bash_completion" ] && . "$NVM_DIR/bash_completion"  # This loads nvm bash_completion
```

因为我用 zsh，还要在 ~/.zshrc文件最后添加一句：
```bash
source ~/.bash_profile
```

这样就大功告成啦，测试一下，发现可以正常使用 <kbd>Tab</kbd> 补全 nvm 的参数了。
