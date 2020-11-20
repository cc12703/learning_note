

# MachO格式



## 基本介绍
* MachO是IOS/MacOS下的可执行文件格式

### 包含部分
* 文件头
* 加载命令
* 文本段
* 数据段
* 动态库加载信息
* 入口函数
* 符号表
* 动态库符号表
* 字符串表


## 文件格式

| 整体结构 |
| -- |
|  Header （头信息）  |
|  Load Commands （加载描述） | 
|  Data (数据)  |

|  加载描述 |
| -- |
| Segment 1 Command (段1描述) |
| Segment 2 Command (段2描述) |
| Segment 3 Command (段3描述) |


| 数据 |
| -- |
| Segment 1 Data (段1数据) |
| --->   Section 1 Data (区1数据) |
| --->   Section 2 Data (区2数据) |
| --->   Section 3 Data (区3数据) |
| Segment 2 Data (段2数据) |
| --->   Section 4 Data (区4数据) |
| --->   Section 5 Data (区5数据) |
| Segment 3 Data (段3数据) |
| --->   Section 6 Data (区6数据) |
| --->   Section 7 Data (区7数据) |
| --->   Section 8 Data (区8数据) |


## 数据结构

### 头信息

| 字段    |    含义| 
| :-------- | --------:|
| magic | 标识符 |
| cpu-type | CPU类型 | 
| cpu-subtype | CPU子类型 |
| file-type | 文件类型|
| ncmds | LoadCommands的个数 |
| size-of-cmds | LoadCommands的长度 | 
| flags | 标识符 | 


#### 文件类型
* MH_OBJECT    编译过程中生成的 *.obj文件
* MH_EXECUTABLE  可执行二进制文件
* MH_CORE   CoreDump文件
* MH_DYLIB  动态库
* MH_DYLINKER  连接器

### 段描述

| 字段    |    含义| 
| :-------- | --------:|
| cmd   |  类型 |
| cmd-size | 大小 （随着类型不同而不同）| 

#### 段类型
* LC_SEGMENT  文件中的段映射到进程地址空间中
* LC_SYMTAB    符号表
* LC_DYSYMTAB  动态符号表
* LC_LOAD_DYLINKER  使用的动态加载库
* LC_UUID   文件的唯一标识
* LC_MAIN   程序的入口地址


### 段数据

#### 要点
* 段内会按不同的功能划分为几个区（section）

#### 类型
* __PAGEZERO 空指针陷阱，用于捕获对null的引用
* __TEXT 包含了执行代码和只读数据
* __DATA  包含了程序数据
* __OBJC  objective-c的运行时支持库

#### segment_command
| 字段    |    含义| 
| :-------- | --------:|
| cmd | 类型值 | 
| cmd-size | 大小 |
| seg-name | 名字 |
| vm-addr | 虚拟内存空间的开始地址 |
| vm-size | 占用内存空间的大小 |
| file-off | 数据所在的文件偏移量 |
| file-size | 占用文件的大小 | 
| max-prot | 虚拟地址保护相关 | 
| init-prot |虚拟地址保护相关 |
| nsects | 区个数 |
| flags | 标识符 | 

**大小等于 sizeof(segment_command) + (sizeof(section) * segment->nsect)**

#### section
| 字段    |    含义| 
| :-------- | --------:|
| sect-name | 区名字 |
| seg-name |  段名字 | 
| addr | 虚拟内存空间的开始地址 |
| size | 占用内存空间的大小 |
| offset | 该段在文件中的偏移量 | 
| align | 对齐值 | 
| rel-off | 第一个重定向项的文件偏移量 | 
| n-reloc | 重定向项的数量 | 
| flags |  标识
