---
title: ubuntu主力系统的日常
layout: post
date: 2016-08-09 00:00:00
tags: [HowTo]
feature-img: "https://images.pexels.com/photos/461497/pexels-photo-461497.jpeg?cs=srgb&dl=bear-business-computer-461497.jpg&fm=jpg"
---

因工作原因，不想把时间浪费在折腾`瘟到死`系统配置的死循环中，想弄个Mac过来装逼，但奉行够(que)用(shi)就(mei)好(qian)的原则，遂把工作机换成了*ubuntu* 为主系统。~~以安装双系统的方式共存，安装方法之前的文章已经有说过了~~。接着是理清下思路，切换工作环境到 `ubuntu 16.04`，需要做哪些准备，是否影响日常的工作呢？

<!--more-->

### 友情提示

 .deb包无法直接在图形界面安装，使用命令安装

```bash
$  dpkg -i your_package_name.deb
 
 # 部分软件缺少信赖关系，系统会提示你解决方法
 $ sudo apt -f install
```

### 远程

1. teamviewer支持linux版本
2. ubuntu系统自带 remmina远程桌面客户端，支持访问windows系统

### 终端

使用 terminator 代替系统的默认终端。贴个自用的配色方案

```shell
...
# 省略部分...
...
[profiles]
  [[default]]
    background_color = "#131313"
    background_darkness = 0.85
    background_image = ""
    background_type = transparent
    cursor_color = "#ff0018"
    font = Liberation Mono 14
    foreground_color = "#d6dbe5"
    palette = "#000000:#ff0000:#38de21:#ffe50a:#008df8:#ff005d:#00bbbb:#bbbbbb:#555555:#f40e17:#3bd01d:#edc809:#0092ff:#ff55ff:#6ae3fa:#ffffff"
    scroll_background = False
    scrollback_lines = 3000
    show_titlebar = False
    use_system_font = False
```

更多颜色方案可参考 [iTerm2-Color-Schemes](https://github.com/mbadolato/iTerm2-Color-Schemes) 。

### SSH-KEY 密码管理

本机上的**SSH-Key** 管理 ，特指key有密码的情况，方法有两种，均是在*.bashrc* 或者 *.zshrc* 中操作

```shell
# 方法一
$ apt install keychain
$ eval `keychain --eval id_rsa`

# 方法二
$ eval "(ssh-agent -s)" && ssh-add ~/.ssh/id_rsa
```

### conky 系统监控

在桌面上显示的系统监控信息：conky

```shell
# 安装
$ sudo apt install conky
$ mkdir -pv ~/.config/conky

# 贴以下配置
$ vim ${HOME}/.config/conky/conky.conf
conky.config = {  
    alignment = 'top_left',  
      
    background = false,  
      
    border_width = 1,  
      
    cpu_avg_samples = 2,  
    net_avg_samples = 2,  
      
    use_xft = true,  
    -- Xft font when Xft is enabled  
    font = 'Sans:size=11',  
    -- Text alpha when using Xft  
    xftalpha = 0.8,  
      
    default_color = 'white',  
    default_outline_color = 'green',  
    default_shade_color = 'green',  
      
    draw_borders = false,  
    draw_graph_borders = true,  
    draw_outline = false,  
    draw_shades = false,  
  
    gap_x = 30,  
    gap_y = 31,  
    minimum_height = 5,  
    minimum_width = 5,  
      
    no_buffers = true,  
    out_to_console = false,  
    out_to_stderr = false,  
    extra_newline = false,  
  
    double_buffer = true,  
    -- Create own window instead of using desktop (required in nautilus)  
    own_window = true,  
    own_window_class = 'Conky',  
    own_window_argb_visual = true,  
    own_window_transparent = true,  
    own_window_hints = 'undecorated,below,sticky,skip_taskbar,skip_pager',  
    own_window_type = 'normal',  
  
    stippled_borders = 0,  
    update_interval = 1.0,  
    uppercase = false,  
    use_spacer = 'none',  
    show_graph_scale = false,  
    show_graph_range = false  
}  
  
conky.text = [[  
${color red}系统信息 ${hr 1}${color}  
主机名: $alignr$nodename  
内核版本: $alignr$kernel  
运行时长: $alignr$uptime  
  
CPU: ${alignr}${freq dyn} MHz  
进程数: ${alignr}$processes ($running_processes running)  
负载: ${alignr}$loadavg  
  
CPU ${alignr}${cpu cpu0}%  
${cpubar 4 cpu0}  
Ram ${alignr}$mem / $memmax ($memperc%)  
${membar 4}  
swap ${alignr}$swap / $swapmax ($swapperc%)  
${swapbar 4}  
  
CPU占用排行 $alignr CPU%  MEM%  
${top name 1}$alignr${top cpu 1}   ${top mem 1}  
${top name 2}$alignr${top cpu 2}   ${top mem 2}  
${top name 3}$alignr${top cpu 3}   ${top mem 3}  
  
内存占用排行 $alignr CPU%  MEM%  
${top_mem name 1}$alignr${top_mem cpu 1}   ${top_mem mem 1}  
${top_mem name 2}$alignr${top_mem cpu 2}   ${top_mem mem 2}  
${top_mem name 3}$alignr${top_mem cpu 3}   ${top_mem mem 3}  
  
${color green}网络信息 ${hr 1}${color}  
Down ${downspeed enp3s0}/s ${alignr}Up ${upspeed enp3s0}/s  
${downspeedgraph enp3s0 24,110} ${alignr}${upspeedgraph enp3s0 24,110}  
Total ${totaldown enp3s0} ${alignr}Total ${totalup enp3s0}  
  
${color pink}硬盘读写 ${hr 1}${color}  
Read ${diskio_read}/s ${alignr}Write ${diskio_write}/s  
${diskiograph_read /dev/sda 25,107} ${alignr}${diskiograph_read /dev/sda 25,107}  

${color yellow}硬盘空间 ${hr 1}${color}  
/home  $color${fs_used /home}/${fs_size /home}   ${fs_bar 5,120 /home}
/data  $color${fs_used /data}/${fs_size /data}   ${fs_bar 5,120 /data}   
/      $color${fs_used /}/${fs_size /}           ${fs_bar 5,120 /}
]]  
```

### 谷歌大法好

详情参见 [科学上网之shadowsocks](/2016/11/29/shadowsocks-installation-and-usage)

### linux机器管理

SecureCRT居然有了linux版本，啧啧。

```shell
# 到官网下载ubuntu版本的CRT
# 下载破解程序
$ wget http://download.boll.me/securecrt_linux_crack.pl

# 执行破解，生成注册信息
$ sudo perl securecrt_linux_crack.pl /usr/bin/SecureCRT  
```

### 文档处理

暂时使用自带的LibreOffice，如果实在不好用，再考虑换wps for linux吧

### 密码管理

~~keepass中文字体显示异常，安装完毕之后，需要进入软件设置，选择google的开源字体: `思源体 - noto sans`~~

```shell
$ sudo apt-add-repository ppa:jtaylor/keepass
$ sudo apt-get update
$ sudo apt-get install keepass2
```

`现推荐 enpass` ，由印度公司开发的，桌面版完全免费，移动端$10 一次性买断，颜值高，界面有抄袭 *1PASSWORD* 的嫌疑，性价比高。[官网下载页](https://www.enpass.io/downloads/) 。

### 坚果云同步

下载deb包，执行`dpkg -i`安装即可

### Pycharm

在 [官网](https://www.jetbrains.com/pycharm/download/#section=mac) 下载社区免费版即可

```shell
# 官网下载压缩包
$ mv pycharm-community-2016.2.tar.gz /opt
$ cd /opt && tar -xf pycharm-community-2016.2.tar.gz
$ sudo ln -s /opt/pycharm-community-2016.2/bin/pycharm.sh /usr/bin/pycharm
```

### Typora

從來不看更新日志的在下，某次心血來潮掃了一下，不得了啊～[*Typora*](https://www.typora.io/) 居然支持 *linux* 了，老子那個激動啊。個人比較喜歡的一款 *markdown* 軟件，三打操作系統平臺均支持。

```shell
# 額外的選項，但強烈建議添加
$ sudo apt-key adv --keyserver keyserver.ubuntu.com --recv-keys BA300B7755AFCFAE

# 添加軟件源
$ sudo add-apt-repository 'deb https://typora.io linux/'
$ sudo apt-get update

# 安裝 typora
$ sudo apt-get install typora
```

### sublime text


`现使用 text 3` 版本，官方的指导安装是

```shell
$ wget -qO - https://download.sublimetext.com/sublimehq-pub.gpg | sudo apt-key add -

$ sudo apt-get install apt-transport-https

# stable
$ echo "deb https://download.sublimetext.com/ apt/stable/" | sudo tee $ /etc/apt/sources.list.d/sublime-text.list

# dev
$ echo "deb https://download.sublimetext.com/ apt/dev/" | sudo tee $ /etc/apt/sources.list.d/sublime-text.list

$ sudo apt-get update && sudo apt-get install sublime-text
```

中文输入问题依旧未解决，但不影响日常使用，大不了注释是英文写咯。

安装包管理控制台：`ctrl+shift+p` => 输入 *Install Package* 。

自动补全插件：**SublimeCodeIntel** 

python集成插件：**Anaconda** ，安装后配置 "python_interpreter" 指定Python路径

添加python3编译环境：

`Tools -> Build System -> New Build System` 

命名为 python3.sublime-build

```shell
{ 
"cmd": ["python3的绝对路径", "-u", "$file"], 
"file_regex": "^[ ]File \”(…?)\”, line ([0-9]*)", 
"selector": “source.python” 
}
```

### Visual Studio Code

*sublime text* 对中文支持略不友好，则可以使用 [*visual studio code*](https://code.visualstudio.com/docs/?dv=linux64_deb) ，一款简洁的代码编辑器，功能强大。

### rime输入法

[Rime](http://rime.io) 小狼毫输入法，*fcitx* 版本，ubuntu 默认安装的即是 *fcitx* 

```shell
# 增加源并安装
$ sudo add-apt-repository ppa:fcitx-team/nightly && sudo apt-get update
$ sudo apt-get install fcitx-rime

# 安装五笔「默认 明月拼音」，并添加到默认的选项中，
$ sudo apt-get install librime-data-wubi 
$ sudo cp /usr/share/rime-data/wubi* /home/your_user_name/.config/fcitx/rime/
$ sudo sed -i 's/cangjie5/wubi86/g' ${USER}/.config/fcitx/rime/default.yaml 
```

### 下载软件

简易的 *HTTP* 下载，使用 *Uget* ；BT 下载则使用 *Qbittorrent* ，这货支持资源网站订阅，亦可通过web访问。

```shell
$ sudo apt -y install uget qbittorrent
```

### 播放器

推荐使用 [smplayer](https://www.smplayer.info/) ，功能强大，无须特别复杂的配置，比 *VLC* 友好得多。

自带的版本比较旧，添加新版本的源

```shell
$ sudo add-apt-repository ppa:rvm/smplayer 
$ sudo apt-get update 
$ sudo apt-get install smplayer smplayer-themes smplayer-skins
```

`更换界面` ：默认界面比较丑，可在 *选项* —> *界面* —> *图标集* 中设定，推荐 `Gartoon`。

`字幕黑边` ：让外挂字幕在黑边中显示，而非视频上方显示「影响观感」，可在 *选项* —> *常规* —> *视频* —> *全屏时添加黑边* 。

### 系统主题

系统默认的主题略简洁，个性化意念较强的，可安装下列主题，首要前提是

```shell
$ sudo apt-get install unity-tweak-tool
```

#### Numix

一款GTK的主题

```shell
$ sudo add-apt-repository ppa:numix/ppa
$ sudo apt-get update
$ sudo apt-get install numix-gtk-theme numix-icon-theme-circle

# 安装配套的壁纸
$ sudo apt-get install numix-wallpaper-*
```

#### [Flatabulous](https://github.com/anmoljagetia/Flatabulous)

一款扁平化的主题

```shell
# 下载源代码
$ wget https://github.com/anmoljagetia/Flatabulous/archive/master.zip

# 移动至相应位置
$ sudo mv Flatabulous-master /usr/share/themes/

# 安装 ultra-flat-icons 图标集
$ sudo add-apt-repository ppa:noobslab/icons
$ sudo apt-get update
$ sudo apt-get install ultra-flat-icons
```

上述主题安装好，使用 *unity-tweak-tool* 设置系统主题和图标集即可。

### QQ与RTX

~~这两个货暂时无解，网上应该有wine方式的安装，暂时不想折腾，于是装个virutalbox虚拟机，装个WIN7，就挂着QQ和RTX。没办法啊，工作要用RTX。~~

虚拟机解决，开启无缝模式即可。

## 尾巴

把上述东东折腾完毕，想着听首歌放松下，立马打开浏览器访问网易云音乐，可惜没装flash，无法在线播放，查看其客户端种类，OMG，居然支持Linux了，啧啧。
对比三四年前实体机安装ubuntu的体验，明显有很大的进步啊。使用linux的人群似乎增多了，很多软件逐步支持linux了，如secureCRT、网易云音乐等等，鹅厂的除外。ubuntu使用体验完全不输于windows，对于开发者而言，ubuntu可能是上不了mac， 下不想win的折衷解决方案了。
