# 系统软件及配置

---

<!--sec data-title="MacOS常用设置" data-id="system_0" data-show=true ces-->

### 1. 显示隐藏文件
* 显示隐藏文件：`defaults write com.apple.finder AppleShowAllFiles -bool true`
* 关闭显示隐藏文件：`defaults write com.apple.finder AppleShowAllFiles -bool false`

### 2. 自动补全忽略大小写
* 打开终端，输入: `nano .inputrc`
* 在里面粘贴上以下语句：
 * `set completion-ignore-case on`
 * `set show-all-if-ambiguous on`
 * `TAB: menu-complete`
* control+o保存，control+x退出，重启终端，OK！

### 3. 开启Apache服务器

开启Web服务器的方法有两种：
1. 打开“系统设置偏好（System Preferences）” -> “共享（Sharing）” -> “Web共享（Web Sharing）”；
2. 通过在terminal终端直接运行Apache的启动命令来打开：sudo apachectl start。

#### 备注：
* Apache服务器默认的web根目录在：/Library/WebServer/Documents。
* Apache的配置文件在：/etc/apache2。
* 重启apache：sudo /usr/sbin/apachectl restart
* 关闭apache：sudo /usr/sbin/apachectl stop
* 开启apache：sudo /usr/sbin/apachectl start

### 4. 开启PHP模块
因Mac OS X已经内置PHP，因此我们只需要在Apache的配置中加载PHP模块即可。方法如：
1. 在终端运行：sudo vi /etc/apache2/httpd.conf，打开Apache配置文件。
2. 找到#LoadModule php5_module libexec/apache2/libphp5.so类似条目，将注释符#去掉，并保存。
3. 终端运行：sudo apachectl restart，重启Apache服务器。

### 5. 安装和启动MySQL
#### 使用Homebrew安装MySQL：

在Mac OS X上安装软件，你可以直接找到相关img安装，也可以像Ubuntu的apt-get类似方便的，可以使用`brew install`进行。

当然，使用此功能，你需要安装Homebrew，安装方法是在终端运行命令：`ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"`。

安装Mysql，在终端运行：`brew install mysql`。brew程序将自动安装mysql的依赖库openssl，然后安装mysql，我的安装的是：mysql-5.6.2。

根据上面安装结束的提示，启动MySQL，在终端运行：`mysql.server start`。启动成功后使用：`mysql -uroot`即可连接到MySQL数据库。

### 6. 安装homebrew和bazel
1. 安装Homebrew：`ruby -e "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/master/install)"`
2. 安装bazel：`brew install bazel`
<!--endsec-->