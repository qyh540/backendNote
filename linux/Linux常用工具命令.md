# 常用 Linux 命令

好用的命令可以帮助我们更好的工作，工作顺了心情也会变好。

## 1. <code>tldr </code>

tldr 是一个帮助文档命令，Linux 自带帮助文档以便于你使用Linux命令，你可以使用 <code>man command</code> 或 <code>command --help</code> 查看这些文档（按 q 退出），但是这些文档过于繁琐，不利于阅读。tldr 文档简单明了，让你很容易上手这些命令。

tldr 意思就是 Too Long Don't Read，可以方便记忆。

使用方法

```shell
tldr command
```

## 2. <code>ripgrep/rg</code>

rg 是一款强大的文本搜索工具，支持正则表达式。他是按行查找的。

(N.B. Various snaps for ripgrep on Ubuntu are also available, but none of them seem to work right and generate a number of very strange bug reports that I don't know how to fix and don't have the time to fix. Therefore, it is no longer a recommended installation option.)

### 基本用法

```shell
# 在文件中查找关键词
# 不指定文件，默认是当前文件夹的所有文件
rg word fileName

# 在 README.md 中查找后面跟着 一个或多个 字母的 fast
# \w 表示单词，+ 表示一个或多个
rg "fast\w+" README.md
```

e.g.

```shell
❯ rg let demo
36:" Enable filetype plugins
37:filetype plugin on
38:filetype indent on
46:let mapleader = ","
66:let $LANG='en'
#==========================================================
❯ rg 'let\w+' demo
36:" Enable filetype plugins
37:filetype plugin on       
38:filetype indent on
```

```shell
# ( 在正则表达式中有特殊作用，需用 \( 转义
# 或者 用 -F
rg 'fn write\(' README.md
rg -F 'fn write('
```

e.g.

```shell
❯ rg 'if has\(' demo
76:if has("win16") || has("win32")
125:if has("gui_macvim")
153:if has("gui_running")
285:if has("mac") || has("macunix")
301:if has("autocmd")
#=========================================================
❯ rg -F 'if has(' demo
76:if has("win16") || has("win32")
125:if has("gui_macvim")
153:if has("gui_running")
285:if has("mac") || has("macunix")
301:if has("autocmd")
```

可以使用正则表达式搜索关键词

```shell
❯ rg 'if has??*??' demo
76:if has("win16") || has("win32")
125:if has("gui_macvim") 
153:if has("gui_running")
285:if has("mac") || has("macunix")
301:if has("autocmd")
```

## 3.<code>awk</code> 

### 基本用法

```shell
awk '{print}' file.txt
# 对 file.txt 中的每一行执行 print 命令

awk -f script.awk hello.txt
# 也可以使用脚本文件，只需用 -f

<script.awk>
{print}
```

### 常用变量

<code>\$n</code> ：表示记录的字段。<code>\$0</code> 比较特殊，代表当前整行。

<code>FS</code> ：字段分割符。

<code>NF</code> ：当前记录中的字段数量。

<code>NR</code> ：当前记录的编号。相当于行号

使用 <code>-F</code> 指定分割符

```shell
awk -F "," '{print $3,$4}' file.txt
```

也可以在输出时插入字符

```shell
awk -F "," '{print "phone"$3,"address"$4}' file.txt
```

如果使用脚本文件，必须使用 <code>FS</code> 指定分隔符。

```shell
<script.awk>
BEGIN {
FS=","
}

{{print "phone"$3,"address"$4}}
```

分割符没有限制为单一字符，也可以是正则表达式。

<code>NF</code> 变量，也叫做“字段数量”变量。awk 会自动将该变量设置成当前记录中的字段数量。可以使用NF变量来只显示某些输入行

```shell
awk 'NF==3 {print $1,$2,$3}' file.txt
```

<code>NR</code> 输出指定行

```shell
awk '(NR < 10)||(NR > 100) {print}'
```

### 高阶

使用正则表达式，输出包含指定字符串的行

```shell
awk '/[0-9]*/ {print}' file.txt
# 输出含有数字的行
```

// TODO 未完



## 4. <code>sed</code> 

### 基本语法

```shell
# 命令格式
sed [option] 'sed command' filename

# 脚本格式
sed [option] -f 'sed script' filename
```

### option

```shell
-n 	# 只打印模式匹配的行
-e	# 默认选项，直接在 sed 命令行进行编辑
-f	# 使用脚本
-r	# 支持扩展表达式
-i	# 直接修改文件内容
```

### 选定范围

```shell
# 打印第二行，会打印两遍，因为 sed 默认打印全部文本
sed '2p' filename

#  打印 1 - 3 行
sed '1,3p' filename

# 打印匹配的行
sed -n '/second/p' filename

# 打印包含两个模式的行，其中之一满足就可打印
sed -n '/data/,/last/p' filename

# 打印 从匹配second到第八行
sed -n '/second/,8p' filename

# 同理
sed -n '3,/last/p' filename

# 使用编辑命令，使用 {} 包含命令，使用 ; 分隔命令
# 
sed -n '1,4{=;p}' filename

# 使用 ! 对前面的匹配模式取反
# 不打印 1 - 2 行
sed -n '1,2!{=;p}'
```

### 正则表达式

匹配模式可以使用正则表达式，略。

扩展正则表达式使用 <code>-r</code> 。

| \<          | 锚点词首----相当于 \b，用法格式：\<pattern  |
| ----------- | ------------------------------------------- |
| \>          | 锚点词尾，用法格式:\>pattern                |
| \<pattern\> | 单词锚点                                    |
|             | 分组，用法格式：pattern，引用\1,\2          |
| []          | 匹配指定范围内的任意单个字符                |
| [^]         | 匹配指定范围外的任意单个字符                |
| [:digit:]   | 所有数字, 相当于0-9， [0-9]---> [[:digit:]] |
| [:lower:]   | 所有的小写字母                              |
| [:upper:]   | 所有的大写字母                              |
| [:alpha:]   | 所有的字母                                  |
| [:alnum:]   | 相当于0-9a-zA-Z                             |
| [:space:]   | 空白字符                                    |
| [:punct:]   | 所有标点符号                                |

### sed 的编辑命令

| 命令                    | 作用                                                         |
| ----------------------- | :----------------------------------------------------------- |
| <code>p</code>          | 打印匹配行（和-n选项一起合用）                               |
| <code>=</code>          | 显示文件行号                                                 |
| <code>a\\</code>        | 在定位行号后附加新文本信息                                   |
| <code>i\\</code>        | 在定位行号后插入新文本信息                                   |
| <code>d</code>          | 删除定位行                                                   |
| <code>c\\</code>        | 用新文本替换定位文本                                         |
| <code>w filename</code> | 写文本到一个文件，类似输出重定向 >                           |
| <code>r filename</code> | 从另一个文件中读文本，类似输入重定向 <                       |
| <code>s</code>          | 使用替换模式替换相应模式                                     |
| <code>q</code>          | 第一个模式匹配完成后退出或立即退出                           |
| <code>l</code>          | 显示与八进制ACSII代码等价的控制符                            |
| <code>{}</code>         | 在定位行执行的命令组，用分号隔开                             |
| <code>n</code>          | 从另一个文件中读文本下一行，并从下一条命令而不是第一条命令开始对其的处理 |
| <code>N</code>          | 在数据流中添加下一行以创建用于处理的多行组                   |
| <code>g</code>          | 将模式2粘贴到/pattern n/                                     |
| <code>y</code>          | 传送字符，替换单个字符                                       |

### 查询

```shell
# 打印非 # 开头的行
sed -n '/^#/!p' filename
# 过滤掉以 # 开头的行和空行
sed -n '/^#/!{/^$/!p}' filename
# 删除以 # 开头的行和空行
sed -e '/^#/d' -e '/^$/d' filename
```

### 添加

```shell
# 匹配含有 world 的行，在行首添加 Li
sed -n '/world/s/^/Li' filename
# 匹配含有 Linux 的行，在 Linux 前添加 Li
sed -n 's/Linux/Li $/' filename
# 在 Linux 之后添加 Li
sed -n 's/Linux/& Li' filename
# 在行位添加 Li
sed -n '/you/s/$/ Li/'
# 在匹配行前一行插入指定内容
sed -n '/are/i hello' filename # 添加 hello
sed -n '/are/i\hello' filename # 添加 hello
sed -n '/are/i/hello' filename # 添加 /hello
# 在匹配行的下一行插入指定内容
# 可以使用转义字符 \n 添加多行
sed -n '/are/a hello' filename # 添加 hello
sed -n '/are/a\hello' filename # 添加 hello
sed -n '/are/a/hello' filename # 添加 /hello
# TODO 未完
```







