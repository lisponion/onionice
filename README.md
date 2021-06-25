# onionice

## require

* libc 
* libuv

## introduction
```text
The name onion-ice is that I think the layers of brackets in lisp are like layers of onion skin, 
and the volatility of the cache database is like melting ice. 

Functionally, it is used to cache k-v data string, is similar to memcached, 
but item has no size limit it can store large values. [key:val(timeout)]
and it can save all kv to file on computer disk.

Structurally, it uses a single-process single-threaded structure, 
similar to redis, but oninoice occupies less CPU and memory. 
Ten million kv occupies 2g of memory, which is one-tenth of the memory used by redis.

but onionice is 10 times slower than memcached. onionice's qps is only 20k to 30k, 
and it is stable at 10k qps for a large number of connections, while memcached is 100k.

In summary, onionice is a single node, stable and slow cache

it can use as __ with __ for example __ :
    kv-cache-db     [set,get,rem]       promotional goods information
    number-counter  [math,more,less]    number of hot goods sold
    message-queue   [push,take]         aynchronous message
    text-reader     [read]              quick read the text string in file
```

## start server
```bash
# onionice [addr] [port]
# it is transmitted in plain text and is not secure
# please use it on your local network

# local ipv4 6666
./onionice

# local ipv4 6060
./onionice 127.0.0.1 6060

# local ipv6 6600
./onionice ::1 6600

# other example
./onionice 66.66.66.66 6666
```

## connect
```python
import socket

def recvall (sock):
    BUFF_SIZE = 4096
    msg = b''
    while True:
        part = sock.recv(BUFF_SIZE)
        msg += part
        if len(part) < BUFF_SIZE:
            break
    return msg

def makeclient ():
    global client
    client = socket.socket(socket.AF_INET,socket.SOCK_STREAM)
    client.connect(("127.0.0.1",6666))
    return 0

def sendstr (str):
    client.send(str.encode('utf-8'))
    data = recvall(client)
    return data.decode()

makeclient()

sendstr(chr(0).join(["set","0","abc","xyz"]))

sendstr(chr(0).join(["get","abc"]))

sendstr(chr(0).join(["del","abc"]))
```

## nosql
```text
use ascii char #\Nul

the hash-table and message-queue is thread-safety
```
```text
set [timeout,key,val]
    timeout 0  is without timeout   
            60 is with a minute timeout  
    key and val is string
send: "set\x0020\x00abc\x00def"
back: "0[onionice✅]:t\r\n"
```
```text
let [timeout0,key0,val0,timeout1,key1,val1,timeout2,key2,val2,......]
    set multi kv at once
send: "let\x000\x00123\x00456\x000\x00231\x00564\x000\x00312\x00645"
back: "0[onionice✅]:t\r\n"
```
```text
get [key]
    get the val in string
    "NIL" means no value
send: "get\x00123"
back: "456\r\n"
```
```text
got [key0,val0,key1,val1,key3,val3,......]
    got multi val at once
    ......\r\n
    ......\r\n
    ......\r\n
    \r\n
send: "got\x00123\x00231\x00312"
back: "456\r\n564\r\n645\r\n\r\n"
```
```text
del [key]
    delete the key and val
send: "del\x00123"
back: "0[onionice✅]:t\r\n"
```
```text
rem [key0,val0,key1,val1,key2,val2,......]
    remove multi kv
send: "rem\x00123\x00231\x00312"
back: "0[onionice✅]:t\r\n"
```
```text
see [] 
    show the information
send: "see"
back: ......
```
```text
push [val]
    push the value to simple mq
send: "push\x00123"
back: "0[onionice✅]:t\r\n"
```
```text
take [] 
    take the value from simple mq
send: "take\x00"
back: "123\r\n"
```
```text
math [timeout,key,val]
    the val type is number
    can use get to get the val
send: "math\x000\x00654\x00321"
back: "0[onionice✅]:t\r\n"
```
```text
more [key]
    the value +1
send: "more\x00654"
back: "322\r\n"
```
```text
less [key]
    the value -1
send: "less\x00654"
back: "321\r\n"
```
```text
save [path]
    save all kv to file path by override 
    will take long when many data
send: "save\x00~/onion.ice"
back: "0[onionice✅]:t\r\n"
```
```text
load [path]
    load all kv from the file path
send: "load\x00~/onion.ice"
back: "0[onionice✅]:t\r\n"
```
```text
read [path] 
    quick read the text string from file
send: "read\x00/home/abc/abc.txt"
back: "123\n\r\n"
```

## package
[the-queue](http://sbcl.org/manual/index.html#Queue)
```common-lisp
(defglobal the-mq (make-queue))

(enqueue "123" the-mq)

(dequeue the-mq)
```
[event-loop](https://github.com/orthecreedence/cl-async)
```common-lisp
(defun abc ()
    ......)
    
(as:start-event-loop #'abc)
```
[handle-string](https://github.com/sharplispers/split-sequence)
```common-lisp
(split-sequence #\Nul "123")
```
[hash-table](https://github.com/diogoalexandrefranco/cl-cache-tables)
```common-lisp
(setf the-cache-table (cache:make-cache-table :test #'equal))

(cache:cache-table-put "key0" "value0" *cache* :expire 0)
```
[cl-store](https://github.com/skypher/cl-store)
```common-lisp
;the-cache-table

(store the-cache-table "~/data")

(setf the-cache-table (restore "~/data"))
```
[mmap](https://github.com/Shinmera/mmap)
```common-lisp
(with-mmap (addr fd size file-path)
  (with-output-to-string (out)
    (......
```

## test
```python3
#!/usr/bin/python3

import pymemcache
import time

mc = pymemcache.Client(('127.0.0.1','12000'))

def runtest():
    time0 = time.time()
    for i in range(100000):
        a = str(i)
        mc.set(a,a)
    time1 = time.time()
    print("100000set: ", time1 - time0)
    time2 = time.time()
    for i in range(100000):
        a = str(i)
        mc.get(a)
    time3 = time.time()
    print("100000get: ", time3 - time2)

# test for memcached
runtest()
```

```python3
#!/usr/bin/python3

import redis
import time

r = redis.StrictRedis(host='localhost', port=6379, db=0)

def runtest():
    time0 = time.time()
    for i in range(100000):
        a = str(i)
        r.set(a,a)
    time1 = time.time()
    print("100000set: ", time1 - time0)
    time2 = time.time()
    for i in range(100000):
        a = str(i)
        r.get(a)
    time3 = time.time()
    print("100000get: ", time3 - time2)

# test for redis
runtest()
```

```python3
#!/usr/bin/python3

import socket
import time

def makeclient ():
   global client
   client = socket.socket(socket.AF_INET,socket.SOCK_STREAM)
   client.connect(("127.0.0.1",6666))
   return 0

def recvall (sock):
    BUFF_SIZE = 4096
    msg = b''
    while True:
        part = sock.recv(BUFF_SIZE)
        msg += part
        if len(part) < BUFF_SIZE:
            break
    return msg

def sendstr (str):
   client.send(str.encode('utf-8'))
   data = recvall(client)
   return data.decode()

def theset(str0,str1,str2):
    message = chr(0).join(["set", str0, str1, str2])
    sendstr(message)
    return

def theget(str0):
    message = chr(0).join(["get", str0])
    sendstr(message)
    return

def runtest():
    time0 = time.time()
    for i in range(100000):
      a = str(i)
      theset("0",a,a) #"0" is the kv without timeout
    time1 = time.time()
    print("100000set: ", time1 - time0)
    time2 = time.time()
    for i in range(100000):
      a = str(i)
      theget(a)
    time3 = time.time()
    print("100000get: ", time3 -time2)

makeclient()

# test for onionice
runtest() 
```
