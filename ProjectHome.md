## 1. Features ##

  * Very simple

  * Very fast, more than 10000 requests/sec

  * High concurrency, support the tens of thousands of concurrent connections.

  * Multiple queue

  * A single queue length maximum support one billion (1,000,000,000).

  * Low memory consumption, mass data storage, storage dozens of GB of data takes less than 100MB of physical memory buffer.

  * Convenient to change the maximum queue length of per-queue.

  * Queue status view

  * Be able to view the contents of the specified queue ID.

  * Multi-Character encoding support

  * Less than 900 lines source code, easy to second development.


---


## 2. Benchmark Test ##

> [Using Apache AB for HTTPSQS Benchmark Test](http://code.google.com/p/httpsqs/wiki/BenchmarkTest)


---


![http://blog.s135.com/attachment/200912/httpsqs_elements.png](http://blog.s135.com/attachment/200912/httpsqs_elements.png)


---


## 3. Server Install ##

> Operating System: CentOS Linux 5.4

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


---


## 4. Server Document ##

> ![http://blog.s135.com/attachment/201006/httpsqs.png](http://blog.s135.com/attachment/201006/httpsqs.png)

> `[root@xoyo ~]# httpsqs -h`

```
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
```

> Example:

```
ulimit -SHn 65535
httpsqs -d -p 1218 -x /data0/queue
```

> or with auth password:

```
ulimit -SHn 65535
httpsqs -d -p 1218 -x /data0/queue -a mypass123
```


> Use command "killall httpsqs", "pkill httpsqs" and "kill `cat /tmp/httpsqs.pid`" to stop httpsqs.

> Please note that don't use the command "pkill -9 httpsqs" and "kill -9 PID of httpsqs", otherwise, the data in memory has not been saved to disk will be lost.


---


## 5. Client Document ##

> Important Notice: The message must be GET before the PUT position to catch up with the GET position in the circle.

#### (1). PUT text message into a queue ####

> HTTP GET protocol (Using curl for example):

```
curl "http://host:port/?name=your_queue_name&opt=put&data=url_encoded_text_message&auth=mypass123"
```

> or

> HTTP POST protocol (Using curl for example):

```
curl -d "url_encoded_text_message" "http://host:port/?name=your_queue_name&opt=put&auth=mypass123"
```

> Using google chrome browser for example:

> ![http://blog.s135.com/attachment/200912/put.png](http://blog.s135.com/attachment/200912/put.png)

> If PUT successful, return:

```
HTTPSQS_PUT_OK
```

> If an error occurs, return:

```
HTTPSQS_PUT_ERROR
```

> If Queue full, return:

```
HTTPSQS_PUT_END
```

> HTTPSQS add a line "Pos: xxx" in the HTTP header and sends that back to the client since version 1.2, the "xxx" is the put pos of the current queue. For example:

```
HTTP/1.1 200 OK
Content-Type: text/plain
Keep-Alive: 120
Pos: 19
Date: Thu, 18 Mar 2010 04:57:08 GMT
Content-Length: 14

HTTPSQS_PUT_OK
```

#### (2). GET text message from a queue ####

> HTTP GET protocol (Using curl for example):

```
curl "http://host:port/?charset=utf-8&name=your_queue_name&opt=get&auth=mypass123"
```

```
curl "http://host:port/?charset=gb2312&name=your_queue_name&opt=get&auth=mypass123"
```

> Using google chrome browser for example:

> ![http://blog.s135.com/attachment/200912/get.png](http://blog.s135.com/attachment/200912/get.png)

> Return the queue contents.

> If there is no unread queue message, return:

```
HTTPSQS_GET_END
```

> HTTPSQS add a line "Pos: xxx" in the HTTP header and sends that back to the client since version 1.2, the "xxx" is the get pos of the current queue. For example:

```
HTTP/1.1 200 OK
Content-Type: text/plain; charset=utf-8
Keep-Alive: 120
Pos: 7
Date: Thu, 18 Mar 2010 04:56:01 GMT
Content-Length: 7

message
```

> #### The charset parameter (Example: /?charset=utf-8): ####
> Documents transmitted with HTTP that are of type text, such as text/html, text/plain, etc., can send a charset parameter in the HTTP header to specify the character encoding of the document.

> It is very important to always label Web documents explicitly. HTTP 1.1 says that the default charset is ISO-8859-1. But there are too many unlabeled documents in other encodings, so browsers use the reader's preferred encoding when there is no explicit charset parameter.

> The line in the HTTP header typically looks like this:

> Content-Type: text/plain; charset=utf-8

> In theory, any character encoding that has been [registered with IANA](http://www.iana.org/assignments/character-sets) can be used, but there is no browser that understands all of them.

#### (3). View queue status ####

> HTTP GET protocol (Using curl for example):

```
curl "http://host:port/?name=your_queue_name&opt=status&auth=mypass123"
```

> Return (for example):

```
HTTP Simple Queue Service v1.3
------------------------------
Queue Name: xoyo
Maximum number of queues: 1000000
Put position of queue (1st lap): 45
Get position of queue (1st lap): 6
Number of unread queue: 39
```

> or if the "Put position of queue" greater than the "Maximum number of queues", it will come back to the pos 1.

```
HTTP Simple Queue Service v1.3
------------------------------
Queue Name: xoyo
Maximum number of queues: 1000000
Put position of queue (2st lap): 4562
Get position of queue (1st lap): 900045
Number of unread queue: 104517
```

> Using google chrome browser for example:

> ![http://blog.s135.com/attachment/200912/status.png](http://blog.s135.com/attachment/200912/status.png)

#### (4). View queue status in json ####

> HTTP GET protocol (Using curl for example):

```
curl "http://host:port/?name=your_queue_name&opt=status_json&auth=mypass123"
```

> Return (for example):

```
{"name":"xoyo","maxqueue":1000000,"putpos":45,"putlap":1,"getpos":6,"getlap":1,"unread":39}
```

> or if the "Put position of queue" greater than the "Maximum number of queues", it will come back to the pos 1.

```
{"name":"xoyo","maxqueue":1000000,"putpos":4562,"putlap":2,"getpos":900045,"getlap":1,"unread":104517}
```

#### (5). View the contents of the specified queue pos (id) ####

> HTTP GET protocol (Using curl for example):

```
curl "http://host:port/?charset=utf-8&name=your_queue_name&opt=view&pos=5&auth=mypass123"
```

```
curl "http://host:port/?charset=gb2312&name=your_queue_name&opt=view&pos=19&auth=mypass123"
```

> pos >=1 and <= 1000000000

> Return the contents of the specified queue pos.

#### (6). Reset the queue ####

> HTTP GET protocol (Using curl for example):

```
curl "http://host:port/?name=your_queue_name&opt=reset&auth=mypass123"
```

> If reset successful, return:

```
HTTPSQS_RESET_OK
```

> If an error occurs, return:

```
HTTPSQS_RESET_ERROR
```

#### (7). Change the maximum queue length of per-queue ####

> Default maximum queue length: 1000000

> HTTP GET protocol (Using curl for example):

```
curl "http://host:port/?name=your_queue_name&opt=maxqueue&num=1000000000&auth=mypass123"
```

> num >=10 and <= 1000000000

> If change the maximum queue length successful, return:

```
HTTPSQS_MAXQUEUE_OK
```

> This operation will be cancelled if "Put position of queue" less than "Get position of queue" (lap of PUT > lap of GET), will return:

```
HTTPSQS_MAXQUEUE_CANCEL
```

#### (8). Change the interval to sync updated contents to the disk ####

> Default interval: 5 seconds or the parameter set by "httpsqs -s second"

> HTTP GET protocol (Using curl for example):

```
curl "http://host:port/?name=your_queue_name&opt=synctime&num=10&auth=mypass123"
```

> num >=1 and <= 1000000000

> If change the interval successful, return:

```
HTTPSQS_SYNCTIME_OK
```

> If "num" not between 1 to 1000000000, will return:

```
HTTPSQS_SYNCTIME_CANCEL
```

#### (9). Authentication failed ####

> If password(/?auth=xxx) authentication failed, return:

```
HTTPSQS_AUTH_FAILED
```

#### (10). Global error ####

> If an global error occurs, return:

```
HTTPSQS_ERROR
```


---


## 6. Client Document ##

#### (1). PHP Client ####

> A、PHP ext: http://code.google.com/p/php-httpsqs-client/

> B、PHP Class: View source code: [httpsqs\_client.php](http://code.google.com/p/httpsqs/source/browse/trunk/client/php/httpsqs_client.php)

> Usage:

```
<?php
include_once("httpsqs_client.php");
$httpsqs = new httpsqs($httpsqs_host, $httpsqs_port, $httpsqs_auth, $httpsqs_charset);

/*
1. PUT text message into a queue.
   If PUT successful, return boolean: true
   If an error occurs, return boolean: false
*/
$result = $httpsqs->put($queue_name, $queue_data);

/*
2. GET text message from a queue.
   Return the queue contents.
   If there is no unread queue message, return text: HTTPSQS_GET_END
   If an error occurs, return boolean: false.
*/
$result = $httpsqs->get($queue_name); 

/*
3. GET text message and pos from a queue.
   Return example: array("pos" => 7, "data" => "text message")
   If there is no unread queue message, return: array("pos" => 0, "data" => "HTTPSQS_GET_END")
   If an error occurs, return boolean: false.
*/
$result = $httpsqs->gets($queue_name);

/*
4. View queue status
*/
$result = $httpsqs->status($queue_name); 

/*
5. View queue status in json
   Return example: {"name":"queue_name","maxqueue":5000000,"putpos":130,"putlap":1,"getpos":120,"getlap":1,"unread":10}
*/
$result = $httpsqs->status_json($queue_name);

/*
6. View the contents of the specified queue pos (id).
   Return the contents of the specified queue pos.
*/
$result = $httpsqs->view($queue_name, $queue_pos);

/*
7. Reset the queue.
   If reset successful, return boolean: true
   If an error occurs, return boolean: false
*/
$result = $httpsqs->reset($queue_name);

/*
8. Change the maximum queue length of per-queue.
   If change the maximum queue length successful, return boolean: true
   If it be cancelled, return boolean: false
*/
$result = $httpsqs->maxqueue($queue_name, $num);

/*
9. Change the interval to sync updated contents to the disk.
   If change the interval successful, return boolean: true
   If it be cancelled, return boolean: false
*/
$result = $httpsqs->synctime($num);
?>
```

#### (2). Perl Client (Third Party, Developer: [tonny0830](http://code.google.com/u/tonny0830/)) ####

> Perl client file path: httpsqs-1.3/client/perl/

> View source code: http://code.google.com/p/httpsqs/source/browse/trunk/client/perl/

#### (3). Java Client (Third Party, Developer: [SuperSNY](http://code.google.com/u/SuperSNY/)) ####

> Download: http://httpsqs.googlecode.com/files/httpsqs4j-java-client-1.0.zip

> New code: svn checkout http://httpsqs.googlecode.com/svn/trunk/client/httpsqs4j/ httpsqs4j

> Documentation: http://blog.s135.com/book/httpsqs/client/httpsqs4j/