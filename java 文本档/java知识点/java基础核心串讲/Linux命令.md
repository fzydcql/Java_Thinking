### Linux初级指令

#### ls-------List  全称list

ls 参数格式

ls [option]... [file]...

ls命令参数

-a 列出指定目录下的所有文件，包括隐藏文件

-c 使用最后一次更改文件状态以进行排序（-t）或长时间打印（-l）的时间

-h 与-l选项一起使用时，请使用单位后缀：Byte，kilobyte，mete，gb，tb和petabyte，以便使用以2为基数的大小将数字减少到3或更少

-l 长格式列表。如果输出到终端，则所有文件大小的总和将输出到长清单前面的一行中

-n 以数字形式显示用户和组id，而不是在长（-l）输出中转换为用户或组名。这个选项默认打开-l选项

-o 以长格式列出，但省略组id

-s 显示每个文件实际使用的文件系统块的数量，以512字节为单位，其中部分单元四舍五入为下一个整数值

-t 在按照字典顺序对操作数排序之前，先按修改的时间排序（最近修改的是first）

-u 使用最后一次访问的时间，而不是最后一次修改文件进行排序

![image-20200421091007784](C:\Users\HITO_JAVA5\AppData\Roaming\Typora\typora-user-images\image-20200421091007784.png)

#### pwd介绍

打印当前工作目录的完整路径名。

参数格式

pwd 用法展示

