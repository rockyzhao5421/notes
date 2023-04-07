## 开始菜单没了

显示器原来用的 HDMI，后面换了 DP 接口，结果开始菜单没了。

原因是原来 panel 是设置输出到 HDMI 的。解决办法很简单，命令行打开 panel：

    $ xfce4-panel --preferences

然后改输出到 DP 就行了。