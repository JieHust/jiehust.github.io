---
layout: post
category : Note
tagline: "中文输入法"
tags : [ubuntu, fcitx, 中文输入法, 英文Ubuntu]
---
{% include JB/setup %}

## 英文 ubuntu 系统中安装 [fcitx](https://wiki.archlinux.org/index.php/Fcitx_(%E7%AE%80%E4%BD%93%E4%B8%AD%E6%96%87))

- 卸载 ibus 输入法，安装 fcitx （小企鹅）输入法

        sudo apt-get remove ibus

- 添加入系统源

        sudo add-apt-repository ppa:fcitx-team/nightly 

- 更新源

        sudo apt-get update

- 安装拼音

        sudo apt-get install fcitx fcitx-config-gtk fcitx-googlepinyin im-switch

`fcitx-sogoupinyin` 可以换成： 搜狗拼音 `fcitx-sogoupinyin`  云拼音 `fcitx-module-cloudpinyin`  孙拼音 `fcitx-sunpinyin`

- 设置为默认的输入法

        sudo im-switch -s fcitx -z default 

- 若上一步出错，则先执行以下指令，再执行以上指令：

        sudo apt-get install fcitx-table-all

- 重装 `unity-control-center`

        sudo apt-get install unity-control-center

由于卸载了 IBus，其依赖包 `unity-control-center`也被卸载掉了。需重新安装

- 重启或注销系统，即完成安装。

---

成功安装于14.04 LTS 32Bit。

