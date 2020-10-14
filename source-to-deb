规范请见：[商店应用打包规范](https://doc.chinauos.com/production/details?id=oV0sb28BqCzb-QvsqVOJ)

本文档将演示根据商店打包规范从源码构建规范的软件包

1. #### **我们从零开始，首先写一个创建一个用来打包的目录和一个c文件**

```bash
➜  ~ mkdir -p build-demo/build-demo-0.0.1/src;cd build-demo/build-demo-0.0.1/src;vim demo.c
➜  src cat demo.c 
#include<stdio.h>
int main()
{
	printf("this is a demo ! \n");
	return 0;
}
```

本文主要演示打出一个规范的deb包，所以，如你所见，这个c文件就是这么简单，用来演示够用了

2. #### **接下来，写Makefile**

   同样简单的Makefile

   ``````bash
   ➜  com.apps.build-demo-0.0.1 ls
   Makefile  src
   ➜  com.apps.build-demo-0.0.1 cat Makefile 
   PREFIX = /opt/apps/com.apps.demo
   
   all: prepare build-bin
   
   prepare:
   	mkdir -p bin
   	
   build-bin:  
   	$(CC)  -o bin/demo src/demo.c
   
   install:
   	mkdir -p ${DESTDIR}${PREFIX}/files/bin
   	install -v bin/demo ${DESTDIR}${PREFIX}/files/bin
   
   clean:
   	rm -rf bin
   
   .PHONY: all clean
   ``````

   **注意**：根据商店规则，软件安装在`/opt/apps/`下，所以Makefile的PREFIX我们给了值为`/opt/apps/com.apps.demo`，软件的可执行文件要安装在 `/opt/apps/$包名$/files/bin/`目录下，所以 Makefile写的安装位置写的是  `${DESTDIR}${PREFIX}/files/bin`，如果你的Makefile之前已经写好，那你可能需要根据商店的规范修改一下

3. 接下来构建一个规范的软件目录，用来按规则放其他的文件，完整的目录结构为：

   ``````bash
   ➜  ~ tree demo/
   demo/					#此目录以包名命名
   ├── entries
   │   ├── applications	#放desktop文件，gui界面应用必须要有此目录和desktop文件
   │   ├── autostart		#放自启动入口文件
   │   ├── GConf			#gseetings文件
   │   ├── glib-2.0		#schame文件
   │   ├── icons			#应用的图标文件,根据大小放进不同的目录下的apps/目录下,支持的分辨率包括:16/24/32/48/128/256/512,svg格式的放在scalable/apps/目录下，gui界面应用必须要有
   │   │   └── hicolor
   │   │       ├── 16x16
   │   │       │   └── apps
   │   │       ├── 24x24
   │   │       │   └── apps
   │   │       ├── 32x32
   │   │       │   └── apps
   │   │       ├── 48x48
   │   │       │   └── apps
   │   │       ├── 128x128
   │   │       │   └── apps
   │   │       ├── 256x256
   │   │       │   └── apps
   │   │       ├── 512x512
   │   │       │   └── apps
   │   │       └── scalable
   │   │           └── apps
   │   ├── locale			#语言包目录
   │   ├── plugins			#插件目录
   │   └── services		#dbus服务目录
   │   └── help			#各种语言的帮助文档
   ├── files				#放其他文件
   └── info				#json格式的应用信息，info文件必须有
   
   23 directories, 1 file
   ``````

   info文件是应用的描述文件，使用`json`格式，完整的info文件格式如下：

   ``````json
   {
       "appid": "com.deepin.demo",
       "name": "Demo",
       "version": "5.0.0.0",
       "arch": ["amd64", "mips64"],
       "permissions": {
           "autostart": false,
           "notification": false,
           "trayicon": false,
           "clipboard": false,
           "account": false,
           "bluetooth": false,
           "camera": false,
           "audio_record": false,
           "installed_apps": false
       },
       "support-plugins": [
           "plugin/demo"
    ],
       "plugins": [
           "plugin/webbrowser",
           "plugin/office"
       ]
   }
   ``````
   
   info文件中各个字段的说明如下：
   
   appid：应用标识
   
   name：应用名称
   
   version：应用版本，格式为 {MAJOR}.{MINOR}.{PATCH}.{BUILD},所有版本号均为纯数字
   
   arch：应用支持架构，当前商店支持如下CPU架构
   
   - amd64：x86架构CPU
   
   - mips64：龙芯系列CPU
   
   - arm64：ARM64位CPU
   
   - sw_64：申威CPU
   
   permissions：应用权限描述。
   
   - autostart：是否允许自启动
   
   - notification：是否允许使用通知
   
   - trayicon：是否运行显示托盘图标
   
   - clipboard：是否允许使用剪切板
   
   - account：是否允许读取登录用户信息
   
   - bluetooth：是否允许使用蓝牙设备
   
   - camera：是否允许使用视频设备
   
   - audio_record：是否允许进行录音
   
   - installed_apps：是否允许读取安装软件列表
   
   support-plugins: 支持的插件类型
   
   plugins：实现的的插件类型，在对应的plugins目录下，按照实现的插件类型放置文件。
   
   在安装时，系统会将插件链接到对应的应用目录。
   
   本软件info内容参考如下：
   
   ``````json
   ➜  com.apps.build-demo-0.0.1 cat com.apps.demo/info
   {
       "appid": "com.apps.demo",
       "name": "Demo",
       "version": "0.0.1",
       "arch": ["amd64"],
       "permissions": {
           "autostart": false,
           "notification": false,
           "trayicon": false,
           "clipboard": false,
           "account": false,
           "bluetooth": false,
           "camera": false,
           "audio_record": false,
           "installed_apps": false
       }
   }
   
   ``````
   
   本演示包我提供了desktop文件和图标，所以本包的软件目录格式为：
   
   ``````bash
   ➜  com.apps.build-demo-0.0.1 tree demo 
   demo
   ├── entries
   │   ├── applications
   │   │   └── com.apps.demo.desktop	#desktop文件必须要以包名命名
   │   └── icons
   │       └── hicolor
   │           ├── 512x512
   │           │   └── apps
   │           │       └── com.apps.demo.png
   │           └── scalable
   │               └── apps
   │                   └── com.apps.demo.svg
   ├── files
   └── info
   
   9 directories, 4 files
   ``````
   
   到此，我们的源码构建完毕，源码树如下：

``````bash
➜  com.apps.build-demo-0.0.1 tree
.
├── demo
│   ├── entries
│   │   ├── applications
│   │   │   └── com.apps.demo.desktop
│   │   └── icons
│   │       └── hicolor
│   │           ├── 512x512
│   │           │   └── apps
│   │           │       └── com.apps.demo.png
│   │           └── scalable
│   │               └── apps
│   │                   └── com.apps.demo.svg
│   ├── files
│   └── info
├── Makefile
└── src
    └── demo.c

11 directories, 6 files
``````

4. 接下来使用`dh_make --createorig -s `命令生成`debian`目录

   当前目录下会自动创建`debian`目录,目录下有很多打包使用的模板文件,以.ex/.EX结尾,对于此软件包,我们不需要这些模板文件,所以全部删掉

   ``````bash
   ➜  com.apps.build-demo-0.0.1 dh_make --createorig -s
   Maintainer Name     : liuyong
   Email-Address       : liuyong@deepin.com
   Date                : Tue, 17 Mar 2020 15:44:30 +0800
   Package Name        : com.apps.build-demo
   Version             : 0.0.1
   License             : blank
   Package Type        : single
   Are the details correct? [Y/n/q]
   Done. Please edit the files in the debian/ subdirectory now.
   
   ➜  com.apps.build-demo-0.0.1 ls debian 
   build-demo.cron.d.ex    changelog  copyright        manpage.xml.ex  postrm.ex   README.Debian  source
   build-demo.doc-base.EX  compat     manpage.1.ex     menu.ex         preinst.ex  README.source  watch.ex
   build-demo-docs.docs    control    manpage.sgml.ex  postinst.ex     prerm.ex    rules
   ➜  com.apps.build-demo-0.0.1 rm debian/*.ex debian/*.EX
   ``````

5. 接下来可以根据自己的需要去编辑`changelog`，control等文件，我只修改了control文件：

   ``````bash
   ➜  com.apps.build-demo-0.0.1 cat debian/control 
   Source: com.apps.build-demo
   Section: utils
   Priority: optional
   Maintainer: liuyong <liuyong@deepin.com>
   Build-Depends: debhelper (>= 11)
   Standards-Version: 4.1.3
   Homepage: http://www.chinauos.com
   #Vcs-Browser: https://salsa.debian.org/debian/build-demo
   #Vcs-Git: https://salsa.debian.org/debian/build-demo.git
   
   Package: com.apps.build-demo
   Architecture: any
   Depends: ${shlibs:Depends}, ${misc:Depends}
   Description: this is a demo;
    <insert long description, indented with spaces>
   
   ``````

   接下来要指定下之前创建的软件包`demo`目录的安装路径，可以在Makefile里面写，也可以使用`debian/install` 来实现，本文使用install来实现

   在`debian`目录下写一个install，内容如下：

   ``````bash
   ➜  com.apps.build-demo-0.0.1 vim debian/install
   ➜  com.apps.build-demo-0.0.1 cat debian/install
   com.apps.demo/ /opt/apps
   ``````

   代表将com.apps.demo目录按照规则装在`/opt/apps/`目录下

6. 然后`debuild`或者`dpkg-buildpackage `打包

   接下来可以看到在上级目录有了打包的deb包，但在`debuild`过程中，包的帮助手册，以及`copyright`和`changelogs`都会在打包过程中安装到`/usr/share/doc`目录下，我们要把这个目录改装在entries目录下，使用`fakeroot dpkg-deb -R  pkg.deb a` 来将名为pkg的deb包解压到 `a` 目录下，doc目录mv到entries目录下，然后删除空的`/usr/share/`目录，使用`fakeroot dpkg-deb -b a pkg.deb`将改好的包格式压入deb包内，可以使用`dpkg-deb -c pkg.deb`命令来查看deb包的安装文件目录，确定所以文件都安装在/opt/apps/包名 这个目录下即可

``````bash
➜  com.apps.build-demo fakeroot dpkg-deb -R com.apps.build-demo_0.0.1-1_amd64.deb a
➜  com.apps.build-demo mv a/usr/share/doc a/opt/apps/com.apps.demo/entries 
➜  com.apps.build-demo rm -rf a/usr
➜  com.apps.build-demo ls a 
DEBIAN  opt  
➜  com.apps.build-demo fakeroot dpkg-deb -b a com.apps.build-demo_0.0.1-1_amd64.deb
dpkg-deb: 正在 'com.apps.build-demo_0.0.1-1_amd64.deb' 中构建软件包 'com.apps.build-demo'。
``````



通往罗马的路不止一条，打包也不止一种方式，示例中的很多打包规则都可以使用其他方法替代，比如desktop文件你可以写脚本让它在打包过程中生成，比如安装文件到指定目录可以用install，也可以在Makefile里写，比如生成在doc目录的copyright和changelogs的压缩文件是由dh_installdocs和dh_installchangedocs生成的，你可以在rules里override掉自己写，让他们直接安装在指定的目录去

**重点是打成的包要符合商店的规范，大致如下：**

包名要使用倒置域名规则

应用的全部安装文件必须在/opt/apps/你的包名/ 这个目录下

包中不允许存在post/pre  inst/rm 这用钩子脚本

应用根目录下要有entries和files目录和json格式的info文件

应用只允许使用普通用户权限启动，禁止应用以任何形式获取root权限

entries目录存放程序的各种入口文件

​		desktop文件放在 entries/applications目录下

​		自启动文件放在 entries/autostart目录下

​		图标根据尺寸大小放在 entries/icons/hicolor/尺寸/apps/目录下，SVG格式图标放在      entries/icons/hicolor/scalable/apps目录下

​		插件放在 entries/plugins目录下

​		dbus服务和systemd服务放在 entries/services目录下

​		schema文件放在 entries/glib-2.0目录下

​		gseetings文件放在 entries/GConf目录下

​		语言包文件放在 entries/locale目录下

​		软件帮助文档放在放在 entries/help目录下

​		其他文件放在files目录下
