目前尝试过安装在arm64和x64的Ubuntu上，Openwrt上，Pve母机的Debian上，openwrt直接跑会出现内存溢出的情况，本文最后说说如何从根本上解决，顺便鄙视一下某个每篇文章都要1.99元的网站
虽然官方有安装脚本，但等于没有一样，并用不了，不要尝试
docker run -d --name hassio_supervisor --privileged --restart always -v /run/docker.sock:/run/docker.sock -v /run/dbus:/run/dbus -v /run/udev:/run/udev -v /root/docker/hassio:/data -e SUPERVISOR_SHARE=/root/docker/hassio -e SUPERVISOR_NAME=hassio_supervisor -e HOMEASSISTANT_REPOSITORY=homeassistant/qemux86-64-homeassistant [ghcr.io/home-assistant/amd64-hassio-supervisor:2023.10.0]( http://ghcr.io/home-assistant/amd64-hassio-supervisor:2023.10.0)
下载
删掉冒号后面的去访问就能知道最新stable版本是多少不下载也行，启动的时候会下载
arm64
docker pull ghcr.io/home-assistant/aarch64-hassio-supervisor:2023.08.1
docker pull ghcr.io/home-assistant/aarch64-homeassistant:2023.8.1
x64
docker pull ghcr.io/home-assistant/amd64-hassio-supervisor:2023.08.1
docker pull ghcr.io/home-assistant/qemux86-64-homeassistant:2023.8.1
启动
不要看官方文档安装他所需要的依赖，因为那些依赖确实没有也行，并且如果在Pve下安装networkmanager会导致网络崩坏
此处 /opt/apps/hassio 是Home Assistant的所有数据，可以改到你想要的地方，迁移的时候也只需要迁移这个文件夹，并卸载重新安装所有插件（插件即容器，数据不会丢）不要问为什么不加--privileged问就是启不来
arm64
docker run -d --name hassio_supervisor --privileged \
--restart unless-stopped \
-v /var/run/docker.sock:/var/run/docker.sock \
-v /var/run/dbus:/var/run/dbus \
-v /opt/apps/hassio:/data \
-e SUPERVISOR_SHARE=/opt/apps/hassio \
-e SUPERVISOR_NAME=hassio_supervisor \
-e HOMEASSISTANT_REPOSITORY=homeassistant/aarch64-homeassistant \
ghcr.io/home-assistant/aarch64-hassio-supervisor:2023.08.1
x64
docker run -d --name hassio_supervisor --privileged \
--restart always \
-v /var/run/docker.sock:/var/run/docker.sock \
-v /var/run/dbus:/var/run/dbus \
-v /opt/apps/hassio:/data \
-e SUPERVISOR_SHARE=/opt/apps/hassio \
-e SUPERVISOR_NAME=hassio_supervisor \
-e HOMEASSISTANT_REPOSITORY=homeassistant/qemux86-64-homeassistant \
ghcr.io/home-assistant/amd64-hassio-supervisor:2023.08.1
然后需要耐心等待，按网速和性能需要蛮长时间的，用这个命令来看安装情况或者直接去portainer看log
docker logs -f hassio_supervisor
另外有人说装完之后找不到Supervisor的菜单，新版的菜单在设置里而不是左边菜单栏上，如果实在找不到，访问路径/hassio/dashboard
更新
发布了新版本，怎么可能不更新呢
这里的容器镜像名记住了替换下面的
docker stop hassio_supervisor
docker stop homeassistant
docker rm hassio_supervisor
docker rm homeassistant
docker images | grep supervisor
上面输出的都挨个删掉
docker rmi ghcr.io/home-assistant/amd64-hassio-supervisor:2022.04.0
docker images | grep 4-homeassistant
上面输出的都挨个删掉
docker rmi ghcr.io/home-assistant/qemux86-64-homeassistant:2022.3.6
这里下载最新版 删掉冒号后面的去访问就能知道最新版本是多少，替换掉
docker pull ghcr.io/home-assistant/amd64-hassio-supervisor:2023.08.1
docker pull ghcr.io/home-assistant/qemux86-64-homeassistant:2022.11.2
然后用上面安装的方式去运行，最后一行用上面pull的代替，SUPERVISOR_SHARE必须是同一个地方，迁移也只需要迁移他
万一 homeassistant 没有启动，那就执行下面这两条再执行上面安装，同时检查下下来的两个镜像是不是最新的稳定版本（没有dev字样）
docker stop hassio_cli hassio_multicast hassio_audio hassio_dns hassio_observer hassio_supervisor homeassistant
docker rm hassio_cli hassio_multicast hassio_audio hassio_dns hassio_observer hassio_supervisor homeassistant
错误和解决方案
	1. 'AddonManager.install' blocked from execution, system is not healthy
hassio文件夹下新建jobs.json并在里面写入
{"ignore_conditions": ["healthy"]}
	2. 在Openwrt下，hassio_audio会以肉眼可见的速度拼命吃内存，原因是因为Openwrt没有/dev/fd
解决方案1（推荐）
ln -s /proc/self/fd /dev/fd
并且写到启动项中（因为是tmpfs所以需要每次开机都建一次软连接）
解决方案2（虽然说这东西没啥用，但hassio_supervisor有守护进程，会自动重启它，起起停停浪费资源）
在计划任务写入
						◊ docker kill hassio_audio
	3. path /opt/apps/hassio/share is mounted on /opt but it is not a shared or slave mount
新版hass对所在分区挂载有要求，需要增加一条开机启动（尚不清楚怎么写fstab）
mount --make-shared /opt
附加组件
既然都用到hassio了，那肯定会用到mqtt吧，那就启动一个emqx吧
docker run -d --name emqx -v /etc/localtime:/etc/localtime -p 1883:1883 -p 18083:18083 emqx/emqx
然后发现启动报错，因为Openwrt没有/etc/localtime和/etc/timezone
opkg install zoneinfo-asia
/etc/init.d/system reload
docker run -d --name hassio_supervisor --privileged 
-v /var/run/docker.sock:/var/run/docker.sock 
-v /var/run/dbus:/var/run/dbus 
-v /opt/hassio:/data 
-e SUPERVISOR_SHARE="/opt/hassio" 
-e SUPERVISOR_NAME=hassio_supervisor 
-e HOMEASSISTANT_REPOSITORY="homeassistant/qemux86-64-homeassistant" 
--restart=always homeassistant/amd64-hassio-supervisor
docker run -d --name hassio_supervisor --privileged --restart always -v /run/docker.sock:/run/docker.sock -v /run/dbus:/run/dbus -v /run/udev:/run/udev -v /root/docker/hassio:/data -e SUPERVISOR_SHARE=/root/docker/hassio -e SUPERVISOR_NAME=hassio_supervisor -e HOMEASSISTANT_REPOSITORY=homeassistant/qemux86-64-homeassistant [ghcr.io/home-assistant/amd64-hassio-supervisor:2023.10.0]( http://ghcr.io/home-assistant/amd64-hassio-supervisor:2023.10.0)
docker run -d --name hassio_supervisor --privileged -v /var/run/docker.sock:/var/run/docker.sock -v /var/run/dbus:/var/run/dbus -v /opt/docker/hassio:/data -e SUPERVISOR_SHARE="/opt/docker/hassio" -e SUPERVISOR_NAME=hassio_supervisor -e HOMEASSISTANT_REPOSITORY="homeassistant/qemux86-64-homeassistant" --restart unless-stopped homeassistant/amd64-hassio-supervisor
停止 hassio-supervisor 系统服务 sudo systemctl stop hassio-supervisor && systemctl disable hassio-supervisor # 停止 homeassistant 的 docker 并且删除 docker stop homeassistant&&docker rm homeassistant # 停止 hassio_supervisor 的 docker 并且删除 docker stop hassio_supervisor docker rm hassio_supervisor # 删除相关目录 rm /etc/hassio.json rm /etc/systemd/system/hassio-* rm -r /usr/share/hassio # 删除相关 docker 镜像 docker ps |grep "homeassistant/" |awk '{print $1}' | xargs docker rm -f docker images |grep "homeassistant/" | awk '{print $3}'| xargs docker rmi
