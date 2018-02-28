# Puppet小结

---

### Puppet简介
puppet是一种Linux、Unix、windows平台的集中配置管理系统，使用自有的puppet描述语言，可管理配置文件、用户、cron任务、软件包、系统服务等。puppet把这些系统实体称之为资源（Resource），puppet的设计目标是简化对这些资源的管理以及妥善处理资源间的依赖关系。

puppet 分为企业版和社区版，社区版是免费的，有些常用的功能，企业版是收费的。

### Puppet整体架构

![puppet architecture](http://rnd-isourceb.huawei.com/images/SZ/20170801/5770447c-74f4-4a4f-99a1-e5facbbe6d32/puppet_architecture.png)
puppet是基于c/s架构的，整体架构如上图所示。服务器端保存着所有对客户端服务器的配置代码，在puppet里面叫做manifest。客户端下载manifest之后，可以根据manifest对服务器进行配置，例如软件包管理，用户管理和文件管理等等。

![puppet working flow](http://rnd-isourceb.huawei.com/images/SZ/20170801/a67fba63-ee46-4705-b46a-35f9ff46e18c/puppet_flow.jpg)
如上图所示，puppet的工作流程如下，所有数据的交互都通过SSL进行，以保证安全：

1. 客户端Puppetd向Master发起认证请求，或使用带签名的证书。
2. Master告诉Client你是合法的。
3. 客户端Puppetd调用Facter，Facter探测出主机的一些变量，例如主机名、内存大小、IP地址等。Puppetd将这些信息通过SSL连接发送到服务器端。
4. 服务器端的Puppet Master检测客户端的主机名，然后找到manifest对应的node配置，并对该部分内容进行解析。Facter送过来的信息可以作为变量处理，node牵涉到的代码才解析，其他没牵涉的代码不解析。解析分为几个阶段，首先是语法检查，如果语法错误就报错；如果语法没错，就继续解析，解析的结 果生成一个中间的“伪代码”（catelog），然后把伪代码发给客户端。
5. 客户端接收到“伪代码”，并且执行。
6. 客户端在执行时判断有没有File文件，如果有，则向fileserver发起请求。
7. 客户端判断有没有配置Report，如果已配置，则把执行结果发送给服务器。
8. 服务器端把客户端的执行结果写入日志，并发送给报告系统。

### Puppet平台构成

* **Puppet**: Puppet is the core of our configuration management platform. It consists of a special programming language for describing desired system states, an agent that can enforce desired states, and several other tools and services.

* **Puppet server**: Puppet Server is the JVM application that provides Puppet’s core HTTPS services. Whenever Puppet agent checks in to request a configuration catalog for a node, it contacts Puppet Server.

* **Facter**: Facter is a system profiling tool. Puppet agent uses it to send important system info to Puppet Server, which can access that info when compiling that node’s catalog.

* **Hiera**: Hiera is a hierarchical data lookup tool. You can use it to configure your Puppet classes.

* **PuppetDB**: PuppetDB collects the data Puppet generates, and offers a powerful query API for analyzing that data. It’s the foundation of the PE console, and you can also use the API to build your own applications. If you’re interacting with PuppetDB directly, you’ll mostly be using the query API.

### Puppet核心概念

* **Puppet Language：**Puppet language的核心是声明Resource，语言的其他部分都是为了增加Resource声明的灵活性和方便性。

 * **Resources：**资源主要描述单个file或package，常见的资源类型包括user, file, package, service, cron, exec等。

 * **Class：**一组资源可以组织成class，class是一个更大的配置单元。一个class可以描述配置一个完成的服务或者应用的所有需要的配置信息，如任意数量的packages, config files, service daemons, 以及maintenance tasks。小的classes可以组合成大的classes，以描述整个系统的角色。

 * **Node：**承担不同角色的节点往往用不同的class的组合来描述，区分节点角色的配置过程称为classification，节点即可以用node definition来区分，也可以用ENC或Hiera来区分。

* **Ordering：**Puppet假设大部分的资源间都是没有关联关系的，如果以需要以特定的顺序管理它们，需要显式地定义这些关系。

* **Files：**使用puppet language定义的文件称为manifests (.pp文件)，这些文件用UTF8编码，puppet永远从一个site manifest/ main manifest开始编译，类似于main函数的入口。**"package/file/service"** 是一种非常有用的配置模式，package资源确保软件已经安装，配置文件依赖于package资源，service资源则订阅配置文件的变化通知。

### 一个简单的例子

```
case $operatingsystem {
  centos, redhat: { $service_name = 'ntpd' }
  debian, ubuntu: { $service_name = 'ntp' }
}

package { 'ntp':
  ensure => installed,
}

service { 'ntp':
  name      => $service_name,
  ensure    => running,
  enable    => true,
  subscribe => File['ntp.conf'],
}

file { 'ntp.conf':
  path    => '/etc/ntp.conf',
  ensure  => file,
  require => Package['ntp'],
  source  => "puppet:///modules/ntp/ntp.conf",
  # This source file would be located on the Puppet master at
  # /etc/puppetlabs/code/modules/ntp/files/ntp.conf
}

```
> 讲的比较好的文章列表：
> * [puppet官网概述](https://docs.puppet.com/pe/latest/puppet_overview.html)
> * [puppet中文wiki](http://puppet.wikidot.com/intro)
> * [集中化运维管理——Puppet管理之路](http://blog.csdn.net/wenhuiqiao/article/details/7998715)