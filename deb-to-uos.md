### 打包示例

本文档使用`eog`包进行演示从旧规则的包改成新规则的包的过程示例

如果是从源码直接编译,请根据[商店打包规范](https://shimo.im/docs/dhQd9R3PYprtPVDj/read) 遵守新规则规范即可



#### 第一步 获取deb包

使用`apt download eog`或者其他方式来下载一个旧规则的`eog`包

然后使用`fakeroot dpkg-deb -R  pkg.deb a` 来将名为pkg的deb包解压到 `a` 目录下



```bash
deepin@deepin-PC:~/test/eog$ apt download eog
获取:1 http://10.0.10.25/uos eagle/main amd64 eog amd64 3.28.4-2+b1 [5,092 kB]
已下载 5,092 kB，耗时 0秒 (108 MB/s)
deepin@deepin-PC:~/test/eog$ fakeroot dpkg-deb -R eog_3.28.4-2+b1_amd64.deb a
deepin@deepin-PC:~/test/eog$ ls
a  eog_3.28.4-2+b1_amd64.deb

```



#### 第二步 改包名和版本号

按照新规则,我们要使用倒置域名规则来命令包名,并且要升级一下版本号

修改a目录下的`DEBIAN`目录下的`control`文件,具体为修改Package字段和Version字段

未修改之前的

```
deepin@deepin-PC:~/test/eog$ cd a
deepin@deepin-PC:~/test/eog/a$ head -n 5 DEBIAN/control 
Package: eog
Source: eog (3.28.4-2)
Version: 3.28.4-2+b1
Architecture: amd64
Maintainer: Debian GNOME Maintainers <pkg-gnome-maintainers@lists.alioth.debian.org>
```

修改之后为

```bash
deepin@deepin-PC:~/test/eog/a$ vim DEBIAN/control 
deepin@deepin-PC:~/test/eog/a$ head -n 5 DEBIAN/control 
Package: org.gnome.eog
Source: eog (3.28.4-2)
Version: 3.28.4-3+b1
Architecture: amd64
Maintainer: Debian GNOME Maintainers <pkg-gnome-maintainers@lists.alioth.debian.org>

```

包名变成了`org.gnome.eog` ,版本号变成了 `3.28.4-3+b1`

**版本号由软件版本号和打包版本号组成,由`-`分割,左边是软件版本号,右边是打包版本号,我们升级版本号只修改打包版本号**

#### 第三部 创建新规则包结构,并且将旧规则的包内容移到新规则包结果里面

新规则包结构大致为

```bash
deepin@deepin-PC:~/test/eog/a$ tree opt/
opt/
└── apps
    └── org.gnome.eog	#此目录以包名命名
        ├── entries
        │   ├── applications	#放desktop文件
        │   ├── autostart		#放自启动入口文件
        │   ├── icons	#应用的图标文件,根据大小放进不同的目录下的apps/目录下,svg格式的放在scalable/apps/目录下
        │   │   └── hicolor
        │   │       ├── 128x128
        │   │       │   └── apps
        │   │       ├── 16x16
        │   │       │   └── apps
        │   │       ├── 24x24
        │   │       │   └── apps
        │   │       ├── 256x256
        │   │       │   └── apps
        │   │       ├── 32x32
        │   │       │   └── apps
        │   │       ├── 48x48
        │   │       │   └── apps
        │   │       ├── 512x512
        │   │       │   └── apps
        │   │       ├── 64x64
        │   │       │   └── apps
        │   │       └── scalable
        │   │           └── apps
        │   ├── plugins		#插件的目录
        │   └── services	#dbus服务目录
        │   └── glib-2.0	#schema文件
        │   └── GConf		#gseetings文件
        │   └── locale		#语言包文件
        ├── files			#其他文件
        └── info			#应用的一些信息

28 directories, 1 file

```



结构的具体解释可以参考  [商店打包规范](https://doc.chinauos.com/production/details?id=oV0sb28BqCzb-QvsqVOJ)

接着查看我们解压出来的除`DEBIAN`目录外的其他目录文件,按照规则放在相应的目录下

`eog`包除`DEBIAN`目录外只有一个`usr`目录,我们查看下`usr`目录

```bash
deepin@deepin-PC:~/test/eog/a/usr$ ls
bin  lib  share
```

bin目录是可执行文件目录,lib是运行依赖库目录,按照规则放在`opt/apps/org.gnome.eog/files/`目录下

```bash
deepin@deepin-PC:~/test/eog/a/usr$ mv bin/ lib/ ../opt/apps/org.gnome.eog/files/
deepin@deepin-PC:~/test/eog/a/usr$ ls ../opt/apps/org.gnome.eog/files/
bin  lib

```

查看剩下的share目录

```bash
deepin@deepin-PC:~/test/eog/a/usr/share$ ls
applications  doc  eog  GConf  glib-2.0  help  icons  locale  man  metainfo
```

按照规则 将 `applications`, `GConf` ,`glib-2.0`,`icons`,`locale`目录放在相应的位置去,剩下的全部放在files目录下

```bash
deepin@deepin-PC:~/test/eog/a/usr/share$ mv applications/ ../../opt/apps/org.gnome.eog/entries/
deepin@deepin-PC:~/test/eog/a/usr/share$ mv GConf/ ../../opt/apps/org.gnome.eog/entries/
deepin@deepin-PC:~/test/eog/a/usr/share$ mv glib-2.0/ ../../opt/apps/org.gnome.eog/entries/
deepin@deepin-PC:~/test/eog/a/usr/share$ mv locale/ ../../opt/apps/org.gnome.eog/entries/
deepin@deepin-PC:~/test/eog/a/usr/share$ mv icons/ ../../opt/apps/org.gnome.eog/entries/
deepin@deepin-PC:~/test/eog/a/usr/share$ mv * ../../opt/apps/org.gnome.eog/files/

```

本包没有自启动文件和services文件,所以不做处理

把usr目录下的所有文件按照规则 放置后,保证usr目录为空后删除usr目录

```bash
deepin@deepin-PC:~/test/eog/a$ tree usr/
usr/
└── share

1 directory, 0 files
deepin@deepin-PC:~/test/eog/a$ rm -rf usr/
```

#### 第四步,编辑info文件

接下来填写info文件的内容,info文件的字段解释在商店打包规范里面有,具体这里不再解释,编辑好的文件如下

```bash
deepin@deepin-PC:~/test/eog/a/opt/apps/org.gnome.eog$ cat info 
{
    "appid": "org.gnome.eog",
    "name": "eog",
    "version": "3.28.4-3+b1",
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

```

#### 第五步 创建启动脚本用来 export `LD_LIBRARY_PATH`

某些包带有自己的运行依赖库,新规则之前都是安装在`/usr/lib/`目录下,在系统启动时可以找到需要的运行库

新规则不允许安装在`/usr/lib/`下,所以我们写一个启动脚本,先使用`LD_LIBRARY_PATH`环境变量来导出运行依赖库的路径,再去执行可执行文件

```bash
deepin@deepin-PC:~/test/eog/a/opt/apps/org.gnome.eog/files$ cd bin/
deepin@deepin-PC:~/test/eog/a/opt/apps/org.gnome.eog/files/bin$ ls
eog
deepin@deepin-PC:~/test/eog/a/opt/apps/org.gnome.eog/files/bin$ vim eog.sh
deepin@deepin-PC:~/test/eog/a/opt/apps/org.gnome.eog/files/bin$ vim eog.sh
deepin@deepin-PC:~/test/eog/a/opt/apps/org.gnome.eog/files/bin$ cat eog.sh 
export LD_LIBRARY_PATH='/opt/apps/org.gnome.eog/files/lib/x86_64-linux-gnu/eog'
/opt/apps/org.gnome.eog/files/bin/eog
deepin@deepin-PC:~/test/eog/a/opt/apps/org.gnome.eog/files/bin$ ll
总用量 20
-rwxr-xr-x 1 deepin deepin 14408 1月  11  2019 eog
-rw-r--r-- 1 deepin deepin   118 1月  17 14:17 eog.sh
deepin@deepin-PC:~/test/eog/a/opt/apps/org.gnome.eog/files/bin$ chmod a+x eog.sh
deepin@deepin-PC:~/test/eog/a/opt/apps/org.gnome.eog/files/bin$ ll
总用量 20
-rwxr-xr-x 1 deepin deepin 14408 1月  11  2019 eog
-rwxr-xr-x 1 deepin deepin   118 1月  17 14:17 eog.sh

```



#### 第六步,编辑desktop文件

将desktop文件重命名为包名.desktop格式

修改exec字段,exec是可执行文件的路径,因为我们改了包的安装位置,所以exec字段也要修改为正确的可执行文件的路径

```bash
deepin@deepin-PC:~/test/eog/a/opt/apps/org.gnome.eog/entries/applications$ mv eog.desktop org.gnome.eog.desktop 
deepin@deepin-PC:~/test/eog/a/opt/apps/org.gnome.eog/entries/applications$ vim org.gnome.eog.desktop 
deepin@deepin-PC:~/test/eog/a/opt/apps/org.gnome.eog/entries/applications$ cat org.gnome.eog.desktop |grep Exec
TryExec=/opt/apps/org.gnome.eog/files/bin/eog.sh
Exec=/opt/apps/org.gnome.eog/files/bin/eog.sh %U

```

#### 第七步,删除钩子脚本

按照新规则，所有的包不可以再使用 `post/pre inst/rm`  等钩子脚本，所以删除`DEBIAN`目录下的钩子脚本，（因本包并无钩子脚本，所以以下演示删除的脚本是我自己touch的）

```bash
deepin@deepin-PC:~/test/a$ cd DEBIAN/
deepin@deepin-PC:~/test/a/DEBIAN$ ls
control  md5sums  postinst  postrm  preinst  prerm
deepin@deepin-PC:~/test/a/DEBIAN$ rm postinst  postrm  preinst  prerm 
deepin@deepin-PC:~/test/a/DEBIAN$ ls
control  md5sums
deepin@deepin-PC:~/test/a/DEBIAN$ 
```



#### 第八步 , 压包

将按照新规则放置好的目录压入deb包内

```bash
deepin@deepin-PC:~/test/eog$ fakeroot dpkg-deb -b a org.gnome.eog_3.28.4-3+b1_amd64.deb
dpkg-deb: 正在 'org.gnome.eog_3.28.4-3+b1_amd64.deb' 中构建软件包 'org.gnome.eog'。
deepin@deepin-PC:~/test/eog$ ls
a  eog_3.28.4-2+b1_amd64.deb  org.gnome.eog_3.28.4-3+b1_amd64.deb

```

#### 最后一步,安装测试包是否能运行,在启动器是否有图标显示

```bash
deepin@deepin-PC:~/test/eog$ sudo dpkg -i org.gnome.eog_3.28.4-3+b1_amd64.deb
deepin@deepin-PC:~/test/eog$ sudo apt install -f
```

如果在终端安装，需要自己解决依赖关系，`apt install -f`是修复依赖关系的
