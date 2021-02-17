# 一：cd

 Linux cd 命令可以说是Linux中最基本的命令语句，其他的命令语句要进行操作，都是建立在使用 cd 命令上的。所以，学习Linux 常用命令，首先就要学好 cd 命令的使用方法技巧。

## 1.命令格式

**cd [**目录名]

## 2.命令功能

切换当前目录至dirName

## 3.常用范例

**3.1** 例一：进入系统根目录

**cd /** 

进入系统根目录可以使用“ cd .. ”一直退，就可以到达根目录 

**例**2：使用 cd 命令进入当前用户主目录

###  命令1：

cd

``` shell
1 [root@localhost soft]# pwd
2 /opt/soft
3 [root@localhost soft]# cd
4 [root@localhost ~]# pwd
5 /root
```

### 命令2：

cd ~

跳转到指定的目录

cd /opt/soft

``` shell
1 [root@localhost ~]# cd /opt/soft
2 [root@localhost soft]# pwd
3 /opt/soft
4 [root@localhost soft]# cd jdk1.6.0_16/
5 [root@localhost jdk1.6.0_16]# pwd
6 /opt/soft/jdk1.6.0_16
7 [root@localhost jdk1.6.0_16]# 
```

跳转到指定目录，从根目录开始，目录名称前加 / ,当前目录内的子目录直接写名称即可

### 命令3：

cd -

**返回进入此目录之前所在的目录**

```shell
1 [root@localhost soft]# pwd
2 /opt/soft
3 [root@localhost soft]# cd -
4 /root
5 [root@localhost ~]# pwd
6 /root
7 [root@localhost ~]# cd -
8 /opt/soft
9 [root@localhost soft]# 
```

### 命令4：

**把上个命令的参数作为cd参数使用。** 

``` shell
1 [root@localhost soft]# cd !$
2 cd -
3 /root
4 [root@localhost ~]# cd !$
5 cd -
6 /opt/soft
7 [root@localhost soft]# 
```



# 二: pwd

Linux中用 pwd 命令来查看”当前工作目录“的完整路径。 简单得说，每当你在终端进行操作时，你都会有一个当前工作目录。 

在不太确定当前位置时，就会使用pwd来判定当前目录在文件系统内的确切位置。

**1．****命令格式：**

​	pwd [选项]

**2．****命令功能：**

​	查看”当前工作目录“的完整路径

**3．****常用参数：**

一般情况下不带任何参数

如果目录是链接时：

格式：pwd -P 显示出实际路径，而非使用连接（link）路径。 

**4．****常用实例：**

​    **实例****1：****用 pwd 命令查看默认工作目录的完整路径**



# 三:mkdir

linux mkdir 命令用来创建指定的名称的目录，要求创建目录的用户在当前目录中具有写权限，并且指定的目录名不能是当前目录中已有的目录。

## 1.命令格式

mkdir [选项] 目录...

## 2.命令功能

通过 mkdir 命令可以实现在指定位置创建以 DirName(指定的文件名)命名的文件夹或目录。要创建文件夹或目录的用户必须对所创建的文件夹的父文件夹具有写权限。并且，所创建的文件夹(目录)不能与其父目录(即父文件夹)中的文件名重名，即同一个目录下不能有同名的(区分大小写)。 

## 3.命令参数

 -m, --mode=模式，设定权限<模式> (类似 chmod)，而不是 rwxrwxrwx 减 umask

 -p, --parents 可以是一个路径名称。此时若路径中的某些目录尚不存在,加上此选项后,系统将自动建立好那些尚不存在的目录,即一次可以建立多个目录; 

 -v, --verbose 每次创建新目录都显示信息

   --help  显示此帮助信息并退出

   --version 输出版本信息并退出

## 4．命令实例

实例1：创建一个空目录 

命令：

mkdir test1

输出：

```shell
[root@localhost soft]# cd test

[root@localhost test]# mkdir test1

[root@localhost test]# ll

总计 4drwxr-xr-x 2 root root 4096 10-25 17:42 test1

[root@localhost test]#
```

实例2：递归创建多个目录 

命令：

mkdir -p test2/test22

输出：

```shell
[root@localhost test]# mkdir -p test2/test22

[root@localhost test]# ll

总计 8drwxr-xr-x 2 root root 4096 10-25 17:42 test1

drwxr-xr-x 3 root root 4096 10-25 17:44 test2

[root@localhost test]# cd test2/

[root@localhost test2]# ll

总计 4drwxr-xr-x 2 root root 4096 10-25 17:44 test22

[root@localhost test2]#
```

实例3：创建权限为777的目录 

命令：

mkdir -m 777 test3

输出：

```shell
[root@localhost test]# mkdir -m 777 test3

[root@localhost test]# ll

总计 12drwxr-xr-x 2 root root 4096 10-25 17:42 test1

drwxr-xr-x 3 root root 4096 10-25 17:44 test2

drwxrwxrwx 2 root root 4096 10-25 17:46 test3

[root@localhost test]#
```

说明：

test3 的权限为rwxrwxrwx

实例4：创建新目录都显示信息

命令：

mkdir -v test4

输出：

```shell
[root@localhost test]# mkdir -v test4

mkdir: 已创建目录 “test4”

[root@localhost test]# mkdir -vp test5/test5-1

mkdir: 已创建目录 “test5”

mkdir: 已创建目录 “test5/test5-1”

[root@localhost test]#
```

实例五：一个命令创建项目的目录结构

参考：http://www.ibm.com/developerworks/cn/aix/library/au-badunixhabits.html 

命令：

mkdir -vp scf/{lib/,bin/,doc/{info,product},logs/{info,product},service/deploy/{info,product}}

输出：

```shell
[root@localhost test]# mkdir -vp scf/{lib/,bin/,doc/{info,product},logs/{info,product},service/deploy/{info,product}}

mkdir: 已创建目录 “scf”

mkdir: 已创建目录 “scf/lib”

mkdir: 已创建目录 “scf/bin”

mkdir: 已创建目录 “scf/doc”

mkdir: 已创建目录 “scf/doc/info”

mkdir: 已创建目录 “scf/doc/product”

mkdir: 已创建目录 “scf/logs”

mkdir: 已创建目录 “scf/logs/info”

mkdir: 已创建目录 “scf/logs/product”

mkdir: 已创建目录 “scf/service”

mkdir: 已创建目录 “scf/service/deploy”

mkdir: 已创建目录 “scf/service/deploy/info”

mkdir: 已创建目录 “scf/service/deploy/product”

[root@localhost test]# tree scf/

scf/

|-- bin

|-- doc

|   |-- info

|   `-- product

|-- lib

|-- logs

|   |-- info

|   `-- product

`-- service

      `-- deploy

        |-- info

         `-- product


12 directories, 0 files

[root@localhost test]#
```

# 四.rm命令

rm是常用的命令，该命令的功能为删除一个目录中的一个或多个文件或目录，它也可以将某个目录及其下的所有文件及子目录均删除。对于链接文件，只是删除了链接，原有文件均保持不变。

rm是一个危险的命令，使用的时候要特别当心，尤其对于新手，否则整个系统就会毁在这个命令（比如在/（根目录）下执行rm * -rf）。所以，我们在执行rm之前最好先确认一下在哪个目录，到底要删除什么东西，操作时保持高度清醒的头脑。

## 1.命令格式

rm [选项] 文件… 

## 2.命令功能

删除一个目录中的一个或多个文件或目录，如果没有使用- r选项，则rm不会删除目录。如果使用 rm 来删除文件，通常仍可以将该文件恢复原状。

## 3.命令参数

  -f, --force  忽略不存在的文件，从不给出提示。

  -i, --interactive 进行交互式删除

  -r, -R, --recursive  指示rm将参数中列出的全部目录和子目录均递归地删除。

  -v, --verbose  详细显示进行的步骤

​    --help   显示此帮助信息并退出

​    --version 输出版本信息并退出

## 4命令实例

**实例一：**删除文件file，系统会先询问是否删除。 

**命令：**

rm 文件名

**输出：**

```shell
[root@localhost test1]# ll

总计 4

-rw-r--r-- 1 root root 56 10-26 14:31 log.log

root@localhost test1]# rm log.log 

rm：是否删除 一般文件 “log.log”? y

root@localhost test1]# ll

总计 0[root@localhost test1]#
```

**说明：**

输入rm log.log命令后，系统会询问是否删除，输入y后就会删除文件，不想删除则数据n。

**实例二：**强行删除file，系统不再提示。 

**命令：**

rm -f log1.log

**输出：**

```shell
[root@localhost test1]# ll

总计 4

-rw-r--r-- 1 root root 23 10-26 14:40 log1.log

[root@localhost test1]# rm -f log1.log 

[root@localhost test1]# ll

总计 0[root@localhost test1]#
```

**实例三：**删除任何.log文件；删除前逐一询问确认 

**命令：**

rm -i *.log

**输出：**

```shell
[root@localhost test1]# ll

总计 8

-rw-r--r-- 1 root root 11 10-26 14:45 log1.log

-rw-r--r-- 1 root root 24 10-26 14:45 log2.log

[root@localhost test1]# rm -i *.log

rm：是否删除 一般文件 “log1.log”? y

rm：是否删除 一般文件 “log2.log”? y

[root@localhost test1]# ll

总计 0[root@localhost test1]#
```

 **实例四：**将 test1子目录及子目录中所有档案删除

**命令：**

rm -r test1

**输出：**

```shell
[root@localhost test]# ll

总计 24drwxr-xr-x 7 root root 4096 10-25 18:07 scf

drwxr-xr-x 2 root root 4096 10-26 14:51 test1

drwxr-xr-x 3 root root 4096 10-25 17:44 test2

drwxrwxrwx 2 root root 4096 10-25 17:46 test3

drwxr-xr-x 2 root root 4096 10-25 17:56 test4

drwxr-xr-x 3 root root 4096 10-25 17:56 test5

[root@localhost test]# rm -r test1

rm：是否进入目录 “test1”? y

rm：是否删除 一般文件 “test1/log3.log”? y

rm：是否删除 目录 “test1”? y

[root@localhost test]# ll

总计 20drwxr-xr-x 7 root root 4096 10-25 18:07 scf

drwxr-xr-x 3 root root 4096 10-25 17:44 test2

drwxrwxrwx 2 root root 4096 10-25 17:46 test3

drwxr-xr-x 2 root root 4096 10-25 17:56 test4

drwxr-xr-x 3 root root 4096 10-25 17:56 test5

[root@localhost test]#
```

**实例五：**rm -rf test2命令会将 test2 子目录及子目录中所有档案删除,并且不用一一确认

**命令：**

rm -rf test2 

**输出：**

```shell
[root@localhost test]# rm -rf test2

[root@localhost test]# ll

总计 16drwxr-xr-x 7 root root 4096 10-25 18:07 scf

drwxrwxrwx 2 root root 4096 10-25 17:46 test3

drwxr-xr-x 2 root root 4096 10-25 17:56 test4

drwxr-xr-x 3 root root 4096 10-25 17:56 test5

[root@localhost test]#
```

**实例六：**删除以 -f 开头的文件

**命令：**

rm -- -f

```shell
[root@localhost test]# touch -- -f

[root@localhost test]# ls -- -f

-f[root@localhost test]# rm -- -f

rm：是否删除 一般空文件 “-f”? y

[root@localhost test]# ls -- -f

ls: -f: 没有那个文件或目录

[root@localhost test]#
```

也可以使用下面的操作步骤:

```shell
[root@localhost test]# touch ./-f

[root@localhost test]# ls ./-f

./-f[root@localhost test]# rm ./-f

rm：是否删除 一般空文件 “./-f”? y

[root@localhost test]#
```

# 五：rm

 rmdir是常用的命令，该命令的功能是删除空目录，一个目录被删除之前必须是空的。（注意，rm - r dir命令可代替rmdir，但是有很大危险性。）删除某目录时也必须具有对父目录的写权限。

## 1．命令格式

rmdir [选项]... 目录...

## 2.命令功能

该命令从一个目录中删除一个或多个子目录项，删除某目录时也必须具有对父目录的写权限。 

## 3.命令参数

\- p 递归删除目录dirname，当子目录删除后其父目录为空时，也一同被删除。如果整个路径被删除或者由于某种原因保留部分路径，则系统在标准输出上显示相应的信息。  

-v, --verbose 显示指令执行过程 

## 4.命令实例

**实例一：**rmdir 不能删除非空目录

**命令：**

   rmdir doc

**输出：**

```shell
[root@localhost scf]# tree

.

|-- bin

|-- doc

|   |-- info

|   `-- product

|-- lib

|-- logs

|   |-- info

|   `-- product

`-- service

    `-- deploy

        |-- info

        `-- product

 

12 directories, 0 files

[root@localhost scf]# rmdir doc

rmdir: doc: 目录非空

[root@localhost scf]# rmdir doc/info

[root@localhost scf]# rmdir doc/product

[root@localhost scf]# tree

.

|-- bin

|-- doc

|-- lib

|-- logs

|   |-- info

|   `-- product

`-- service

    `-- deploy

        |-- info

        `-- product

 

10 directories, 0 files
```

**说明：**

rmdir 目录名 命令不能直接删除非空目录

**实例2：**rmdir -p 当子目录被删除后使它也成为空目录的话，则顺便一并删除 

**命令：**

rmdir -p logs

**输出：**

```shell
[root@localhost scf]# tree

.

|-- bin

|-- doc

|-- lib

|-- logs

|   `-- product

`-- service

    `-- deploy

        |-- info

        `-- product

 

10 directories, 0 files

[root@localhost scf]# rmdir -p logs

rmdir: logs: 目录非空

[root@localhost scf]# tree

.

|-- bin

|-- doc

|-- lib

|-- logs

|   `-- product

`-- service

    `-- deploy

        |-- info

        `-- product

 

9 directories, 0 files

[root@localhost scf]# rmdir -p logs/product

[root@localhost scf]# tree

.

|-- bin

|-- doc

|-- lib

`-- service

`-- deploy

        |-- info

        `-- product

 

7 directories, 0 files
```

# 六：mv

mv命令是move的缩写，可以用来移动文件或者将文件改名（move (rename) files），是Linux系统下常用的命令，经常用来备份文件或者目录。

## 1.命令格式

  mv [选项] 源文件或目录 目标文件或目录

## 2.命令功能

视mv命令中第二个参数类型的不同（是目标文件还是目标目录），mv命令将文件重命名或将其移至一个新的目录中。当第二个参数类型是文件时，mv命令完成文件重命名，此时，源文件只能有一个（也可以是源目录名），它将所给的源文件或目录重命名为给定的目标文件名。当第二个参数是已存在的目录名称时，源文件或目录参数可以有多个，mv命令将各参数指定的源文件均移至目标目录中。在跨文件系统移动文件时，mv先拷贝，再将原有文件删除，而链至该文件的链接也将丢失。

## 3．命令参数

-b ：若需覆盖文件，则覆盖前先行备份。 

-f ：force 强制的意思，如果目标文件已经存在，不会询问而直接覆盖；

-i ：若目标文件 (destination) 已经存在时，就会询问是否覆盖！

-u ：若目标文件已经存在，且 source 比较新，才会更新(update)

​	  -t  ： --target-directory=DIRECTORY move all SOURCE arguments into DIRECTORY，即指定mv的目标目录，该选项适用于移动多个源文件到一个目录的情况，此时目标目录在前，源文件在后。

**4．****命令实例：**

**实例一：文件改名**

**命令：**

```shell
mv test.log test1.txt
```

**输出：**

```shell
[root@localhost test]# ll

总计 20drwxr-xr-x 6 root root 4096 10-27 01:58 scf

drwxrwxrwx 2 root root 4096 10-25 17:46 test3

drwxr-xr-x 2 root root 4096 10-25 17:56 test4

drwxr-xr-x 3 root root 4096 10-25 17:56 test5

-rw-r--r-- 1 root root   16 10-28 06:04 test.log

[root@localhost test]# mv test.log test1.txt

[root@localhost test]# ll

总计 20drwxr-xr-x 6 root root 4096 10-27 01:58 scf

-rw-r--r-- 1 root root   16 10-28 06:04 test1.txt

drwxrwxrwx 2 root root 4096 10-25 17:46 test3

drwxr-xr-x 2 root root 4096 10-25 17:56 test4

drwxr-xr-x 3 root root 4096 10-25 17:56 test5
```

**说明：**

将文件test.log重命名为test1.txt

**实例二：移动文件**

**命令：**

```shell
mv test1.txt test3
```

**输出：**

```shell
[root@localhost test]# ll

总计 20drwxr-xr-x 6 root root 4096 10-27 01:58 scf

-rw-r--r-- 1 root root   29 10-28 06:05 test1.txt

drwxrwxrwx 2 root root 4096 10-25 17:46 test3

drwxr-xr-x 2 root root 4096 10-25 17:56 test4

drwxr-xr-x 3 root root 4096 10-25 17:56 test5

[root@localhost test]# mv test1.txt test3

[root@localhost test]# ll

总计 16drwxr-xr-x 6 root root 4096 10-27 01:58 scf

drwxrwxrwx 2 root root 4096 10-28 06:09 test3

drwxr-xr-x 2 root root 4096 10-25 17:56 test4

drwxr-xr-x 3 root root 4096 10-25 17:56 test5

[root@localhost test]# cd test3

[root@localhost test3]# ll

总计 4

-rw-r--r-- 1 root root 29 10-28 06:05 test1.txt

[root@localhost test3]#
```

**说明**：

将test1.txt文件移到目录test3中

**实例三：**将文件log1.txt,log2.txt,log3.txt移动到目录test3中。 

**命令：**

```shell
mv log1.txt log2.txt log3.txt test3
```

**输出：**

```shell
[root@localhost test]# ll

总计 28

-rw-r--r-- 1 root root    8 10-28 06:15 log1.txt

-rw-r--r-- 1 root root   12 10-28 06:15 log2.txt

-rw-r--r-- 1 root root   13 10-28 06:16 log3.txt

drwxrwxrwx 2 root root 4096 10-28 06:09 test3

[root@localhost test]# mv log1.txt log2.txt log3.txt test3

[root@localhost test]# ll

总计 16drwxrwxrwx 2 root root 4096 10-28 06:18 test3

[root@localhost test]# cd test3/

[root@localhost test3]# ll

总计 16

-rw-r--r-- 1 root root  8 10-28 06:15 log1.txt

-rw-r--r-- 1 root root 12 10-28 06:15 log2.txt

-rw-r--r-- 1 root root 13 10-28 06:16 log3.txt

-rw-r--r-- 1 root root 29 10-28 06:05 test1.txt

[root@localhost test3]#

[root@localhost test3]# ll

总计 20

-rw-r--r-- 1 root root    8 10-28 06:15 log1.txt

-rw-r--r-- 1 root root   12 10-28 06:15 log2.txt

-rw-r--r-- 1 root root   13 10-28 06:16 log3.txt

drwxr-xr-x 2 root root 4096 10-28 06:21 logs

-rw-r--r-- 1 root root   29 10-28 06:05 test1.txt

[root@localhost test3]# mv -t /opt/soft/test/test4/ log1.txt log2.txt  log3.txt 

[root@localhost test3]# cd ..

[root@localhost test]# cd test4/

[root@localhost test4]# ll

总计 12

-rw-r--r-- 1 root root  8 10-28 06:15 log1.txt

-rw-r--r-- 1 root root 12 10-28 06:15 log2.txt

-rw-r--r-- 1 root root 13 10-28 06:16 log3.txt

[root@localhost test4]#
```

mv log1.txt log2.txt log3.txt test3 命令将log1.txt ，log2.txt， log3.txt 三个文件移到 test3目录中去，mv -t /opt/soft/test/test4/ log1.txt log2.txt log3.txt 命令又将三个文件移动到test4目录中去

**实例四：****将文件file1改名为file2，如果file2已经存在，则询问是否覆盖**

命令：

```shell
mv -i log1.txt log2.txt
```

输出：

```shell
[root@localhost test4]# ll

总计 12

-rw-r--r-- 1 root root  8 10-28 06:15 log1.txt

-rw-r--r-- 1 root root 12 10-28 06:15 log2.txt

-rw-r--r-- 1 root root 13 10-28 06:16 log3.txt

[root@localhost test4]# cat log1.txt 

odfdfs

[root@localhost test4]# cat log2.txt 

ererwerwer

[root@localhost test4]# mv -i log1.txt log2.txt 

mv：是否覆盖“log2.txt”? y

[root@localhost test4]# cat log2.txt 

odfdfs

[root@localhost test4]#
```

**实例五：****将文件file1改名为file2，即使file2存在，也是直接覆盖掉。**

**命令：**

```shell
mv -f log3.txt log2.txt
```

**输出：**

```shell
[root@localhost test4]# ll

总计 8

-rw-r--r-- 1 root root  8 10-28 06:15 log2.txt

-rw-r--r-- 1 root root 13 10-28 06:16 log3.txt

[root@localhost test4]# cat log2.txt 

odfdfs

[root@localhost test4]# cat log3

cat: log3: 没有那个文件或目录

[root@localhost test4]# ll

总计 8

-rw-r--r-- 1 root root  8 10-28 06:15 log2.txt

-rw-r--r-- 1 root root 13 10-28 06:16 log3.txt

[root@localhost test4]# cat log2.txt 

odfdfs

[root@localhost test4]# cat log3.txt 

dfosdfsdfdss

[root@localhost test4]# mv -f log3.txt log2.txt 

[root@localhost test4]# cat log2.txt 

dfosdfsdfdss

[root@localhost test4]# ll

总计 4

-rw-r--r-- 1 root root 13 10-28 06:16 log2.txt

[root@localhost test4]#
```

**说明**：

log3.txt的内容直接覆盖了log2.txt内容，-f 这是个危险的选项，使用的时候一定要保持头脑清晰，一般情况下最好不用加上它。

**实例六：目录的移动**

**命令：**

```shell
mv dir1 dir2 
```

**输出：**

```shell
[root@localhost test4]# ll

-rw-r--r-- 1 root root 13 10-28 06:16 log2.txt

[root@localhost test4]# ll

-rw-r--r-- 1 root root 13 10-28 06:16 log2.txt

[root@localhost test4]# cd ..

[root@localhost test]# ll

drwxr-xr-x 6 root root 4096 10-27 01:58 scf

drwxrwxrwx 3 root root 4096 10-28 06:24 test3

drwxr-xr-x 2 root root 4096 10-28 06:48 test4

drwxr-xr-x 3 root root 4096 10-25 17:56 test5

[root@localhost test]# cd test3

[root@localhost test3]# ll

drwxr-xr-x 2 root root 4096 10-28 06:21 logs

-rw-r--r-- 1 root root   29 10-28 06:05 test1.txt

[root@localhost test3]# cd ..

[root@localhost test]# mv test4 test3

[root@localhost test]# ll

drwxr-xr-x 6 root root 4096 10-27 01:58 scf

drwxrwxrwx 4 root root 4096 10-28 06:54 test3

drwxr-xr-x 3 root root 4096 10-25 17:56 test5

[root@localhost test]# cd test3/

[root@localhost test3]# ll

drwxr-xr-x 2 root root 4096 10-28 06:21 logs

-rw-r--r-- 1 root root   29 10-28 06:05 test1.txt

drwxr-xr-x 2 root root 4096 10-28 06:48 test4

[root@localhost test3]#
```

**说明：**

如果目录dir2不存在，将目录dir1改名为dir2；否则，将dir1移动到dir2中。

**实例****7：移动当前文件夹下的所有文件到上一级目录**

**命令：**

```shell
mv * ../
```

**输出：**

```shell
[root@localhost test4]# ll

-rw-r--r-- 1 root root 25 10-28 07:02 log1.txt

-rw-r--r-- 1 root root 13 10-28 06:16 log2.txt

[root@localhost test4]# mv * ../

[root@localhost test4]# ll

[root@localhost test4]# cd ..

[root@localhost test3]# ll

-rw-r--r-- 1 root root   25 10-28 07:02 log1.txt

-rw-r--r-- 1 root root   13 10-28 06:16 log2.txt

drwxr-xr-x 2 root root 4096 10-28 06:21 logs

-rw-r--r-- 1 root root   29 10-28 06:05 test1.txt

drwxr-xr-x 2 root root 4096 10-28 07:02 test4
```

**实例八：把当前目录的一个子目录里的文件移动到另一个子目录里**

**命令：**

```shell
mv test3/*.txt test5
```

**输出：**

```shell
[root@localhost test]# ll

drwxr-xr-x 6 root root 4096 10-27 01:58 scf

drwxrwxrwx 4 root root 4096 10-28 07:02 test3

drwxr-xr-x 3 root root 4096 10-25 17:56 test5

[root@localhost test]# cd test3

[root@localhost test3]# ll

-rw-r--r-- 1 root root   25 10-28 07:02 log1.txt

-rw-r--r-- 1 root root   13 10-28 06:16 log2.txt

drwxr-xr-x 2 root root 4096 10-28 06:21 logs

-rw-r--r-- 1 root root   29 10-28 06:05 test1.txt

drwxr-xr-x 2 root root 4096 10-28 07:02 test4

[root@localhost test3]# cd ..

[root@localhost test]# mv test3/*.txt test5

[root@localhost test]# cd test5

[root@localhost test5]# ll

-rw-r--r-- 1 root root   25 10-28 07:02 log1.txt

-rw-r--r-- 1 root root   13 10-28 06:16 log2.txt

-rw-r--r-- 1 root root   29 10-28 06:05 test1.txt

drwxr-xr-x 2 root root 4096 10-25 17:56 test5-1

[root@localhost test5]#  cd ..

[root@localhost test]# cd test3/

[root@localhost test3]# ll

drwxr-xr-x 2 root root 4096 10-28 06:21 logs

drwxr-xr-x 2 root root 4096 10-28 07:02 test4

[root@localhost test3]#
```

**实例九：文件被覆盖前做简单备份，前面加参数-b**

**命令：**

```shell
mv log1.txt -b log2.txt
```

**输出：**

```shell
[root@localhost test5]# ll

-rw-r--r-- 1 root root   25 10-28 07:02 log1.txt

-rw-r--r-- 1 root root   13 10-28 06:16 log2.txt

-rw-r--r-- 1 root root   29 10-28 06:05 test1.txt

drwxr-xr-x 2 root root 4096 10-25 17:56 test5-1

[root@localhost test5]# mv log1.txt -b log2.txt

mv：是否覆盖“log2.txt”? y

[root@localhost test5]# ll

-rw-r--r-- 1 root root   25 10-28 07:02 log2.txt

-rw-r--r-- 1 root root   13 10-28 06:16 log2.txt~

-rw-r--r-- 1 root root   29 10-28 06:05 test1.txt

drwxr-xr-x 2 root root 4096 10-25 17:56 test5-1

[root@localhost test5]#
```

**说明：**

-b 不接受参数，mv会去读取环境变量VERSION_CONTROL来作为备份策略。

--backup该选项指定如果目标文件存在时的动作，共有四种备份策略：

1.CONTROL=none或off : 不备份。

2.CONTROL=numbered或t：数字编号的备份

3.CONTROL=existing或nil：如果存在以数字编号的备份，则继续编号备份m+1...n：

执行mv操作前已存在以数字编号的文件log2.txt.~1~，那么再次执行将产生log2.txt~2~，以次类推。如果之前没有以数字编号的文件，则使用下面讲到的简单备份。

4.CONTROL=simple或never：使用简单备份：在被覆盖前进行了简单备份，简单备份只能有一份，再次被覆盖时，简单备份也会被覆盖。

# 七：cp

cp命令用来复制文件或者目录，是Linux系统中最常用的命令之一。一般情况下，shell会设置一个别名，在命令行下复制文件时，如果目标文件已经存在，就会询问是否覆盖，不管你是否使用-i参数。但是如果是在shell脚本中执行cp时，没有-i参数时不会询问是否覆盖。这说明命令行和shell脚本的执行方式有些不同。 

## 1.命令格式

用法：

​	  cp [选项]... [-T] 源 目的

​    或：cp [选项]... 源... 目录

​    或：cp [选项]... -t 目录 源...

## 2.命令功能

将源文件复制至目标文件，或将多个源文件复制至目标目录。

## 3.命令参数

-a, --archive  等于-dR --preserve=all

  --backup[=CONTROL  为每个已存在的目标文件创建备份

-b        类似--backup 但不接受参数

  --copy-contents    在递归处理是复制特殊文件内容

-d        等于--no-dereference --preserve=links

-f, --force    如果目标文件无法打开则将其移除并重试(当 -n 选项

​          存在时则不需再选此项)

-i, --interactive    覆盖前询问(使前面的 -n 选项失效)

-H        跟随源文件中的命令行符号链接

-l, --link      链接文件而不复制

-L, --dereference  总是跟随符号链接

-n, --no-clobber  不要覆盖已存在的文件(使前面的 -i 选项失效)

-P, --no-dereference  不跟随源文件中的符号链接

-p        等于--preserve=模式,所有权,时间戳

  --preserve[=属性列表  保持指定的属性(默认：模式,所有权,时间戳)，如果

​        可能保持附加属性：环境、链接、xattr 等

-R, -r, --recursive 复制目录及目录内的所有项目

## 4.命令实例

**实例一：复制单个文件到目标目录，文件在目标文件中不存在**

**命令：**

```shell
cp log.log test5
```

**输出：**

```shell
[root@localhost test]# cp log.log test5

[root@localhost test]# ll

-rw-r--r-- 1 root root    0 10-28 14:48 log.log

drwxr-xr-x 6 root root 4096 10-27 01:58 scf

drwxrwxrwx 2 root root 4096 10-28 14:47 test3

drwxr-xr-x 2 root root 4096 10-28 14:53 test5

[root@localhost test]# cd test5

[root@localhost test5]# ll

-rw-r--r-- 1 root root 0 10-28 14:46 log5-1.log

-rw-r--r-- 1 root root 0 10-28 14:46 log5-2.log

-rw-r--r-- 1 root root 0 10-28 14:46 log5-3.log

-rw-r--r-- 1 root root 0 10-28 14:53 log.log
```

**说明：**

在没有带-a参数时，两个文件的时间是不一样的。在带了-a参数时，两个文件的时间是一致的。 



**实例二：目标文件存在时，会询问是否覆盖**

**命令：**

```shell
cp log.log test5
```

**输出：**

```shell
[root@localhost test]# cp log.log test5

cp：是否覆盖“test5/log.log”? n

[root@localhost test]# cp -a log.log test5

cp：是否覆盖“test5/log.log”? y

[root@localhost test]# cd test5/

[root@localhost test5]# ll

-rw-r--r-- 1 root root 0 10-28 14:46 log5-1.log

-rw-r--r-- 1 root root 0 10-28 14:46 log5-2.log

-rw-r--r-- 1 root root 0 10-28 14:46 log5-3.log

-rw-r--r-- 1 root root 0 10-28 14:48 log.log
```

八：head

head 与 tail 就像它的名字一样的浅显易懂，它是用来显示开头或结尾某个数量的文字区块，head 用来显示档案的开头至标准输出中，而 tail 想当然尔就是看档案的结尾。 

**1．****命令格式：**

head [参数]... [文件]... 

**2．****命令功能：**

head 用来显示档案的开头至标准输出中，默认head命令打印其相应文件的开头10行。 

**3．****命令参数：**

-q 隐藏文件名

-v 显示文件名

-c<字节> 显示字节数

-n<行数> 显示的行数

**4．****使用实例：**

**实例1：显示文件的前n行**

**命令：**

``` shell
head -n 5 log2014.log
```

**输出：**

**实例2：显示文件前n个字节**

```shell
head -c 20 log2014.log
```

**实例3：****文件的除了最后****n****个字节以外的内容** 

```shell
head -c -32 log2014.log
```

# 八：tail命令

tail 命令从指定点开始将文件写到标准输出.使用tail命令的-f选项可以方便的查阅正在改变的日志文件,tail -f filename会把filename里最尾部的内容显示在屏幕上,并且不但刷新,使你看到最新的文件内容. 

**1．****命令格式****;**

tail[必要参数][选择参数][文件]  

**2．****命令功能：**

用于显示指定文件末尾内容，不指定文件时，作为输入信息进行处理。常用查看日志文件。

**3．****命令参数：**

-f 循环读取

-q 不显示处理信息

-v 显示详细的处理信息

-c<数目> 显示的字节数

-n<行数> 显示行数

--pid=PID 与-f合用,表示在进程ID,PID死掉之后结束. 

-q, --quiet, --silent 从不输出给出文件名的首部 

-s, --sleep-interval=S 与-f合用,表示在每次反复的间隔休眠S秒 

**4．****使用实例：**

**实例1：显示文件末尾内容**

```shell
tail -n 5 log2014.log
```

**实例2：循环查看文件内容**

```shell
tail -f test.log
```

**实例3：从第5行开始显示文件**

**命令：**

tail -n +5 log2014.log

# 九：head命令

head 与 tail 就像它的名字一样的浅显易懂，它是用来显示开头或结尾某个数量的文字区块，head 用来显示档案的开头至标准输出中，而 tail 想当然尔就是看档案的结尾。 

**1．****命令格式：**

head [参数]... [文件]... 

**2．****命令功能：**

head 用来显示档案的开头至标准输出中，默认head命令打印其相应文件的开头10行。 

**3．****命令参数：**

-q 隐藏文件名

-v 显示文件名

-c<字节> 显示字节数

-n<行数> 显示的行数

**4．****使用实例：**

**实例1：显示文件的前n行**

**命令：**

head -n 5 log2014.log

**实例2：显示文件前n个字节**

**命令：**

head -c 20 log2014.log

**实例3：****文件的除了最后****n****个字节以外的内容** 

**命令：**

head -c -32 log2014.log

**实例****4：输出文件除了最后n行的全部内容**

**命令：**

head -n -6 log2014.log



# 十 : less命令

less 工具也是对文件或其它输出进行分页显示的工具，应该说是linux正统查看文件内容的工具，功能极其强大。less 的用法比起 more 更加的有弹性。在 more 的时候，我们并没有办法向前面翻， 只能往后面看，但若使用了 less 时，就可以使用 [pageup] [pagedown] 等按键的功能来往前往后翻看文件，更容易用来查看一个文件的内容！除此之外，在 less 里头可以拥有更多的搜索功能，不止可以向下搜，也可以向上搜。

**1．****命令格式：**

less [参数] 文件 

**2．****命令功能：**

less 与 more 类似，但使用 less 可以随意浏览文件，而 more 仅能向前移动，却不能向后移动，而且 less 在查看之前不会加载整个文件。

**3．****命令参数：**

-b <缓冲区大小> 设置缓冲区的大小

-e 当文件显示结束后，自动离开

-f 强迫打开特殊文件，例如外围设备代号、目录和二进制文件

-g 只标志最后搜索的关键词

-i 忽略搜索时的大小写

-m 显示类似more命令的百分比

-N 显示每行的行号

-o <文件名> 将less 输出的内容在指定文件中保存起来

-Q 不使用警告音

-s 显示连续空行为一行

-S 行过长时间将超出部分舍弃

-x <数字> 将“tab”键显示为规定的数字空格

/字符串：向下搜索“字符串”的功能

?字符串：向上搜索“字符串”的功能

n：重复前一个搜索（与 / 或 ? 有关）

N：反向重复前一个搜索（与 / 或 ? 有关）

b 向后翻一页

d 向后翻半页

h 显示帮助界面

Q 退出less 命令

u 向前滚动半页

y 向前滚动一行

空格键 滚动一行

回车键 滚动一页

[pagedown]： 向下翻动一页

[pageup]：  向上翻动一页



**4．****使用实例：**

**实例1：查看文件**

**命令：**

less log2013.log

**5．****附加备注**

**1.全屏导航**

ctrl + F - 向前移动一屏

ctrl + B - 向后移动一屏

ctrl + D - 向前移动半屏

ctrl + U - 向后移动半屏

 

**2.单行导航**

j - 向前移动一行

k - 向后移动一行

 

**3.其它导航**

G - 移动到最后一行

g - 移动到第一行

q / ZZ - 退出 less 命令

 

**4.其它有用的命令**

v - 使用配置的编辑器编辑当前文件

h - 显示 less 的帮助文档

&pattern - 仅显示匹配模式的行，而不是整个文件

 

**5.标记导航**

当使用 less 查看大文件时，可以在任何一个位置作标记，可以通过命令导航到标有特定标记的文本位置：

ma - 使用 a 标记文本的当前位置

'a - 导航到标记 a 处



# 十一 whereis

whereis命令只能用于程序名的搜索，而且只搜索二进制文件（参数-b）、man说明文件（参数-m）和源代码文件（参数-s）。如果省略参数，则返回所有信息。

和find相比，whereis查找的速度非常快，这是因为linux系统会将 系统内的所有文件都记录在一个数据库文件中，当使用whereis和下面即将介绍的locate时，会从数据库中查找数据，而不是像find命令那样，通 过遍历硬盘来查找，效率自然会很高。 

但是该数据库文件并不是实时更新，默认情况下时一星期更新一次，因此，我们在用whereis和locate 查找文件时，有时会找到已经被删除的数据，或者刚刚建立文件，却无法查找到，原因就是因为数据库文件没有被更新。 

**1．****命令格式：**

whereis [-bmsu] [BMS 目录名 -f ] 文件名

**2．****命令功能：**

whereis命令是定位可执行文件、源代码文件、帮助文件在文件系统中的位置。这些文件的属性应属于原始代码，二进制文件，或是帮助文件。whereis 程序还具有搜索源代码、指定备用搜索路径和搜索不寻常项的能力。

**3．****命令参数：**

-b  定位可执行文件。

-m  定位帮助文件。

-s  定位源代码文件。

-u  搜索默认路径下除可执行文件、源代码文件、帮助文件以外的其它文件。

-B  指定搜索可执行文件的路径。

-M  指定搜索帮助文件的路径。

-S  指定搜索源代码文件的路径。

**4．****使用实例：**

**实例1：将和\**文件相关的文件都查找出来**

**命令：**

whereis svn

**实例2：****只将二进制文件 查找出来** 

**命令：**

whereis -b svn

# 十二  which

我们经常在linux要查找某个文件，但不知道放在哪里了，可以使用下面的一些命令来搜索： 
    which 查看可执行文件的位置。
    whereis 查看文件的位置。 
    locate  配合数据库查看文件位置。
    find  实际搜寻硬盘查询文件名称。

which命令的作用是，在PATH变量指定的路径中，搜索某个系统命令的位置，并且返回第一个搜索结果。也就是说，使用which命令，就可以看到某个系统命令是否存在，以及执行的到底是哪一个位置的命令。 

**1．****命令格式：**

which 可执行文件名称 

**2．****命令功能：**

which指令会在PATH变量指定的路径中，搜索某个系统命令的位置，并且返回第一个搜索结果。

**3．****命令参数：**

-n 指定文件名长度，指定的长度必须大于或等于所有文件中最长的文件名。

-p 与-n参数相同，但此处的包括了文件的路径。

-w 指定输出时栏位的宽度。

-V 显示版本信息

**4．****使用实例：**

**实例1：查找文件、显示命令路径**

**命令：**

which lsmod

# 十三 locate命令





## 十四find命令

