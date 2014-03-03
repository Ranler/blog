title: Device Path For Linux
date: 2014-02-27 16:44:05
categories: Storage
---

Linux 设备路径的命名方式有：

- Standard Path
- udev: by-id by-path by-uuid
- MultiPath

### 1. standard path

Linux设备的标准路径就在`/dev/*`下面，包括常用的

- 磁盘，如`/dev/sda`
- 光驱，如`/dev/cdrom`

### 2. udev

udev是Linux内核的设备管理器，用于代替`devfs`来管理`/dev`目录下的设备文件，
和代替`hotplug`管理用户空间的插拔设备行为。
udev可能使用sysfs位于`/sys`下的信息。

udev可以配置规则用于设备的命名。
默认规则保存在`/lib/udev/rules.d/`下，
用户可以在`/etc/udev/rules.d/`目录下定义自己的规则。
规则修改后无需重启udev，即可适用于新添加的设备。
规则的编写可以参考[Writing udev rules](http://www.reactivated.net/writing_udev_rules.html)。

udev在`/dev/disk/`目录下为存储设备预设了的一些命名规则，
包括by-id, by-path, by-label, by-uui四种命名方式。
这些四种命名方式通过软链接映射到`/dev/sd*`下。

udev相关命令可以参考`udevadm`。


### 3. Multipath

Linux中同一个Volume的不同物理路径映射到了文件系统的不同设备路径上。
Multipath可以为这些路径提供统一的命名。
这样可以提供：

- 冗余性(Reducdancy)：如果一条物理路径不同，可以走另一条物理路径，完成容错/高可用性/鲁棒性；
- 性能：多物理路径可以进行IO负载平衡，提高性能；

Linux中Multipath模块叫做DM-Multipath，安装相应包之后，
最终会映射到`/dev/mapper/mpath*`位置。
Multipath路径也被udev管理，可以通过规则映射到其它目录下。

一个简单的例子如下，已存在两个设备路径`/dev/sdb`和`/dev/sdc`，
分别是FC链路和iSCSI链路连接同一磁盘。
安装mutlipath包之后，配置/etc/multipath.conf，
修改为以下字段，把不需要设为multipath的磁盘屏蔽，然后重启。

```
devnode_blacklist {
        devnode "^sda"
}
```

接下来设置MultiPath：

``` sh
# modprobe dm-multipath
# modprobe dm-round-robin
# multipath -v2
# service multipathd start
```

最终检查MultiPath的路径：

``` sh
# ls /dev/mapper/
control  mpath0  VolGroup00-LogVol00  VolGroup00-LogVol01

# multipath -l
mpath0 (3600000e00d1100000011158100500000) dm-2 FUJITSU,ETERNUS_DXL
[size=200M][features=1 queue_if_no_path][hwhandler=0][rw]
\_ round-robin 0 [prio=0][active]
 \_ 1:0:0:0 sdc 8:32  [active][undef]
\_ round-robin 0 [prio=0][enabled]
 \_ 0:0:1:0 sdb 8:16  [active][undef]
```

`/dev/mapper/mpath0`就是新的multipath路径。

