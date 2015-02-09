---
layout: post
category : Notes
tagline: "中文输入法"
tags : [ubuntu, fcitx, 中文输入法, 英文Ubuntu]
---

### 英文 ubuntu 系统中安装 [fcitx](https://wiki.archlinux.org/index.php/Fcitx_(%E7%AE%80%E4%BD%93%E4%B8%AD%E6%96%87))

- 卸载 ibus 输入法，安装 fcitx （小企鹅）输入法

        sudo apt-get remove ibus

- 添加入系统源

        sudo add-apt-repository ppa:fcitx-team/nightly 

- 更新源

        sudo apt-get update

- 安装拼音

        sudo apt-get install fcitx fcitx-config-gtk fcitx-googlepinyin 

`fcitx-sogoupinyin` 可以换成： 搜狗拼音 `fcitx-sogoupinyin`  云拼音 `fcitx-module-cloudpinyin`  孙拼音 `fcitx-sunpinyin`。**安装完后重启或注销**。

- 设置fcitx为默认的输入框架

		在 language support 中， 设置 fcitx 为默认输入法

> 设置为默认输入法，以前一直使用的是 `im-switch`，但事实上，`im-switch` 已经被默认的系统包给取代了，安装 `im-switch` 会导致系统的 language support 消失，如果安装过请先卸载 `im-switch`，然后安装 `im-config` 与 `language-selector-gnome`，这样系统的 language support 就恢复了


- fcitx中加入已安装的输入法

        在 fcitx 的配置中，点击 "+"，加入已安装的输入法。注意将 "Only show current language" 的复选框去掉

- 重装 `unity-control-center`

		sudo apt-get install unity-control-center

> 由于卸载了 IBus，其依赖包 `unity-control-center`也被卸载掉了。需重新安装

---

成功安装于14.04 LTS 32Bit。

