部署命令如下

docker run -d \
--name=typecho-blog \
--restart always \
--mount type=tmpfs,destination=/tmp \
-v /volume1/docker/typecho:/data \
-e PHP_TZ=Asia/Shanghai \
-e PHP_MAX_EXECUTION_TIME=600 \
-p 90:80 \
80x86/typecho:latest
