# 基于Openwrt的校园网单线多拨方案

> ***作者：kikyo   Email：sjh10075@outlook.com***  QQ:2712403268

  笔者所在CQUPT校园网每台设备限速50Mbps，每个账号限制两台设备同时在线，但是在2024年这个网速还是十分影响体验的，同时由于我有三台设备（手机、电脑、平板），需要同时上网也不希望频繁切换登录设备，遂购入一台红米AC2100，刷入openwrt固件，开启单线多拨，多线程网速提升至100Mbps，基本满足上网需求。

## 教程

需要准备的东西：

> [!TIP]
>
>  > 1. 一台可以刷Openwrt的路由器(推荐红米AC2100)
>  >
>  > 2. 一根网线（最好是CAT5E及以上的）
>  >
>  > 3. 一台电脑（带RJ45网口）（亦可通过C口转接）
>  >
>  > 4. Xterminal 软件
>  >
>  > 5. 一双灵巧的手和聪慧的大脑
>
> **PS：本教程用到的软件、固件包等会在文章末分享**
>
> ***本教程针对红米AC2100，其他路由器请自行查阅资料刷入breed，成功刷入breed后的步骤相同.***

------

## 1. 刷入breed

> Breed【不死】，是一个由国内个人hackpascal开发的Bootloader引导程序，成功刷入breed后，就可以借助它刷入第三方路由器固件
>
> <u>**刷入breed的方法较多，本文介绍两个相对简单的方法，其他方法请自行查阅相关教程**</u>

------

> (1). 直接刷入法
>
> a. 固件降级
>
> ```txt
> 下载完成后进入后台 192.168.31.1->常用设置->系统状态->手动升级
> 加载固件，可以保留数据->开始升级
> ```
>
> b. 直接刷入breed
>
> > [!CAUTION]
> >
> >   首先需要确保路由器有网络，有网络才能自动下载breed。网络可以连接网线，并使用手机连接路由器WiFi并登录，如果在学校没有网线的话，
> > 可以手机开热点，然后打开路由器的无线桥连（中继）功能，让路由器连接到自己的手机热点，就可以使路由器有网络。
>
> ```html
> 电脑浏览器（推荐）进入 192.168.31.1 ，复制自己的stok，即网页地址栏出现的 192.168.31.1/……/stok=stok值（这部分）
> 用复制的stok替换下列代码中的stok值
> http://192.168.31.1/cgi-bin/luci/;stok=stok值/api/misystem/set_config_iotdev?bssid=Xiaomi&user_id=longdike&ssid=%0A%5B%20-z%20%22%24(dmesg%20%7C%20grep%20ESMT)%22%20%5D%20%26%26%20B%3D%22Toshiba%22%20%7C%7C%20B%3D%22ESMT%22%0Auci%20set%20wireless.%24(uci%20show%20wireless%20%7C%20awk%20F%20'.'%20'%2Fwl1%2F%20%7Bprint%20%242%7D').ssid%3D%22%24B%20%24(dmesg%20%7C%20awk%20'%2FBad%2F%20%7Bprint%20%245%7D')%22%0A%2Fetc%2Finit.d%2Fnetwork%20restart%0A
> 回车，浏览器会显示 {"CODE:0"}    如果显示其他代码，可能是你还没降级固件或者stok 过期，也可以恢复出厂设置尝试
> 此代码是用来检查NAND坏块，运行代码后，你路由器的2.4g WiFi名称会改名成：比如 “ESMT”，“Toshiba”，“Toshiba 90 768”。 90和768是坏块。 如果ESMT或者Toshiba后面没数字，代表没有坏块。坏块基本不会影响下面的操作，但是存在部分设备异常的情况，如存在坏块且无法刷入breed，考虑更换路由器尝试。
> http://192.168.31.1/cgi-bin/luci/;stok=stok值/api/misystem/set_config_iotdev?bssid=Xiaomi&user_id=longdike&ssid=%0Acd%20%2Ftmp%0Acurl%20-o%20B%20-O%20https%3A%2F%2Fbreed.hackpascal.net%2Fbreed-mt7621-xiaomi-r3g.bin%20-k%0A%5B%20-z%20%22%24(sha256sum%20B%20%7C%20grep%20242d42eb5f5aaa67ddc9c1baf1acdf58d289e3f792adfdd77b589b9dc71eff85)%22%20%5D%20%7C%7C%20mtd%20-r%20write%20B%20Bootloader%0A ```
> 浏览器地址栏输入上面的代码（记得替换stok），回车
> 不出意外的话，你的路由器会在60秒内重启，system指示灯变为黄色，最终变蓝成功进入系统，代表刷入breed成功
> 接下来，将路由器断电，按住reset，接通电源，等待10秒，路由器蓝色灯光闪烁，代表进入breed，用网线将路由器和电脑连接，浏览器地址栏输入192.168.1.1进入breed控制台
> ```

------

> (2)ssh刷入
>
> a.固件降级
>
> ```txt
> 下载完成后进入后台 192.168.31.1->常用设置->系统状态->手动升级
> 加载固件，可以保留数据->开始升级
> ```
>
> b.获取stok
>
> ```
> 电脑浏览器（推荐）进入 192.168.31.1 ，复制自己的stok，即网页地址栏出现的 192.168.31.1/……/stok=stok值（这部分）
> http://192.168.31.1/cgi-bin/luci/;stok=<STOK值>/api/misystem/set_config_iotdev?bssid=Xiaomi&user_id=longdike&ssid=-h%3B%20nvram%20set%20ssh_en%3D1%3B%20nvram%20commit%3B%20sed%20-i%20's%2Fchannel%3D.*%2Fchannel%3D%5C%22debug%5C%22%2Fg'%20%2Fetc%2Finit.d%2Fdropbear%3B%20%2Fetc%2Finit.d%2Fdropbear%20start%3B
> 浏览器输入上面的代码（记得替换stok），回车，浏览器会显示{"CODE:0"},代表成功
> 如果返回其他代码，请检查你的stok和网址是否正确
> 等待60秒路由器重启，已开启ssh权限
> ```
>
> c. ssh连接刷入breed
>
> ```bash
> 找到路由器背面的SN码
> 访问https://miwifi.dev/ssh
> 输入SN码可获取root密码
> 打开Xterminal，新建服务器，22端口，账户为root，密码为刚刚获取的密码
> 将breed-mt7621-xiaomi-r3g.bin文件上传到根目录下的tmp文件夹（注意是根目录下的，不是tmp文件夹下的tmp文件夹）
> 打开终端执行 mtd -r write /tmp/breed-mt7621-xiaomi-r3g.bin Bootloader
> 等待5分钟，成功刷入breed
> 接下来，将路由器断电，按住reset，接通电源，等待10秒，路由器蓝色灯光闪烁，代表进入breed，用网线将路由器和电脑连接，浏览器地址栏输入192.168.1.1进入breed控制台
> ```

------

## 2. 刷入openwrt

> ```bash
> 进入breed控制台后，在环境变量中添加xiaomi.r3g.bootfw 值为2.
> 文件里有两个固件包，带DE字样的是防校园网检测设备共享固件，如你的校园网不检测网络共享，可以刷入不带DE的固件
> 点击更新固件，内存布局选择R3G_openwrt，上传kenel 和rootfs文件，勾选自动重启，确认等待自动更新，并开机，第一次开机所需的时间较长，请耐心等待
> 开机后WiFi会出现openwrt_5G和Openwrt_2G两个wifi,将路由器连接至网络（可以有线接入，手机连接路由器WiFi会弹出登录页面）,电脑连接路由器
> 打开Xterminal，打开终端，输入 ssh root@192.168.1.1
> 提示输入密码 默认密码为 password （输入密码过程中看不到密码，请输入后按回车即可）
> 进入控制台
> ```
>
> > [!WARNING]
> >
> > 警告：DE版本的固件无法自动开启WiFi，第一次刷入固件开机后，有线连接电脑，进入`192.168.1.1`，路由器后台，密码为 `password`
> >
> > 在 系统->启动项->本地启动脚本，在`exit0`前加入如下语句 `/sbin/mtkwifi up` 保存应用重启路由器即可
>
> 

## 3. 单线多拨

------

> 1. 添加虚拟网卡
>
> ```bash
> opkg update
> opkg install kmod-macvlan
> ```
>
> 2. 添加启用虚拟网卡
>
>   ```bash
>   ip link add link eth1 name veth0 type macvlan # 添加一个类型为macvlan，名字为veth0的虚拟网卡，并通过虚拟链路和eth1连接起来
>   ifconfig veth0 up # 启用创建的veth0网卡
>   ```
>
>   注意`eth0`是接口中物理WAN卡的名称，如果提示找你不到接口，请检查名称
>
>   以上方法创建的虚拟网卡会在重启后失效，如需永久设置，参考下面的方法
>
>   ```bash
>   在/etc/config/network中
>    添加以下代码
>   config device 'veth0'
>       option name 'veth0'
>       option ifname 'eth1'
>       option type 'macvlan'
>   ```
>
>   输入 `ipconfig`检查列表中是否有添加的虚拟网卡
>
> 3. 虚拟网卡配置
>
>    	进入Openwrt后台，点击网络→接口，暂时将WAN口禁用，避免虚拟网卡获取不到ip地址，添加新接口，设备选择虚拟网卡veth0，然后创建接口。名称建议设置为 vwan1，设备为虚拟网卡veth0，协议为DHCP服务器，网关跃点随便设置。但是不要和其他WAN口重复，防火墙选择wan，保存即可。
>    					
>    	此时将校园网后台设备下线，Vwan1连接到校园网，成功获取ip后，手机连接路由器WiFi登校园网，检查网络是否正常。
>    					
>    	再创建一个接口，按同样的方法再创建一个虚拟网卡vwan2。这时vwan2还未连上校园网，只是自动获取了IP地址
>
> 4. 负载均衡	
>
>    终端输入`opkg install mwan3 luci-app-mwan3`	
>
>    ```
>    安装完后到网络→负载均衡界面，把接口、成员、策略、规则里面的配置全部删掉
>    在接口里面新增vwan1，名字要和在网络→接口添加的接
