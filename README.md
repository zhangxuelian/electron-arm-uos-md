本文主要演示从带二进制源码打出符合UOS商店规范的deb包

商店打包规范请参考uos[官网文档](https://doc.chinauos.com/production/details?id=oV0sb28BqCzb-QvsqVOJ)

### **第一步,配置好环境变量值**

环境变量值  在家目录下的`.bashrc`（如果你使用的bash shell）文件中加入如下三行,根据个人信息替换

我使用的`zsh` 所以在`.zshrc`文件下配置

``````bash
➜  ~ head -n 3 .zshrc
DEBFULLNAME="liuyong"
DEBEMAIL="liuyong@deepin.com"
export DEBFULLNAME DEBEMAIL

``````

构建deb包可分为从源码编译构建和从二进制包直接构建,二进制包已有编译好的二进制程序,可直接运行,无需再编译,直接构建成deb包即可,本文讲从二进制包构建deb包

本文的实例包为`jetbrains`公司的`IDE` -- `Clion`     [Clion下载地址](https://www.jetbrains.com/clion/)

下载完后，接下来我们来将他构建为一个deb包

1. ####  首先创建一个用来构建deb包的目录,目录要以 包名-版本号 的格式

   ``````bash
   ➜  ~ mkdir -p clion/com.jetbrains.clion-2019.3.5
   ``````

2. #### 接下来构建一个规范的软件目录，用来按规则放其他的文件，完整的目录结构为：

   ``````bash
   ➜  ~ cd clion/com.jetbrains.clion-2019.3.5
   ➜  com.jetbrains.clion-2019.3.5 tree com.jetbrains.clion
   com.jetbrains.clion		#此目录以包名命名
   ├── entries				#此目录必须有
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
   ├── files				#放其他文件	此目录必须有
   └── info				#json格式的应用信息，info文件必须有
   
   23 directories, 1 file
   ``````

   **将二进制包解压，并将对应的文件放入相应的位置**

   ``````bash
   ➜  com.jetbrains.clion tar -xvzf ~/Downloads/CLion-2019.3.5.tar.gz
   ➜  com.jetbrains.clion ls
   clion-2019.3.5  entries  files  info
   ➜  com.jetbrains.clion cp clion-2019.3.5/bin/clion.svg 	entries/icons/hicolor/scalable/apps #svg图标按照规则放这个目录
   ➜  com.jetbrains.clion cp clion-2019.3.5/bin/clion.png entries/icons/hicolor/128x128/apps  #png格式按照尺寸放
   ➜  com.jetbrains.clion mv clion-2019.3.5/* files   #其他文件放files目录
   ➜  com.jetbrains.clion rmdir clion-2019.3.5 
   ➜  com.jetbrains.clion ls files 
   bin  build.txt  help  Install-Linux-tar.txt  jbr  lib  license  plugins  product-info.json
   ``````

3. #### 写一个规范的info文件

   info文件是应用的描述文件，使用`json`格式，完整的info文件格式如下：

   ``````bash
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

   **info文件中各个字段的说明如下：**

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
	
	**本软件info内容参考如下：**
	
	``````bash
	➜  com.jetbrains.clion cat info 
	{
	 "appid": "com.jetbrains.clion",
	 "name": "CLion",
	 "version": "2019.3.5",
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
	
4. #### 写一个desktop文件以包名.desktop为格式，本包写好的文件格式如下

   ``````bash
   ➜  com.jetbrains.clion vim entries/applications/com.jetbrains.clion.desktop
   ➜  com.jetbrains.clion cat entries/applications/com.jetbrains.clion.desktop
   [Desktop Entry]
   Name=CLion
   Comment=The Drive to Develop
   Exec=/opt/apps/com.jetbrains.clion/files/bin/clion.sh
   Icon=clion
   Terminal=false
   Categories=Development
   Type=Application
   
   ``````

   desktop[官方文档](https://specifications.freedesktop.org/desktop-entry-spec/desktop-entry-spec-1.1.html)

      **desktop语法解释**

      [Desktop Entry]	文件头

      Name		  			应用名称

      Comment   			软件描述信息

      Exec 						软件的可执行文件（可以是二进制或者脚本）

      Icon						  图标名

      Terminal				  是否使用终端

      Type						 启动器类型

      Categories			  应用的类型 [可用的字段参考](https://specifications.freedesktop.org/menu-spec/latest/apa.html)

5. #### 接下来,回到上层目录，使用 dh_make --createorig -s 命令创建debian目录,并在上层目录初始化出来源码包

   ``````bash
   ➜  com.jetbrains.clion-2019.3.5 dh_make --createorig -s
   Maintainer Name     : liuyong
   Email-Address       : liuyong@deepin.com
   Date                : Wed, 25 Mar 2020 14:27:06 +0800
   Package Name        : com.jetbrains.clion
   Version             : 2019.3.5
   License             : blank
   Package Type        : single
   Are the details correct? [Y/n/q]
   
   ``````

   确认信息无误输入y即可

   当前目录下会自动创建debian目录,目录下有很多打包使用的模板文件,以.ex/.EX结尾,具体用途参考 [模板文件参考](https://www.debian.org/doc/manuals/maint-guide/dother.zh-cn.html),对于此软件包,我们不需要这些模板文件,所以全部删掉

   ```````bash
   ➜  com.jetbrains.clion-2019.3.5 ls
   com.jetbrains.clion  debian
   ➜  com.jetbrains.clion-2019.3.5 ls debian 
   changelog                        control          menu.ex      README.Debian
   com.jetbrains.clion.cron.d.ex    copyright        postinst.ex  README.source
   com.jetbrains.clion.doc-base.EX  manpage.1.ex     postrm.ex    rules
   com.jetbrains.clion-docs.docs    manpage.sgml.ex  preinst.ex   source
   compat                           manpage.xml.ex   prerm.ex     watch.ex
   ➜  com.jetbrains.clion-2019.3.5 rm debian/*.EX debian/*.ex
   ➜  com.jetbrains.clion-2019.3.5 ls debian                 
   changelog                      compat   copyright      README.source  source
   com.jetbrains.clion-docs.docs  control  README.Debian  rules
   ```````

   **然后修改control文件**

   对各个Control文件的具体描述说明,参考[Control文件说明](https://docs.deepin.io/?p=3738),我们本次打包,只需修改sections字段,homepage字段,Architecture字段,Description字段,修改完后如下格式

   ``````bash
   ➜  com.jetbrains.clion-2019.3.5 vim debian/control 
   ➜  com.jetbrains.clion-2019.3.5 cat debian/control
   Source: com.jetbrains.clion
   Section: devel
   Priority: optional
   Maintainer: liuyong <liuyong@deepin.com>
   Build-Depends: debhelper (>= 11)
   Standards-Version: 4.1.3
   Homepage: http://jetbrains.com
   #Vcs-Browser: https://salsa.debian.org/debian/com.jetbrains.clion
   #Vcs-Git: https://salsa.debian.org/debian/com.jetbrains.clion.git
   
   Package: com.jetbrains.clion
   Architecture: amd64
   Depends: ${shlibs:Depends}, ${misc:Depends}
   Description: C/C++ smart IDE
    <insert long description, indented with spaces>
   ``````

   **本次修改的几个字段解释**

   sections  为软件分类,字段值参考 [sections字段值参考](https://packages.debian.org/en/sid/)

   homepage  软件的主页的网址

   Architecture  支持的架构,因为该二进制为64位,所以只用写amd64

   descriptions  软件的描述信息

   

   接下来在debian目录创建install文件,install文件可以指定各个文件的安装路径,[官方文档](http://www.tin.org/bin/man.cgi?section=1&topic=dh_install) 本包的install文件参考：

   ``````bash
   ➜  com.jetbrains.clion-2019.3.5 vim debian/install                       
   ➜  com.jetbrains.clion-2019.3.5 cat debian/install
   com.jetbrains.clion/ /opt/apps
   ``````

   

   因为是从二进制包构建的,不用编译和生成共享库等行为,所以我们可以在rules文件里覆盖掉这些指令,让他什么都不做,本包的rules文件参考如下：（出了标注出来的行，其他为默认生成） [debian官方文档](https://www.debian.org/doc/manuals/maint-guide/dreq.zh-cn.html#rules)

   ``````bash
   ➜  com.jetbrains.clion-2019.3.5 cat debian/rules
   #!/usr/bin/make -f
   # See debhelper(7) (uncomment to enable)
   # output every command that modifies files on the build system.
   export DH_VERBOSE = 1
   
   
   # see FEATURE AREAS in dpkg-buildflags(1)
   #export DEB_BUILD_MAINT_OPTIONS = hardening=+all
   
   # see ENVIRONMENT in dpkg-buildflags(1)
   # package maintainers to append CFLAGS
   #export DEB_CFLAGS_MAINT_APPEND  = -Wall -pedantic
   # package maintainers to append LDFLAGS
   #export DEB_LDFLAGS_MAINT_APPEND = -Wl,--as-needed
   
   #for mutil-arch packages
   #source_tar = ../
   #source_dir = ../
   #ifneq ($(DEB_HOST_ARCH),amd64)
   #	source_tar = ../
   #	source_dir = ../
   #endif
   
   %:
   	dh $@
   
   override_dh_auto_build:				#这是加入的行
   
   override_dh_shlibdeps:				#这是加入的行
   
   override_dh_strip:					#这是加入的行
   
   
   # dh_make generated override targets
   # This is example for Cmake (See https://bugs.debian.org/641051 )
   #override_dh_auto_configure:
   #	dh_auto_configure -- #	-DCMAKE_LIBRARY_PATH=$(DEB_HOST_MULTIARCH)
   
   
   ``````

   接下来执行`dpkg-source -b .`和 `dpkg-buildpackage -us -uc -nc`在打出deb包

   ```````bash
   ➜  com.jetbrains.clion-2019.3.5 dpkg-source -b .
   dpkg-source: info: using source format '3.0 (quilt)'
   dpkg-source: info: building com.jetbrains.clion using existing ./com.jetbrains.clion_2019.3.5.orig.tar.xz
   dpkg-source: info: building com.jetbrains.clion in com.jetbrains.clion_2019.3.5-1.debian.tar.xz
   dpkg-source: info: building com.jetbrains.clion in com.jetbrains.clion_2019.3.5-1.dsc
   ➜  com.jetbrains.clion-2019.3.5 dpkg-buildpackage -us -uc -nc
   ➜  com.jetbrains.clion-2019.3.5 ls ..
   com.jetbrains.clion-2019.3.5
   com.jetbrains.clion_2019.3.5-1_amd64.buildinfo
   com.jetbrains.clion_2019.3.5-1_amd64.changes
   com.jetbrains.clion_2019.3.5-1_amd64.deb
   com.jetbrains.clion_2019.3.5-1.debian.tar.xz
   com.jetbrains.clion_2019.3.5-1.dsc
   com.jetbrains.clion_2019.3.5.orig.tar.xz
   ```````

   但在构建deb包的过程中，包的帮助手册，以及`copyright`和`changelogs`都会在打包过程中安装到`/usr/share/doc`目录下，我们要把这个目录改装在entries目录下，使用`fakeroot dpkg-deb -R  pkg.deb a` 来将名为pkg的deb包解压到 `a` 目录下，doc目录mv到entries目录下，然后删除空的`/usr/share/`目录，使用`fakeroot dpkg-deb -b a pkg.deb`将改好的包格式压入deb包内，可以使用`dpkg-deb -c pkg.deb`命令来查看deb包的安装文件目录，确定所以文件都安装在/opt/apps/包名 这个目录下即可

   ``````bash
   ➜  clion fakeroot dpkg-deb -R com.jetbrains.clion_2019.3.5-1_amd64.deb a
   ➜  clion mv a/usr/share/doc a/opt/apps/com.jetbrains.clion/files
   ➜  clion rm -rf a/usr 
   ➜  clion fakeroot dpkg-deb -b a com.jetbrains.clion_2019.3.5-1_amd64.deb
   ``````

   然后可以通过`dpkg -i`测试安装 ，看软件包是否可以在启动器显示图标，正常运行即可
