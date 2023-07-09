---
title: "macOS catalina beta anaconda3 zsh command not found: conda 的一种解决方法"
date: 2019-07-02T10:27:00+08:00
draft: false
---

​
问题：

更新了macOS catalina 10.15 beta之后，terminal提示说要把bash换成zsh，更换之后发现conda指令不见了。

<!--more-->

解决方法：

按照[这里](https://github.com/conda/conda/blob/main/CHANGELOG.md#440-2017-12-20)的方法：

1. 在terminal中输入

```bash
vi ～/.zshrc
```

2. 按`i`进入编辑模式

3. 加入以下代码，注意，代码中的`/Users/username/anaconda3`要是你的conda的安装目录。

```
. /Users/username/anaconda3/etc/profile.d/conda.sh
conda activate base
```

4. 按 esc 退出编辑模式

5. 输入

```
:wq
```

来保存并退出。

6. 输入

```bash
source ~/.zshrc
```

7. 尝试输入 `conda --version`, 如果可以正常使用就说明问题解决了。

原理：

告诉zsh，每次启动zsh的时候自动启动conda中名为base的环境。

​