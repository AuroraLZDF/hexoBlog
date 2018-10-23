---
title: linux(deepin) 下mysql自动备份
date: 2017-10-18 16:02:11
tags: [Linux, MySQL]
toc: true
---

### 创建并编辑文件 /home/aurora/backup/backup_mysql_aurora.sh，命令：

```bash
	cd /home/aurora/backup
	vim backup_mysql_aurora.sh
```

添加内容

```bash
	#!/bin/bash
	db_user=root
	db_passwd=root
	db_name=aurora
	backup_dir=/home/aurora/backup/mysql_backup/
	#time="$(date + %Y%m%d%H%M%S)"
	time=$(date)

	mysqldump -u$db_user -p$db_passwd $db_name > "$backup_dir$db_name"_"$time".sql                     # 备份保存为sql文件
	#mysqldump -u$db_user -p$db_passwd $db_name | gzip > "$backup_dir$db_name"_"$time".sql.gz          # 备份并打包文件
	#mysqldump --defaults-extra-file=/etc/mysql/my.cnf $db_name >"$backup_dir$db_name"_"$time".sql     # 调用默认配置文件备份数据库文件（默认使用用户名、密码会报【warning】错误）

```

### 修改文件bakmysql属性，使其可执行

赋予执行权限

```bash
	chmod +x backup_mysql_aurora.sh
```

### 创建定时任务，定时备份

使用`crontab -e`命令编辑定时任务

```bash
	--------- crontab -e --------
	|		定时任务文件内容   	|
	-----------------------------

	# Edit this file to introduce tasks to be run by cron.
	# 
	# Each task to run has to be defined through a single line
	# indicating with different fields when the task will be run
	# and what command to run for the task
	# 
	# To define the time you can provide concrete values for
	# minute (m), hour (h), day of month (dom), month (mon),
	# and day of week (dow) or use '*' in these fields (for 'any').# 
	# Notice that tasks will be started based on the cron's system
	# daemon's notion of time and timezones.
	# 
	# Output of the crontab jobs (including errors) is sent through
	# email to the user the crontab file belongs to (unless redirected).
	# 
	# For example, you can run a backup of all your user accounts
	# at 5 a.m every week with:
	# 0 5 * * 1 tar -zcf /var/backups/home.tgz /home/
	# 
	# For more information see the manual pages of crontab(5) and cron(8)
	# 
	# m h  dom mon dow   command

	#一分钟执行一次
	*/1 * * * * /home/aurora/backup/backup_mysql_aurora.sh 2>&1

```

*查看结果如图*
<img src="/images/mysql_backup.png">