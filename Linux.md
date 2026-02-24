

一切皆文件

Linux的一切皆文件是指，Linux世界中的所有、任意、一切东西都可以通过文件的方式访问、管理。




```mermaid
graph LR
    Root("/ 根目录 （公司大楼）") --> Boot[ /boot（机房：内核与引导）]
    Root --> ETC[ /etc（行政部：配置文件） ]
    Root --> BIN[ /bin（公共文具：普通命令） ]
    Root --> SBIN[ /sbin （行政钥匙：管理命令） ]
    Root --> LIB[ /lib & /lib64 （耗材仓库：系统库） ]
    Root --> DEV[ /dev （监控屏：硬件设备） ]
    Root --> PROC[ /proc （实时监控表：内核/进程） ]
    Root --> SYS[ /sys （硬件拓扑图：总线信息） ]
    Root --> HOME[ /home （员工工位：普通用户） ]
    Root --> ROOT_HOME[ /root （董事长办公室：root用户） ]
    Root --> OPT[ /opt （专卖店：第三方软件） ]
    Root --> VAR[ /var （仓库/日志室：可变数据） ]
    Root --> TMP[ /tmp （公共垃圾桶：临时文件） ]
    Root --> MEDIA[ /media （USB接口：可移动设备） ]
    Root --> USR[ /usr （公共资源楼：共享数据） ]
    
    USR --> USR_BIN[ /usr/bin （更多公共文具） ]
    USR --> USR_LIB[ /usr/lib （更多仓库耗材） ]
    USR --> USR_LOCAL[ /usr/local （手工安装的软件） ]
```

`
