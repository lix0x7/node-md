# docker
- `docker inspect <container-name>`
- `docker pull`
- `docker ps`



# k8s


# gcc


- `-o`设置目标文件名
- `-I`设置需要引入的头文件夹
- `-L`设置需要引入的库文件夹
- `-Wall`打印所有 `warning`信息
- `-S`仅输出汇编代码：`gcc -S main.c > main.s`



例如，编译liburing的sample：
```shell
gcc webserver_liburing.c -L/lib/liburing/src -L/lib/liburing/ -I/lib/liburing/src/include/ -o webserver_liburing
```


ref: [https://man7.org/linux/man-pages/man1/gcc.1.html](https://man7.org/linux/man-pages/man1/gcc.1.html)


# Maven

- `mvn clean package`
- `mvn clean deploy`



# Linux


- `top`
   - -H 查看线程
- `ll --block-size m`
- `grep`
   - -i 忽略大小写
   - -o 仅输出匹配部分
   - -e 正则匹配
- `netstat -apno`
   - -a all
   - -p 打印关联的进程
   - -n 显示ip而非域名
   - -o 显示timer列，显示长短连接和长连接保活时间
- `ps -eLf`
   - -e 打印所有进程
   - -L 打印线程信息
   - -f 打印所有可用信息



## apt

- `apt update`
- `apt install -y`
- `apt remove <PackageName>`: remove bin files
- `apt purge <PackageName>`: remove bin and data files
- `apt autoremove`: remove useless dependencies



### dpkg

- 安装 `dpkg -i *.deb`
- 卸载 `dpkg -r`
- 列表 `dpkg -l`
- [https://man7.org/linux/man-pages/man1/dpkg.1.html](https://man7.org/linux/man-pages/man1/dpkg.1.html)



# MySQL

- `mysql -h <host> -P <port> -u <root> -p`之后提示输入密码
- `show databases;`
- `show tables;`
- `desc <table_name>;`



# Redis CLI

- `redis-cli -h host -p port -a password -n database`
- `ttl`



# keymaps


On macOS, use the `command`as `ctrl`and the`option`as `alt`.


对于 `ctrl+left`这类快捷键，使用系统级改键的方式实现而不是修改每一个软件的快捷键是出于如下考虑：这是一个系统级的操作，去逐一修改软件是不现实的，大多软件以此为默认操作，并且不提供修改方式。



| 编辑操作 |  |  |
| --- | --- | --- |
| ctrl + d | 删除当前行 | 备注 |
| ctrl + q / cmd + 1 | 选中下一个匹配区域 | cmd+1 是为了避免 macOS 上的 cmd+q 冲突 |
| ctrl + c | 复制当前行（未选中状态） |  |
| ctrl + x | 剪切当前行（未选中状态） |  |
| TODO | 格式化选中块 |  |
| alt + down | 当前行下移 |  |
| alt + up | 当前行上移 |  |
| shift + enter | 下一行插入新行 |  |
| ctrl + / | 注释、取消注释 |  |
| ctrl + r | 重构 |  |
| ctrl + shift + f | 在目录中查找 |  |
| ctrl + shif + r (refactor) | 重构 - 重命名（修改 函数、变量、类、文件 名） |  |
| ctrl + p | 参数提示 |  |
| alt + enter | context action |  |
| ctrl + space
 | intellisense |  |
| ctrl + [ | 操作记录中的前一个光标位置 |  |
| crtl + ] | 操作记录中的后一个光标位置 |  |
| ctrl + ⬅️ | 光标到前一个单词 | 再增加shift键实现选中功能。macOS平台默认为 `option+left`，需要通过改键映射 |
| ctrl + ➡️ | 光标到后一个单词 | 同上 |
| ctrl + delete | 删除前一个单词 | macOS平台默认为 `option+delete` ，需要通过改键映射 |
| ctrl + button 1 | 跳转到「定义」，Definition |  |
| ctrl + shift + button 1 | 跳转到「实现」，Implementation |  |
| 窗口操作 |  |  |
| ctrl + ` | Open terminal pannel |  |
| ctrl + \ | 垂直分屏 |  |
| ctrl + shift + \ | 水平分屏 |  |
| ctrl + alt + 上下左右 | 焦点在tab页中（上下左右）移动 |  |
| ctrl + e | 最近使用文件 |  |
| double shift | Search Everywhere | IDEA |
| 调试运行 |  |  |
| F8 | Continue Debug. |  |
| F10 | Step over. |  |
| F11 | Step into. |  |
| Shift + F11 | Stop out. |  |



