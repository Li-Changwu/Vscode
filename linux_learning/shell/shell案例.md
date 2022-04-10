# shell案例

## 1.归档文件

案例描述：实现一个每天对指定目录归档备份的脚本，输入一个目录名称（末尾不带/），将目录下所有文件按天归档保存，并将归档日期附加在归档文件名上，放在/root/archive 下。
tips：
这里用到了归档命令：tar，后面可以加上-c 选项表示归档，加上-z 选项表示同时进行压缩，得到的文件后缀名为.tar.gz。

案例实现：

1. 编写脚本

    ```shell
    #!/bin/bash

    # 首先判断参数个数是否正确
    if [ $# -ne 1 ]
    then
    echo "参数个数错误，请输入一个参数表示归档目录名"
    exit
    fi

    # 获取目录名称
    if [ -d $1 ]
    then
    echo
    else
    echo "目录不存在"
    exit
    fi

    # 获取目录名和绝对路径
    DIR_NAME=$(basename $1)
    DIR_PATH=$(cd $(dirname $1); pwd)

    # 获取当前日期
    DATA=$(date +%y%m%d)

    # 定义归档文件名称
    FILE=archieve_${DIR_NAME}_${DATA}.tar.gz
    DEST=/root/archieve/$FILE

    # 开始归档文件

    tar -czf $DEST $DIR_PATH/$DIR_NAME

    if [ $? -eq 0 ]
    then
    echo
    echo "归档成功"
    echo "归档文件为:$DEST"
    else
    echo "归档失败"
    fi
    exit
    ```

2. 设置定时任务

    ```shell
    # 编写定时任务，每天2：00执行脚本
    [root@hadoop100 scripts]# crontab -e
    # 查看定时任务
    [root@hadoop100 scripts]# crontab -l
    0 2 * * * /root/scripts/archieve.sh /root/scripts
    ```

## 2.发送消息

案例描述：
案例实现：
