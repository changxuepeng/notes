# [CentOS之yum安装JDK](https://www.linuxprobe.com/centos-yum-jdk.html)

https://www.linuxprobe.com/centos-yum-jdk.html

**1.查看云端目前支持安装的jdk版本**

```xml
[root@localhost ~]# yum search java|grep jdk
ldapjdk-javadoc.noarch : Javadoc for ldapjdk
java-1.6.0-openjdk.x86_64 : OpenJDK Runtime Environment
java-1.6.0-openjdk-demo.x86_64 : OpenJDK Demos
java-1.6.0-openjdk-devel.x86_64 : OpenJDK Development Environment
java-1.6.0-openjdk-javadoc.x86_64 : OpenJDK API Documentation
java-1.6.0-openjdk-src.x86_64 : OpenJDK Source Bundle
java-1.7.0-openjdk.x86_64 : OpenJDK Runtime Environment
java-1.7.0-openjdk-accessibility.x86_64 : OpenJDK accessibility connector
java-1.7.0-openjdk-demo.x86_64 : OpenJDK Demos
java-1.7.0-openjdk-devel.x86_64 : OpenJDK Development Environment
java-1.7.0-openjdk-headless.x86_64 : The OpenJDK runtime environment without
java-1.7.0-openjdk-javadoc.noarch : OpenJDK API Documentation
java-1.7.0-openjdk-src.x86_64 : OpenJDK Source Bundle
java-1.8.0-openjdk.i686 : OpenJDK Runtime Environment
java-1.8.0-openjdk.x86_64 : OpenJDK Runtime Environment
java-1.8.0-openjdk-accessibility.i686 : OpenJDK accessibility connector
java-1.8.0-openjdk-accessibility.x86_64 : OpenJDK accessibility connector
java-1.8.0-openjdk-accessibility-debug.i686 : OpenJDK accessibility connector
java-1.8.0-openjdk-accessibility-debug.x86_64 : OpenJDK accessibility connector
java-1.8.0-openjdk-debug.i686 : OpenJDK Runtime Environment with full debug on
java-1.8.0-openjdk-debug.x86_64 : OpenJDK Runtime Environment with full debug on
java-1.8.0-openjdk-demo.i686 : OpenJDK Demos
java-1.8.0-openjdk-demo.x86_64 : OpenJDK Demos
java-1.8.0-openjdk-demo-debug.i686 : OpenJDK Demos with full debug on
java-1.8.0-openjdk-demo-debug.x86_64 : OpenJDK Demos with full debug on
java-1.8.0-openjdk-devel.i686 : OpenJDK Development Environment
java-1.8.0-openjdk-devel.x86_64 : OpenJDK Development Environment
java-1.8.0-openjdk-devel-debug.i686 : OpenJDK Development Environment with full
java-1.8.0-openjdk-devel-debug.x86_64 : OpenJDK Development Environment with
java-1.8.0-openjdk-headless.i686 : OpenJDK Runtime Environment
java-1.8.0-openjdk-headless.x86_64 : OpenJDK Runtime Environment
java-1.8.0-openjdk-headless-debug.i686 : OpenJDK Runtime Environment with full
java-1.8.0-openjdk-headless-debug.x86_64 : OpenJDK Runtime Environment with full
java-1.8.0-openjdk-javadoc.noarch : OpenJDK API Documentation
java-1.8.0-openjdk-javadoc-debug.noarch : OpenJDK API Documentation for packages
java-1.8.0-openjdk-javadoc-zip.noarch : OpenJDK API Documentation compressed in
java-1.8.0-openjdk-javadoc-zip-debug.noarch : OpenJDK API Documentation
java-1.8.0-openjdk-src.i686 : OpenJDK Source Bundle
java-1.8.0-openjdk-src.x86_64 : OpenJDK Source Bundle
java-1.8.0-openjdk-src-debug.i686 : OpenJDK Source Bundle for packages with
java-1.8.0-openjdk-src-debug.x86_64 : OpenJDK Source Bundle for packages with
ldapjdk.noarch : The Mozilla LDAP Java SDK
```

**2.选择版本后，安装（执行以下[命令](https://www.linuxcool.com/)会自动安装jdk相关依赖**

```
[root@localhost ~]#  yum install -y java-1.8.0-openjdk
已加载插件：fastestmirror
Loading mirror speeds from cached hostfile
 * base: mirrors.zju.edu.cn
 * extras: mirror.bit.edu.cn
 * updates: mirrors.aliyun.com
正在解决依赖关系
--> 正在检查事务
---> 软件包 java-1.8.0-openjdk.x86_64.1.1.8.0.151-5.b12.el7_4 将被 安装
--> 正在处理依赖关系 java-1.8.0-openjdk-headless(x86-64) = 1:1.8.0.151-5.b12.el7_4，它被软件包 1:java-1.8.0-openjdk-1.8.0.151-5.b12.el7_4.x86_64 需要
--> 正在处理依赖关系 xorg-x11-fonts-Type1，它被软件包 1:java-1.8.0-openjdk-1.8.0.151-5.b12.el7_4.x86_64 需要
--> 正在处理依赖关系 libpng15.so.15(PNG15_0)(64bit)，它被软件包 1:java-1.8.0-openjdk-1.8.0.151-5.b12.el7_4.x86_64 需要
--> 正在处理依赖关系 libjvm.so(SUNWprivate_1.1)(64bit)，它被软件包 1:java-1.8.0-openjdk-1.8.0.151-5.b12.el7_4.x86_64 需要
--> 正在处理依赖关系 libjpeg.so.62(LIBJPEG_6.2)(64bit)，它被软件包 1:java-1.8.0-openjdk-1.8.0.151-5.b12.el7_4.x86_64 需要
--> 正在处理依赖关系 libjli.so(SUNWprivate_1.1)(64bit)，它被软件包 1:java-1.8.0-openjdk-1.8.0.151-5.b12.el7_4.x86_64 需要
--> 正在处理依赖关系 libjava.so(SUNWprivate_1.1)(64bit)，它被软件包 1:java-1.8.0-openjdk-1.8.0.151-5.b12.el7_4.x86_64 需要
--> 正在处理依赖关系 fontconfig(x86-64)，它被软件包 1:java-1.8.0-openjdk-1.8.0.151-5.b12.el7_4.x86_64 需要
--> 正在处理依赖关系 libpng15.so.15()(64bit)，它被软件包 1:java-1.8.0-openjdk-1.8.0.151-5.b12.el7_4.x86_64 需要
--> 正在处理依赖关系 libjvm.so()(64bit)，它被软件包 1:java-1.8.0-openjdk-1.8.0.151-5.b12.el7_4.x86_64 需要
--> 正在处理依赖关系 libjpeg.so.62()(64bit)，它被软件包 1:java-1.8.0-openjdk-1.8.0.151-5.b12.el7_4.x86_64 需要
--> 正在处理依赖关系 libjli.so()(64bit)，它被软件包 1:java-1.8.0-openjdk-1.8.0.151-5.b12.el7_4.x86_64 需要
--> 正在处理依赖关系 libjava.so()(64bit)，它被软件包 1:java-1.8.0-openjdk-1.8.0.151-5.b12.el7_4.x86_64 需要
--> 正在处理依赖关系 libgif.so.4()(64bit)，它被软件包 1:java-1.8.0-openjdk-1.8.0.151-5.b12.el7_4.x86_64 需要
--> 正在处理依赖关系 libawt.so()(64bit)，它被软件包 1:java-1.8.0-openjdk-1.8.0.151-5.b12.el7_4.x86_64 需要
--> 正在处理依赖关系 libXtst.so.6()(64bit)，它被软件包 1:java-1.8.0-openjdk-1.8.0.151-5.b12.el7_4.x86_64 需要
--> 正在处理依赖关系 libXrender.so.1()(64bit)，它被软件包 1:java-1.8.0-openjdk-1.8.0.151-5.b12.el7_4.x86_64 需要
--> 正在处理依赖关系 libXi.so.6()(64bit)，它被软件包 1:java-1.8.0-openjdk-1.8.0.151-5.b12.el7_4.x86_64 需要
--> 正在处理依赖关系 libXext.so.6()(64bit)，它被软件包 1:java-1.8.0-openjdk-1.8.0.151-5.b12.el7_4.x86_64 需要
--> 正在处理依赖关系 libXcomposite.so.1()(64bit)，它被软件包 1:java-1.8.0-openjdk-1.8.0.151-5.b12.el7_4.x86_64 需要
--> 正在处理依赖关系 libX11.so.6()(64bit)，它被软件包 1:java-1.8.0-openjdk-1.8.0.151-5.b12.el7_4.x86_64 需要
... ...
```

**3.安装完成，验证是否安装成功**

```
[root@localhost ~]# java -version
openjdk version "1.8.0_151"
OpenJDK Runtime Environment (build 1.8.0_151-b12)
OpenJDK 64-Bit Server VM (build 25.151-b12, mixed mode)
```

**4.通过搜索java文件，查找jdk默认安装目录**

```
[root@localhost ~]# find / -name 'java'
/etc/pki/ca-trust/extracted/java
/etc/pki/java
/etc/java
/etc/alternatives/java
/var/lib/alternatives/java
/usr/bin/java
/usr/lib/java
/usr/lib/jvm/java-1.8.0-openjdk-1.8.0.151-5.b12.el7_4.x86_64/jre/bin/java
/usr/share/java
```

