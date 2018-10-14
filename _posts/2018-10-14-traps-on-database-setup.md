---
layout: post
title:  "Traps on setting the database(MySQL & Redis)"
date:   2018-10-14 13:51:51 +0800
categories: Tricks
tags: Database MySQL Redis
description: Traps I met when set/get key-values when using MySQL and Redis. The tricks can be achieved from Google but I summarized a little bit here.
---

<script type="text/javascript" async src="//cdn.bootcss.com/mathjax/2.7.0/MathJax.js?config=TeX-AMS-MML_HTMLorMML"></script>
<script type="text/javascript" async src="https://cdnjs.cloudflare.com/ajax/libs/mathjax/2.7.1/MathJax.js?config=TeX-MML-AM_CHTML"></script>

# Foreword

Finally I have moved forward and had the first step on my new job. The most recent task is to design a **Task Scheduler** for assigning tasks optimally to the straddle carriers to reduce the total cost. The basic structure is to read TOS data and save it in the MySQL database and covert it to task data c++ struct in Redis. Then the **Task Scheduler** will read the task data and calculate a optimized assignment with genetic algorithms.

The structure sounds not that complicated but everything has its dark side. When I try to move forward I meet a lot of traps in the following parts:

* MySQL loses connection;
* Data can not be SET and GET in the correct format;
* I got confused in the usage of GAlib.

In this article I will focus on the first two bullets. For the third bullet it is worth of writting a new article to describe it.

# Problem Description

For the MySQL:
* Can not reset the password;
* Can not start and show the exception message:
```
ERROR 2002 (HY000): Can't connect to local MySQL server through socket /var/run/mysqld/mysqld.sock' (2)
```

For the Redis:
* When use **RPUSH** to push data in the Redis, can not use **GET** to read the data;
* Even use **SET** to push, can also not use **GET** to read;
* Can push and read in one session, but can not read the data in another session.

# Prerequisites

The environment is Ubuntu 16.04 with MySQL and Redis installed, simply use
```
sudo apt-get install mysql-server mysql-client
sudo apt-get install redis-server
```
In addition, in order to call redis in c++ , you also need **hiredis** installed.

# Solution

## Cannot Reset MySQL Password

When you install MySQL in the first time, you need to run
```
#root mysqladmin -u root password 'your_password'
```
to reset the user and password. But in my case I suffered
```
mysqladmin: connect to server at 'localhost' failed
error: 'Access denied for user 'root'@'localhost' (using password: NO)
```
To solve it, you need first stop the MySQL service and then start with safe mode
```
#root /etc/init.d/mysqld stop           \\ Stop MySQL service

#root mysqld_safe --skip-grant-tables &     \\ Start safe mode
```

Then you can login with root user without password
```
#root /usr/bin/mysql -u root -p
Enter password:                         \\ Here directly type enter to login
```

Now in the MySQL command you can reset the password
```
mysql> update user set password=password("your_password") where user='root' and host='localhost';
```
After changes you need one more command
```
mysql> flush privileges;
```

Now restart the service
```
#root /etc/init.d/mysqld start
```
and you can login with root user and the password you set.

[Reference](https://blog.csdn.net/mchdba/article/details/10630701)

## Can not Find Sock

Sometimes you may meet the exception massage as I mentioned in the second bullet. The first thing you should do is to check the MySQL config files
```
/etc/my.conf
/etc/mysql/my.conf
/usr/etc/my.conf
~/.my.conf
```
and find
```
socket = /var/run/mysqld/mysqld.sock
```
My problem is that everytime when I restart, the `mysqld.sock` will be lost. Hence, I change it with another directory and establish a soft link
```
#root ln -s /your_directory/mysql.sock /var/run/mysqld/mysqld.sock
```
then problem solved.

## Redis RPUSH and GET

In **hiredis**, if you would like to execute a redis command, you need write in the following style
```
redisReply *reply = (redisReply *)redisCommand(ctx, "RPUSH %s %s", KEY_NAME, VALUE_DATA);
```
When I push values in a key, I use
```
reply = (redisReply *)redisCommand(ctx, "GET %s", KEY_NAME);
```
to read it but got a exception message
```
WRONGTYPE Operation against a key holding the wrong kind of value
```
this type of error is because of the mistasks of the data type. If you use 
```
reply = (redisReply *)redisCommand(ctx, "TYPE %s", KEY_NAME);
```
to check the data type of the key, you may find that **RPUSH** will push a list but **GET** can only read string. To read a list, you may use
```
reply = (redisReply *)redisCommand(ctx, "LRANGE %s 0 -1", KEY_NAME);
cout << reply -> element[i] -> str << endl;
```
Actually here `reply` is a pointer to a struct `redisReply`. The API and usage of it can be achieved from the [reference](http://www.fpstop.com/redis/hiredis1/).

## Redis SET and GET

As I mentioned in the previous part, you can use **SET** to push a string and use **GET** to read it. But the traps I met is, I can't even push and read with the following format
```
redisReply *reply = (redisReply *)redisCommand(ctx, "SET KEY_NAME VALUE_NAME");

redisReply *reply = (redisReply *)redisCommand(ctx, "SET %s %s", KEY_NAME, VALUE_DATA);

reply = (redisReply *)redisCommand(ctx, "GET %s", KEY_NAME);
```
All formats above may cause problems when you push and read. After lots of trials, I found the flawless format would be
```
redisReply *reply = (redisReply *)redisCommand(ctx, "RPUSH %s %s", KEY_NAME.c_str(), VALUE_DATA.c_str());

reply = (redisReply *)redisCommand(ctx, "GET %s", KEY_NAME.c_str());
```
you can also use `KEY_NAME.data()` instead. The problem may be caused by the difference on string format for both Redis and c++. You have to use this format to push and read smoothly.

## Redis Push and Read in Different Session
This problem is also caused by the string difference. If you push data in a key in the wrong format, the key's name would be a messy code in redis if you type
```
(redis)127.0.0.1:6379> keys *
 1) "`Y9\xbb\xff\x7f"
 2) "\xe0\x9b\x8c\x81\xff\x7f"
 3) ""
 4) "\xf0\x97\x96\x99\xff\x7f"
 5) "\xd0A\x0b\xfd\xfe\x7f"
 6) " Y9\xbb\xff\x7f"
 7) "\xc06&\xcf\xfd\x7f"
```
it shows clearly that you can't recongnize any key's name in redis. Consequently, you can't get the right key's name in another session and also can not get the right value.

# Summary
* Reset MySQL needs to stop the service, reset in safe mode and restart service;
* The lost sock can be solved with soft link;
* Push and read data in Redis with **CORRECT** format;
* Reply pointer is the best way for debugging.
