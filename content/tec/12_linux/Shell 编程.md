# Ref


- [http://www.runoob.com/linux/linux-shell-variable.html](http://www.runoob.com/linux/linux-shell-variable.html)



# 变量

- 使用$访问变量，但是赋值不需要该符号，且**和等号间不需要空格**，例如下例：
```bash
name=lix7
# 上面的语句不能加空格写成 name = lix7，此时 shell 会将其识别成调用 name 命令，以 `=` 和 `lix7` 作为参数
echo ${name}
```
大括号可加可不加，大括号的是为了帮助shell识别变量的边界。


- 使用unset删除变量，删除后不能再使用。
```bash
name=lix7
unset name
```


- 使用readonly设置变量为只读变量
```bash
name=lix7
readonly name
```


# shell字符串


shell中既可以使用单引号又可以使用双引号来声明字符串，但是有所区别。


- 单引号
**单引号中不能使用变量**；两个单引号中间不能包含单独的单引号（即使使用了转义字符），可以出现成对的单引号，用于拼接字符串。
- 双引号
**双引号中可以使用环境变量**，可以使用转义字符。
```bash
your_name="runoob"
# 使用双引号拼接
greeting="hello, "$your_name" !"
greeting_1="hello, ${your_name} !"
echo $greeting  $greeting_1
# 使用单引号拼接
greeting_2='hello, '$your_name' !'
greeting_3='hello, ${your_name} !'
echo $greeting_2  $greeting_3

### result ###
hello, runoob ! hello, runoob !
hello, runoob ! hello, ${your_name} !
```


## 获取字符串长度


```bash
string="abcd"
echo ${#string} #输出 4
```


## 提取字符串


```bash
string="runoob is a great site"
echo ${string:1:4} # 输出 unoo
```


# 数组


- 声明
shell中使用小括号定义数组，数组元素间使用空格分割，例如`array_name=(value0 value1 value2 value3)`。
- 赋值
`array_name[1]=value1`
- 读取
`valuen=${array_name[n]}`，或者使用`@`角标获取所有元素`echo ${array_name[@]}`
- 数组元素数量
`length=${#array_name[@]}`



# shell脚本参数变量


- 通过`$0`访问当前脚本的文件名。
- 通过`$1`访问脚本的第一个参数，之后的参数以此类推
- 通过`$#`获取参数个数
- 通过`$*`获取所有参数组成的一个字符串
- 通过`$@`获取参数组成的数组
- 通过 `$?` 获取上一个命令的返回值



```bash
echo "-- \$* 演示 ---"
for i in "$*"; do
	echo $i
done

echo "-- \$@ 演示 ---"
for i in "$@"; do
	echo $i
done
```


运行结果如下：


```
$ chmod +x test.sh 
$ ./test.sh 1 2 3
-- $* 演示 ---
1 2 3
-- $@ 演示 ---
1
2
3
```


# shell基本运算符


## 算术运算符


bash原生不支持算术运算符，使用expr指令可以得到类似效果。


注意下面用的是反引号，代表通过bash执行其中语句；而且加号左右的空格不能缺，这代表着expr后边跟随了三个参数：


```bash
val=`expr $a + $b`
echo "a + b : $val"
```


## 关系运算符


条件表达式要放在方括号之间，并且要有空格，例如: `[$a==$b]` 是错误的，必须写成 `[ $a == $b ]`，关系运算符只支持数字，不支持字符串，除非字符串的值是数字。


下表列出了常用的关系运算符，假定变量 a 为 10，变量 b 为 20：

| 运算符 | 说明 | 举例 |
| --- | --- | --- |
| -eq | 检测两个数是否相等，相等返回 true。 | [ $a -eq $b ] 返回 false。 |
| -ne | 检测两个数是否不相等，不相等返回 true。 | [ $a -ne $b ] 返回 true。 |
| -gt | 检测左边的数是否大于右边的，如果是，则返回 true。 | [ $a -gt $b ] 返回 false。 |
| -lt | 检测左边的数是否小于右边的，如果是，则返回 true。 | [ $a -lt $b ] 返回 true。 |
| -ge | 检测左边的数是否大于等于右边的，如果是，则返回 true。 | [ $a -ge $b ] 返回 false。 |
| -le | 检测左边的数是否小于等于右边的，如果是，则返回 true。 | [ $a -le $b ] 返回 true。 |
| == | 相等。用于比较两个数字，相同则返回 true。 | [ $a == $b ] 返回 false。 |
| != | 不相等。用于比较两个数字，不相同则返回 true。 | [ $a != $b ] 返回 true。 |



## 布尔运算符


下表列出了常用的布尔运算符，假定变量a为10，变量b为20：

| 运算符 | 说明 | 举例 |
| --- | --- | --- |
| ! | 非运算，表达式为 true 则返回 false，否则返回 true。 | [ ! false ] 返回 true。 |
| -o | 或运算，有一个表达式为 true 则返回 true。 | [ $a -lt 20 -o $b -gt 100 ] 返回 true。 |
| -a | 与运算，两个表达式都为 true 才返回 true。 | [ $a -lt 20 -a $b -gt 100 ] 返回 false。 |



## 逻辑运算符


以下介绍 Shell 的逻辑运算符，假定变量 a 为 10，变量 b 为 20:

| 运算符 | 说明 | 举例 |
| --- | --- | --- |
| && | 逻辑的 AND | [[ $a -lt 100 && $b -gt 100 ]] 返回 false |
| &#124;&#124; | 逻辑的 OR | [[ $a -lt 100 &#124;&#124; $b -gt 100 ]] 返回 true |



## 字符串运算符


下表列出了常用的字符串运算符，假定变量 a 为 "abc"，变量 b 为 "efg"：

| 运算符 | 说明 | 举例 |
| --- | --- | --- |
| = | 检测两个字符串是否相等，相等返回 true。 | [ $a = $b ] 返回 false。 |
| != | 检测两个字符串是否相等，不相等返回 true。 | [ $a != $b ] 返回 true。 |
| -z | 检测字符串长度是否为0，为0返回 true。 | [ -z $a ] 返回 false。 |
| -n | 检测字符串长度是否为0，不为0返回 true。 | [ -n "$a" ] 返回 true。 |
| str | 检测字符串是否为空，不为空返回 true。 | [ $a ] 返回 true。 |



# echo


# printf


# test


## 字符串测试
| 参数 | 说明 |
| --- | --- |
| = | 等于则为真 |
| != | 不相等则为真 |
| -z 字符串 | 字符串的长度为零则为真 |
| -n 字符串 | 字符串的长度不为零则为真 |



## 文件测试
| 参数 | 说明 |
| --- | --- |
| -e 文件名 | 如果文件存在则为真 |
| -r 文件名 | 如果文件存在且可读则为真 |
| -w 文件名 | 如果文件存在且可写则为真 |
| -x 文件名 | 如果文件存在且可执行则为真 |
| -s 文件名 | 如果文件存在且至少有一个字符则为真 |
| -d 文件名 | 如果文件存在且为目录则为真 |
| -f 文件名 | 如果文件存在且为普通文件则为真 |
| -c 文件名 | 如果文件存在且为字符型特殊文件则为真 |
| -b 文件名 | 如果文件存在且为块特殊文件则为真 |



# 控制流


## if


shell中的else从句不能为空，如果没有else就不要写，例如：


```bash
if condition
then
	command1 
	command2
	...
	commandN 
fi
```


此处fi就是if倒过来写，后文的case会有一致的规约。
完整的if-else_if-else如下：


```bash
if condition1
then
	command1
elif condition2 
then 
	command2
else
	commandN
fi
```


## for循环


for循环格式如下：


```bash
for var in item1 item2 ... itemN
do
	command1
	command2
	...
	commandN
done
```


例如，顺序输出数字：


```bash
for loop in 1 2 3 4 5
do
	echo "The value is: $loop"
done
```


## while循环


```bash
while condition
do
	command
done
```


## until循环


```bash
until condition
do
	command
done
```


## case


Shell case语句为多选择语句。可以用case语句匹配一个值与一个模式，如果匹配成功，执行相匹配的命令。case语句格式如下：


```bash
case 值 in
模式1)
	command1
	command2
	...
	commandN
	;;
模式2)
	command1
	command2
	...
	commandN
	;;
esac
```


注意shell与一般语言的case不同之处如下：


case工作方式如上所示。取值后面必须为单词`in`，每一模式必须以右括号结束。取值可以为变量或常数。匹配发现取值符合某一模式后，其间所有命令开始执行直至 `;;`。


取值将检测匹配的每一个模式。一旦模式匹配，则执行完匹配模式相应命令后不再继续其他模式。如果无一匹配模式，使用星号 `*` 捕获该值，再执行后面的命令。


case的实例如下：


```bash
echo '输入 1 到 4 之间的数字:'
echo '你输入的数字为:'
read aNum
case $aNum in
	1)  echo '你选择了 1'
	;;
	2)  echo '你选择了 2'
	;;
	3)  echo '你选择了 3'
	;;
	4)  echo '你选择了 4'
	;;
	*)  echo '你没有输入 1 到 4 之间的数字'
	;;
esac
```


## break/continue


# 输入输出重定向


重定向命令列表如下：

| 命令 | 说明 | 示例 | 解释 |
| --- | --- | --- | --- |
| command > file | 将输出重定向到 file。 | who > users | 重定向who命令的输出为users文件 |
| command < file | 将输入重定向到 file。 | wc -l < users | 重定向wc命令的输入为users文件 |
| command >> file | 将输出以追加的方式重定向到 file。 |  |  |
| n > file | 将文件描述符为 n 的文件重定向到 file。 |  |  |
| n >> file | 将文件描述符为 n 的文件以追加的方式重定向到 file。 |  |  |
| n >& m | 将输出文件 m 和 n 合并。 | command >> file 2>&1 | 将stdout和stderr合并后以追加写方式重定向到file |
| n <& m | 将输入文件 m 和 n 合并。 |  |  |
| << tag | 将开始标记 tag 和结束标记 tag 之间的内容作为输入。 | wc -l << EOF
 hello
 lix
 hello
 EOF | 将EOF中间的三行作为输入，输出结果为3。 |



需要注意的是文件描述符 0 通常是标准输入（STDIN），1 是标准输出（STDOUT），2 是标准错误输出（STDERR）。
