1.在线编译地址https://firmware-selector.openwrt.org/?version=23.05.5&target=x86%2F64&id=generic
![image](https://github.com/user-attachments/assets/6ca5a382-aacf-4fe5-be50-4bca55017e62)
注意上方🔝截图，有一个展开的按钮。点击之后，就能自定义参数了。如下你可以 在预装软件中增加自己需要的app
luci-i18n-firewall-zh-cn luci-i18n-filebrowser-zh-cn luci-app-argon-config luci-i18n-argon-config-zh-cn luci-i18n-opkg-zh-cn luci-i18n-ttyd-zh-cn luci-i18n-passwall-zh-cn luci-app-openclash luci-i18n-homeproxy-zh-cn openssh-sftp-server
上面⬆️的软件可以根据需求 自行添加，如果不知道软件名称 可以在仓库搜索和查询，点击下方直达仓库搜索
https://mirrors.vsean.net/openwrt/releases/23.05.4/packages/x86_64/luci/?sort=size&order=desc
备注：如果你是编译luci 24的版本，上述代码中 要将luci-i18n-opkg-zh-cn替换为luci-i18n-package-manager-zh-cn 因为新版本里它的名称发生变化。
首次启动时运行的脚本（uci-defaults）增加
# 设置默认wan口防火墙打开 方便虚拟机用户首次访问webui
uci set firewall.@zone[1].input='ACCEPT'
uci commit firewall
# 设置主机名映射 解决安卓原生TV首次连不上网的问题
uci add dhcp domain
uci set "dhcp.@domain[-1].name=time.android.com"
uci set "dhcp.@domain[-1].ip=203.107.6.88"
uci commit dhcp
# 根据网卡数量配置网络
count=0
for iface in $(ls /sys/class/net | grep -v lo); do
  # 检查是否有对应的设备，并且排除无线网卡
  if [ -e /sys/class/net/$iface/device ] && [[ $iface == eth* || $iface == en* ]]; then
    count=$((count + 1))
  fi
done
if [ "$count" -eq 1 ]; then
    # 单个网卡，设置为 DHCP 模式
    uci set network.lan.proto='dhcp'
    uci commit network
elif [ "$count" -gt 1 ]; then
    # 多个网卡，保持静态 IP
    uci set network.lan.ipaddr='192.168.100.1'
    uci commit network
fi
3.都填写完成后，点击 【请求构建】就开始在线编译了。编译完成后下载即可

4.借助iStoreOS环境给我们下载的固件img扩容（其他linux也行）
我推荐使用iStoreOS系统来处理这个img镜像。因为其他OpenWrt可能没有大的空间来处理。

如果你想使用其他openwrt来处理，并且保证你有大于2GB以上的空间，也不是不行，需要额外安装2个组件
opkg update
opkg install parted
opkg install openssh-sftp-server
如果想增加2GB空间 则可用如下方法设置，遇到提示就按照要求输入Fix 和 OK

# 解压缩镜像文件
gzip -kd immortalwrt.img.gz
# 给img镜像末尾填充2GB空数据 (count=2000 表示增加 2GB 的空间)
dd if=/dev/zero bs=1M count=2000 >> immortalwrt.img



# 为了将2GB空间真正赋予具体的分区，使用分区工具parted操作镜像。
parted immortalwrt.img


# 查看分区情况
print
# 调整分区大小 (将第 2 个分区扩展至镜像文件的 100%)
resizepart 2 100%
# 查看是否扩展成功
print
# 退出分区工具
quit
更换软件源（可选 非必需）
sed -e 's,https://downloads.immortalwrt.org,https://mirrors.cernet.edu.cn/immortalwrt,g' \
    -e 's,https://mirrors.vsean.net/openwrt,https://mirrors.cernet.edu.cn/immortalwrt,g' \
    -i.bak /etc/opkg/distfeeds.conf
运行时扩容（luci-app-partexp）
sirpdboy 的项目源码⬇️⬇️⬇️

https://github.com/sirpdboy/luci-app-partexp

利用sirpdboy 的项目源码+kiddin9编译的ipk，我们可以一键安装分区扩容app
https://dl.openwrt.ai/23.05/packages/x86_64/kiddin9/
一键安装 分区扩容 app

# 一键安装 sirpdboy分区扩容 app
wget -O install.sh https://cafe.vip.cpolar.top/wkdaily/OneKeyExpand/raw/branch/main/install.sh && chmod +x install.sh && ./install.sh

# 如果上述代码访问不到。可直接复制下面的内容。
opkg update
wget https://cafe.vip.cpolar.top/wkdaily/OneKeyExpand/raw/branch/main/luci-app-partexp_all.ipk
opkg install luci-app-partexp_all.ipk 2>/dev/null
echo "安装成功"
![image](https://github.com/user-attachments/assets/ceff161b-3d38-4ca9-b463-7da0a3a0e90f)
一键备份OpenWrt（仅 squashfs 格式的镜像文件有效）
项目地址：https://github.com/wukongdaily/OpenBackRestore/
网友灵魂拷问,为什么不把docker也加进去编译
回答：immortalwrt在云端imagebuilder 并没有开放ROOTFS_PARTSIZE 这个定义软件包分区大小的参数。所以我们就不能自己定义大小。而docker 并非一个小组件，docker 及其依赖，全都安装完毕，可能需要1GB甚至更多。所以我们不能将docker加进去。其他组件都比较小。大家可以酌情根据自己的需求来添加。至于说docker，我们完全可以在扩容之后再安装。各位这次懂了吧～

如果各位的需求就是如此，那么你就可以考虑使用kwrt了。因为它可以定义软件包大小。而且默认就1GB 也不小了。它的网址如下

https://openwrt.ai/?target=x86%2F64&id=generic

根据我视频里提到的关于升级的问题。升级后是新固件的大小。而immortalwrt仓库提供了一个【值守式更新】的功能。如果用户可以自由调整软件包的大小，又能够任意升级。那么就会面临着从“大固件”升级到“小固件”的风险。这样就尴尬了，相当于数据丢失、或者干脆是升级失败的。所以我觉得他没有开放这个参数，是有这个原因的。另外这是一个很多人在维护的项目。若期待调整默认大小是1GB软件包，估计也需要其他开发者同意才行。也就是说这不是一个人说了算的，是一个团队。
常见编译错误
可能服务器为了避免重复存储，做了一些限制吧。如果都用同样的参数，可能产生的固件是相同的。因此可以稍微增减参数。这样才能触发编译。如果网盘有的，建议就别重复编译了。咱们也稍微节省点人家服务器的资源。
![image](https://github.com/user-attachments/assets/2ded61aa-075a-4ad2-b4fc-901ad7e2d36f)

如何安装docker
确定软件包是有足够的空间的，如果你安装的是我扩容之后的固件，自然没问题。docker和相关组件可能占用1g

luci-i18n-dockerman-zh-cn

上述名称是docker的名称。我们可以在软件包那里先更新列表，然后搜索该组件，并安装即可。比较大，需要好几分钟。

另外一个方法是采用命令行安装。
opkg update 
opkg install luci-i18n-dockerman-zh-cn
安装成功后，会出现在左侧菜单栏。首次使用建议重启一次路由器。


