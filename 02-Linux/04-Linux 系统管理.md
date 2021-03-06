## 系统管理命令

| 命令       | 说明                        |
|:---------|:--------------------------|
| stat     | 显示指定文件的相关信息,比 ls 命令显示内容更多 |
| who      | 显示在线登录用户                  |
| hostname | 显示主机名称                    |
| uname    | 显示系统信息                    |
| top      | 显示当前系统中耗费资源最多的进程          |
| ps       | 显示瞬间的进程状态                 |
| du       | 显示指定的文件（目录）已使用的磁盘空间的总量    |
| df       | 显示文件系统磁盘空间的使用情况           |
| free     | 显示当前内存和交换空间的使用情况          |
| ifconfig | 显示网络接口信息                  |
| ping     | 测试网络的连通性                  |
| netstat  | 显示网络状态信息                  |
| clear    | 清屏                        |
| kill     | 杀死一个进程                    |

## 开关机命令

`shutdown` 命令可以用来进行关机程序，并且在关机以前传送讯息给所有使用者正在执行的程序，`shutdown` 也可以用来重开机。

| 命令       | 语法                                              | 参数         | 参数说明                             |
|:---------|:------------------------------------------------|:-----------|:---------------------------------|
| shutdown | shutdown [-t seconds] [-rkhncfF] time [message] |            |                                  |
|          |                                                 | -t seconds | 设定在几秒钟之后进行关机程序                   |
|          |                                                 | -k         | 并不会真的关机，只是将警告讯息传送给所有只用者          |
|          |                                                 | -r         | 关机后重新开机（重启）                      |
|          |                                                 | -h         | 关机后停机                            |
|          |                                                 | -n         | 不采用正常程序来关机，用强迫的方式杀掉所有执行中的程序后自行关机 |
|          |                                                 | -c         | 取消目前已经进行中的关机动作                   |
|          |                                                 | -f         | 关机时，不做 fcsk 动作(检查 Linux 档系统)     |
|          |                                                 | -F         | 关机时，强迫进行 fsck 动作                 |
|          |                                                 | time       | 设定关机的时间                          |
|          |                                                 | message    | 传送给所有使用者的警告讯息                    |

**重启**

- `sudo reboot`

- `sudo shutdown -r now`

**关机**

- `shutdown -h now`