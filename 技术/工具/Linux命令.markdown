## 内部命令与外部命令

### 内部命令

shell自带的命令

### 外部命令

由用户安装的命令

当我们登录到linux时，就进入了bash shell程序

### bash shell 执行命令

cd /etc/

1. 根据空格来切割字符串，把第一个位置认为是命令，后面的座位参数
2. 判断命令是内部命令还是外部命令通过type来判断
3. 外部命令，寻找这个命令的执行文件，执行命令速度并不慢，因为这个有path环境变量（对于外部命令的优化）

## hash优化查询时间



## 常用命令

1. whereis 查找命令的位置
2. type  是否是内部命令
3. echo 相当于system.out.println
4. man + 命令
5. ifconfig 查看网卡信息
6. a = 1 -- echo $a arr = (1 2 3) echo${arr[1]}
7. ehco $$ 打印bash shell进程号
8. kill -9 进程号

## 文件系统

df -h 查看磁盘挂载情况

/boot 系统启动相关的文件

/dev 设备文件

/lib 库文件

/opt 第三方程序的安装目录

/var 可变化的文件

/bin  可执行文件，用户命令

/sbin 管理命令

/proc 临时文件系统

## 文件系统相关命令

du 显示文件系统使用情况

ls 显示目录