# docker

这个很全，可以直接使用： [wsargent / docker-cheat-sheet](https://github.com/wsargent/docker-cheat-sheet/blob/master/zh-cn/README.md)

下面列一些个人常用的：
- `docker inspect <container-name>`
- `docker pull`
- `docker ps -a`
- `docker run -rm --name <container-name> -it -v <volume-name>:<in-container-path>:<host-path> <img>`: 交互式运行镜像
  - `-rm`: 容器退出即删除

## container lifecycle
![](https://miro.medium.com/max/2258/1*vca4e-SjpzSL5H401p4LCg.png)

# k8s

简要版：
- 配置：`kubectl apply`
- 列表：`kubectl get <resource>`
- 详情：`kubectl descibe <resource> <resource-name>`
- 删除：`kubectl delete -f resource.yml` / `kubectl delete deployment nginx`

主要命令备忘如下：

- 根据文件配置资源：`kubectl apply -f resource.yml`
- 查看 pods 列表： `kubectl get pods`
- 查看 pod 详情：`kubectl describe pods <pod-name>`
- 查看 deployments 列表：`kubectl get deployments`
- 查看 deployments 详情：`kubectl describe deployment <deplyoment-name>`
- 端口转发：`kubectl port-forward service/nginx 8080:80`

官方的备忘单很完整： [kubectl 备忘单 | Kubernetes](https://kubernetes.io/zh/docs/reference/kubectl/cheatsheet/)

参考的应用 yaml

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
spec:
  selector:
    matchLabels:
      app: nginx
  replicas: 2 # tells deployment to run 2 pods matching the template
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.14.2
        ports:
        - containerPort: 80
---
apiVersion: v1
kind: Service
metadata:
  name: nginx
  labels:
    app: nginx
spec:
  ports:
  - port: 80
    protocol: TCP
  selector:
    app: nginx
```

# gcc

- `-o` 设置目标文件名
- `-I` 设置需要引入的头文件夹
- `-L` 设置需要引入的库文件夹
- `-Wall` 打印所有 `warning` 信息
- `-S` 仅输出汇编代码：`gcc -S main.c > main.s`

例如，编译 liburing 的 sample：
```shell
gcc webserver_liburing.c -L/lib/liburing/src -L/lib/liburing/ -I/lib/liburing/src/include/ -o webserver_liburing
```

ref: [https://man7.org/linux/man-pages/man1/gcc.1.html](https://man7.org/linux/man-pages/man1/gcc.1.html)

# Maven
- `mvn clean package`
- `mvn clean deploy`
- `mvn dependency: tree`

# Linux

- `top`
  - -H 查看线程
- `ll --block-size m`
- `grep`
  - -i 忽略大小写
  - -o 仅输出匹配部分
  - -e 正则匹配
  - -v 仅输出不匹配部分
- `netstat -apno`
  - -a all
  - -p 打印关联的进程
  - -n 显示 ip 而非域名
  - -o 显示 timer 列，显示长短连接和长连接保活时间
- `ps -eLf`
  - -e 打印所有进程
  - -L 打印线程信息
  - -f 打印所有可用信息
- `awk`
- `tee <file>` 同时到标准输出和文件
- `curl -X POST www.qq.com -d '{"param": "1234"}' -H 'content-type: application/json'`

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

- `mysql -h <host> -P <port> -u <root> -p` 之后提示输入密码
- `show databases;`
- `show tables;`
- `desc <table_name>;`

# Redis CLI

- `redis-cli -h <host> -p <port> -a <password> -n <database>`
- `ttl`

# Mongo CLI

- `mongo 127.0.0.1:12345/db -u <username> -p <password>`
- `use <db>`
- `db.collection.find({})`

# iTerm

`cmd + shift + i`: 复制输出到所有 console，再次输入取消

# keymaps

对于 `ctrl+left` 这类快捷键，使用系统级改键的方式实现而不是修改每一个软件的快捷键是出于如下考虑：这是一个系统级的操作，去逐一修改软件是不现实的，大多软件以此为默认操作，并且不提供修改方式。

在 mac 上，使用 `command` 替代 `ctrl`；使用 `option` 替代 `alt`。

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
| ctrl + shift + r (refactor) | 重构 - 重命名（修改 函数、变量、类、文件 名） |  |
| ctrl + p | 参数提示 |  |
| alt + enter | context action |  |
| ctrl + space
| intellisense |  |
| ctrl + [ | 操作记录中的前一个光标位置 |  |
| ctrl + ] | 操作记录中的后一个光标位置 |  |
| ctrl + ⬅️ | 光标到前一个单词 | 再增加 shift 键实现选中功能。macOS 平台默认为 `option+left`，需要通过改键映射 |
| ctrl + ➡️ | 光标到后一个单词 | 同上 |
| ctrl + delete | 删除前一个单词 | macOS 平台默认为 `option+delete` ，需要通过改键映射 |
| ctrl + button 1 | 跳转到「定义」，Definition |  |
| ctrl + shift + button 1 | 跳转到「实现」，Implementation |  |
| 窗口操作 |  |  |
| ctrl + ` | Open terminal pannel |  |
| ctrl + \ | 垂直分屏 |  |
| ctrl + shift + \ | 水平分屏 |  |
| ctrl + alt + 上下左右 | 焦点在 tab 页中（上下左右）移动 |  |
| ctrl + e | 最近使用文件 |  |
| double shift | Search Everywhere | IDEA |
| 调试运行 |  |  |
| F8 | Continue Debug. |  |
| F10 | Step over. |  |
| F11 | Step into. |  |
| Shift + F11 | Stop out. |  |
