清除堆叠配置

方法一：
执行命令reset stack configuration清除堆叠相关配置。包括：交换机槽位号、堆叠优先级、堆叠保留VLAN、系统MAC切换时间、堆叠口配置、堆叠口速率配置。例如：
<HUAWEI> system-view [HUAWEI] reset stack configuration Warning: This operation will clear all stack configurations and may lead to the loss of the slot ID configuration and cause the device to reset immediately. Are you sure you want to continue? [Y/N]:y
执行该命令后会导致原有堆叠系统分裂，设备重启。
方法二：
不支持命令reset stack configuration，只能使用配置命令分别进行清除。清除配置需要重启设备才生效。
	• 恢复堆叠优先级：
<HUAWEI> system-view
[HUAWEI] stack slot 2 priority 100 //将2槽位号的优先级恢复为缺省值100
	• 恢复槽位号：
<HUAWEI> system-view
[HUAWEI] stack slot 2 renumber 0 //将设备槽位号修改为0
	• 恢复堆叠保留VLAN：
<HUAWEI> system-view
[HUAWEI] stack reserved-vlan 4093 //将堆叠保留VLAN恢复到缺省值4093
	• 恢复系统MAC切换时间：
<HUAWEI> system-view
[HUAWEI] undo stack timer mac-address switch-delay
堆叠配置
1.主
int xg0/0/3 shutdown int xg0/0/4 shutdown 先关闭配置接口
[HUAWEI] interface stack-port 0/1 进入堆叠接口组01
port interface xg0/0/3 enable 将接口加入堆叠接口组 已经加入的不可再次添加
[HUAWEI-stack-port0/1]po int XGigabitEthernet 0/0/3 enable Warning: Enabling stack function may cause configuration loss on the interface. Continue? [Y/N]:Y Info: This operation may take a few seconds. Please wait.... Nov 24 2023 16:23:18+08:00 HUAWEI %%01IFNET/4/IF_DISABLE(l)[17]:Interface XGigabitEthernet0/0/3 has been unavailable...
y 按照提示确认
[HUAWEI-stack-port0/1]int stack-port 0/2
[HUAWEI-stack-port0/2] [HUAWEI-stack-port0/2] [HUAWEI-stack-port0/2] [HUAWEI-stack-port0/2] [HUAWEI-stack-port0/2]po in [HUAWEI-stack-port0/2]po interface xg [HUAWEI-stack-port0/2]po interface XGigabitEthernet 0/0/4 EN [HUAWEI-stack-port0/2]po interface XGigabitEthernet 0/0/4 enable Warning: Enabling stack function may cause configuration loss on the interface. Continue? [Y/N]:Y Info: This operation may take a few seconds. Please wait.... Nov 24 2023 16:24:11+08:00 HUAWEI %%01IFNET/4/IF_DISABLE(l)[19]:Interface XGigabitEthernet0/0/4 has been unavailable.. [HUAWEI-stack-port0/2] Nov 24 2023 16:24:12+08:00 HUAWEI %%01IFNET/4/IF_ENABLE(l)[20]:Interface XGigabitEthernet0/0/4 has been available.
stack slot 0 priority 200设置主的优先级
save y 保存退出
按照要求启用对应主备接口
int xg0/0/3 undo shutdown int xg0/0/4 undo shutdown
插入堆叠线，重启交换机
2.备
[HUAWEI-stack-port0/1]int stack-port 0/1
[HUAWEI-stack-port0/2] [HUAWEI-stack-port0/2] [HUAWEI-stack-port0/2] [HUAWEI-stack-port0/2] [HUAWEI-stack-port0/2]po in [HUAWEI-stack-port0/2]po interface xg [HUAWEI-stack-port0/2]po interface XGigabitEthernet 0/0/3 EN [HUAWEI-stack-port0/2]po interface XGigabitEthernet 0/0/3 enable Warning: Enabling stack function may cause configuration loss on the interface. Continue? [Y/N]:Y Info: This operation may take a few seconds. Please wait.... Nov 24 2023 16:24:11+08:00 HUAWEI %%01IFNET/3/IF_DISABLE(l)[19]:Interface XGigabitEthernet0/0/4 has been unavailable.. [HUAWEI-stack-port0/2] Nov 24 2023 16:24:12+08:00 HUAWEI %%01IFNET/3/IF_ENABLE(l)[20]:Interface XGigabitEthernet0/0/3 has been available.
[HUAWEI-stack-port0/1]int stack-port 0/2
[HUAWEI-stack-port0/2] [HUAWEI-stack-port0/2] [HUAWEI-stack-port0/2] [HUAWEI-stack-port0/2] [HUAWEI-stack-port0/2]po in [HUAWEI-stack-port0/2]po interface xg [HUAWEI-stack-port0/2]po interface XGigabitEthernet 0/0/4 EN [HUAWEI-stack-port0/2]po interface XGigabitEthernet 0/0/4 enable Warning: Enabling stack function may cause configuration loss on the interface. Continue? [Y/N]:Y Info: This operation may take a few seconds. Please wait.... Nov 24 2023 16:24:11+08:00 HUAWEI %%01IFNET/4/IF_DISABLE(l)[19]:Interface XGigabitEthernet0/0/4 has been unavailable.. [HUAWEI-stack-port0/2] Nov 24 2023 16:24:12+08:00 HUAWEI %%01IFNET/4/IF_ENABLE(l)[20]:Interface XGigabitEthernet0/0/4 has been available.
stack slot 0 renumber 1 设置备机的优先级
save y 保存退出
按照要求启用对应主备接口
int xg0/0/3 undo shutdown int xg0/0/4 undo shutdown
插入堆叠线，重启交换机
查看重启后的状态 dis stack 如图所示正常

