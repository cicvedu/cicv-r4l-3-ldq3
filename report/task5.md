字符设备的注册通过分配或指定主、次设备号来实现

当创建一个字符设备时，一般会在/dev目录下生成一个设备文件，Linux用户层的程序就可以通过这个设备文件来操作这个字符设备

字符设备驱动所做的工作：
- 申请、释放设备号
- 填充file_operations结构体中的功能函数，比如open()、read()、write()、close()等
- 添加、初始化、删除cdev结构体