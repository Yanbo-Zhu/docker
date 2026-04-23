
https://github.com/jesseduffield/lazydocker?tab=readme-ov-file

https://www.youtube.com/watch?v=NICqQPxwJWw


# 1 安装 

windows 
choco install lazydocker

安装在了 C:\ProgramData\chocolatey\lib\lazydocker\tools

# 2 配置文件 

https://github.com/jesseduffield/lazydocker/blob/master/docs/Config.md


- OSX: `~/Library/Application Support/jesseduffield/lazydocker/config.yml`
- Linux: `~/.config/lazydocker/config.yml`
- Windows: `C:\\Users\\<User>\\AppData\\Roaming\\jesseduffield\\lazydocker\\config.yml` (I think)
    - c:\Users\yzh\AppData\Roaming\lazydocker\config.yml


## 2.1 get text to wrap in my main panel

`gui.wrapMainPanel: true`


## 2.2 How do you select text?

Because we support mouse events, you will need to hold option while dragging the mouse to indicate you're trying to select text rather than click on something. Alternatively you can disable mouse events via the `gui.ignoreMouseEvents` config value.

Mac Users: See [Issue #190](https://github.com/jesseduffield/lazydocker/issues/190) for other options.



# 3 Keybindings

https://github.com/jesseduffield/lazydocker/blob/master/docs/keybindings/Keybindings_zh.md


<pre>
  <kbd>e</kbd>: 编辑lazydocker配置
  <kbd>o</kbd>: 打开lazydocker配置
  <kbd>m</kbd>: 查看日志
  <kbd>enter</kbd>: 聚焦主面板
  <kbd>[</kbd>: 上一个选项卡
  <kbd>]</kbd>: 下一个选项卡
</pre>


## 3.1 主要

<pre>
  <kbd>esc</kbd>: 返回
</pre>

## 3.2 全局

<pre>
  <kbd>+</kbd>: 下一个屏幕模式（正常/半屏/全屏）
  <kbd>_</kbd>: 上一个屏幕模式
</pre>


## 3.3 容器

<pre>
  <kbd>d</kbd>: 移除
  <kbd>e</kbd>: 隐藏/显示已停止的容器
  <kbd>p</kbd>: 暂停
  <kbd>s</kbd>: 停止
  <kbd>r</kbd>: 重新启动
  <kbd>a</kbd>: attach
  <kbd>m</kbd>: 查看日志
  <kbd>E</kbd>: 执行shell
  <kbd>c</kbd>: 运行预定义的自定义命令
  <kbd>b</kbd>: 查看批量命令
  <kbd>w</kbd>: 在浏览器中打开(第一个端口为http)
  <kbd>enter</kbd>: 聚焦主面板
  <kbd>[</kbd>: 上一个选项卡
  <kbd>]</kbd>: 下一个选项卡
  <kbd>/</kbd>: 过滤列表
</pre>

## 3.4 服务

<pre>
  <kbd>u</kbd>: 启动服务
  <kbd>d</kbd>: 移除容器
  <kbd>s</kbd>: 停止
  <kbd>p</kbd>: 暂停
  <kbd>r</kbd>: 重新启动
  <kbd>S</kbd>: 启动项目
  <kbd>a</kbd>: attach
  <kbd>m</kbd>: 查看日志
  <kbd>U</kbd>: 创建并启动容器
  <kbd>D</kbd>: 停止并移除容器
  <kbd>R</kbd>: 查看重启选项
  <kbd>c</kbd>: 运行预定义的自定义命令
  <kbd>b</kbd>: 查看批量命令
  <kbd>E</kbd>: 执行shell
  <kbd>w</kbd>: 在浏览器中打开(第一个端口为http)
  <kbd>enter</kbd>: 聚焦主面板
  <kbd>[</kbd>: 上一个选项卡
  <kbd>]</kbd>: 下一个选项卡
  <kbd>/</kbd>: 过滤列表
</pre>

## 3.5 镜像

<pre>
  <kbd>c</kbd>: 运行预定义的自定义命令
  <kbd>d</kbd>: 移除镜像
  <kbd>b</kbd>: 查看批量命令
  <kbd>enter</kbd>: 聚焦主面板
  <kbd>[</kbd>: 上一个选项卡
  <kbd>]</kbd>: 下一个选项卡
  <kbd>/</kbd>: 过滤列表
</pre>

## 3.6 卷

<pre>
  <kbd>c</kbd>: 运行预定义的自定义命令
  <kbd>d</kbd>: 移除卷
  <kbd>b</kbd>: 查看批量命令
  <kbd>enter</kbd>: 聚焦主面板
  <kbd>[</kbd>: 上一个选项卡
  <kbd>]</kbd>: 下一个选项卡
  <kbd>/</kbd>: 过滤列表
</pre>

## 3.7 网络

<pre>
  <kbd>c</kbd>: 运行预定义的自定义命令
  <kbd>d</kbd>: 移除网络
  <kbd>b</kbd>: 查看批量命令
  <kbd>enter</kbd>: 聚焦主面板
  <kbd>[</kbd>: 上一个选项卡
  <kbd>]</kbd>: 下一个选项卡
  <kbd>/</kbd>: 过滤列表
</pre>


