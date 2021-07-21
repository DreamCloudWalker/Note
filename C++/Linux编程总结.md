* 超级用户#，普通用户$
* 常用快捷键介绍：
    * ctrl+A    // 光标到最前
    * ctrl+E    // 光标到最后
    * ctrl+U    // 当光标在最后时清空
    * lsp_release -a    // 查看是Ubuntu还是centos
* 环境变量
    * 临时环境变量（关闭窗口消失）
    ```
    export AAAA=1234567
    echo $AAAA
    ```
    * 永久性环境变量
    ```
    vim /etc/profile 加入 export AAAA=1234567
    source profile 【让修改的生效】
    echo $AAAA
    ```
* Linux文件读取操作
    * cat file05.txt 【快速查看，只读的】
    * tac file05.txt 【倒序查看内容】
    * more file05.txt 【百分百查看 相当于分页 敲回车分页】
    * head -2 profile【查看两行 前面的】
    * head -2 profile【查看两行 前面的】
* vim快捷键
    * 常用
        * u 【撤销 恢复】
        * i 【输入 ，光标不动】
        * I 【输入，光标前面】
        * a 【输入 ，光标退后一个】
        * A 【输入 ，光标到行末尾】
        * s 【输入 ，光标删除所在字符】 
        * S 【输入 ，光标删除所在整行】
    * 跳转和删除
        * u 【撤销 恢复】
        * h 【左】
        * j 【下】
        * k 【上】
        * l 【右】
        * 5G 【跳转到多行】 【不常用】 
        * gg 【第一行】
        * G 【最后一行】
        * :set number 【显示行号】
        * $ 【行尾】
        * 0 【行首】
        * dw 【删除单词】
        * dd 【删除一行】
        * 3dd 【删除三行】
    * 复制粘贴
        * yy 【复制一行】
        * dd 【剪切一行】
        * p 【刚刚yy、dd复制/剪切的，粘贴到当前光标行】 
        * P 【刚刚yy，dd复制剪切的，粘贴到下一行】
    * 查找替换
        * /define【查找 define内容】
        * r + p 【把当前光标的字符替换成 p】
        * :s /printf/printxxx 【光标所在行，替换成printxxx】
        * 1,6s /printf/printxxx/g 【1 ~ 6行 替换成printxxx，默认从第一行开始】 
        * :%s /printf/printxxx/g 【整个代码，把所有的printf 替换 printxxx】
    * 分屏操作
        * :vsp 【左右两个屏幕】 
        * :sp 【上下两个屏幕】 
        * ctrl + ww 【切换屏幕】 
        * q 【退出当前屏幕】 
        * wqall 【退出全部屏幕】
* Shell语法
    * 弱类型
    ```
    #!bin/bash
    # 相当于弱类型的语法，注意:不能搞中文输入法 # 我才是注释
    #A=10 A=10 错误的写法
    A=10
    echo "Derry Success"
    echo A==$A
    
    echo 当前Shell脚本的名称是: $0 
    echo 参数一:$1
    echo 参数二:$2
    ```
    * if
    ```
    echo 当前Shell脚本的名称是: $0 
    echo 参数一:$1
    echo 参数二:$2
    
    echo "本次执行状态如下:" 
    if(($?));then
        echo "本次执行失败" 
    else
        echo "本次执行成功" 
    fi
    
    echo "外界传递了参数内容是:this\ is $*" 
    echo "外界传递了参数的数量: this\ is $#"
    ```
    * 循环
    ```
    # 使用 `seq 1 20` 作为我们的数据源，然后我们遍历这个数据源，学习循环操作
    for i in `seq 1 20` 
    do
        echo "遍历的数字是:$i" 
    done
    
    # 完成一个，类似于系统提供的功能
    [root@iZuf65b1ojlcudtv4gurq7Z NDK25]# expr 200 + 300 【内置的 做运算】 500
    
    
    #!/bin/bash
    # 做累加操作
    # 这里不要有空格
    a=0 
    for((f=0;f<=100;f++)) 
    do
        # 注意:加的时候，必须加空格，否则有问题 
        a=`expr $f + $a`
    done
    echo "最后累加1~100的值是：$a"
    ```
    * 打包压缩
    ```
    tar czf all.taz *.sh 【把所有的 sh文件，压缩成包 all.taz】
    
    #!/bin/bash
    # 查找当前目录下，所有的.sh文件，进行打包操作
    for i in `find . -name "*.sh"` 
    do
        tar -czf derryAll.tgz $i 
    done
    
    
    #!/bin/bash
    # 查找当前目录下，所有的.sh文件，进行打包操作 
    a=0
    for i in `find . -name "*.sh"`
    do
        a=`expr $a + 1`
        tar -czf derryAll+$a.tgz $i
    done
    ```
    * while循环
    ```
    #!/bin/bash
    i=0
    while((i<100))
    do
        i=`expr $i + 1`
        echo "遍历的值:$i" 
    done
    ```
    * 读文件
    ```
    #!/bin/bash
    # 读取文件里面的内容，然后打印出来
    # 得到当前的目录路径 
    echo $PWD
    echo `pwd`
    # --------------
    while read lineVarAA
    do
        echo $lineVarAA 
    done<`pwd`/file01.txt
    ```
    * if语句 语法详解
    ```
    #!/bin/bash
    NUM1=100
    NUM2=200
    if(($NUM1>$NUM2));then
        echo "OK"
    else
        echo "else"
    fi
    
    if(($NUM1<$NUM2));then
        echo "OK"
    fi
    ```
    * 算数运算符
    ```
    [root@iZuf65b1ojlcudtv4gurq7Z NDK25]# result=$(( 100 + 100 ))
    [root@iZuf65b1ojlcudtv4gurq7Z NDK25]# echo $result
    200
    [root@iZuf65b1ojlcudtv4gurq7Z NDK25]#
    [root@iZuf65b1ojlcudtv4gurq7Z NDK25]# result=`expr 88 + 14` [root@iZuf65b1ojlcudtv4gurq7Z NDK25]# echo $result
    102
    [root@iZuf65b1ojlcudtv4gurq7Z NDK25]#
    
    #!/bin/bash
    var1="abcde"
    var2="zzzzz"
    # 判断是否相等
    if [ $var1 = $var2 ]
    then
        echo "等于" 
    else
        echo "不等于" 
    fi
    ```
    * 重定向
    ```
    屏幕 < 内容 【右到左 重定向】
    cat 屏幕的内容 AAAAAAAAAAAAAAAAAAAAA
    cat 0< file01.txt 【0代表屏幕的标记 < 把内容重定向到 屏幕 所以显示】
    
    屏幕 > 内容 【左到右 重定向】
    echo CCCCCCCCCCCCCC > file02.txt
    ```
    * 函数
    ```
    #!/bin/bash
    # 函数的学习
    function test01() { 
        echo "我是一个函数"
    }
    test01 # 调用此函数
    
    # 函数里面是可以定义变量的，不要去考虑，堆 栈 弹栈，他就是脚本 
    function test02() {
        var1="Kevin"
        var2="Derry"
        echo $var1
        echo $var2
        echo "我是一个函数"
    }
    
    # 函数传递参数 重点 
    function test03() {
        echo "我是一个函数，参数是:`expr $1`"
    }
    
    test03 99999 # 内置传递参数
    
    # 函数传递参数 重点 
    function test04() {
        echo "我是一个函数，参数是:`expr $1`"
    }
    
    test04 $1 # 脚本配合 内置传递参数
    
    function test05() {
        echo "我是一个函数，参数是:`expr $1`"
    }
    
    test05 $1 # 脚本配合 内置传递参数
    ```