
命令基本格式： command  +[ 选项 ]  +[参数（ 操作对象的路径 ）]   例如：ls -a  /etc

说明：
1、极个别不遵循。
2、当有多个选项时，可以写在一起，例如：ls -alt，就是-a 、-l、-t组合在一起的|
3、有简化选项和完整选项，简化选项用 "-a"表示，完整选项用"--all"表示，如：du --max -depth=1 -ah
# 一. 文件管理系统

一切皆文件

Linux的一切皆文件是指，Linux世界中的所有、任意、一切东西都可以通过文件的方式访问、管理。
Linux 基金会发布的 FHS 标准: FHS（Filesystem Hierarchy Standard），文件系统层次化标准，该标准规定了 Linux 系统中所有一级目录以及部分二级目录（/usr 和 /var）的用途。



```mermaid
graph LR
    Root("/ 根目录 （公司大楼）") --> Boot[ /boot（机房：内核与引导）]
    Root --> ETC[ /etc（配置文件） ]
    Root --> BIN[ /bin（普通命令） ]
    Root --> SBIN[ /sbin （管理命令） ]
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
```mermaid

flowchart LR
    subgraph A [第一类：核心启动区]
        direction LR
        boot[“/boot<br>内核 & 引导”]
        efi[“/efi<br>UEFI启动”]
    end
    
    subgraph B [第二类：管理员专用]
        direction LR
        etc[“/etc<br>配置文件”]
        sbin[“/sbin<br>管理命令”]
        lib[“/lib<br>系统库”]
    end
    
    subgraph C [第三类：硬件/内核交互]
        direction LR
        dev[“/dev<br>设备文件”]
        proc[“/proc<br>进程信息”]
        sys[“/sys<br>硬件状态”]
    end
    
    subgraph D [第四类：用户数据]
        direction LR
        home[“/home<br>普通用户家”]
        root[“/root<br>管理员家”]
        opt[“/opt<br>第三方软件”]
    end
    
    subgraph E [第五类：动态/临时]
        direction LR
        var[“/var<br>日志/缓存”]
        tmp[“/tmp<br>临时文件”]
        run[“/run<br>运行时数据”]
    end
    
    subgraph F [第六类：挂载点]
        direction LR
        media[“/media<br>自动挂载”]
        mnt[“/mnt<br>手动挂载”]
    end
    
    User(("用户<br>（你是谁？）")) --> A
    User --> B
    User --> C
    User --> D
    User --> E
    User --> F
```