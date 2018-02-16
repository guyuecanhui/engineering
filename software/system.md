# 系统软件及配置

---

<!--sec data-title="MacOS常用设置" data-id="system_0" data-show=true ces-->

### 显示隐藏文件
* 显示隐藏文件：`defaults write com.apple.finder AppleShowAllFiles -bool true`
* 关闭显示隐藏文件：`defaults write com.apple.finder AppleShowAllFiles -bool false`

### 自动补全忽略大小写
* 打开终端，输入: `nano .inputrc`
* 在里面粘贴上以下语句：
 * `set completion-ignore-case on`
 * `set show-all-if-ambiguous on`
 * `TAB: menu-complete`
* control+o保存，control+x退出，重启终端，OK！

<!--endsec-->