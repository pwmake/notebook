---
layout: post
title: Ubuntu 开启历史命令记录
date: 2018-06-27
tags: [Linux]
feature-img: https://images.pexels.com/photos/430208/pexels-photo-430208.jpeg?cs=srgb&dl=camera-equipment-pavement-430208.jpg&fm=jpg
---

之前有介绍过的，但那种需要重新编译 `bash` 来实现，略为麻烦。现在的这种方法，借助 `rsyslog` 来实现，只需要添加几个脚本即可。

<!--more-->

## rsyslog设定

添加一个新的 `bash` 配置到 `rsyslog` ，文件路径为：`/etc/rsyslog.d/bash.conf`

```shell
local6.*  /var/log/commands.log
```

## 设置登录时加载

>  bash 必须是4.1或以上的版本

通过 **bash** 的 `prompt_command` 变量来实现记录 历史命令 日志，新建文件：`/etc/profile.d/prompt_command.sh`

```shell
# 获取 NAME_OF_KEY 变量 与 CLIENT_IP 变量
test -f /etc/bash_tt && . /etc/bash_tt

# 使用logger将命令记录到上述指定的日志
export PROMPT_COMMAND='\
if [ ! -z "${LAST_CMD}" ] && [ "$(history 1)" != "${LAST_CMD}" ];then
logger -p local6.debug "PPID=$$ USER=$(whoami) IP=${CLIENT_IP} KEY=${NAME_OF_KEY} CMD=[$(history 1 | sed "s/^[ ]*[0-9]\+[ ]*//" )] ";
fi;
export LAST_CMD="$(history 1)";'
```

## 两个变量的获取

脚本 `/etc/bash_tt` 的实际内容如下

```shell
#! /bin/bash

authorized_keys="$HOME/.ssh/authorized_keys"
ssh_key_log="${HOME}/.${USER}_key.log"
ssh_key_tmp="${HOME}/.${USER}_key.tmp"

# check authorized_keys file and finger_print file
if [ ! -f ${authorized_keys} ]
then
   touch ${authorized_keys}
fi

test -f ${ssh_key_log} || touch ${ssh_key_log}

cat ${authorized_keys} | while read LINE
do
    if [ -z "${LINE}" ] || [[ ${LINE} == '#'* ]]
    then
        continue
    fi
    NAME=$(echo $LINE | awk '{print $3}') # jeff@zshmobi
    echo $LINE > ${ssh_key_tmp}
    KEY=$(ssh-keygen -l -f ${ssh_key_tmp} | awk '{print $2}')
    /bin/grep -q "$KEY $NAME" ${ssh_key_log} || echo "$KEY $NAME" >> ${ssh_key_log}
done

if [ $UID == 0 ]
then
    ppid=${PPID}
else
    ppid=$(ps -ef | grep ${PPID} |grep 'sshd:' |awk '{print $3}')
fi

log_line=$(sudo /bin/grep 'Accepted publickey for' /var/log/auth.log|/bin/grep "${ppid}")
RSA_KEY=$(echo ${log_line}|awk '{print $NF}'|tail -1)
CLIENT_IP=$(echo ${log_line}|awk '{print $11}'| tail -1)

# when someone change his role to another one
if [[ ${RSA_KEY} == "" ]]
then
    old_user=$(ps -ef |grep ${ppid}| grep -v grep|awk '{print $1,$2}'| grep ${ppid}|cut -d ' ' -f 1)
    pp_pid=$(ps -ef | grep ${ppid} |grep -v grep |awk '{print $3}'| grep -v ${ppid})
    ppp_pid=$(ps -ef | grep ${pp_pid} |grep -v grep |awk '{print $3}'| grep -v ${pp_pid})
    pppp_pid=$(ps -ef | grep ${ppp_pid} |grep -v grep |awk '{print $3}'| grep -v ${ppp_pid})
    log_line=$(sudo /bin/grep 'Accepted publickey for' /var/log/auth.log|/bin/grep "${pppp_pid}")
    RSA_KEY=$(echo ${log_line}|awk '{print $NF}'|tail -1)
    CLIENT_IP=$(echo ${log_line}|awk '{print $11}'| tail -1)

    if [[ ${old_user}} != ${USER} ]]
    then
        ssh_key_log="$(grep ${old_user} /etc/passwd| cut -d ':' -f 6)/.${old_user}_key.log"
    fi
    NAME_OF_KEY=$(grep "$RSA_KEY" ${ssh_key_log}|awk '{print $NF}')
else
    NAME_OF_KEY=$(grep "$RSA_KEY" ${ssh_key_log}|awk '{print $NF}')
fi


#把NAME_OF_KEY设置为只读
readonly NAME_OF_KEY
export NAME_OF_KEY

# 客户端的连接IP
readonly CLIENT_IP
export CLIENT_IP
```
`ubuntu` 与 `centos` 的会有小小的区别。

现在 `tail -f /var/log/command.log` 试试，实时查看命令历史吧。实际输出如下

```shell
Jun 26 14:24:48 myserver jeff: PID=29009 USER=jeff IP=11.22.33.44 KEY=jeff@test CMD=[date]
Jun 26 14:27:32 myserver jeff: PID=29009 USER=jeff IP=11.22.33.44 KEY=jeff@test CMD=[echo 'This is a test']
```

## 小结

既已经实现之，说说存在问题吧。

再者是关于日志文件的安全性，例如有人改动，或者直接删除之，那就麻烦了。可以考虑使用 `audit` 审计作日志的管理，或者其他方式实时同步历史命令日志。此处暂时不表之。
