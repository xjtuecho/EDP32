# EDP32 Command Line Reference

EDP32内置一个命令解释器，通讯协议设置为TERM时，可以通过超级终端（或者Putty、SecrueCRT）
等软件来连接该命令解释器，实现类似Linux Bash的交互界面。

默认串口参数：`波特率115200、8位数据、1位停止、无校验、无流控`。

本文档基于EDP32固件v20.3.24，其余固件版本仅供参考。

## getui

获取电压、电流、功率、温度等信息。

命令格式：`getui` 

```
getui
 Ui=1.1085V 12.19V 0 AD=0x2AF4 0x0564
 Uo=0.4540V  4.99V 0 AD=0x1198 0x0232
 Io=0.0489V 0.000A 0 AD=0x01E6 0x0049
 Vt=1.5168V   29.4oC AD=0x3AC6 0x0753
 Vd=3.3035V   1200mV AD=0x0000
```

数据分5行：

- 1行Ui为输入电压，包括：控制器管脚电压、实际电压、当前电压档位、AD采样原始值；
- 2行Uo为输出电压，包括：控制器管脚电压、实际电压、当前电压档位、AD采样原始值；
- 3行Io为输出电流，包括：控制器管脚电压、实际电流、当前电流档位、AD采样原始值；
- 4行Vt为主板温度，包括：控制器管脚电压、实际温度、AD采样原始值；
- 5行Vd为3.3V电压，由用户校准时设置；

第三方工具可以使用该命令采集当前数据。


## clear

清除设备当前时间和电量信息。

设备上电以后，运行时间和Ah电量、Wh电量会一直累计，执行该命令以后，运行时间、Ah电量、Wh电量全部归零。

其余数据为实时更新，该命令无影响。

在电量测试界面下长按**下键（DN）**可清除当前时间和电量信息。

## log

操作离线数据。

命令格式：`log [dump|auto|append|uart|file|int|flush] Operate data logs.`

不带参数的log命令输出当前设置，如下所示：
```
log
log [dump|auto|append|uart|file|int|flush] Operate data logs.
 LOG=0x0034 AUTO=0 APPEND=1 UART=1 FILE=1 INT=0 FLUSH=100
```
具体意义见以下子命令说明。

### log dump

log dump子命令导出`record.csv`文件中记录的离线数据。

命令格式：`log dump`.

```
log dump
     0,  5529,12.20, 0.00,0.000, 29.1
     1,  5529,12.20, 0.00,0.000, 29.1
     2,  5530,12.20, 0.00,0.000, 29.1
     3,  5530,12.20, 0.00,0.000, 29.1
     4,  5530,12.20, 0.00,0.000, 29.1
     5,  5531,12.20, 0.00,0.000, 29.1
     6,  5531,12.19, 0.00,0.000, 29.1
     7,  5531,12.20, 0.00,0.000, 29.1
```

输出数据为CSV格式，共六列：

- 第一列为数据索引；
- 第二列为设备记录数据时的相对时间（单位秒）；
- 第三列为输入电压，单位V；
- 第四列为输出电压，单位V；
- 第五列为输出电流，单位A；
- 第六列为主板温度，单位℃；
 
可以借助超级终端捕获文字功能保存离线数据。菜单：`传送(T)->捕获文字(C)…`打开超级终端的捕获文字功能，
启动捕获文字，执行完log dump命令，然后停止保存离线记录数据。

### log auto

打开或者关闭AUTO模式（自动记录）。

默认情况下，数据记录功能需要用户来手动启动。如果用户需要上电后自动开始记录，可以设置AUTO=1打开自动记录功能。

使用`log auto 1`命令打开自动记录功能。

使用`log auto 0`命令关闭自动记录功能。

设置以后立即生效，保存参数需要执行`param save`命令。

### log append

打开或者关闭追加记录模式。

每次启动数据记录，记录数据默认以截断模式写入`record.csv`文件，即会删除`record.csv`文件中的已有内容。
打开追加记录模式以后，不删除`record.csv`原有内容，新的数据追加到原有数据之后。

使用`log append 1`命令打开追加记录模式。

使用`log append 0`命令关闭追加记录模式。

设置以后立即生效，保存参数需要执行`param save`命令。

### log uart

记录数据是否发送到串口。

打开以后，记录数据的同时将数据打印到UART串口。

使用`log uart 1`命令打开UART功能。

使用`log uart 0`命令关闭UART功能。

设置以后立即生效，保存参数需要执行`param save`命令。

### log file

记录数据是否写入`record.csv`文件。

打开以后，记录数据写入设备内置存储的`record.csv`文件。

使用`log file 1`命令打开FILE功能。

使用`log file 0`命令关闭FILE功能。

设置以后立即生效，保存参数需要执行`param save`命令。

UART和FILE两个选项配合可以实现设备数据记录功能：

- UART=1，FILE=0，数据只发送到串口。
- UART=0，FILE=1，数据只记录到内部文件。
- UART=1，FILE=1，数据记录到内部文件，同时发送到串口。
- UART=0，FILE=0，不记录。

### log int

设置离线记录周期，单位秒，如：`log int 10`，设置离线记录间隔为10秒。

设置为0时以最快3Hz记录数据，最大周期255秒。

设置以后立即生效，保存参数需要执行`param save`命令。

### log flush

设置内部缓存刷新间隔。

命令格式：`log flush [dec flush] Set log flush interval.`

数据记录过程中如果突然掉电会丢失缓存中的数据，设置记录间隔可以尽量减少数据损失，
默认100条刷新一次，设置为0关闭定时刷新功能。

设置以后立即生效，保存参数需要执行`param save`命令。

## param

操作用户参数。

命令格式：`param [load|save|restore] Operate parameters.`

param命令带三个子命令：load、save、restore。

`param load`命令从内置EEPROM加载保存的参数。

`param save`命令将参数保存到内置EEPROM。

`param restore`命令恢复默认参数。同时按住上下键上电也可以恢复默认参数。

## uiset

输入电压相关参数设置。

命令格式：`uiset [adj|zero|cali|vdd] [dat] set Uin param.`

不带参数的uiset命令输出当前电压电流采样所有信息。

```
uiset
uiset [adj|zero|cali|vdd] [dat] set Uin param.
 UI=1.0000 1.0000 1.0000 1.0000     0
 UO=1.0000 1.0000 1.0000 1.0000     0
 IO=0.9727 0.9746 0.9746 0.9746   484
 UVP=  4.80V OVP= 31.00V OCP= 5.500A OPP=100.0W
 OTP= 80.0oC OAP=  0.0Ah OWP=  0.0Wh OHP=00h00m
 USET= 5.00V ISET=5.100A USTEP=    2 ISTEP=   64
 UMAX=30.00V UMIN= 0.00V UHYS= 0.50V
```

### uiset adj

设置输入电压增益校正系数。

电压增益校正系数是一个1附近的数值，设置数值扩大10000倍。

如设置输入电压增益系数为1.0023，命令为：`uiset adj 10023`

### uiset zero

设置输入电压零偏校正系数。

如电压零偏校正系数为480，命令为：`uiset zero 480`。

### uiset cali

输入电压快速校准。

首先使用`uiset adj 10000`命令将输入电压增益系数设置为1.

使用高位电压表测量设备实际输入电压，执行命令`uset cali [实际电压x100]`，设备自动计算输入电压增益系数。

设置以后立即生效，保存参数需要执行`param save`命令。

### uiset vdd

设置3.3V参考电压实际值。

如设置3.3V实际电压为3.2777V，命令为：`uiset vdd 32777`

## uoset

输出电压相关参数设置。

命令格式：`uoset [adj|zero|cali|set] [dat] set Uout param.`

不带参数的uoset命令输出当前电压电流采样所有信息。

```
uoset
uoset [adj|zero|cali|set] [dat] set Uout param.
 UI=1.0000 1.0000 1.0000 1.0000     0
 UO=1.0000 1.0000 1.0000 1.0000     0
 IO=0.9727 0.9746 0.9746 0.9746   484
 UVP=  4.80V OVP= 31.00V OCP= 5.500A OPP=100.0W
 OTP= 80.0oC OAP=  0.0Ah OWP=  0.0Wh OHP=00h00m
 USET= 5.00V ISET=5.100A USTEP=    2 ISTEP=   64
 UMAX=30.00V UMIN= 0.00V UHYS= 0.50V
```

### uoset adj

设置输出电压增益校正系数。

电压增益校正系数是一个1附近的数值，设置数值扩大10000倍。

如设置输出电压增益系数为1.0023，命令为：`uoset adj 10023`

### uoset zero

设置输出电压零偏校正系数。

如电压零偏校正系数为480，命令为：`uoset zero 480`。

### uoset cali

输出电压快速校准。

首先使用`uoset adj 10000`命令将输出电压增益系数设置为1.

使用高位电压表测量设备实际输出电压，执行命令`uoset cali [实际电压x100]`，设备自动计算输出电压增益系数。

设置以后立即生效，保存参数需要执行`param save`命令。

### uoset set

设置输出电压值，实际电压扩大100倍。

如设置输出电压为3.30V，命令为：`uoset set 330`

## ioset

输出电流相关参数设置。

命令格式：`ioset [adj|zero|cali|set] [dat] set Iout param.`

不带参数的ioset命令输出当前电压电流采样所有信息。

```
ioset
ioset [adj|zero|cali|set] [dat] set Iout param.
 UI=1.0000 1.0000 1.0000 1.0000     0
 UO=1.0000 1.0000 1.0000 1.0000     0
 IO=0.9727 0.9746 0.9746 0.9746   484
 UVP=  4.80V OVP= 31.00V OCP= 5.500A OPP=100.0W
 OTP= 80.0oC OAP=  0.0Ah OWP=  0.0Wh OHP=00h00m
 USET= 1.00V ISET=5.100A USTEP=    2 ISTEP=   64
 UMAX=30.00V UMIN= 0.00V UHYS= 0.50V
```

### ioset adj

设置输出电流增益校正系数。

电流增益校正系数是一个1附近的数值，设置数值扩大10000倍。

如设置输出电流增益系数为1.0023，命令为：`ioset adj 10023`

### ioset zero

设置输出电流零偏校正系数。

如电流零偏校正系数为480，命令为：`ioset zero 480`。

### ioset cali

输出电流快速校准。

首先使用`ioset adj 10000`命令将输出电流增益系数设置为1.

使用高位电流表测量设备实际输出电流，执行命令`ioset cali [实际电流x1000]`，设备自动计算输出电压增益系数。

设置以后立即生效，保存参数需要执行`param save`命令。

### ioset set

设置输出电流值，实际电流扩大1000倍。

如设置输出电流为3.000A，命令为：`ioset set 3000`

## info

查看设置系统信息。

命令格式：`info -> info [func|dev|log|lcd|addr|baud|bklt] Display/Set system Info.`

不带参数的info命令显示当前系统信息，如下所示：

```
info
 FUNC=0x0201 ECHO=1 TEST=0 PWRON=0 LOCK=0
 DEV=0x0310 BUZ=0 BAUD=3 115200 PROP=1 TERM
 LOG=0x0130 AUTO=0 APPEND=0 UART=1 FILE=1 INT=1
 LCD=0x0089 DIR=2 BKLT=8 PAGE=0
 FLAG=0x0100 ADDR=1 AUTO=60
```

子命令介绍如下：

- func、dev、log、lcd为设备内部使用的寄存器，通过面板可以设置，通常无需用户修改。
- addr为MODBUS地址，通过`info addr [1-247]`命令进行修改，推荐通过面板设置。
- auto为参数自动保存时间，单位秒，通过`info auto [10-3600]`命令进行修改。
- baud为波特率寄存器，推荐通过面板设置。
- bklt为LCD背光亮度寄存器，推荐通过面板设置。

## ctrl

设置控制命令。

命令格式：`ctrl [echo|test|led|buz|main|time] [param] Device Control.`。

- `ctrl echo [0-1]`控制命令行回显开关。
- `ctrl test [0-1]`控制测试开关，仅用来测试。
- `ctrl led [0-1]`控制面板LED，仅用来测试，需要TEST=1.
- `ctrl buz [0-1]`控制面板蜂鸣器，仅用来测试，需要TEST=1，**实际硬件无蜂鸣器**。
- `ctrl main [0-1]`总输出开关控制，与按下面板编码器等效。
- `ctrl time [dec time]`设置设备运行时间，参数单位为秒。

## lfs

操作文件系统。

命令格式：`lfs [umount|mount|format] Operate File System`

带三个子命令：

- `lfs umount`卸载文件系统。
- `lfs mount`挂载文件系统。
- `lfs format`格式化文件系统。

用户通常无需自己挂载、卸载文件系统；更换存储芯片或文件系统遇到问题，可尝试`lfs format`命令重新格式化文件系统。

## ls

列出当前文件。

举例如下：

```
ls
d        0 .
d        0 ..
-  7488975 01.csv
-   738504 02.csv
-     1404 record.csv
```

列出的内容中：

- 第一列为文件类型，`d`表示目录，`-`表示文件；
- 第二列为文件大小，单位字节(Byte)；
- 第三列为文件名，只支持`8.3`格式。

EDP32默认不支持多层目录。

## df

查看存储占用情况。

举例如下：

```
df
Block Size:4096 Total:2048 Used:2018 Rate:98.5%
```

以上df命令的输出表示存储空间由2048个4kB块组成，已经使用2018个，使用率98.5%。

## rm

删除文件或者目录。

命令格式：`rm [file|dir] remove FILE or DIR.`

删除`record.csv`文件，举例如下：

```
ls
d        0 .
d        0 ..
-  7488975 01.csv
-   738504 02.csv
-      702 record.csv
rm record.csv
ls
d        0 .
d        0 ..
-  7488975 01.csv
-   738504 02.csv
```

## mv

文件或者目录重命名。

命令格式：`mv [src] [dst] move or rename FILE or DIR.`

将`record.csv`文件重命名为`02.csv`文件，举例如下：

```
ls
d        0 .
d        0 ..
-  7488975 01.csv
-   738504 record.csv
mv record.csv 02.csv
ls
d        0 .
d        0 ..
-  7488975 01.csv
-   738504 02.csv
```

EDP32记录数据时会将数据写入`record.csv`文件，默认为截断模式，即每次记录删除文件中的已有数据，
用户如果想保存已有记录数据，可已使用mv命令将`record.csv`文件修改为其它名字。

## cat

查看文件内容。

命令格式：`cat [file] Show File Contents.`

使用该命令查看任意名称记录文件内容，可以配合**超级终端**的`传送->捕获文字`功能，将记录文件导入到计算机。

**注**：`log dump`命令与`cat record.csv`命令是等效的。

## reboot

重启系统。

可以带一个延时参数，单位ms，如`reboot 900`延时900ms以后重启。

命令输出如下：

```
reboot
 rebooting...
 EDP32 v20.3.24 SN:6C5D31363735000141305741
 ECHO Studio <echo.xjtu@gmail.com>. All Rights Reserved.
reboot 900
 rebooting...
 EDP32 v20.3.24 SN:6C5D31363735000141305741
 ECHO Studio <echo.xjtu@gmail.com>. All Rights Reserved.

## help

获取在线帮助。

命令输出如下：

```
help
 getui -> get U I P R Info.
 clear -> clear power and time Info.
 log -> log [dump|auto|append|uart|file|int|flush] Operate data logs.
 param -> param [load|save|restore] Operate parameters.
 uiset -> uiset [adj|zero|cali|vdd] [dat] set Uin param.
 uoset -> uoset [adj|zero|cali|set] [dat] set Uout param.
 ioset -> ioset [adj|zero|cali|set] [dat] set Iout param.
 info -> info [func|dev|log|lcd|addr|baud|bklt] Display/Set system Info.
 ctrl -> ctrl [echo|test|led|buz|main|time] [param] Device Control.
 lfs -> lfs [umount|mount|format] Operate File System.
 ls -> ls list DIR or FILE.
 df -> df Show Disk Usage.
 rm -> rm [file|dir] remove FILE or DIR.
 mv -> mv [src] [dst] move or rename FILE or DIR.
 cat -> cat [file] Show File Contents.
 reboot -> reboot [delay ms] Restart system.
 help -> help Info.
 version -> display SW version and SN.
```

## version

获取固件和设备序列号等信息。

命令输出如下：

```
version
 EDP32 v20.3.24 SN:6C5D31363735000141305741
 ECHO Studio <echo.xjtu@gmail.com>. All Rights Reserved.
```
