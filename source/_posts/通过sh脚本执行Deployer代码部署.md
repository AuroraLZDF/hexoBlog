---
title: 通过sh脚本执行Deployer代码部署
date: 2020-04-03 16:32:16
categories: [Deployer]
tags: [Deployer]
---

# 场景
在上一篇[Deployer部署项目](/2020/04/02/Deployer部署项目/)中，我们编写单个 `Deployer` PHP脚本来实现无需登陆远程服务器，通过 `ssh` 方式将 `git` 仓库代码上传到远程服务器。
极大的方便了运维代码发布的操作和时间的节约。
当手中有多个项目需要代码维护时，虽然可以对不同项目添加不同的部署文件，通过 `dep deploy --file="test.php"` 来部署项目。但，这里希望实现如下需求：
- 运维每次执行一个命令，即可实现项目部署
- 支持通过传递参数，来区分不同项目
- 要能够发布指定分支的代码

通过一下脚本来实现

# sh脚本
```bash
#!/bin/bash
#  $1：项目名，如：erp|api|member|crm
#  $2：环境，如：ta|tb|tc
#  $3：分支，如：task111

confirm() {
sleep 1
read -p $'\x0a确认是否执行:(y/n)' v_q
if [[ "$v_q" == 'y' ]]; then
        echo -e "\033[32m开始执行 \033[0m"
        sleep 1
else
        echo -e "\033[32m退出执行 \033[0m"
        exit 0;
fi
}


if ! [ $# -eq 3 ];then
        echo -e "\033[31m 告警: 必须输入3个参数 \033[0m"
        echo -e "\033[32m 如: $0  项目名  环境  分支 \033[0m"
        echo -e "\033[32m 如: $0  erp    ta   master \033[0m"
        exit 0;
fi

if ! [ -d /home/www/log/"$2" ];then
        mkdir -p /home/www/log/"$2"
fi


start_time=`date '+%Y_%m_%d'_'%H:%M:%S'`
echo -e "\033[32m"开始时间: $start_time" \033[0m" >> /home/www/log/"$2"/"$1".log

case $1 in

www|erp|srt|api)
        confirm;
        dep deploy --file=/home/test/deployer/"$1".php --branch=task1111 -vvv | tee -a /home/test/log/"$2"/"$1".log
;;

*)
        echo -e "\033[32m输入错误\033[0m"
;;

esac
```
终端输入：
```bash
$ chmod +x deploy.sh    # 赋予脚本可执行权限
$ ./deploy.sh
告警: 必须输入3个参数 
 如: ./deploy.sh  项目名  环境  分支 
 如: ./deploy.sh  erp    ta   master
 $ ./deploy.sh erp ta task111
 ✈︎ Deploying task111 on 192.168.15.75
• done on [192.168.15.75]
➤ Executing task deploy:prepare
[192.168.15.75] > echo $0
[192.168.15.75] < ssh multiplexing initialization
[192.168.15.75] < bash
[192.168.15.75] > if [ ! -d /opt/molbase.inc/data_app/dev13/srt.molbase.org ]; then mkdir -p /opt/molbase.inc/data_app/dev13/srt.molbase.org; fi
[192.168.15.75] > if [ ! -L /opt/molbase.inc/data_app/dev13/srt.molbase.org/current ] && [ -d /opt/molbase.inc/data_app/dev13/srt.molbase.org/current ]; then echo 'true'; fi
[192.168.15.75] > cd /opt/molbase.inc/data_app/dev13/srt.molbase.org && if [ ! -d .dep ]; then mkdir .dep; fi
[192.168.15.75] > cd /opt/molbase.inc/data_app/dev13/srt.molbase.org && if [ ! -d releases ]; then mkdir releases; fi
[192.168.15.75] > cd /opt/molbase.inc/data_app/dev13/srt.molbase.org && if [ ! -d shared ]; then mkdir shared; fi
• done on [192.168.13.202]
...
```
这样，在每次维护完一个项目的 `Deployer` 文件，只要执行 `deploy.sh`脚本，带上要发布的项目（寻找对应项目的 `Deployer` 文件）、发布位置（具体位置，以 `Deployer` 文件中定义的变量 `deploy_path` 为准）、对应分支（存在的git分支）即可。
