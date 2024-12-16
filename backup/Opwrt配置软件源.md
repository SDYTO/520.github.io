⚠️ 注意事项1
有些小伙伴反馈，代码执行成功，iStore也更新成功，但是依旧无法安装。那么你可以在b站视频下方留言并截图。

此前，OPKG 的软件源，如果你更改过。那么大概率是这里的问题.

系统——软件包——OPKG配置，你检查一下 /etc/opkg/distfeeds.conf 此文本框里的软件源是不是被你该动过。

还有就是第三方的自定义软件源也看看。位于 /etc/opkg/compatfeeds.conf 这文本框里。

如果你此前没有备份过这些信息，如果被自己改乱了。那么方法一是你重置iStoreOS系统。方法二是找到原本的软件源信息。

我这里给懒人一个选项吧。我将arm 和 x86 两种平台的iStoreOS 默认软件源 贴出来，你可以自己复制粘贴。

注意，这是最新版iStoreOS的软件源信息。你的iStoreOS 不能太老。最好是2024年的。因为通常情况下，iStoreOS 是一年一次大版本更新。

ARM64 平台
系统——软件包——OPKG配置

/etc/opkg/distfeeds.conf

src/gz openwrt_base https://mirrors.tuna.tsinghua.edu.cn/openwrt/releases/22.03.7/packages/aarch64_cortex-a53/base
src/gz openwrt_luci https://mirrors.tuna.tsinghua.edu.cn/openwrt/releases/22.03.7/packages/aarch64_cortex-a53/luci
src/gz openwrt_packages https://mirrors.tuna.tsinghua.edu.cn/openwrt/releases/22.03.7/packages/aarch64_cortex-a53/packages
src/gz openwrt_routing https://mirrors.tuna.tsinghua.edu.cn/openwrt/releases/22.03.7/packages/aarch64_cortex-a53/routing
src/gz openwrt_telephony https://mirrors.tuna.tsinghua.edu.cn/openwrt/releases/22.03.7/packages/aarch64_cortex-a53/telephony
X86-64 平台
系统——软件包——OPKG配置

/etc/opkg/distfeeds.conf

src/gz openwrt_core https://mirrors.cernet.edu.cn/openwrt/releases/22.03.7/targets/x86/64/packages
src/gz openwrt_base https://mirrors.cernet.edu.cn/openwrt/releases/22.03.7/packages/x86_64/base
src/gz openwrt_luci https://mirrors.cernet.edu.cn/openwrt/releases/22.03.7/packages/x86_64/luci
src/gz openwrt_packages https://mirrors.cernet.edu.cn/openwrt/releases/22.03.7/packages/x86_64/packages
src/gz openwrt_routing https://mirrors.cernet.edu.cn/openwrt/releases/22.03.7/packages/x86_64/routing
src/gz openwrt_telephony https://mirrors.cernet.edu.cn/openwrt/releases/22.03.7/packages/x86_64/telephony
⚠️注意事项2
有些小伙伴执行修复代码的时候 访问不到。那么可能是你的dns出错了。也可能是科学插件等代理造成的。你可以尝试先关闭科学插件。

然后在执行修复的代码。可以使用ping fw.koolcenter.com 来ping一下。看看能否ping通。总之 此域名是正常的。一定是网络原因造成的。如果就是访问不到，你还可以将sh文件下载到本地，在上传到软路由里。通过sh xxx.sh 来执行该修复脚本。

如果这些都觉得不方便的话，那就干脆直接复制粘贴脚本里的实际内容。我贴出来，你可以直接复制粘贴到软路由终端命令行。

该脚本的内容如下⬇️

sed -i 's/istore.linkease.com/istore.istoreos.com/g' /bin/is-opkg
sed -i 's/istore.linkease.com/istore.istoreos.com/g' /etc/opkg/compatfeeds.conf
sed -i 's/istore.linkease.com/istore.istoreos.com/g' /www/luci-static/istore/index.js
is-opkg update
如何恢复原状
sed -i 's/istore.istoreos.com/istore.linkease.com/g' /bin/is-opkg
sed -i 's/istore.istoreos.com/istore.linkease.com/g' /etc/opkg/compatfeeds.conf
sed -i 's/istore.istoreos.com/istore.linkease.com/g' /www/luci-static/istore/index.js
is-opkg update