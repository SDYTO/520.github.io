
客户端与服务端下载地址：https://github.com/ehang-io/nps/releases
github镜像 下载地址 https://githubfast.com/ehang-io/nps/releases/ 
1.下载解压文件
mkdir nps && cd nps
# 下载服务端
wget https://github.com/ehang-io/nps/releases/download/v0.26.10/linux_amd64_server.tar.gz
# 解压
tar -zxvf linux_amd64_server.tar.gz
2.常用配置
● • http_proxy_port：域名代理http代理监听端口。
● • https_proxy_port：域名代理https代理监听端口。
● • https_just_proxy：为false时仅代理https转发，为true则可以配置站点的ssl证书。
● • web_port：web管理端口。
● • web_username：web界面管理账号。
● • web_password：web界面管理密码。
3. 安装服务端【在解压目录下执行】
# 进行安装
./nps install
# 启动nps
nps start
安装完成后，nps的配置文件会存放在/etc/nps目录下。
4.
编辑配置文件

使用记事本或者你任意喜欢的文本编辑器编辑conf/nps.conf文件，配置文件的内容与Linux中部署是一样的道理。
访问web控制台
通过访问http://127.0.0.1:8080来进入到控制台登录界面，使用配置的web_username和web_password来进行登录。
创建客户端
一台需要映射的机器视作一个客户端，我们需要先在控台创建好客户端，才能对需要穿透的主机进行穿透。我们点击左侧的客户端进入到客户端管理界面。点击新增客户端添加一个需要映射客户端的基本信息：

 
填写备注基本就可以了，填写完成点击新增按钮即可添加成功，新增完会自动回到客户端列表，此时刷新即可看到新增的客户端：

1. 1. ID：id是用来区分主机的，胚子的域名绑定和端口映射都是跟id挂钩的。
2. 2. 备注：用于方便区分不同的机器。
3. 3. 版本：内网机器安装的客户端版本。
4. 4. 唯一验证密钥：内网机器安装的客户端通过这个密码可以找到与平台挂钩的映射配置，唯一且只能被一个客户端引用。
配置端口映射
在每个客户端后面也就是右侧中有两个按钮，在隧道中可以配置端口映射规则，进入到隧道我们新增一个隧道：

依次来看：
1. 1. 模式：映射的端口走的什么协议，比如（TCP，UDP）等等。一般映射网站也可以使用TCP协议。
2. 2. 客户端ID：这个映射规则被哪个客户端所使用，一般情况下也可以直接修改现有规则的客户端ID将规则移动到指定客户端下。
3. 3. 备注：给自己看的，用于分别映射用途。
4. 4. 服务端端口：即搭建NPS服务器的自身端口，不可重复使用同一个端口。
5. 5. 目标(IP:端口)：指内网客户端需要映射的端口，如（127.0.0.1:3389），也可以是内网其他机器如（192.168.1.100:3389）。
点击下方新增按钮即可立即生效。
配置域名访问
如果内网存在网站这种服务，想要使用域名访问内网的网站则可以配置主机映射，同样的在客户单列表中选择指定的客户端右侧的主机按钮进入到域名配置页面，点击新增按钮：
续来看每一个作用重复的内容不再赘述：
1. 1. 主机：即需要是什么域名来访问咱们的服务，填写域名即可。
2. 2. 模式：这里分（http、https）。
3. 3. URL路由：则是区分不同的路由访问不同的服务类似nginx代理（http://a.com/a、http://a.com/b）。
如果经常使用域名访问服务，还是比较推荐使用域名泛解析*来解析道NPS服务器的IP上。