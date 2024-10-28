# linux常用命令

# linux高级命令

| 序号 | 命令                              | 命令解释                               |
| ---- | --------------------------------- | -------------------------------------- |
| 1    | top                               | 查看内存                               |
| 2    | df -h                             | 查看磁盘存储情况                       |
| 3    | iotop                             | 查看磁盘IO读写(yum install iotop安装） |
| 4    | iotop -o                          | 直接查看比较高的磁盘读写程序           |
| 5    | netstat -tlnp \| grep 端口号      | 查看端口占用情况                       |
| 6    | uptime                            | 查看报告系统运行时长及平均负载         |
| 7    | ps -aux     或  ps -ef\|grep java | 查看进程                               |

## 查看端口的进程

### 1. netstat -tlnp|grep 端口号

``` shell
netstat -tlnp|grep 3306
```

### 2. ss -ltnp | grep 端口号

``` shell
ss -ltnp|grep 3306
```

### 3. lsof -i :端口号

