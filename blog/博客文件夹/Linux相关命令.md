# Linux相关命令

1. 查看 内存、CPU占用情况命令：
   * ps -eo pid,user,%mem,%cpu,comm --sort=-%mem | head
   * free -m
   * ps aux --sort -rss | head
   * du -ah --max-depth=1   // 查看根目录下各个文件占用情况，找到占用空间较大的文件
2. 查看jvm运行情况
   * jmap -histo:live 95786 | head -n 20
3. 查看linux内存占用详情
   * dmesg
4. 如何定位最大文件目录
   输入命令： cd / 进入根目录
   输入命令：du -h --max-depth=1 寻找当前目录，哪个文件夹占用空间最大
   如何定位最大文件
   输入命令：ls –lhS 将文件以从大到小顺序展现
