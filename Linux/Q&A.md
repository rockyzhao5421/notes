## 命令行打开显示器设置

    xfce4-display-settings --minimal
    
## 开始菜单没了

显示器原来用的 HDMI，后面换了 DP 接口，结果开始菜单没了。

原因是原来 panel 是设置输出到 HDMI 的。解决办法很简单，命令行打开 panel：

    $ xfce4-panel --preferences

然后改输出到 DP 就行了。

## 鼠标闪烁

双显示器后，发现其中一个显示器鼠标闪烁，尤其是在浏览器上更严重。

原因是开源显卡驱动垃圾，换 N 卡官方驱动：

    sudo mhwd -a pci nonfree 0300

重启之后确认用的是新驱动：

    [rocky@manjaro Desktop]$ inxi -G
    Graphics:
    Device-1: NVIDIA GK104 [GeForce GTX 760] driver: nvidia v: 470.182.03
    Display: x11 server: X.Org v: 21.1.8 driver: X: loaded: nvidia gpu: nvidia
        resolution: 1: 3840x2160~60Hz 2: 1620x2880~60Hz
    API: OpenGL Message: Unable to show GL data. Required tool glxinfo
        missing.

## 关机、重启无效

关机，重启都没反应，用命令关机、重启提示

     Failed to hibernate system via logind: There's already a shutdown or
      sleep operation in progress

解决办法：

    systemctl cancel

原因可能是电源管理的 suspend 造成的，关掉试试先。


# vim 开启鼠标右键

    :set mouse-=a
