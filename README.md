# httpsqs
Automatically exported from code.google.com/p/httpsqs


## Installation

```
ulimit -SHn 65535

wget http://httpsqs.googlecode.com/files/libevent-2.0.12-stable.tar.gz
tar zxvf libevent-2.0.12-stable.tar.gz
cd libevent-2.0.12-stable/
./configure --prefix=/usr/local/libevent-2.0.12-stable/
make
make install
cd ../

wget http://httpsqs.googlecode.com/files/tokyocabinet-1.4.47.tar.gz
tar zxvf tokyocabinet-1.4.47.tar.gz
cd tokyocabinet-1.4.47/
./configure --prefix=/usr/local/tokyocabinet-1.4.47/
#Note: In the 32-bit Linux operating system, compiler Tokyo cabinet, please use the ./configure --enable-off64 instead of ./configure to breakthrough the filesize limit of 2GB.
#./configure --enable-off64 --prefix=/usr/local/tokyocabinet-1.4.47/
make
make install
cd ../

wget http://httpsqs.googlecode.com/files/httpsqs-1.7.tar.gz
tar zxvf httpsqs-1.7.tar.gz
cd httpsqs-1.7/
make
make install
cd ../
```

# How to use

```
[root@xoyo ~]# httpsqs -h
 -l <ip_addr>  interface to listen on, default is 0.0.0.0
 -p <num>      TCP port number to listen on (default: 1218)
 -x <path>     database directory (example: /opt/httpsqs/data)
 -t <second>   timeout for an http request (default: 3)
 -s <second>   the interval to sync updated contents to the disk (default: 5)
 -c <num>      the maximum number of non-leaf nodes to be cached (default: 1024)
 -m <size>     database memory cache size in MB (default: 100)
 -i <file>     save PID in <file> (default: /tmp/httpsqs.pid)
 -a <auth>     the auth password to access httpsqs (example: mypass123)
 -d            run as a daemon
 -h            print this help and exit
 
Example:
ulimit -SHn 65535
httpsqs -d -p 1218 -x /data0/queue
or with auth password:
ulimit -SHn 65535
httpsqs -d -p 1218 -x /data0/queue -a mypass123
```

请使用命令“killall httpsqs”、“pkill httpsqs”和“kill `cat /tmp/httpsqs.pid`”来停止httpsqs。

注意：请不要使用命令“pkill -9 httpsqs”和“kill -9  httpsqs的进程ID”来结束httpsqs，否则，内存中尚未保存到磁盘的数据将会丢失。


Use command "killall httpsqs", "pkill httpsqs" and "kill cat /tmp/httpsqs.pid" to stop httpsqs.
Please note that don't use the command "pkill -9 httpsqs" and "kill -9 PID of httpsqs", otherwise, the data in memory has not been saved to disk will be lost.
